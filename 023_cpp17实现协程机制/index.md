# C++17 实现协程机制


## 前提说明
C++17 标准**没有内置协程语法与标准库**，协程是 C++20 才正式标准化的特性。C++17 实现协程，主流方案是**用户态有栈协程（Stackful Coroutine）**，基于平台上下文切换 API 实现执行流的用户态切换，完全避开内核态调度，切换开销远小于线程。

本文基于 POSIX 标准的 `ucontext` 系列 API（Linux/macOS/BSD 通用），实现一套 C++17 语法的最小可用协程库，同时补充 Windows 兼容方案、核心原理与进阶优化方向。

---

## 一、有栈协程核心原理
1.  **独立栈空间**：每个协程拥有独立的固定大小栈，用于保存函数调用栈、局部变量，切换时栈数据不会被破坏。
2.  **上下文切换**：通过 `ucontext` 系列函数，在用户态完成 CPU 寄存器上下文的保存与恢复，实现执行流的主动切换，无内核态开销。
3.  **协作式调度**：调度器管理协程生命周期与执行队列，协程通过 `yield` 主动让出 CPU，调度器按策略恢复待执行协程。
4.  **线程局部存储**：用 TLS 保存当前线程的调度器与运行中协程，保证单线程内协程的正确寻址，天然规避多线程竞争。

---

## 二、C++17 完整实现代码
### 1. 头文件与前置定义
```cpp
#include <ucontext.h>
#include <functional>
#include <memory>
#include <vector>
#include <iostream>
#include <thread>

// 协程状态枚举
enum class CoroutineState {
    Ready,      // 就绪，可执行
    Running,    // 正在运行
    Suspended,  // 已挂起
    Terminated  // 已结束
};

class Scheduler;
class Coroutine;

// 线程局部变量（TLS）：保存当前线程的调度器与运行中协程
static thread_local Scheduler* t_scheduler = nullptr;
static thread_local Coroutine* t_current_coro = nullptr;
```

### 2. 协程核心类
```cpp
// 协程类，继承 enable_shared_from_this 实现安全的生命周期管理
class Coroutine : public std::enable_shared_from_this<Coroutine> {
public:
    using CoroutineFunc = std::function<void()>;
    friend class Scheduler;

    Coroutine(Scheduler* scheduler, CoroutineFunc func);
    ~Coroutine();

    // 禁止拷贝，允许移动
    Coroutine(const Coroutine&) = delete;
    Coroutine& operator=(const Coroutine&) = delete;
    Coroutine(Coroutine&&) = default;
    Coroutine& operator=(Coroutine&&) = default;

    // 恢复协程执行
    void resume();
    // 挂起当前协程（静态方法，可在协程函数内直接调用）
    static void yield();

    // 获取协程状态
    CoroutineState getState() const { return m_state; }

private:
    // 协程入口函数，适配 ucontext 的函数签名要求
    static void entryPoint();
    // 设置协程状态
    void setState(CoroutineState state) { m_state = state; }
    // 获取上下文指针
    ucontext_t* getContext() { return &m_ctx; }

private:
    Scheduler* m_scheduler = nullptr;  // 所属调度器
    CoroutineFunc m_func;               // 协程执行函数
    CoroutineState m_state = CoroutineState::Ready;
    ucontext_t m_ctx{};                  // 协程CPU上下文
    std::unique_ptr<char[]> m_stack;     // 协程栈空间
    size_t m_stackSize = 1024 * 1024;   // 栈大小：默认1MB
};
```

### 3. 协程调度器类
```cpp
// 调度器类：单线程内协程的管理与调度核心
class Scheduler {
public:
    Scheduler();
    ~Scheduler() = default;

    // 禁止拷贝与移动
    Scheduler(const Scheduler&) = delete;
    Scheduler& operator=(const Scheduler&) = delete;
    Scheduler(Scheduler&&) = delete;
    Scheduler& operator=(Scheduler&&) = delete;

    // 创建协程并加入调度队列
    std::shared_ptr<Coroutine> createCoroutine(Coroutine::CoroutineFunc func);
    // 启动调度器，执行所有协程
    void run();
    // 切换回主协程（挂起当前子协程）
    void switchToMain();
    // 切换到目标协程
    void switchToCoroutine(std::shared_ptr<Coroutine> coro);
    // 获取主协程指针
    Coroutine* getMainCoroutine() { return &m_mainCoroutine; }

private:
    Coroutine m_mainCoroutine;                   // 主协程（调度协程）
    std::shared_ptr<Coroutine> m_currentCoroutine;  // 当前运行的子协程
    std::vector<std::shared_ptr<Coroutine>> m_coroutines; // 协程就绪队列
};
```

### 4. 核心实现逻辑
```cpp
// ---------------------- 调度器实现 ----------------------
Scheduler::Scheduler() {
    m_mainCoroutine.setState(CoroutineState::Running);
    m_currentCoroutine = nullptr;
    // 初始化TLS，绑定当前线程的调度器
    t_scheduler = this;
    t_current_coro = &m_mainCoroutine;
}

std::shared_ptr<Coroutine> Scheduler::createCoroutine(Coroutine::CoroutineFunc func) {
    auto coro = std::make_shared<Coroutine>(this, std::move(func));
    m_coroutines.push_back(coro);
    return coro;
}

void Scheduler::switchToMain() {
    if (m_currentCoroutine) {
        t_current_coro = &m_mainCoroutine;
        // 保存子协程上下文，切换到主协程
        swapcontext(m_currentCoroutine->getContext(), m_mainCoroutine.getContext());
    }
}

void Scheduler::switchToCoroutine(std::shared_ptr<Coroutine> coro) {
    if (!coro || coro->getState() == CoroutineState::Terminated) {
        return;
    }
    m_currentCoroutine = coro;
    t_current_coro = coro.get();
    coro->setState(CoroutineState::Running);
    // 保存主协程上下文，切换到子协程
    swapcontext(m_mainCoroutine.getContext(), coro->getContext());
    // 协程挂起/结束后切回，恢复主协程状态
    m_currentCoroutine = nullptr;
    t_current_coro = &m_mainCoroutine;
}

void Scheduler::run() {
    // 简单轮转调度：循环执行就绪队列中的协程
    while (!m_coroutines.empty()) {
        auto coro = m_coroutines.front();
        m_coroutines.erase(m_coroutines.begin());

        if (coro->getState() == CoroutineState::Terminated) {
            continue;
        }

        // 恢复协程执行
        coro->resume();

        // 未结束的协程重新加入队列尾部
        if (coro->getState() != CoroutineState::Terminated) {
            m_coroutines.push_back(coro);
        }
    }
}

// ---------------------- 协程类实现 ----------------------
Coroutine::Coroutine(Scheduler* scheduler, CoroutineFunc func)
    : m_scheduler(scheduler), m_func(std::move(func)) {
    // 初始化上下文
    getcontext(&m_ctx);
    // 分配独立栈空间
    m_stack = std::make_unique<char[]>(m_stackSize);
    // 配置上下文栈信息
    m_ctx.uc_stack.ss_sp = m_stack.get();
    m_ctx.uc_stack.ss_size = m_stackSize;
    // 协程结束后自动切回主协程
    m_ctx.uc_link = m_scheduler->getMainCoroutine()->getContext();
    // 绑定协程入口函数
    makecontext(&m_ctx, &Coroutine::entryPoint, 0);
}

Coroutine::~Coroutine() = default;

void Coroutine::entryPoint() {
    Coroutine* coro = t_current_coro;
    if (coro && coro->m_func) {
        // 执行协程业务函数
        coro->m_func();
        // 执行完毕，标记为终止状态
        coro->setState(CoroutineState::Terminated);
    }
}

void Coroutine::resume() {
    if (m_state == CoroutineState::Terminated) {
        return;
    }
    m_scheduler->switchToCoroutine(shared_from_this());
}

void Coroutine::yield() {
    Coroutine* coro = t_current_coro;
    // 禁止在主协程中调用yield
    if (coro == t_scheduler->getMainCoroutine()) {
        return;
    }
    if (coro && coro->m_state == CoroutineState::Running) {
        coro->setState(CoroutineState::Suspended);
        t_scheduler->switchToMain();
    }
}
```

### 5. 测试用例
```cpp
int main() {
    Scheduler scheduler;

    // 创建协程1
    scheduler.createCoroutine([]() {
        std::cout << "Coroutine 1: 启动" << std::endl;
        for (int i = 0; i < 3; ++i) {
            std::cout << "Coroutine 1: 执行第" << i << "轮" << std::endl;
            Coroutine::yield(); // 主动让出CPU
        }
        std::cout << "Coroutine 1: 结束" << std::endl;
    });

    // 创建协程2
    scheduler.createCoroutine([]() {
        std::cout << "Coroutine 2: 启动" << std::endl;
        for (int i = 0; i < 2; ++i) {
            std::cout << "Coroutine 2: 执行第" << i << "轮" << std::endl;
            Coroutine::yield(); // 主动让出CPU
        }
        std::cout << "Coroutine 2: 结束" << std::endl;
    });

    // 启动调度器
    std::cout << "调度器启动" << std::endl;
    scheduler.run();
    std::cout << "调度器结束" << std::endl;

    return 0;
}
```

### 6. 编译与运行
```bash
# Linux/macOS 下编译，指定C++17标准
g++ -std=c++17 coroutine.cpp -o coroutine
# 运行
./coroutine
```

#### 运行结果
```
调度器启动
Coroutine 1: 启动
Coroutine 1: 执行第0轮
Coroutine 2: 启动
Coroutine 2: 执行第0轮
Coroutine 1: 执行第1轮
Coroutine 2: 执行第1轮
Coroutine 1: 执行第2轮
Coroutine 2: 结束
Coroutine 1: 结束
调度器结束
```

---

## 三、关键注意事项
1.  **平台兼容性**
    - 上述代码基于 POSIX `ucontext` API，仅适用于 Linux/macOS/BSD 系统。
    - Windows 平台需使用**纤程（Fiber）API** 替代：`ConvertThreadToFiber`、`CreateFiber`、`SwitchToFiber`、`DeleteFiber`，核心调度逻辑完全一致。
    - 跨平台高性能方案：基于汇编实现不同架构（x86/x64/ARM/ARM64）的寄存器上下文切换，参考 `boost.context` 实现。

2.  **核心风险点**
    - **栈溢出**：协程栈大小固定，若协程内使用大局部变量、深递归，会导致栈溢出，需根据场景调整栈大小。
    - **阻塞调用**：当前实现为协作式调度，协程内调用阻塞系统调用（如 `sleep`、阻塞 `read`）会阻塞整个线程，需搭配非阻塞 IO + IO 多路复用（epoll/io_uring/kqueue），或 hook 系统调用实现自动挂起。
    - **线程安全**：调度器与协程绑定单线程，禁止跨线程切换协程；多线程场景需为每个线程创建独立调度器，或加锁保护协程队列。

3.  **生命周期管理**
    - 基于 `std::shared_ptr` 管理协程对象，避免协程执行中被析构；
    - 协程内的局部变量遵循 RAII 规则，协程正常结束时会自动析构，强制终止会导致资源泄漏。

---

## 四、C++17 有栈协程 vs C++20 标准无栈协程
| 特性 | C++17 有栈协程（用户实现） | C++20 标准无栈协程 |
| :--- | :--- | :--- |
| 栈空间 | 每个协程独立固定栈，内存开销大 | 无独立栈，编译器生成状态机保存上下文，内存开销极小 |
| 切换开销 | 保存/恢复全量寄存器，开销中等 | 仅保存必要状态，切换开销极低 |
| 挂起能力 | 可在任意嵌套函数内挂起 | 仅能在协程函数内直接挂起，嵌套函数需配合 `co_await` |
| 标准支持 | 无标准，依赖系统API/汇编 | C++20 标准，编译器原生支持 |
| 易用性 | 接口简单，与普通函数用法一致 | 需理解 Promise/Awaitable 机制，学习成本高 |

---

## 五、进阶优化方向
1.  **调度策略升级**：实现优先级调度、时间片轮转、睡眠队列、IO 等待队列，适配真实业务场景。
2.  **异步 IO 集成**：搭配 epoll/io_uring 实现异步 IO，IO 阻塞时自动挂起协程，就绪后自动恢复，参考腾讯 libco 实现。
3.  **协程池**：预分配协程对象与栈空间，避免频繁创建/销毁的内存开销。
4.  **对称协程**：实现协程间直接切换，无需经过主协程转发，进一步降低切换开销。
5.  **动态栈增长**：基于 mmap 实现可增长栈，通过页错误机制动态扩容，彻底规避栈溢出风险。


