# C++11 新特性详细总结


# C++ 多线程编程详细总结
C++11 首次引入了**标准跨平台多线程库**，彻底替代了此前 Windows、Linux 等平台各自的原生线程API（如`CreateThread`、`pthread`），后续 C++14/17/20 持续补充完善，形成了完整的现代C++多线程体系。本文从基础概念到高级特性、从核心用法到避坑指南，进行全面系统的总结。

## 一、基础概念与核心头文件
### 1. 核心术语区分
| 术语 | 核心定义 |
|------|----------|
| 进程 | 操作系统资源分配的最小单位，拥有独立的地址空间，进程间隔离性强，切换开销大 |
| 线程 | 操作系统调度执行的最小单位，隶属于进程，共享进程的地址空间，切换开销极小 |
| 并发 | 多个任务在宏观上同时推进，微观上可通过时间片轮转交替执行（单核CPU也可实现并发） |
| 并行 | 多个任务在物理核心上同时执行，必须依赖多核CPU |
| 同步 | 线程间按预定的先后顺序执行，通过互斥、等待-通知等机制协调执行节奏 |
| 异步 | 任务执行无需等待前序任务完成，可独立推进，结果通过回调、future等方式传递 |
| 数据竞争 | 多个线程同时访问同一个可变共享数据，且至少有一个线程是写操作，未做同步保护，会触发未定义行为 |

### 2. 标准库核心头文件
| 头文件 | 核心功能 |
|--------|----------|
| `<thread>` | 线程核心类`std::thread`、C++20`std::jthread`，线程ID、硬件并发数查询等基础接口 |
| `<mutex>` | 互斥量全系列（`std::mutex`、递归锁、超时锁）、RAII锁包装器（`lock_guard`、`unique_lock`、`scoped_lock`） |
| `<shared_mutex>` | C++17 读写锁`std::shared_mutex`，C++14`std::shared_timed_mutex`，适配读多写少场景 |
| `<condition_variable>` | 条件变量`std::condition_variable`、`std::condition_variable_any`，实现线程间等待-通知机制 |
| `<future>` | 异步任务体系：`std::promise`/`std::future`/`std::shared_future`、`std::packaged_task`、`std::async` |
| `<atomic>` | 原子操作模板类`std::atomic<T>`，6种内存顺序，无锁编程核心支持 |
| `<stop_token>` | C++20 线程中断机制：`stop_token`/`stop_source`/`stop_callback`，配合`jthread`使用 |
| `<semaphore>` | C++20 信号量，轻量级并发数限制工具 |
| `<latch>`/`<barrier>` | C++20 线程屏障，实现多线程协同等待 |

## 二、线程的创建与生命周期管理
### 1. std::thread 核心特性
- **不可拷贝，仅可移动**：拷贝构造函数和拷贝赋值运算符被`delete`，仅支持移动构造和移动赋值，确保一个线程对象唯一对应一个执行线程。
- **可结合性**：线程对象默认是`joinable`（可结合）状态，必须在对象销毁前调用`join()`或`detach()`，否则程序会直接调用`std::terminate`异常终止。

### 2. 线程的5种创建方式
线程的执行入口是**可调用对象**，支持以下5种主流创建方式，覆盖绝大多数场景：

#### （1）普通函数/全局函数
```cpp
#include <thread>
#include <iostream>

void func(int a, const std::string& str) {
    std::cout << "线程执行: " << a << ", " << str << std::endl;
}

int main() {
    std::thread t(func, 10, "hello"); // 传入函数名+参数
    t.join(); // 等待线程执行完毕
    return 0;
}
```

#### （2）Lambda表达式
最灵活的创建方式，支持值捕获、引用捕获，适配闭包场景：
```cpp
int main() {
    int x = 20;
    // 值捕获
    std::thread t1([x](){ std::cout << "值捕获: " << x << std::endl; });
    // 引用捕获，必须确保变量生命周期长于线程
    std::thread t2([&x](){ x++; std::cout << "引用捕获: " << x << std::endl; });
    t1.join();
    t2.join();
    return 0;
}
```

#### （3）类成员函数
需传入成员函数指针、类对象指针（或引用），再传入函数参数：
```cpp
class Test {
public:
    void member_func(int a) {
        std::cout << "成员函数执行: " << a << std::endl;
    }
    static void static_func() {
        std::cout << "静态成员函数执行" << std::endl;
    }
};

int main() {
    Test obj;
    // 非静态成员函数，需传入对象指针
    std::thread t1(&Test::member_func, &obj, 30);
    // 静态成员函数，无需对象
    std::thread t2(&Test::static_func);
    t1.join();
    t2.join();
    return 0;
}
```

#### （4）仿函数（函数对象）
重载`operator()`的类对象，可作为线程入口：
```cpp
class Functor {
public:
    void operator()(int a) {
        std::cout << "仿函数执行: " << a << std::endl;
    }
};

int main() {
    Functor func;
    std::thread t(func, 40);
    t.join();
    return 0;
}
```

#### （5）智能指针管理的可调用对象
配合`std::move`传递不可拷贝的对象（如`std::unique_ptr`）：
```cpp
#include <memory>

int main() {
    std::unique_ptr<int> ptr = std::make_unique<int>(50);
    // 必须用std::move转移所有权，unique_ptr不可拷贝
    std::thread t([p = std::move(ptr)](){ std::cout << "智能指针: " << *p << std::endl; });
    t.join();
    return 0;
}
```

### 3. 线程核心操作与参数传递坑点
#### （1）核心操作接口
| 接口 | 功能说明 |
|------|----------|
| `join()` | 阻塞当前线程，等待目标线程执行完毕，执行后线程变为`non-joinable`状态 |
| `detach()` | 分离线程，目标线程的生命周期与线程对象完全解绑，后台独立执行，执行后变为`non-joinable`状态 |
| `joinable()` | 判断线程是否为可结合状态，仅`joinable`的线程可调用`join()`/`detach()` |
| `get_id()` | 获取线程唯一ID，`std::thread::id`类型，可用于区分线程 |
| `std::this_thread::get_id()` | 获取当前线程的ID |
| `std::this_thread::sleep_for()` | 当前线程休眠指定时长（如`std::chrono::milliseconds(100)`） |
| `std::this_thread::yield()` | 放弃当前CPU时间片，主动让调度器切换其他线程 |
| `std::thread::hardware_concurrency()` | 静态方法，返回CPU支持的硬件并发数（逻辑核心数），失败返回0 |

#### （2）参数传递的核心坑点
1. **引用传递必须用`std::ref`/`std::cref`**
`std::thread`的构造函数会默认将参数**拷贝到线程私有栈空间**，即使函数形参是引用，也只会引用拷贝后的临时对象，而非原变量。必须通过`std::ref`包装才能传递真正的引用：
```cpp
void func(int& a) { a++; }

int main() {
    int x = 0;
    // std::thread t(func, x); // 错误：x被拷贝，函数内修改的是临时变量，原x不变
    std::thread t(func, std::ref(x)); // 正确：传递原变量的引用
    t.join();
    std::cout << x << std::endl; // 输出1
    return 0;
}
```

2. **detach线程严禁引用局部变量**
`detach`后的线程会在后台执行，若捕获/引用了主线程的局部变量，主线程退出后变量会被销毁，线程访问悬空引用会触发未定义行为。**非必要场景严禁使用detach**，优先使用`join()`。

3. **不可拷贝对象必须用`std::move`转移所有权**
如`std::unique_ptr`、`std::thread`本身，不可拷贝，必须通过`std::move`转移到线程中。

### 4. 线程局部存储 thread_local
C++11 引入`thread_local`关键字，声明**线程私有变量**：每个线程拥有该变量的独立副本，互不影响，无需加锁即可安全访问，彻底避免数据竞争。
```cpp
#include <thread>
#include <iostream>

// 每个线程有独立的count副本
thread_local int count = 0;

void add() {
    count++;
    std::cout << "线程" << std::this_thread::get_id() << ": count=" << count << std::endl;
}

int main() {
    std::thread t1(add); // 输出count=1
    std::thread t2(add); // 输出count=1
    t1.join();
    t2.join();
    std::cout << "主线程: count=" << count << std::endl; // 输出count=0
    return 0;
}
```
适用场景：线程私有的计数器、随机数生成器、线程池的任务上下文、避免加锁的单线程缓存。

### 5. C++20 std::jthread 新一代线程
`std::jthread`（joining thread）是`std::thread`的增强版，彻底解决了原生线程的核心痛点，**现代C++优先推荐使用jthread**：
1. **自动join**：析构时会自动调用`join()`等待线程结束，无需手动处理，避免程序异常终止。
2. **协作式线程中断**：内置`stop_token`/`stop_source`中断机制，无需自定义标志位即可优雅停止线程。
3. **完全兼容std::thread的接口**，可无缝替换。

#### 核心用法示例
```cpp
#include <thread>
#include <iostream>
#include <chrono>

int main() {
    // 自动处理join，无需手动调用
    std::jthread t([](std::stop_token stoken) {
        // 循环判断中断请求
        while (!stoken.stop_requested()) {
            std::cout << "线程运行中..." << std::endl;
            std::this_thread::sleep_for(std::chrono::milliseconds(100));
        }
        std::cout << "线程收到中断请求，优雅退出" << std::endl;
    });

    // 主线程运行500ms
    std::this_thread::sleep_for(std::chrono::milliseconds(500));
    // 发送中断请求，jthread析构时也会自动发送
    t.request_stop();

    // 无需手动join，t析构时自动等待
    return 0;
}
```

## 三、线程间同步与互斥原语
多线程的核心问题是**共享可变数据的安全访问**，同步原语的核心作用是避免数据竞争、协调线程执行顺序。

### 1. 互斥量（Mutex）全系列
互斥量是最基础的同步工具，核心原理是**互斥访问**：同一时间只有一个线程能持有锁，持有锁的线程可安全访问共享数据，其他线程必须等待锁释放。

| 互斥量类型 | 核心特性 | 适用场景 |
|------------|----------|----------|
| `std::mutex` | 基础互斥量，不可递归加锁，支持`lock()`/`unlock()`/`try_lock()` | 绝大多数常规互斥场景，首选 |
| `std::recursive_mutex` | 递归互斥量，同一线程可多次`lock()`，需对应次数`unlock()`才会释放 | 类内多个成员函数互相调用，且都需要加锁的场景（不推荐滥用） |
| `std::timed_mutex` | 带超时的基础互斥量，新增`try_lock_for()`/`try_lock_until()`，支持超时等待 | 避免无限阻塞，需要设置等待超时的场景 |
| `std::recursive_timed_mutex` | 递归+超时特性结合 | 递归加锁+超时等待的复合场景 |
| `std::shared_mutex` (C++17) | 读写锁，支持两种模式：<br>1. 独占模式（写）：同一时间仅一个线程持有<br>2. 共享模式（读）：同一时间多个线程持有 | 读多写少场景（如配置缓存、元数据管理），大幅提升并发性能 |

### 2. RAII锁包装器
**严禁手动调用`lock()`/`unlock()`**，手动解锁极易因异常、分支跳转导致锁未释放，引发死锁。C++标准库提供了基于RAII（资源获取即初始化）的锁包装器，构造时加锁，析构时自动解锁，确保锁的安全释放。

| 锁包装器 | 核心特性 | 适用场景 |
|----------|----------|----------|
| `std::lock_guard` (C++11) | 最简RAII锁，不可移动、不可复制，构造时强制加锁，析构时解锁，无额外接口 | 单锁、固定作用域的场景，首选，开销最小 |
| `std::unique_lock` (C++11) | 灵活的RAII锁，支持移动、不可复制，支持延迟加锁、超时加锁、手动解锁、解锁后重新加锁 | 条件变量配合、需要灵活控制锁生命周期的场景 |
| `std::shared_lock` (C++14) | 共享模式RAII锁，配合`std::shared_mutex`使用，构造时加共享读锁，析构时解锁 | 读写锁的读场景，线程安全的并发读 |
| `std::scoped_lock` (C++17) | 多锁RAII包装器，支持同时对多个互斥量加锁，自动避免死锁，构造时加锁，析构时解锁 | 多个互斥量同时加锁的场景，替代C++11的`std::lock`，彻底解决多锁死锁问题 |

#### 核心用法示例
```cpp
#include <mutex>
#include <iostream>
#include <thread>

std::mutex mtx;
int shared_data = 0;

// lock_guard 基础用法
void add() {
    for (int i = 0; i < 10000; i++) {
        std::lock_guard<std::mutex> lock(mtx); // 构造加锁
        shared_data++; // 临界区：仅持有锁的线程可访问
        // 析构自动解锁，无论函数正常返回还是异常退出
    }
}

// scoped_lock 多锁安全用法，自动避免死锁
std::mutex mtx1, mtx2;
void multi_lock_func() {
    // 同时对多个互斥量加锁，无需考虑顺序，自动避免死锁
    std::scoped_lock lock(mtx1, mtx2);
    std::cout << "多锁安全访问" << std::endl;
}

int main() {
    std::thread t1(add);
    std::thread t2(add);
    t1.join();
    t2.join();
    std::cout << "最终结果: " << shared_data << std::endl; // 输出20000，无数据竞争
    return 0;
}
```

### 3. 条件变量（condition_variable）
条件变量用于实现**线程间的等待-通知机制**，解决“线程需要等待某个条件满足才能继续执行”的场景，避免忙等（while循环轮询）导致的CPU资源浪费。

#### 核心接口
| 接口 | 功能说明 |
|------|----------|
| `wait(unique_lock<mutex>& lock)` | 阻塞当前线程，释放持有的锁，直到被其他线程唤醒；唤醒后会重新加锁 |
| `wait(unique_lock<mutex>& lock, 谓词)` | 重载版本，唤醒后会执行谓词，仅当谓词返回`true`时才退出等待，**彻底解决虚假唤醒问题** |
| `wait_for()`/`wait_until()` | 带超时的等待，超时后自动退出，返回是否因条件满足而唤醒 |
| `notify_one()` | 唤醒一个正在等待的线程 |
| `notify_all()` | 唤醒所有正在等待的线程 |

#### 核心坑点：虚假唤醒
操作系统实现中，`wait()`可能在没有线程调用`notify_*()`的情况下提前返回（虚假唤醒），若直接认为条件满足，会触发未定义行为。
**解决方案**：必须使用带谓词的`wait`重载版本，或把`wait`放在循环中，每次唤醒后都检查条件是否满足。

#### 经典示例：生产者消费者模型
```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <queue>
#include <chrono>

const int MAX_QUEUE_SIZE = 5;
std::queue<int> task_queue;
std::mutex mtx;
std::condition_variable cv_producer; // 生产者等待队列有空位
std::condition_variable cv_consumer; // 消费者等待队列有数据

// 生产者：生产数据放入队列
void producer(int id) {
    int data = 0;
    while (true) {
        std::unique_lock<std::mutex> lock(mtx);
        // 队列满了，等待空位，带谓词解决虚假唤醒
        cv_producer.wait(lock, [](){ return task_queue.size() < MAX_QUEUE_SIZE; });

        // 生产数据
        task_queue.push(data);
        std::cout << "生产者" << id << "生产数据: " << data << std::endl;
        data++;

        // 通知消费者有新数据
        cv_consumer.notify_one();
        lock.unlock(); // 提前解锁，减少临界区

        std::this_thread::sleep_for(std::chrono::milliseconds(100));
    }
}

// 消费者：从队列取出数据消费
void consumer(int id) {
    while (true) {
        std::unique_lock<std::mutex> lock(mtx);
        // 队列空了，等待数据，带谓词解决虚假唤醒
        cv_consumer.wait(lock, [](){ return !task_queue.empty(); });

        // 消费数据
        int data = task_queue.front();
        task_queue.pop();
        std::cout << "消费者" << id << "消费数据: " << data << std::endl;

        // 通知生产者有空位
        cv_producer.notify_one();
        lock.unlock();

        std::this_thread::sleep_for(std::chrono::milliseconds(200));
    }
}

int main() {
    std::thread p1(producer, 1);
    std::thread c1(consumer, 1);
    std::thread c2(consumer, 2);

    p1.join();
    c1.join();
    c2.join();
    return 0;
}
```

### 4. 死锁：成因、必要条件与避免方案
死锁是指多个线程互相等待对方持有的锁，导致所有线程永久阻塞，无法继续执行。

#### （1）死锁的四个必要条件
四个条件必须同时满足才会发生死锁，打破任意一个即可避免死锁：
1. **互斥条件**：资源同一时间只能被一个线程持有（互斥量的固有特性，无法打破）。
2. **持有并等待**：线程持有至少一个资源，同时请求其他被持有的资源，且不释放已持有的资源。
3. **不可剥夺**：资源只能由持有线程主动释放，其他线程无法强制剥夺。
4. **循环等待**：线程间形成环形的资源等待链（T1等待T2的锁，T2等待T3的锁，T3等待T1的锁）。

#### （2）死锁的避免方案
1. **优先使用`std::scoped_lock`**：同时对多个互斥量加锁，自动避免循环等待，是多锁场景的首选方案。
2. **固定加锁顺序**：所有线程必须按照完全相同的顺序对多个互斥量加锁，彻底打破循环等待。
3. **避免锁嵌套**：尽量不要在持有一个锁的情况下，再去申请另一个锁，从根源上避免多锁冲突。
4. **一次性申请所有资源**：打破持有并等待条件，要么一次性获取所有需要的锁，要么一个都不获取。
5. **设置超时机制**：使用带超时的互斥量，若等待超时则释放已持有的锁，打破不可剥夺条件。
6. **最小化临界区**：锁的范围只包含必须的共享数据访问，不要在临界区执行耗时操作、IO、回调函数。

## 四、异步任务与future/promise体系
原生`std::thread`无法直接获取线程函数的返回值，也无法跨线程传递异常，C++11 引入的`<future>`头文件提供了一套完整的异步任务体系，解决了异步结果获取、异常传递的核心问题。

### 1. 核心组件：promise/future/shared_future
三者是配套使用的基础组件，核心是**共享状态**：`promise`负责写入结果/异常，`future`负责读取结果/异常。

#### （1）std::promise
异步结果的“生产者”，用于手动设置异步任务的结果或异常，核心接口：
- `get_future()`：获取与该promise绑定的`future`对象，用于读取结果。
- `set_value()`：设置结果值，设置后共享状态变为就绪。
- `set_exception()`：设置异常，异常会在`future`调用`get()`时抛出。

#### （2）std::future
异步结果的“消费者”，用于获取异步任务的结果，核心接口：
- `get()`：阻塞等待结果就绪，获取结果值（或抛出异常），**仅可调用一次**，调用后future变为无效状态。
- `wait()`：仅阻塞等待结果就绪，不获取结果。
- `wait_for()`/`wait_until()`：带超时的等待，返回`std::future_status`，判断是就绪、超时还是延迟执行。
- `valid()`：判断future是否持有有效共享状态。

#### （3）std::shared_future
可复制、可多次调用`get()`的future，支持多个线程同时等待同一个异步结果，解决`std::future`只能移动、只能读取一次的限制。可通过`future.share()`转换得到。

#### 基础用法示例
```cpp
#include <iostream>
#include <thread>
#include <future>
#include <stdexcept>

// 线程函数中通过promise设置结果
void async_task(std::promise<int> promise) {
    try {
        int result = 100 + 200;
        // 设置结果
        promise.set_value(result);
        // 若发生异常，设置异常
        // throw std::runtime_error("任务执行失败");
    } catch (...) {
        // 捕获所有异常，设置到promise中
        promise.set_exception(std::current_exception());
    }
}

int main() {
    // 创建promise和对应的future
    std::promise<int> promise;
    std::future<int> future = promise.get_future();

    // 启动线程，将promise移动到线程中
    std::thread t(async_task, std::move(promise));

    try {
        // 阻塞等待结果，获取值，异常会在这里抛出
        int result = future.get();
        std::cout << "异步任务结果: " << result << std::endl;
    } catch (const std::exception& e) {
        std::cout << "异步任务异常: " << e.what() << std::endl;
    }

    t.join();
    return 0;
}
```

### 2. std::packaged_task
包装一个可调用对象，将其与`future`绑定，可将任务包装后放入线程池、任务队列中延迟执行，执行后自动将结果写入共享状态，无需手动调用`set_value`。
```cpp
#include <iostream>
#include <thread>
#include <future>

int add(int a, int b) {
    return a + b;
}

int main() {
    // 包装函数，绑定future
    std::packaged_task<int(int, int)> task(add);
    std::future<int> future = task.get_future();

    // 启动线程执行任务，task不可拷贝，必须move
    std::thread t(std::move(task), 100, 200);

    // 获取结果
    std::cout << "任务结果: " << future.get() << std::endl;
    t.join();
    return 0;
}
```

### 3. std::async 异步任务高级接口
`std::async`是对线程、packaged_task、promise的高层封装，无需手动管理线程，一行代码即可启动异步任务，是C++中执行异步任务的首选方案。

#### （1）启动策略
`std::async`支持两种启动策略，可手动指定，也可使用默认策略：
| 策略 | 核心行为 |
|------|----------|
| `std::launch::async` | 立即创建新线程，异步执行任务，任务一定会在新线程中执行 |
| `std::launch::deferred` | 延迟执行，任务不会启动新线程，仅当调用`future`的`get()`/`wait()`时，才会在调用线程中同步执行 |
| 默认策略 `async | deferred` | 由编译器和操作系统决定，可能是异步执行，也可能是延迟同步执行，**存在极大的坑点** |

#### （2）核心用法与坑点
```cpp
#include <iostream>
#include <future>
#include <chrono>

int async_func() {
    std::cout << "异步任务执行，线程ID: " << std::this_thread::get_id() << std::endl;
    std::this_thread::sleep_for(std::chrono::milliseconds(500));
    return 100;
}

int main() {
    std::cout << "主线程ID: " << std::this_thread::get_id() << std::endl;

    // 正确用法1：指定async策略，确保异步执行
    std::future<int> f1 = std::async(std::launch::async, async_func);
    // 正确用法2：默认策略，必须保存future，否则会阻塞
    // std::future<int> f2 = std::async(async_func);

    // 超级大坑：临时future对象会立即析构，析构时会阻塞等待任务完成，变成同步执行
    // std::async(std::launch::async, async_func); // 这行代码会阻塞，完全失去异步效果

    std::cout << "主线程继续执行..." << std::endl;
    // 获取结果
    std::cout << "结果: " << f1.get() << std::endl;

    return 0;
}
```

#### 关键避坑指南
1.  **必须明确指定`std::launch::async`**，除非你明确需要延迟执行，否则默认策略可能不会启动新线程，导致异步任务变成同步执行。
2.  **必须保存`std::async`返回的future对象**，临时future对象的析构函数会阻塞等待任务执行完毕，完全失去异步效果。
3.  `future`的`get()`只能调用一次，多次调用会触发未定义行为，需要多次读取请使用`shared_future`。

## 五、内存模型与原子操作
C++11 首次定义了**标准化内存模型**，明确了多线程环境下的内存访问规则，而`std::atomic`原子操作是无锁编程的核心，提供了比互斥量更轻量的同步机制，保证操作的不可分割性，避免数据竞争。

### 1. 原子操作基础
原子操作是**不可分割的操作**，要么完全执行，要么完全不执行，不会被线程调度打断，多个线程同时访问同一个原子变量不会触发数据竞争，无需互斥量即可安全访问。

#### （1）std::atomic 核心特性
- 模板类`std::atomic<T>`，支持整型、指针类型，C++20 扩展支持浮点型、`std::shared_ptr`等类型。
- 对原子变量的读写操作默认使用最强的顺序一致性内存序，保证线程安全。
- 支持无锁操作：`is_lock_free()`判断该类型的原子操作是否是无锁的，绝大多数平台的整型原子操作都是无锁的。

#### （2）核心接口
| 接口 | 功能说明 |
|------|----------|
| `store(val, order)` | 原子写入值，支持指定内存序 |
| `load(order)` | 原子读取值，支持指定内存序 |
| `exchange(val, order)` | 原子替换值，返回旧值 |
| `compare_exchange_weak(expected, desired, order)` | 弱比较交换：若当前值等于expected，则写入desired，返回true；否则将expected更新为当前值，返回false。可能出现虚假失败，需放在循环中 |
| `compare_exchange_strong(expected, desired, order)` | 强比较交换：不会虚假失败，无需循环，性能略低于weak版本 |
| `fetch_add()`/`fetch_sub()` | 原子加减，返回旧值 |
| `fetch_and()`/`fetch_or()`/`fetch_xor()` | 原子位运算，返回旧值 |
| `operator++`/`operator--` | 重载自增自减运算符，原子操作 |

#### 基础用法示例
```cpp
#include <iostream>
#include <thread>
#include <atomic>

// 原子计数器，无需加锁，线程安全
std::atomic<int> count = 0;

void add() {
    for (int i = 0; i < 10000; i++) {
        count++; // 原子自增，无数据竞争
        // 等价于 count.fetch_add(1, std::memory_order_seq_cst);
    }
}

int main() {
    std::thread t1(add);
    std::thread t2(add);
    t1.join();
    t2.join();
    std::cout << "最终计数: " << count << std::endl; // 输出20000
    std::cout << "是否无锁: " << count.is_lock_free() << std::endl; // 绝大多数平台输出1
    return 0;
}
```

### 2. 内存顺序（Memory Order）
内存顺序是C++多线程最复杂的核心知识点，用于控制编译器和CPU对指令的重排，以及多线程间的内存可见性。

#### （1）为什么需要内存顺序
单线程环境下，编译器和CPU会对指令进行重排，只要保证单线程的执行结果不变，即可优化性能。但多线程环境下，指令重排会导致其他线程看到的内存访问顺序与预期不符，引发未定义行为。内存顺序就是用来约束重排行为，保证多线程间的内存可见性。

#### （2）6种内存顺序详解
C++标准定义了6种内存顺序，分为4大类，按性能从高到低（约束从弱到强）排序：

| 内存顺序 | 核心约束 | 适用场景 |
|----------|----------|----------|
| `std::memory_order_relaxed` | 松散序：仅保证操作的原子性，不保证任何指令重排约束，不保证多线程间的内存可见性 | 无需同步的原子操作，如独立的计数器、统计量 |
| `std::memory_order_consume` | 消费序：仅保证依赖于该原子变量的操作不能重排到它之前，仅保证依赖数据的可见性 | 极少使用，编译器大多将其实现为`acquire`，不推荐使用 |
| `std::memory_order_acquire` | 获取序：用于load读操作，本线程中所有后续的读写操作不能重排到该acquire操作之前；能看到对应release操作之前的所有写入 | 与release配对使用，实现线程间的同步 |
| `std::memory_order_release` | 释放序：用于store写操作，本线程中所有之前的读写操作不能重排到该release操作之后；写入的内容会被对应acquire操作的线程看到 | 与acquire配对使用，实现线程间的同步 |
| `std::memory_order_acq_rel` | 获取释放序：用于读-改-写操作（如exchange、fetch_add），同时具备acquire和release的约束 | 同时需要读写同步的场景，如自旋锁 |
| `std::memory_order_seq_cst` | 顺序一致性：默认内存序，最强约束。所有线程看到的所有原子操作的顺序完全一致，同时具备acq_rel的所有约束 | 绝大多数场景的首选，最安全，无需考虑重排问题，仅性能瓶颈时再优化 |

#### （3）核心同步关系
- **synchronizes-with**：一个线程的release操作，与另一个线程读取到该写入结果的acquire操作，形成同步关系。
- **happens-before**：C++内存模型的核心关系，若A操作happens-before B操作，则A操作的结果对B操作完全可见。

#### （4）acquire-release 配对使用示例
```cpp
#include <iostream>
#include <thread>
#include <atomic>
#include <chrono>

std::atomic<bool> ready = false;
int data = 0; // 非原子变量，通过acquire-release保证可见性

void writer() {
    data = 100; // 写操作，必须在release之前
    // release操作：保证之前的所有写入都不会重排到该操作之后
    ready.store(true, std::memory_order_release);
}

void reader() {
    // acquire操作：保证之后的所有读取都不会重排到该操作之前
    while (!ready.load(std::memory_order_acquire)) {
        std::this_thread::sleep_for(std::chrono::milliseconds(10));
    }
    // 一定能看到writer线程中data=100的写入
    std::cout << "data: " << data << std::endl;
}

int main() {
    std::thread t1(writer);
    std::thread t2(reader);
    t1.join();
    t2.join();
    return 0;
}
```

### 3. 无锁编程注意事项
1. **无锁编程门槛极高，极易出错**，仅在性能瓶颈明确、互斥量开销无法接受时使用，优先使用互斥量保证线程安全。
2. **ABA问题**：无锁编程中，变量的值从A变为B，又变回A，此时compare_exchange会认为值没有变化，导致错误。解决方案：使用带版本号的双字原子操作、避免重复使用内存地址。
3. **内存序误用**：弱内存序的使用需要极其谨慎，错误的内存序会导致同步失效，引发难以复现的bug，默认使用`seq_cst`。
4. **无锁编程无法解决所有同步问题**，复杂场景下，互斥量的可维护性和稳定性远优于无锁代码。

## 六、C++20+ 多线程新特性
C++20 对多线程库进行了重大升级，补充了大量实用的同步原语，简化了多线程编程的复杂度。

### 1. 信号量（semaphore）
`<semaphore>`头文件提供了轻量级信号量，用于限制并发访问的线程数量，核心是一个计数器，`acquire()`减少计数（计数为0时阻塞），`release()`增加计数。
- `std::counting_semaphore<N>`：计数信号量，最大计数为N。
- `std::binary_semaphore`：二进制信号量，最大计数为1，等价于`counting_semaphore<1>`，可实现轻量级互斥锁。

```cpp
#include <iostream>
#include <thread>
#include <semaphore>
#include <chrono>

// 限制最多2个线程同时执行
std::counting_semaphore<2> sem(2);

void task(int id) {
    sem.acquire(); // 获取信号量，计数-1，计数为0则阻塞
    std::cout << "线程" << id << "进入临界区" << std::endl;
    std::this_thread::sleep_for(std::chrono::milliseconds(500));
    std::cout << "线程" << id << "退出临界区" << std::endl;
    sem.release(); // 释放信号量，计数+1
}

int main() {
    std::thread threads[5];
    for (int i = 0; i < 5; i++) {
        threads[i] = std::thread(task, i+1);
    }
    for (auto& t : threads) {
        t.join();
    }
    return 0;
}
```

### 2. 闩锁（latch）与屏障（barrier）
用于实现多线程的协同等待，让多个线程到达指定点后再一起继续执行。

#### （1）std::latch
一次性的线程屏障，初始化时指定计数，线程调用`arrive_and_wait()`减少计数并等待，直到计数变为0，所有等待的线程被唤醒。**不可重置，用完即毁**。
适用场景：等待多个线程完成初始化后，主线程再继续执行。

#### （2）std::barrier
可复用的线程屏障，初始化时指定计数，所有线程到达后，执行可选的回调函数，然后唤醒所有线程，自动重置计数，可进入下一轮等待。
适用场景：多轮迭代的并行计算，如并行排序、矩阵运算。

### 3. 原子操作增强
1. 原子等待/通知机制：`std::atomic`新增`wait()`/`notify_one()`/`notify_all()`接口，无需条件变量即可实现线程间的等待-通知，简化无锁同步。
2. 扩展支持类型：支持浮点型原子操作、`std::shared_ptr`的原子操作。
3. 增强的原子位操作：新增`atomic_fetch_modify`等通用原子修改接口。

## 七、经典多线程设计模式与实现
### 1. 线程安全单例模式
C++11 之后，**局部静态变量的初始化是线程安全的**（Magic Static特性），是实现单例模式的最简、最优方案，无需手动加锁：
```cpp
#include <mutex>

class Singleton {
public:
    // 禁用拷贝和移动
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;
    Singleton(Singleton&&) = delete;
    Singleton& operator=(Singleton&&) = delete;

    // 全局唯一访问入口，线程安全
    static Singleton& get_instance() {
        static Singleton instance; // C++11后初始化线程安全
        return instance;
    }

private:
    // 私有构造函数
    Singleton() = default;
    ~Singleton() = default;
};
```

### 2. 线程池设计与实现
线程池是多线程开发中最常用的组件，核心是复用固定数量的线程处理任务队列，避免频繁创建销毁线程的开销，适用于大量短任务的并发处理。

核心组成：
1. 任务队列：存储待执行的任务，线程安全。
2. 工作线程组：固定数量的线程，循环从任务队列取任务执行。
3. 同步机制：互斥量+条件变量，实现任务的添加和线程的唤醒。
4. 停止机制：优雅关闭线程池，等待所有任务执行完毕。

### 3. 读写锁模型
基于`std::shared_mutex`实现读多写少场景的高并发访问，读操作使用共享锁，可并发执行；写操作使用独占锁，保证线程安全：
```cpp
#include <shared_mutex>
#include <unordered_map>
#include <string>

class Cache {
private:
    std::unordered_map<std::string, std::string> data;
    mutable std::shared_mutex mtx; // mutable允许const成员函数加锁

public:
    // 读操作：加共享锁，多个线程可同时读
    std::string get(const std::string& key) const {
        std::shared_lock<std::shared_mutex> lock(mtx);
        auto it = data.find(key);
        return it != data.end() ? it->second : "";
    }

    // 写操作：加独占锁，同一时间仅一个线程可写
    void set(const std::string& key, const std::string& value) {
        std::unique_lock<std::shared_mutex> lock(mtx);
        data[key] = value;
    }

    // 删除操作：加独占锁
    void remove(const std::string& key) {
        std::unique_lock<std::shared_mutex> lock(mtx);
        data.erase(key);
    }
};
```

## 八、多线程编程常见坑与避坑指南
| 坑点 | 触发场景 | 避坑方案 |
|------|----------|----------|
| joinable线程销毁导致程序terminate | std::thread对象销毁前未调用join/detach | 优先使用std::jthread，自动join；用RAII包装std::thread |
| 数据竞争与未定义行为 | 多个线程同时访问可变共享数据，无同步保护 | 共享数据必须用mutex保护，或使用原子操作；尽量减少共享可变数据 |
| 死锁 | 多线程循环等待锁、持有并等待、锁嵌套 | 优先使用scoped_lock；固定加锁顺序；避免锁嵌套；最小化临界区 |
| 虚假唤醒 | 条件变量wait未放在循环中，未使用谓词 | 必须使用带谓词的wait重载版本，每次唤醒都检查条件 |
| detach线程悬空引用 | detach线程引用了主线程的局部变量，变量销毁后线程仍在访问 | 非必要严禁使用detach；优先使用join；确保捕获的变量生命周期长于线程 |
| async异步变同步 | async返回的future未保存，临时对象析构阻塞；默认启动策略 | 必须保存async返回的future；明确指定std::launch::async策略 |
| 线程参数引用失效 | 线程函数参数为引用，未使用std::ref包装 | 引用传递必须用std::ref/cref；确保引用的变量生命周期 |
| 异常未捕获导致程序终止 | 线程函数抛出异常未捕获，会直接terminate | 线程函数内必须捕获所有异常；用future/promise跨线程传递异常 |
| 递归mutex滥用 | 递归锁导致代码逻辑混乱，死锁风险提升 | 尽量不使用递归mutex，重构代码避免递归加锁 |
| 临界区过大 | 锁的范围包含耗时操作、IO、回调，导致并发性能下降，死锁风险提升 | 最小化临界区，仅保护共享数据访问，非共享操作放在锁外 |

## 九、最佳实践与性能优化
### 1. 编码最佳实践
1. **优先使用高层抽象**：优先使用`std::async`、线程池，而非手动管理`std::thread`，减少手动管理的复杂度和出错概率。
2. **优先使用C++20 jthread**：替代原生`std::thread`，避免手动join的问题，内置中断机制，简化线程停止逻辑。
3. **最小化共享数据**：遵循“共享可变数据最小化”原则，优先使用消息传递（如任务队列）替代共享内存，从根源上减少同步需求。
4. **严禁手动lock/unlock**：始终使用RAII锁包装器（`lock_guard`、`scoped_lock`、`unique_lock`），确保异常安全。
5. **最小化临界区**：临界区仅包含必须的共享数据访问，不要在临界区执行耗时计算、IO操作、未知回调函数。
6. **异常安全**：确保所有线程内的异常都被捕获，跨线程异常传递使用`future`体系。
7. **避免无锁编程**：除非性能瓶颈明确，否则优先使用互斥量，无锁编程的调试和维护成本极高。
8. **线程数合理设置**：CPU密集型任务，线程数约等于CPU逻辑核心数（`hardware_concurrency()`）；IO密集型任务，线程数可适当增加，避免核心空闲。
9. **避免频繁创建销毁线程**：使用线程池复用线程，减少线程创建销毁的开销。

### 2. 性能优化技巧
1. **读写分离**：读多写少场景使用`std::shared_mutex`，大幅提升读并发性能。
2. **减少锁竞争**：批量处理数据，减少加锁解锁的次数；使用线程局部存储`thread_local`缓存数据，减少共享数据访问。
3. **锁粒度优化**：细粒度锁替代粗粒度锁，不同的共享数据使用独立的互斥量，减少锁竞争范围。
4. **避免伪共享**：多线程频繁访问的不同变量位于同一个CPU缓存行，导致缓存行频繁失效。使用`alignas(64)`对齐变量，确保每个变量独占一个缓存行。
5. **内存序优化**：性能瓶颈场景下，在保证同步正确的前提下，使用更弱的内存序（如acquire-release）替代默认的seq_cst，减少内存屏障的开销。
6. **无锁数据结构**：极端性能场景下，使用成熟的无锁队列、无锁哈希表，而非手动实现无锁逻辑。

### 3. 调试与问题排查工具
1. **ThreadSanitizer (TSAN)**：谷歌开发的线程检测工具，编译时添加`-fsanitize=thread`，可精准检测数据竞争、死锁、锁误用等问题，是多线程调试的首选工具。
2. **valgrind-helgrind/drd**：Linux平台下的多线程错误检测工具，可检测数据竞争、死锁等问题。
3. **Windows Application Verifier**：Windows平台下的应用程序验证工具，可检测多线程相关的错误。
4. **gdb/lldb**：支持多线程调试，可切换线程、查看线程栈、断点调试多线程代码。

## 十、平台相关扩展
C++标准库提供了跨平台的核心能力，部分平台特有能力需通过原生API实现：
1. **线程亲和性**：绑定线程到指定的CPU核心，减少线程切换开销，Linux使用`pthread_setaffinity_np`，Windows使用`SetThreadAffinityMask`。
2. **线程优先级**：设置线程的调度优先级，Linux使用`pthread_setschedparam`，Windows使用`SetThreadPriority`，**严禁依赖优先级保证同步逻辑**。
3. **线程栈大小**：设置线程的栈空间大小，标准库无统一接口，Linux通过`pthread_attr_setstacksize`设置，Windows通过`CreateThread`参数设置。
4. **线程强制终止**：标准库仅支持协作式中断，不支持强制终止线程，平台原生API（如`pthread_cancel`、`TerminateThread`）可强制终止，但会导致资源泄漏、死锁等问题，**严禁使用**。


