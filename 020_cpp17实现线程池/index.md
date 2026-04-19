# C++17 实现线程池


### C++17 线程池完整实现
该实现基于C++17标准特性，支持任意可调用对象、带返回值任务、线程安全的任务调度与优雅退出，兼容主流编译器（GCC/Clang/MSVC）。

#### 完整代码
```cpp
#include <vector>
#include <queue>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <future>
#include <functional>
#include <stdexcept>
#include <atomic>

// C++17 线程池实现
class ThreadPool {
public:
    // 构造函数：指定线程数，默认使用CPU核心数
    explicit ThreadPool(size_t num_threads = std::thread::hardware_concurrency())
        : stop(false) {
        if (num_threads == 0) {
            throw std::invalid_argument("Thread number must be greater than 0");
        }

        // 初始化工作线程
        workers.reserve(num_threads);
        for (size_t i = 0; i < num_threads; ++i) {
            workers.emplace_back([this]() {
                // 工作线程主循环
                for (;;) {
                    std::function<void()> task;

                    // 加锁访问任务队列
                    {
                        std::unique_lock<std::mutex> lock(queue_mutex);
                        // 等待条件：线程池停止 或 队列非空
                        cv.wait(lock, [this]() {
                            return stop.load() || !tasks.empty();
                        });

                        // 线程池停止且无剩余任务，退出线程
                        if (stop.load() && tasks.empty()) {
                            return;
                        }

                        // 取出队首任务
                        task = std::move(tasks.front());
                        tasks.pop();
                    }

                    // 执行任务（解锁后执行，避免锁占用过久）
                    task();
                }
            });
        }
    }

    // 禁用拷贝与移动（线程池不可拷贝/移动）
    ThreadPool(const ThreadPool&) = delete;
    ThreadPool& operator=(const ThreadPool&) = delete;
    ThreadPool(ThreadPool&&) = delete;
    ThreadPool& operator=(ThreadPool&&) = delete;

    // 析构函数：优雅关闭所有线程
    ~ThreadPool() {
        // 原子设置停止标志
        stop.store(true);
        // 唤醒所有等待的线程
        cv.notify_all();
        // 等待所有线程执行完毕
        for (std::thread& worker : workers) {
            if (worker.joinable()) {
                worker.join();
            }
        }
    }

    // 提交任务到线程池，返回future用于获取返回值
    template<typename F, typename... Args>
    auto submit(F&& func, Args&&... args)
        -> std::future<std::invoke_result_t<F, Args...>> {

        using ReturnType = std::invoke_result_t<F, Args...>;

        // 包装任务为packaged_task，支持获取返回值
        // 使用shared_ptr解决packaged_task不可拷贝的问题
        auto task = std::make_shared<std::packaged_task<ReturnType()>>(
            std::bind(std::forward<F>(func), std::forward<Args>(args)...)
        );

        std::future<ReturnType> result_future = task->get_future();

        // 加锁将任务加入队列
        {
            std::unique_lock<std::mutex> lock(queue_mutex);
            // 线程池停止后禁止提交新任务
            if (stop.load()) {
                throw std::runtime_error("Submit task to stopped ThreadPool");
            }
            tasks.emplace([task]() { (*task)(); });
        }

        // 唤醒一个等待的工作线程
        cv.notify_one();

        return result_future;
    }

private:
    // 工作线程数组
    std::vector<std::thread> workers;
    // 任务队列
    std::queue<std::function<void()>> tasks;

    // 同步原语
    std::mutex queue_mutex;
    std::condition_variable cv;
    // 线程池停止标志（原子类型保证多线程可见性）
    std::atomic<bool> stop;
};

// ------------------- 测试示例 -------------------
#include <iostream>
#include <chrono>

int main() {
    // 创建4线程的线程池
    ThreadPool pool(4);

    // 1. 提交无返回值任务
    pool.submit([]() {
        std::cout << "Hello from thread: " << std::this_thread::get_id() << "\n";
    });

    // 2. 提交带参数、有返回值的任务
    auto add_future = pool.submit([](int a, int b) { return a + b; }, 2, 3);
    std::cout << "2 + 3 = " << add_future.get() << "\n";

    // 3. 批量提交任务
    std::vector<std::future<int>> futures;
    futures.reserve(10);
    for (int i = 0; i < 10; ++i) {
        futures.emplace_back(pool.submit([i]() {
            std::this_thread::sleep_for(std::chrono::milliseconds(100));
            return i * i;
        }));
    }

    // 获取批量任务结果
    std::cout << "Batch task results: ";
    for (auto& fut : futures) {
        std::cout << fut.get() << " ";
    }
    std::cout << "\n";

    return 0;
}
```

---

### 核心特性与实现说明
#### 1. C++17 特性使用
- `std::invoke_result_t`：替代C++11弃用的`std::result_of`，安全推导可调用对象的返回值类型
- 完美转发与折叠表达式：支持任意数量、任意类型的参数传递
- `std::atomic<bool>`：保证停止标志的多线程内存可见性，避免指令重排导致的异常

#### 2. 线程安全设计
- 任务队列的所有访问都通过`std::mutex`加锁保护
- `std::condition_variable`实现线程的等待/唤醒机制，避免忙等占用CPU
- 任务执行在锁外完成，最大程度减少锁的持有时间，提升并发性能

#### 3. 任务调度机制
- 支持任意可调用对象（普通函数、lambda、函数对象、类成员函数）
- 通过`std::packaged_task` + `std::future`实现任务返回值的传递与异常捕获
- 任务队列采用FIFO调度策略，唤醒单个线程处理新任务，避免惊群效应

#### 4. 优雅退出机制
- 析构函数先设置停止标志，再唤醒所有线程
- 工作线程会执行完队列中所有剩余任务后再退出，不会丢弃已提交的任务
- 通过`join()`等待所有线程正常退出，避免资源泄漏与僵尸线程

---

### 编译与使用说明
1. **编译要求**：启用C++17及以上标准，Linux下需链接`pthread`库
   ```bash
   # GCC/Clang 编译命令
   g++ -std=c++17 -pthread main.cpp -o thread_pool
   # MSVC 编译命令
   cl /std:c++17 main.cpp
   ```
2. **使用注意事项**
   - 禁止在线程池任务中提交子任务并同步等待结果，可能导致死锁（所有线程都被阻塞，子任务无法执行）
   - 任务中抛出的异常会被封装在`future`中，调用`get()`时会触发异常，需用户自行捕获处理
   - 线程池对象的生命周期必须长于所有提交的任务，否则会触发未定义行为
   - 线程数建议设置为CPU核心数（默认值），CPU密集型任务不宜超过核心数，IO密集型任务可适当增加


