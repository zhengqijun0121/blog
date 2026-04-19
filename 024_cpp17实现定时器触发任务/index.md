# C++17 实现定时器触发任务


## C++17 定时器实现（支持单次/循环任务、线程安全）
该实现完全基于C++17标准特性，核心采用**单调时钟steady_clock**（避免系统时间修改导致的计时偏差）、优先队列管理任务（按触发时间排序，效率最优），支持线程安全的任务添加、取消、定时器启停，同时处理了任务异常、计时累积误差等常见问题。

---

### 完整代码实现
```cpp
#include <chrono>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <queue>
#include <functional>
#include <atomic>
#include <memory>
#include <unordered_map>
#include <stdexcept>

// 定时器类，C++17兼容，线程安全
class Timer {
public:
    // 构造函数：启动定时器工作线程
    Timer() {
        m_running.store(true);
        m_worker = std::thread(&Timer::worker_thread, this);
    }

    // 析构函数：自动停止定时器，释放线程资源
    ~Timer() {
        stop();
    }

    // 禁止拷贝、移动（mutex和thread不可拷贝/移动）
    Timer(const Timer&) = delete;
    Timer& operator=(const Timer&) = delete;
    Timer(Timer&&) = delete;
    Timer& operator=(Timer&&) = delete;

    /**
     * @brief 添加单次执行任务
     * @param delay 延迟执行时间（毫秒）
     * @param task 待执行的任务函数
     * @return 任务唯一ID，失败返回0
     */
    uint64_t add_once_task(std::chrono::milliseconds delay, std::function<void()> task) {
        if (!m_running.load() || !task) {
            return 0;
        }

        uint64_t task_id = m_next_task_id.fetch_add(1);
        auto trigger_time = std::chrono::steady_clock::now() + delay;
        auto is_valid = std::make_shared<std::atomic<bool>>(true);

        TimerTask new_task{
            task_id,
            trigger_time,
            std::chrono::milliseconds(0),
            false,
            std::move(task),
            is_valid
        };

        {
            std::lock_guard<std::mutex> lock(m_mutex);
            m_task_queue.push(std::move(new_task));
            m_task_valid_map[task_id] = is_valid;
        }
        m_cv.notify_one(); // 唤醒工作线程，更新等待时间
        return task_id;
    }

    /**
     * @brief 添加循环执行任务
     * @param interval 循环执行间隔（毫秒）
     * @param task 待执行的任务函数
     * @param first_delay 首次执行延迟（毫秒），默认0
     * @return 任务唯一ID，失败返回0
     */
    uint64_t add_repeat_task(std::chrono::milliseconds interval,
                              std::function<void()> task,
                              std::chrono::milliseconds first_delay = std::chrono::milliseconds(0)) {
        if (!m_running.load() || !task || interval.count() <= 0) {
            return 0;
        }

        uint64_t task_id = m_next_task_id.fetch_add(1);
        auto trigger_time = std::chrono::steady_clock::now() + first_delay;
        auto is_valid = std::make_shared<std::atomic<bool>>(true);

        TimerTask new_task{
            task_id,
            trigger_time,
            interval,
            true,
            std::move(task),
            is_valid
        };

        {
            std::lock_guard<std::mutex> lock(m_mutex);
            m_task_queue.push(std::move(new_task));
            m_task_valid_map[task_id] = is_valid;
        }
        m_cv.notify_one();
        return task_id;
    }

    /**
     * @brief 取消指定任务
     * @param task_id 任务ID
     * @return 取消成功返回true，任务不存在/已取消返回false
     */
    bool cancel_task(uint64_t task_id) {
        std::lock_guard<std::mutex> lock(m_mutex);
        auto it = m_task_valid_map.find(task_id);
        if (it == m_task_valid_map.end()) {
            return false;
        }

        // 标记任务无效（惰性删除，出队时才会清理）
        auto is_valid = it->second.lock();
        if (is_valid) {
            is_valid->store(false);
        }
        m_task_valid_map.erase(it);
        return true;
    }

    /**
     * @brief 停止定时器，终止工作线程，清空所有任务
     */
    void stop() {
        // 标记停止状态
        m_running.store(false);
        m_cv.notify_all(); // 唤醒所有等待，退出线程

        // 等待线程安全退出
        if (m_worker.joinable()) {
            m_worker.join();
        }

        // 清空剩余任务
        std::lock_guard<std::mutex> lock(m_mutex);
        while (!m_task_queue.empty()) {
            m_task_queue.pop();
        }
        m_task_valid_map.clear();
    }

private:
    // 定时任务结构体
    struct TimerTask {
        uint64_t task_id;                                  // 任务唯一ID
        std::chrono::steady_clock::time_point trigger_time; // 触发时间点
        std::chrono::milliseconds interval;                // 循环间隔
        bool is_repeat;                                     // 是否循环执行
        std::function<void()> task;                         // 任务函数
        std::shared_ptr<std::atomic<bool>> is_valid;       // 任务有效标记（用于取消）
    };

    // 优先队列比较器：构建小顶堆，最早触发的任务在堆顶
    struct TaskComparator {
        bool operator()(const TimerTask& a, const TimerTask& b) const {
            return a.trigger_time > b.trigger_time;
        }
    };

    // 工作线程核心循环
    void worker_thread() {
        while (m_running.load()) {
            std::unique_lock<std::mutex> lock(m_mutex);

            // 等待：队列非空 或 定时器停止（处理虚假唤醒）
            m_cv.wait(lock, [this]() {
                return !m_task_queue.empty() || !m_running.load();
            });

            // 定时器停止，退出循环
            if (!m_running.load()) {
                break;
            }

            // 检查堆顶任务是否到触发时间
            const auto& top_task = m_task_queue.top();
            auto now = std::chrono::steady_clock::now();

            // 未到触发时间，等待到触发时间点（或被新任务/停止唤醒）
            if (top_task.trigger_time > now) {
                m_cv.wait_until(lock, top_task.trigger_time);
                continue; // 等待结束后重新校验队列状态
            }

            // 取出到期任务
            auto task = std::move(m_task_queue.top());
            m_task_queue.pop();

            // 执行任务前解锁，避免任务执行阻塞其他操作
            lock.unlock();

            // 执行有效任务，捕获异常避免线程崩溃
            if (task.is_valid->load()) {
                try {
                    task.task();
                } catch (...) {
                    // 可在此处添加异常日志输出
                }
            }

            // 循环任务：更新触发时间，重新入队（无累积误差）
            if (task.is_repeat && task.is_valid->load()) {
                // 用原触发时间+间隔，避免任务执行耗时导致的计时偏差
                task.trigger_time += task.interval;
                {
                    std::lock_guard<std::mutex> lock2(m_mutex);
                    m_task_queue.push(std::move(task));
                }
                m_cv.notify_one();
            } else {
                // 单次任务/已取消任务，清理映射表
                std::lock_guard<std::mutex> lock2(m_mutex);
                m_task_valid_map.erase(task.task_id);
            }
        }
    }

private:
    std::priority_queue<TimerTask, std::vector<TimerTask>, TaskComparator> m_task_queue;
    std::mutex m_mutex;
    std::condition_variable m_cv;
    std::thread m_worker;
    std::atomic<bool> m_running{false};               // 定时器运行状态
    std::atomic<uint64_t> m_next_task_id{1};          // 自增任务ID（0为无效ID）
    std::unordered_map<uint64_t, std::weak_ptr<std::atomic<bool>>> m_task_valid_map;
};

// 测试示例
#include <iostream>
int main() {
    Timer timer;

    // 1. 添加单次任务：1秒后执行
    timer.add_once_task(std::chrono::milliseconds(1000), []() {
        std::cout << "[单次任务] 执行完成，时间戳："
                  << std::chrono::duration_cast<std::chrono::milliseconds>(
                      std::chrono::steady_clock::now().time_since_epoch()
                  ).count() << std::endl;
    });

    // 2. 添加循环任务：每500ms执行一次，无首次延迟
    auto repeat_task_id = timer.add_repeat_task(std::chrono::milliseconds(500), []() {
        std::cout << "[循环任务] 执行完成，时间戳："
                  << std::chrono::duration_cast<std::chrono::milliseconds>(
                      std::chrono::steady_clock::now().time_since_epoch()
                  ).count() << std::endl;
    });

    // 3秒后取消循环任务
    std::this_thread::sleep_for(std::chrono::seconds(3));
    timer.cancel_task(repeat_task_id);
    std::cout << ">>> 已取消循环任务 <<<" << std::endl;

    // 再运行2秒后停止定时器
    std::this_thread::sleep_for(std::chrono::seconds(2));
    timer.stop();
    std::cout << ">>> 定时器已停止 <<<" << std::endl;

    return 0;
}
```

---

### 核心特性说明
1. **C++17标准兼容**：所有特性均为C++17原生支持，无第三方依赖，兼容C++11/14/17标准
2. **高精度计时**：采用`std::chrono::steady_clock`单调时钟，不受系统时间修改、时区调整影响，无计时漂移
3. **无累积误差**：循环任务采用「原触发时间+间隔」的方式更新下一次执行时间，避免任务执行耗时导致的计时偏差
4. **线程安全**：所有对外接口均加锁保护，支持多线程并发添加、取消任务
5. **异常安全**：任务执行时捕获所有异常，避免单个任务异常导致定时器线程崩溃
6. **资源安全**：析构函数自动停止线程并释放资源，无内存泄漏、野线程问题

---

### 使用注意事项
1. **任务耗时限制**：定时器为单线程执行，若任务执行耗时过长，会阻塞后续任务的触发。耗时任务建议在任务内部开启异步线程/线程池执行
2. **时间精度**：计时精度取决于操作系统，Linux系统通常可达1ms，Windows系统默认精度为10-15ms
3. **任务取消**：采用惰性删除机制，取消任务仅标记为无效，不会立即从队列中删除，待任务到期出队时自动清理，不影响正常使用
4. **生命周期**：定时器对象销毁时会自动停止所有任务，请勿在任务中直接销毁定时器对象本身

---

## 核心实现原理
本方案基于Linux特有的`epoll`（IO多路复用）、`timerfd`（定时器文件描述符）、`eventfd`（事件唤醒）实现，结合C++17特性完成线程安全的定时器任务调度，核心设计如下：
1.  **单timerfd+最小堆**：仅用1个定时器fd，通过优先队列（最小堆）管理所有定时任务，避免大量fd占用，保证每次取到最早到期的任务。
2.  **单调时钟**：使用`std::chrono::steady_clock`，不受系统时间跳变影响，保证定时器精度。
3.  **eventfd唤醒机制**：解决epoll阻塞时新增更早任务无法及时生效的问题，同时用于安全停止事件循环。
4.  **线程安全**：互斥锁保护任务队列操作，支持多线程添加/取消任务；回调执行时解锁，避免死锁。
5.  **ET边缘触发+非阻塞IO**：降低epoll事件触发频率，提升性能，避免水平触发的持续通知问题。
6.  **惰性删除**：通过有效标记实现任务取消，解决优先队列不支持随机删除的问题。

---

### 完整C++17实现代码
```cpp
#include <sys/epoll.h>
#include <sys/timerfd.h>
#include <sys/eventfd.h>
#include <unistd.h>
#include <fcntl.h>
#include <cerrno>
#include <cstring>
#include <chrono>
#include <functional>
#include <queue>
#include <mutex>
#include <atomic>
#include <memory>
#include <unordered_map>
#include <optional>
#include <vector>
#include <iostream>
#include <thread>

class TimerEpoll {
public:
    using Clock = std::chrono::steady_clock;
    using TimePoint = Clock::time_point;
    using Duration = Clock::duration;
    using TaskCallback = std::function<void()>;

    // 构造函数：初始化epoll、timerfd、eventfd
    TimerEpoll() : is_running_(false), next_task_id_(1) {
        // 1. 创建epoll实例，EPOLL_CLOEXEC避免fd泄漏
        epoll_fd_ = epoll_create1(EPOLL_CLOEXEC);
        if (epoll_fd_ < 0) {
            throw std::runtime_error("epoll_create1 failed: " + std::string(strerror(errno)));
        }

        // 2. 创建timerfd：单调时钟、非阻塞、CLOEXEC
        timer_fd_ = timerfd_create(CLOCK_MONOTONIC, TFD_NONBLOCK | TFD_CLOEXEC);
        if (timer_fd_ < 0) {
            close(epoll_fd_);
            throw std::runtime_error("timerfd_create failed: " + std::string(strerror(errno)));
        }

        // 3. 创建eventfd：用于唤醒epoll_wait，非阻塞、CLOEXEC
        wakeup_fd_ = eventfd(0, EFD_NONBLOCK | EFD_CLOEXEC);
        if (wakeup_fd_ < 0) {
            close(timer_fd_);
            close(epoll_fd_);
            throw std::runtime_error("eventfd_create failed: " + std::string(strerror(errno)));
        }

        // 4. 注册timerfd到epoll：边缘触发ET，监听可读事件
        epoll_event ev{};
        ev.events = EPOLLIN | EPOLLET;
        ev.data.fd = timer_fd_;
        if (epoll_ctl(epoll_fd_, EPOLL_CTL_ADD, timer_fd_, &ev) < 0) {
            close(wakeup_fd_);
            close(timer_fd_);
            close(epoll_fd_);
            throw std::runtime_error("epoll_ctl add timerfd failed: " + std::string(strerror(errno)));
        }

        // 5. 注册wakeup_fd到epoll：边缘触发ET，监听可读事件
        ev.data.fd = wakeup_fd_;
        if (epoll_ctl(epoll_fd_, EPOLL_CTL_ADD, wakeup_fd_, &ev) < 0) {
            close(wakeup_fd_);
            close(timer_fd_);
            close(epoll_fd_);
            throw std::runtime_error("epoll_ctl add wakeup_fd failed: " + std::string(strerror(errno)));
        }
    }

    // 析构函数：停止循环，释放fd资源
    ~TimerEpoll() {
        Stop();
        close(wakeup_fd_);
        close(timer_fd_);
        close(epoll_fd_);
    }

    // 禁止拷贝构造和赋值
    TimerEpoll(const TimerEpoll&) = delete;
    TimerEpoll& operator=(const TimerEpoll&) = delete;

    // 启动事件循环（阻塞运行，建议在独立线程中调用）
    void Start() {
        if (is_running_.exchange(true)) {
            return; // 已在运行，直接返回
        }

        constexpr int MAX_EVENTS = 16;
        epoll_event events[MAX_EVENTS];

        while (is_running_.load()) {
            // 永久阻塞，直到有事件触发
            int nfds = epoll_wait(epoll_fd_, events, MAX_EVENTS, -1);
            if (nfds < 0) {
                if (errno == EINTR) continue; // 被信号中断，继续循环
                throw std::runtime_error("epoll_wait failed: " + std::string(strerror(errno)));
            }

            // 遍历处理所有触发的事件
            for (int i = 0; i < nfds; ++i) {
                const auto& ev = events[i];
                if (ev.data.fd == timer_fd_) {
                    HandleTimerFdEvent(); // 处理定时器到期事件
                } else if (ev.data.fd == wakeup_fd_) {
                    HandleWakeupFdEvent(); // 处理唤醒事件
                }
            }

            // 重新设置timerfd的超时时间
            ResetTimerFd();
        }
    }

    // 停止事件循环
    void Stop() {
        if (!is_running_.exchange(false)) return;
        Wakeup(); // 唤醒epoll_wait，让循环立即退出
    }

    // 添加一次性定时任务，返回任务ID（用于取消），失败返回nullopt
    std::optional<uint64_t> AddTask(Duration delay, TaskCallback callback) {
        if (!callback || !is_running_.load()) return std::nullopt;
        TimePoint expire_time = Clock::now() + delay;
        return AddTaskInternal(expire_time, Duration::zero(), std::move(callback));
    }

    // 添加周期性定时任务，返回任务ID（用于取消），失败返回nullopt
    std::optional<uint64_t> AddPeriodicTask(Duration period, TaskCallback callback) {
        if (!callback || period <= Duration::zero() || !is_running_.load()) return std::nullopt;
        TimePoint first_expire = Clock::now() + period;
        return AddTaskInternal(first_expire, period, std::move(callback));
    }

    // 取消指定ID的任务，成功返回true，失败返回false
    bool CancelTask(uint64_t task_id) {
        std::lock_guard<std::mutex> lock(mutex_);
        auto it = task_map_.find(task_id);
        if (it == task_map_.end()) return false;
        it->second->is_valid = false;
        Wakeup(); // 唤醒epoll，重新处理队列
        return true;
    }

private:
    // 定时任务结构体
    struct TimerTask {
        uint64_t task_id;
        TimePoint expire_time;
        Duration period;
        TaskCallback callback;
        bool is_valid;

        TimerTask(uint64_t id, TimePoint expire, Duration p, TaskCallback cb)
            : task_id(id), expire_time(expire), period(p), callback(std::move(cb)), is_valid(true) {}
    };

    // 优先队列比较器，实现最小堆（最早到期的任务在队首）
    struct TaskComparator {
        bool operator()(const std::shared_ptr<TimerTask>& a, const std::shared_ptr<TimerTask>& b) const {
            return a->expire_time > b->expire_time;
        }
    };

    // 任务添加内部实现
    std::optional<uint64_t> AddTaskInternal(TimePoint expire_time, Duration period, TaskCallback callback) {
        uint64_t task_id = next_task_id_++;
        auto task = std::make_shared<TimerTask>(task_id, expire_time, period, std::move(callback));

        {
            std::lock_guard<std::mutex> lock(mutex_);
            task_map_.emplace(task_id, task);
            task_queue_.push(task);
        }

        Wakeup(); // 唤醒epoll，更新定时器超时时间
        return task_id;
    }

    // 处理timerfd可读事件
    void HandleTimerFdEvent() {
        // ET模式必须读完所有数据，直到EAGAIN
        uint64_t count;
        while (read(timer_fd_, &count, sizeof(count)) > 0);
        // 处理所有到期任务
        HandleExpiredTasks();
    }

    // 处理wakeup_fd可读事件
    void HandleWakeupFdEvent() {
        uint64_t count;
        while (read(wakeup_fd_, &count, sizeof(count)) > 0);
    }

    // 唤醒epoll_wait
    void Wakeup() {
        uint64_t one = 1;
        write(wakeup_fd_, &one, sizeof(one));
    }

    // 处理所有到期的任务
    void HandleExpiredTasks() {
        std::lock_guard<std::mutex> lock(mutex_);
        TimePoint now = Clock::now();

        // 遍历所有到期的任务
        while (!task_queue_.empty()) {
            auto top_task = task_queue_.top();
            // 队首任务未到期，退出循环
            if (top_task->expire_time > now) break;
            task_queue_.pop();

            // 惰性删除：跳过已取消的无效任务
            if (!top_task->is_valid) {
                task_map_.erase(top_task->task_id);
                continue;
            }

            // 拷贝回调，提前处理周期性任务的重入
            TaskCallback callback = top_task->callback;
            bool is_periodic = top_task->period > Duration::zero();

            if (is_periodic) {
                // 周期性任务：更新到期时间，重新入队
                top_task->expire_time = now + top_task->period;
                task_queue_.push(top_task);
            } else {
                // 一次性任务：从map中移除
                task_map_.erase(top_task->task_id);
            }

            // 解锁后执行回调，避免死锁和阻塞队列操作
            mutex_.unlock();
            try {
                callback();
            } catch (...) {
                std::cerr << "Task callback exception, task_id: " << top_task->task_id << std::endl;
            }
            mutex_.lock(); // 重新加锁，继续处理队列
        }
    }

    // 重新设置timerfd的超时时间
    void ResetTimerFd() {
        std::lock_guard<std::mutex> lock(mutex_);
        itimerspec new_spec{};

        // 无任务时，关闭定时器
        if (task_queue_.empty()) {
            timerfd_settime(timer_fd_, 0, &new_spec, nullptr);
            return;
        }

        // 取队首任务的到期时间，设置为绝对超时时间
        auto top_task = task_queue_.top();
        auto expire_sec = std::chrono::duration_cast<std::chrono::seconds>(top_task->expire_time.time_since_epoch());
        auto expire_nsec = std::chrono::duration_cast<std::chrono::nanoseconds>(top_task->expire_time.time_since_epoch() - expire_sec);

        new_spec.it_value.tv_sec = expire_sec.count();
        new_spec.it_value.tv_nsec = expire_nsec.count();
        new_spec.it_interval.tv_sec = 0;
        new_spec.it_interval.tv_nsec = 0;

        // TFD_TIMER_ABSTIME：使用绝对时间，匹配单调时钟
        if (timerfd_settime(timer_fd_, TFD_TIMER_ABSTIME, &new_spec, nullptr) < 0) {
            std::cerr << "timerfd_settime failed: " << strerror(errno) << std::endl;
        }
    }

private:
    int epoll_fd_ = -1;
    int timer_fd_ = -1;
    int wakeup_fd_ = -1;
    std::atomic<bool> is_running_;
    std::atomic<uint64_t> next_task_id_;

    std::mutex mutex_;
    // 任务最小堆：按到期时间升序排列
    std::priority_queue<std::shared_ptr<TimerTask>, std::vector<std::shared_ptr<TimerTask>>, TaskComparator> task_queue_;
    // 任务ID映射：用于快速取消任务
    std::unordered_map<uint64_t, std::shared_ptr<TimerTask>> task_map_;
};

// 测试代码
int main() {
    TimerEpoll timer;

    // 启动定时器事件循环，在独立线程运行
    std::thread timer_thread([&timer]() {
        std::cout << "Timer event loop start" << std::endl;
        timer.Start();
        std::cout << "Timer event loop stop" << std::endl;
    });

    // 等待循环启动
    std::this_thread::sleep_for(std::chrono::milliseconds(100));

    // 1. 添加1秒后执行的一次性任务
    timer.AddTask(std::chrono::seconds(1), []() {
        std::cout << "One-shot task: 1s expired, timestamp: "
                  << std::chrono::duration_cast<std::chrono::milliseconds>(
                      std::chrono::steady_clock::now().time_since_epoch()
                  ).count() << std::endl;
    });

    // 2. 添加3秒后执行的一次性任务（后续会取消）
    auto task_to_cancel = timer.AddTask(std::chrono::seconds(3), []() {
        std::cout << "This task will be canceled, will not print" << std::endl;
    });

    // 3. 添加500ms周期执行的任务
    auto periodic_task = timer.AddPeriodicTask(std::chrono::milliseconds(500), []() {
        static int count = 0;
        std::cout << "Periodic task: count=" << ++count << ", timestamp: "
                  << std::chrono::duration_cast<std::chrono::milliseconds>(
                      std::chrono::steady_clock::now().time_since_epoch()
                  ).count() << std::endl;
    });

    // 2秒后取消3秒的一次性任务
    std::this_thread::sleep_for(std::chrono::seconds(2));
    if (task_to_cancel.has_value()) {
        bool ret = timer.CancelTask(task_to_cancel.value());
        std::cout << "Cancel task " << task_to_cancel.value() << ", result: " << (ret ? "success" : "fail") << std::endl;
    }

    // 再运行3秒后，取消周期性任务
    std::this_thread::sleep_for(std::chrono::seconds(3));
    if (periodic_task.has_value()) {
        bool ret = timer.CancelTask(periodic_task.value());
        std::cout << "Cancel periodic task " << periodic_task.value() << ", result: " << (ret ? "success" : "fail") << std::endl;
    }

    // 再运行2秒后退出程序
    std::this_thread::sleep_for(std::chrono::seconds(2));
    timer.Stop();
    timer_thread.join();

    return 0;
}
```

---

### 编译与运行
1.  **环境要求**：Linux系统（内核≥2.6.25，支持timerfd、eventfd、epoll），编译器支持C++17标准（g++7+、clang++5+）。
2.  **编译命令**：
    ```bash
    g++ -std=c++17 timer_epoll.cpp -o timer_epoll -lpthread
    ```
3.  **运行命令**：
    ```bash
    ./timer_epoll
    ```

---

### 关键说明与注意事项
1.  **C++17特性使用**
    - `std::optional`：安全处理任务添加的返回值，避免无效ID的误用。
    - `std::atomic`：无锁保证循环状态和任务ID的线程安全。
    - 移动语义：`std::move`转移回调函数，减少拷贝开销。
    - 类型安全的时间处理：`std::chrono`系列接口，避免原始时间类型的转换错误。

2.  **核心约束与优化**
    - **回调执行**：用户回调应避免耗时操作，否则会阻塞事件循环导致定时器延迟；耗时任务建议丢入线程池执行。
    - **时间精度**：默认毫秒级精度，微秒级高精度需要配合Linux实时内核和线程优先级调整。
    - **平台限制**：仅支持Linux系统，epoll/timerfd/eventfd均为Linux特有API，Windows/macOS不兼容。
    - **线程安全**：`AddTask`/`CancelTask`支持多线程并发调用，`Start`仅允许在单线程运行。

3.  **可扩展方向**
    - 集成线程池，处理耗时回调任务；
    - 支持任务暂停/恢复、优先级调度；
    - 增加任务执行完成的回调通知；
    - 优化无效任务的清理机制，减少堆内存占用。

