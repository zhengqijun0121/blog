# C++17 实现内存池


# C++17 内存池实现
本文基于C++17标准特性，实现了**固定大小内存池**（最常用、性能最优的内存池类型），包含单线程基础版、类型化对象池、线程安全版，同时讲解核心原理、C++17特性的应用，以及标准库内置方案。

## 一、内存池核心原理
内存池的核心目标是**减少频繁`new/delete`的系统调用开销、缓解内存碎片、提升内存分配/释放速度**，核心设计：
1. 预分配一大块连续内存（内存块`Block`），切分为多个固定大小的内存槽`Slot`
2. 用**嵌入式空闲链表**管理空闲槽（无需额外容器，直接用空闲槽存储下一个空闲槽的地址）
3. 内存耗尽时自动扩容（分配新的内存块）
4. 析构时批量释放所有内存块，避免内存泄漏

## 二、C++17 完整实现
### 2.1 基础工具与编译期计算
首先实现编译期对齐计算函数，以及依赖的标准头文件，完全基于C++17标准：
```cpp
#include <cstddef>      // std::size_t、std::align_val_t
#include <cstdlib>      // std::aligned_alloc、std::free
#include <new>          // std::bad_alloc、std::launder
#include <type_traits>  // std::is_trivially_destructible、if constexpr
#include <stdexcept>    // std::runtime_error
#include <mutex>        // 线程安全版本使用
#include <utility>      // std::forward

// 编译期向上对齐计算（C++17 constexpr支持）
// 要求Align必须是2的幂（符合C++标准对齐要求）
template <std::size_t Size, std::size_t Align>
constexpr std::size_t AlignUp() noexcept {
    static_assert((Align & (Align - 1)) == 0, "Align must be power of 2");
    return (Size + Align - 1) & ~(Align - 1);
}
```

### 2.2 单线程固定大小内存池（核心实现）
纯原始内存管理，无类型绑定，适用于通用固定大小内存分配，完全使用C++17特性优化：
```cpp
template <
    std::size_t SlotSize,          // 单个槽的大小
    std::size_t SlotAlign = alignof(std::max_align_t),  // 槽的对齐要求，默认最大基本对齐
    std::size_t SlotsPerBlock = 1024  // 每个内存块包含的槽数量（扩容粒度）
>
class FixedMemoryPool {
private:
    // 内存块结构体：串联所有内存块，用于析构时批量释放
    struct Block {
        Block* next;
    };

    // 编译期计算核心参数（零运行时开销）
    // 实际槽大小：至少能存下一个指针（空闲链表用），且满足对齐要求
    static constexpr std::size_t RealSlotSize = AlignUp<std::max(SlotSize, sizeof(void*)), SlotAlign>();
    // 单个内存块的数据区总大小
    static constexpr std::size_t BlockDataSize = RealSlotSize * SlotsPerBlock;
    // 内存块的对齐要求：取Block自身对齐和槽对齐的最大值
    static constexpr std::size_t BlockAlign = std::max(SlotAlign, alignof(Block));

    Block* m_block_list = nullptr;  // 所有内存块的链表头
    void* m_free_list = nullptr;     // 空闲槽链表头
    std::size_t m_free_slots = 0;    // 剩余空闲槽总数（统计用）

    // 扩容：分配新的内存块，并初始化空闲链表
    bool grow() {
        // 计算内存块总大小（头部+数据区，保证对齐）
        constexpr std::size_t block_header_size = AlignUp<sizeof(Block), BlockAlign>();
        constexpr std::size_t total_block_size = block_header_size + BlockDataSize;

        // C++17标准对齐内存分配，替代平台相关的posix_memalign/_aligned_malloc
        void* block_ptr = std::aligned_alloc(BlockAlign, total_block_size);
        if (!block_ptr) {
            return false;
        }

        // placement new构造内存块头部
        Block* new_block = new (block_ptr) Block;
        new_block->next = m_block_list;
        m_block_list = new_block;

        // 计算数据区起始地址
        std::byte* data_start = static_cast<std::byte*>(block_ptr) + block_header_size;

        // 初始化空闲链表：把所有槽串联起来
        for (std::size_t i = 0; i < SlotsPerBlock; ++i) {
            void* slot = data_start + i * RealSlotSize;
            *static_cast<void**>(slot) = static_cast<std::byte*>(slot) + RealSlotSize;
        }
        // 最后一个槽的next置空
        *static_cast<void**>(data_start + (SlotsPerBlock - 1) * RealSlotSize) = nullptr;

        // 更新空闲链表
        m_free_list = data_start;
        m_free_slots += SlotsPerBlock;

        return true;
    }

public:
    // 构造与析构
    FixedMemoryPool() = default;
    ~FixedMemoryPool() {
        // 遍历释放所有内存块
        Block* curr = m_block_list;
        while (curr) {
            Block* next = curr->next;
            curr->~Block();  // 手动析构placement new的对象
            std::free(curr); // aligned_alloc分配的内存必须用free释放
            curr = next;
        }
    }

    // 禁用拷贝与移动（避免双重释放）
    FixedMemoryPool(const FixedMemoryPool&) = delete;
    FixedMemoryPool& operator=(const FixedMemoryPool&) = delete;
    FixedMemoryPool(FixedMemoryPool&&) = delete;
    FixedMemoryPool& operator=(FixedMemoryPool&&) = delete;

    // 分配一个槽的内存
    [[nodiscard]] void* allocate() {
        if (!m_free_list) {
            if (!grow()) {
                throw std::bad_alloc();
            }
        }
        // 取出空闲链表头节点
        void* slot = m_free_list;
        m_free_list = *static_cast<void**>(slot);
        --m_free_slots;
        return slot;
    }

    // 归还内存到内存池
    void deallocate(void* ptr) noexcept {
        if (!ptr) return;
        // 插入到空闲链表头部（O(1)复杂度）
        *static_cast<void**>(ptr) = m_free_list;
        m_free_list = ptr;
        ++m_free_slots;
    }

    // 统计辅助函数
    std::size_t free_slots() const noexcept { return m_free_slots; }
    std::size_t total_slots() const noexcept {
        std::size_t count = 0;
        Block* curr = m_block_list;
        while (curr) {
            count += SlotsPerBlock;
            curr = curr->next;
        }
        return count;
    }
};
```

### 2.3 类型化对象池（上层封装）
基于原始内存池，封装为针对具体类型的对象池，自动处理对象的构造与析构，利用C++17特性做零成本优化：
```cpp
template <typename T, std::size_t SlotsPerBlock = 1024>
class ObjectPool {
public:
    using value_type = T;

    // 构造与析构
    ObjectPool() = default;
    ~ObjectPool() = default;

    // 禁用拷贝与移动
    ObjectPool(const ObjectPool&) = delete;
    ObjectPool& operator=(const ObjectPool&) = delete;
    ObjectPool(ObjectPool&&) = delete;
    ObjectPool& operator=(ObjectPool&&) = delete;

    // 构造对象：完美转发参数，支持任意构造函数
    template <typename... Args>
    [[nodiscard]] T* construct(Args&&... args) {
        void* slot = m_pool.allocate();
        // placement new构造对象
        T* obj = new (slot) T(std::forward<Args>(args)...);
        // C++17 std::launder：解决对象生命周期带来的指针别名问题
        return std::launder(obj);
    }

    // 销毁对象：自动调用析构函数，归还内存
    void destroy(T* obj) noexcept {
        if (!obj) return;
        // C++17 if constexpr：平凡析构类型直接跳过析构调用，零成本优化
        if constexpr (!std::is_trivially_destructible_v<T>) {
            obj->~T();
        }
        m_pool.deallocate(obj);
    }

    // 统计辅助函数
    std::size_t free_count() const noexcept { return m_pool.free_slots(); }
    std::size_t total_count() const noexcept { return m_pool.total_slots(); }

private:
    FixedMemoryPool<sizeof(T), alignof(T), SlotsPerBlock> m_pool;
};
```

### 2.4 线程安全版本
在基础内存池上增加互斥锁，保证多线程环境下的分配/释放安全，适用于多线程场景：
```cpp
template <
    std::size_t SlotSize,
    std::size_t SlotAlign = alignof(std::max_align_t),
    std::size_t SlotsPerBlock = 1024
>
class ThreadSafeFixedMemoryPool {
private:
    struct Block {
        Block* next;
    };

    static constexpr std::size_t RealSlotSize = AlignUp<std::max(SlotSize, sizeof(void*)), SlotAlign>();
    static constexpr std::size_t BlockDataSize = RealSlotSize * SlotsPerBlock;
    static constexpr std::size_t BlockAlign = std::max(SlotAlign, alignof(Block));

    Block* m_block_list = nullptr;
    void* m_free_list = nullptr;
    std::size_t m_free_slots = 0;
    mutable std::mutex m_mutex;  // 互斥锁，mutable支持const函数加锁

    bool grow() {
        constexpr std::size_t block_header_size = AlignUp<sizeof(Block), BlockAlign>();
        constexpr std::size_t total_block_size = block_header_size + BlockDataSize;

        void* block_ptr = std::aligned_alloc(BlockAlign, total_block_size);
        if (!block_ptr) return false;

        Block* new_block = new (block_ptr) Block;
        new_block->next = m_block_list;
        m_block_list = new_block;

        std::byte* data_start = static_cast<std::byte*>(block_ptr) + block_header_size;
        for (std::size_t i = 0; i < SlotsPerBlock; ++i) {
            void* slot = data_start + i * RealSlotSize;
            *static_cast<void**>(slot) = static_cast<std::byte*>(slot) + RealSlotSize;
        }
        *static_cast<void**>(data_start + (SlotsPerBlock - 1) * RealSlotSize) = nullptr;

        m_free_list = data_start;
        m_free_slots += SlotsPerBlock;
        return true;
    }

public:
    ThreadSafeFixedMemoryPool() = default;
    ~ThreadSafeFixedMemoryPool() {
        Block* curr = m_block_list;
        while (curr) {
            Block* next = curr->next;
            curr->~Block();
            std::free(curr);
            curr = next;
        }
    }

    ThreadSafeFixedMemoryPool(const ThreadSafeFixedMemoryPool&) = delete;
    ThreadSafeFixedMemoryPool& operator=(const ThreadSafeFixedMemoryPool&) = delete;
    ThreadSafeFixedMemoryPool(ThreadSafeFixedMemoryPool&&) = delete;
    ThreadSafeFixedMemoryPool& operator=(ThreadSafeFixedMemoryPool&&) = delete;

    [[nodiscard]] void* allocate() {
        std::lock_guard<std::mutex> lock(m_mutex);
        if (!m_free_list && !grow()) {
            throw std::bad_alloc();
        }
        void* slot = m_free_list;
        m_free_list = *static_cast<void**>(slot);
        --m_free_slots;
        return slot;
    }

    void deallocate(void* ptr) noexcept {
        if (!ptr) return;
        std::lock_guard<std::mutex> lock(m_mutex);
        *static_cast<void**>(ptr) = m_free_list;
        m_free_list = ptr;
        ++m_free_slots;
    }

    std::size_t free_slots() const noexcept {
        std::lock_guard<std::mutex> lock(m_mutex);
        return m_free_slots;
    }

    std::size_t total_slots() const noexcept {
        std::lock_guard<std::mutex> lock(m_mutex);
        std::size_t count = 0;
        Block* curr = m_block_list;
        while (curr) {
            count += SlotsPerBlock;
            curr = curr->next;
        }
        return count;
    }
};
```

## 三、C++17 核心特性应用说明
1. **`std::byte`**：替代传统`char*`表示原始内存，提供更强的类型安全，避免无意的算术运算，符合C++17的内存语义。
2. **`if constexpr`**：编译期分支判断，对平凡析构类型跳过析构调用，实现零成本抽象，无任何运行时开销。
3. **`std::aligned_alloc`**：C++17标准引入的对齐内存分配函数，替代平台相关接口，保证跨平台兼容性。
4. **`std::launder`**：解决placement new后带const/引用成员的对象指针别名问题，符合C++17对象模型规范。
5. **`[[nodiscard]]`**：C++17属性，标记分配函数的返回值不可丢弃，编译期预警内存泄漏风险。
6. **`constexpr`编译期计算**：所有大小、对齐参数均在编译期完成计算，无运行时开销，极致优化性能。
7. **`noexcept`规范标注**：对无异常抛出的函数明确标注，帮助编译器做更激进的优化。

## 四、使用示例
### 4.1 原始内存池使用
```cpp
int main() {
    // 定义16字节大小、8字节对齐的内存池
    FixedMemoryPool<16, 8> pool;

    // 分配内存
    void* p1 = pool.allocate();
    void* p2 = pool.allocate();

    // 使用内存（示例：写入数据）
    new (p1) int(123);
    new (p2) double(3.14);

    // 归还内存
    pool.deallocate(p1);
    pool.deallocate(p2);

    return 0;
}
```

### 4.2 对象池使用
```cpp
#include <string>
#include <cstdio>

int main() {
    // 定义std::string对象池，每个块512个对象
    ObjectPool<std::string, 512> str_pool;

    // 构造对象，支持任意构造函数
    std::string* s1 = str_pool.construct("Hello, C++17 Memory Pool!");
    std::string* s2 = str_pool.construct(10, 'a');

    // 使用对象
    printf("%s\n", s1->c_str());
    printf("%s\n", s2->c_str());

    // 销毁对象（自动调用析构函数，归还内存）
    str_pool.destroy(s1);
    str_pool.destroy(s2);

    return 0;
}
```

### 4.3 线程安全版本多线程使用
```cpp
#include <thread>
#include <vector>

int main() {
    // 64字节、Cache Line对齐的线程安全内存池
    ThreadSafeFixedMemoryPool<64, 64> pool;

    constexpr int thread_count = 8;
    constexpr int alloc_per_thread = 10000;
    std::vector<std::thread> threads;
    threads.reserve(thread_count);

    // 多线程并发分配与释放
    for (int i = 0; i < thread_count; ++i) {
        threads.emplace_back([&]() {
            std::vector<void*> ptrs;
            ptrs.reserve(alloc_per_thread);
            // 批量分配
            for (int j = 0; j < alloc_per_thread; ++j) {
                ptrs.push_back(pool.allocate());
            }
            // 批量释放
            for (void* p : ptrs) {
                pool.deallocate(p);
            }
        });
    }

    // 等待所有线程结束
    for (auto& t : threads) {
        t.join();
    }

    // 验证：所有内存已归还
    printf("Free slots: %zu\n", pool.free_slots());
    printf("Total slots: %zu\n", pool.total_slots());

    return 0;
}
```

## 五、关键注意事项
1. **生命周期管理**：内存池的生命周期必须长于所有从它分配的对象，否则会出现野指针和未定义行为。
2. **对象析构**：使用`ObjectPool`时，必须手动调用`destroy`销毁对象，否则对象的析构函数不会执行，会导致资源泄漏。
3. **固定大小限制**：本实现为固定大小内存池，仅能分配指定大小的内存；如需可变大小分配，可基于多个不同槽大小的固定内存池实现slab分配器。
4. **性能优化**：高并发场景下，可基于线程局部存储（TLS）实现无锁内存池，每个线程持有独立的内存池，减少锁竞争。
5. **标准库优先**：C++17标准库已提供成熟的内存池组件，生产环境优先使用标准实现，避免重复造轮子。

## 六、C++17 标准库内置内存池（PMR）
C++17引入了**多态内存资源（PMR）**，在`<memory_resource>`头文件中提供了开箱即用的内存池实现，经过高度优化，兼容STL容器，生产环境优先使用：
```cpp
#include <memory_resource>
#include <vector>
#include <string>

int main() {
    // 1. 线程安全的池化内存资源（多线程场景）
    std::pmr::synchronized_pool_resource sync_pool;
    // 2. 单线程无锁池化内存资源（单线程场景，性能更高）
    std::pmr::unsynchronized_pool_resource unsync_pool;
    // 3. 单调递增内存池（适合频繁分配、无需单独释放的场景，性能极致）
    std::pmr::monotonic_buffer_resource mono_pool;

    // 绑定内存池到STL容器
    std::pmr::vector<int> vec(&sync_pool);
    vec.reserve(1000);
    for (int i = 0; i < 1000; ++i) {
        vec.push_back(i);
    }

    std::pmr::string str("Hello, C++17 PMR!", &sync_pool);

    // 无需手动释放，内存池析构时自动释放所有内存
    return 0;
}
```


