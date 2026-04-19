# C++20 新特性详细总结


# C++20 新特性详细总结
C++20（ISO/IEC 14882:2020）是继C++11之后最具革命性的C++标准版本，于2020年12月正式发布。它通过**四大核心支柱特性**彻底革新了C++的编程范式，同时引入了数十项语言与标准库增强，大幅提升了代码的类型安全、编译效率、表达力、可维护性与性能，解决了传统C++长期存在的诸多历史痛点。

## 一、四大核心支柱特性
这是C++20最具颠覆性的核心能力，重构了C++的核心编程模型。

### 1. Concepts（概念）：泛型编程的革命
**核心目标**：解决传统模板SFINAE机制晦涩难懂、错误信息冗长、接口约束不明确的痛点，为模板参数提供**显式、语义化、可组合的编译期类型约束**，让泛型代码像普通代码一样易读、易调试。

**核心能力**：
- 用`concept`关键字定义具名的类型约束，基于`requires`表达式描述类型必须满足的语法/语义要求
- 支持约束的合取（`&&`）、析取（`||`）、继承（细化），可组合构建复杂约束体系
- 模板声明处直接使用约束，支持**缩写函数模板**简化写法，大幅降低泛型代码的编写门槛
- 编译器在模板实例化前执行约束检查，输出精准友好的错误信息，彻底告别SFINAE的“天书式报错”
- 标准库预定义了大量基础概念：`std::integral`、`std::floating_point`、`std::equality_comparable`、`std::sortable`等

**示例代码**：
```cpp
// 定义概念：可加法运算的类型
template<typename T>
concept Addable = requires(T a, T b) {
    { a + b } -> std::convertible_to<T>;
};

// 用概念约束模板
template<Addable T>
T add(T a, T b) {
    return a + b;
}

// 缩写函数模板写法（C++20简化）
auto multiply(std::integral auto a, std::integral auto b) {
    return a * b;
}
```

### 2. Modules（模块）：终结头文件时代
**核心目标**：彻底替代C/C++沿用数十年的`#include`头文件包含机制，解决头文件重复解析、宏污染、循环依赖、编译速度慢、接口与实现耦合的核心痛点。

**核心能力**：
- 引入`module`、`export`、`import`三个关键字，将代码组织为独立的编译单元模块
- 模块仅需编译一次，编译结果缓存，大型项目编译速度可提升数倍
- 显式控制符号导出，非导出符号对外部不可见，彻底解决宏污染和命名空间冲突问题
- 支持模块分区、头文件单元导入，完美兼容现有代码体系

**示例代码**：
```cpp
// math.ixx - 模块定义文件
export module math; // 声明模块名

// 导出命名空间与符号
export namespace math {
    constexpr double PI = 3.141592653589793;
    double circle_area(double r) { return PI * r * r; }
}

// 非导出函数，外部无法访问
namespace {
    double helper(double x) { return x * 2; }
}

// main.cpp - 模块使用
import <iostream>;
import math; // 导入模块

int main() {
    std::cout << math::circle_area(2.0) << std::endl;
    return 0;
}
```

### 3. Coroutines（协程）：轻量级异步与生成器编程范式
**核心目标**：打破传统函数“一次性执行完毕”的限制，实现可暂停、可恢复的函数执行模型，简化异步IO、事件驱动、生成器、状态机等场景的代码编写，彻底解决回调地狱问题。

**核心能力**：
- 引入`co_await`、`co_yield`、`co_return`三个关键字，分别标记协程的**暂停等待**、**生成值并挂起**、**返回结果并终止**操作
- 采用无栈协程模型，上下文切换成本极低，远低于线程切换，支持海量并发任务
- 基于`promise_type`协议自定义协程行为，标准库提供了协程基础设施，第三方库（cppcoro、folly等）基于此实现了完整的异步框架

**典型场景**：序列生成器、异步网络IO、异步文件操作、UI事件循环、游戏状态机

**示例代码（生成器）**：
```cpp
#include <coroutine>
#include <iostream>

// 生成器协程返回类型
template<typename T>
struct Generator {
    struct promise_type;
    using handle = std::coroutine_handle<promise_type>;

    struct promise_type {
        T current_value;
        auto get_return_object() { return Generator{handle::from_promise(*this)}; }
        auto initial_suspend() { return std::suspend_always{}; }
        auto final_suspend() noexcept { return std::suspend_always{}; }
        void unhandled_exception() { std::terminate(); }
        auto yield_value(T value) {
            current_value = value;
            return std::suspend_always{};
        }
    };

    handle coro;
    Generator(handle h) : coro(h) {}
    ~Generator() { if (coro) coro.destroy(); }

    // 迭代器接口，支持范围for循环
    bool next() {
        coro.resume();
        return !coro.done();
    }
    T value() { return coro.promise().current_value; }
};

// 斐波那契数列生成器
Generator<int> fibonacci(int max) {
    co_yield 0;
    int a = 0, b = 1;
    while (b <= max) {
        co_yield b;
        std::tie(a, b) = std::tuple{b, a + b};
    }
}

int main() {
    auto fib = fibonacci(100);
    while (fib.next()) {
        std::cout << fib.value() << " ";
    }
    return 0;
}
```

### 4. Ranges（范围库）：STL算法与容器的现代化重构
**核心目标**：彻底重构传统STL算法与迭代器的设计，解耦算法与容器，提供更安全、更优雅、更高效的序列数据处理能力。

**核心能力**：
- 定义了`range`核心概念，统一了容器、视图、生成器等所有可迭代序列的接口，算法直接接受range对象，无需传入`begin()/end()`迭代器对
- 引入**视图（Views）** 机制：惰性求值、非拥有式、零开销的序列包装器，支持管道运算符`|`链式调用，避免中间临时容器，大幅提升性能和代码可读性
- 提供了丰富的视图适配器：`filter`、`transform`、`take`、`drop`、`reverse`、`split`、`join`等，可自由组合构建数据处理流水线
- 与Concepts深度结合，算法约束更严格，错误信息更精准

**示例代码**：
```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> nums = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

    // 管道式操作：过滤偶数 → 平方 → 取前3个 → 逆序
    auto result = nums
        | std::views::filter([](int n) { return n % 2 == 0; })
        | std::views::transform([](int n) { return n * n; })
        | std::views::take(3)
        | std::views::reverse;

    // 惰性求值：遍历的时候才执行计算，无中间临时容器
    for (int x : result) {
        std::cout << x << " "; // 输出：36 16 4
    }

    // 直接用range调用算法，无需迭代器对
    std::ranges::sort(nums);
    return 0;
}
```

## 二、其他核心语言特性
### 1. 三路比较运算符（<=>，太空船运算符）与默认比较
- 统一了对象的相等性与序关系比较，简化比较运算符的重载代码，一行实现全量比较运算符
- 语法：`auto operator<=>(const T&) const = default;` 可自动生成`==`、`!=`、`<`、`<=`、`>`、`>=`全部6个比较运算符
- 定义了强序、弱序、偏序三种比较类别，对应不同的相等性语义，配套`<compare>`头文件提供标准工具

### 2. consteval、constinit与constexpr增强
| 关键字 | 核心作用 | 核心区别 |
|--------|----------|----------|
| `consteval` | 声明**立即函数**，函数必须在编译期执行，运行时调用直接编译报错 | 比`constexpr`更严格，强制编译期执行，无运行时 fallback |
| `constinit` | 强制变量在**编译期完成静态初始化**，禁止动态初始化 | 解决静态变量初始化顺序问题（静态初始化顺序惨败），不具备const属性 |
| `constexpr` | 函数/变量可在编译期或运行时执行，兼顾编译期与运行时场景 | 兼容编译期和运行时，C++20大幅扩展了其能力边界 |

- **constexpr增强**：支持constexpr虚函数、constexpr try-catch块、constexpr union成员切换、constexpr `dynamic_cast`/`typeid`，大幅扩展了编译期编程的能力
- 新增`std::is_constant_evaluated()`：判断当前是否处于编译期常量求值上下文，实现编译期/运行期双路径代码

### 3. 有符号整数正式规定为二进制补码表示
终结了C++历史上有符号整数表示的实现定义行为，正式规定所有有符号整数必须采用二进制补码表示，大幅提升了代码的可移植性。

### 4. 指定初始化器（Designated Initializers）
支持C语言风格的按名称初始化结构体/联合体成员，提升代码可读性，未指定的成员执行零初始化：
```cpp
struct Point { int x; int y; int z = 0; };
Point p{.y = 42, .x = 10}; // x=10, y=42, z=0
```

### 5. 范围for循环增强
支持在范围for循环中添加初始化语句，和普通for循环一致，避免循环外定义临时变量：
```cpp
for (auto vec = GetVector(); int n : vec) {
    // 直接使用vec和n，vec的生命周期覆盖整个循环
}
```

### 6. char8_t与UTF-8支持增强
- 新增`char8_t`类型，专门用于表示UTF-8代码单元，解决了`char`类型既用于字符又用于数值的歧义问题
- 配套新增`u8""`UTF-8字符串字面量、`std::mbrtoc8`/`std::c8rtomb`编码转换函数，完善了C++的Unicode支持

### 7. lambda表达式增强
- 支持模板形参列表的泛型lambda：`[]<typename T>(T x) { ... }`，解决了auto泛型lambda无法获取具体类型的痛点
- 支持lambda在不求值上下文（如decltype）中使用
- 支持无状态lambda的默认构造与赋值
- 支持lambda初始化捕获中的参数包展开：`[...args = std::move(args)]() { ... }`
- 修正了捕获歧义：明确`[=]`默认不捕获this指针，必须显式写`[=, this]`捕获

### 8. 新增属性
- `[[likely]]`/`[[unlikely]]`：给编译器提供分支预测提示，优化热路径代码性能
- `[[no_unique_address]]`：标记空成员变量不占用内存空间，优化空基类优化（EBO），常用于分配器、特征类等无状态成员

### 9. explicit(bool)：条件显式构造函数
支持根据模板参数的布尔表达式，动态决定构造函数是否为explicit，解决了泛型代码中构造函数显式性无法动态控制的痛点：
```cpp
template<typename T>
struct Wrapper {
    // 当T是算术类型时，构造函数为隐式；否则为显式
    explicit(std::is_arithmetic_v<T>) Wrapper(T value) : val(value) {}
    T val;
};
```

### 10. 其他语言细节增强
- 聚合体支持圆括号初始化
- 移除了多数场景下必须使用`typename`关键字消除歧义的要求
- 支持`using enum`，将枚举成员导入当前作用域，简化枚举成员的使用
- 数组new支持推导数组大小
- 位域支持默认成员初始化器
- 统一了位运算符的行为规范
- 结构化绑定支持lambda捕获、static/thread_local变量，放宽了自定义点查找规则

## 三、标准库新增与增强特性
C++20新增了15个标准库头文件，同时对现有库进行了全面增强。

### 新增核心头文件
`<bit>`、`<compare>`、`<concepts>`、`<coroutine>`、`<format>`、`<numbers>`、`<ranges>`、`<source_location>`、`<span>`、`<syncstream>`、`<version>`、`<barrier>`、`<latch>`、`<semaphore>`、`<stop_token>`

### 核心库增强详解
#### 1. 格式库（std::format）
类型安全、高性能、可扩展的字符串格式化库，彻底替代不安全的printf系列函数，比C++流格式化更简洁高效，采用Python风格的格式化语法：
```cpp
#include <format>
#include <string>
#include <iostream>

int main() {
    // 基础格式化
    std::string s1 = std::format("Hello, {}! The answer is {:.2f}", "World", 3.14159);
    // 索引占位符
    std::string s2 = std::format("{0} + {1} = {1} + {0}", 1, 2);
    std::cout << s1 << std::endl;
    return 0;
}
```

#### 2. std::span（跨度）
非拥有式的连续内存序列视图，替代裸指针+长度的组合，提供安全、零开销的连续内存访问接口，完美解决数组退化为指针后丢失长度信息的安全问题。
- 支持静态长度和动态长度的span
- 可用于函数参数接收数组、vector、array等连续容器，无需模板，避免代码膨胀

```cpp
#include <span>
#include <vector>
#include <iostream>

// 无需模板，接收任意连续内存序列
void print_ints(std::span<const int> sp) {
    for (int n : sp) std::cout << n << " ";
}

int main() {
    int arr[] = {1,2,3};
    std::vector<int> vec = {4,5,6};
    print_ints(arr);
    print_ints(vec);
    return 0;
}
```

#### 3. 并发编程库全面增强
- **std::jthread**：可自动join、支持取消的线程类，替代std::thread，解决了std::thread析构时未join会导致程序终止的痛点
- **停止令牌机制**：`std::stop_token`/`std::stop_source`/`std::stop_callback`，通用的线程取消/协作中断机制
- **新增同步原语**：
  - `std::latch`：一次性同步屏障，支持多个线程等待直到计数归零
  - `std::barrier`：可复用的线程同步点，支持阶段完成回调
  - `std::counting_semaphore`/`std::binary_semaphore`：信号量，用于控制资源访问的并发数
- **原子操作增强**：支持浮点原子、`std::shared_ptr`/`std::weak_ptr`原子操作

#### 4. chrono时间库增强
新增完整的日历与时区库，支持年、月、日、星期、时区的计算与转换，彻底解决了C++长期以来日期时间处理能力不足的问题：
```cpp
#include <chrono>
#include <iostream>

using namespace std::chrono;

int main() {
    // 公历日期创建
    year_month_day date = 2024y/10/1;
    // 日期运算
    auto next_week = date + weeks{1};
    // 星期计算
    weekday wd = date;
    std::cout << date << " is " << wd << std::endl;
    return 0;
}
```

#### 5. 其他核心库增强
- **数学常数库**：`<numbers>`头文件提供预定义的编译期数学常量，如π、e、√2、ln2等，无需手动定义
- **位操作库**：`<bit>`头文件提供`bit_cast`（类型安全的二进制重解释）、旋转、计数前导零、字节序判断等跨平台位操作工具
- **源码位置库**：`std::source_location`替代`__FILE__`、`__LINE__`等宏，提供类型安全、可传递的源码位置信息，适配日志、断言场景
- **统一容器擦除**：`std::erase`/`std::erase_if`，一行代码实现容器元素批量删除，替代传统的erase-remove惯用法
- **字符串增强**：`std::string`/`std::string_view`新增`starts_with`/`ends_with`成员函数，快速判断前缀后缀
- **同步输出流**：`std::osyncstream`解决多线程环境下标准输出流的乱序问题，保证线程安全的输出

## 四、弃用与移除的特性
- 正式移除了`auto_ptr`、`bind1st`/`bind2nd`、`mem_fun`等早已弃用的特性
- 移除了`throw()`异常说明符，完全替代为`noexcept`
- 大幅缩减了`volatile`的用法，仅保留内存序相关的语义
- 弃用了`std::allocator<void>`特化
- 弃用了C风格内存分配函数（malloc/realloc等）在常量表达式中的使用
- 弃用了部分旧的iostream库特性

## 编译器支持情况
主流编译器已全面支持C++20：
- GCC 10+ 支持核心特性，GCC 11+ 实现完整标准支持
- Clang 13+ 支持核心特性，Clang 14+ 实现完整标准支持
- MSVC 2019 16.11+ 支持核心特性，MSVC 2022 实现完整标准支持


