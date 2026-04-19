# GoogleTest 使用详细总结


# GoogleTest（gtest）使用详细总结
GoogleTest（简称gtest）是Google开源的跨平台C++单元测试框架，遵循xUnit测试架构，是C++项目单元测试的工业级主流方案。它提供了自动化测试用例管理、丰富的断言机制、测试环境复用、参数化测试、死亡测试等核心能力，配套GoogleMock（gmock）可实现接口Mock，完成测试依赖隔离，支持全平台编译与CI/CD流水线集成。

## 一、环境搭建与项目集成
### 1.1 核心依赖要求
- 编译器：支持C++11及以上（最新版本推荐C++17）
- 构建工具：CMake 3.11+（官方推荐），也支持Make、VS、Xcode等
- 源码仓库：https://github.com/google/googletest

### 1.2 推荐集成方式：CMake FetchContent（零环境依赖）
这是官方推荐的现代集成方式，无需手动下载编译gtest，CMake配置阶段自动拉取对应版本源码，可移植性最强，无系统环境差异问题。

项目根目录`CMakeLists.txt`完整配置：
```cmake
cmake_minimum_required(VERSION 3.14)
project(gtest_demo)

# 配置C++标准，与gtest编译版本保持一致
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Windows平台特殊配置：避免覆盖父项目的编译器/运行时设置
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

# 引入FetchContent模块，自动拉取gtest源码
include(FetchContent)
FetchContent_Declare(
  googletest
  GIT_REPOSITORY https://github.com/google/googletest.git
  GIT_TAG        v1.15.2  # 替换为最新稳定版本
)
FetchContent_MakeAvailable(googletest)

# 1. 待测试的业务代码（示例：math模块）
add_library(math STATIC math.cpp math.h)

# 2. 测试可执行文件
add_executable(math_test math_test.cpp)
# 链接gtest核心库、内置main函数库、待测试业务库
target_link_libraries(math_test PRIVATE math GTest::gtest GTest::gtest_main)

# 3. 集成到CTest，自动发现测试用例
include(GoogleTest)
gtest_discover_tests(math_test)
```

### 1.3 其他集成方式
#### 系统级安装（Linux/Mac）
适合本地固定环境使用，步骤如下：
```bash
# 1. 拉取源码
git clone https://github.com/google/googletest.git
cd googletest
# 2. 编译安装
cmake -B build -DCMAKE_CXX_STANDARD=17 -DINSTALL_GTEST=ON
cmake --build build
sudo cmake --install build
```
安装后通过`find_package(GTest REQUIRED)`引入项目，链接`GTest::gtest`即可。

#### Windows平台
- 方式1：vcpkg一键安装：`vcpkg install gtest:x64-windows`
- 方式2：CMake生成VS工程，编译后手动配置头文件和库路径

## 二、核心概念澄清
使用前必须明确核心术语，避免新旧版本命名混淆：
| 术语 | 含义 |
|------|------|
| **断言（Assertion）** | gtest核心校验语句，用于验证代码行为是否符合预期，分为致命/非致命两类 |
| **测试用例（Test）** | 最小测试单元，对应`TEST`宏包裹的代码块，包含一组断言，验证一个具体业务场景 |
| **测试套件（Test Suite）** | 一组相关测试用例的集合，对应`TEST`宏第一个参数，用于按模块/类分类管理用例（旧版本称为Test Case，注意API兼容） |
| **测试固件（Test Fixture）** | 复用测试初始化/清理逻辑的基类，继承自`::testing::Test`，实现多个用例共享相同测试环境 |
| **测试程序** | 包含多个测试套件、main函数的可执行文件，运行后自动执行所有注册的测试用例 |

> 命名规范：测试套件名、测试用例名必须是合法C++标识符，**禁止包含下划线**（gtest内部以下划线分隔命名，避免冲突）。

## 三、核心基础用法
### 3.1 断言机制（核心基础）
断言是gtest的核心，用于校验代码执行结果是否符合预期，分为两大系列，核心区别是失败后的执行行为：
- **`ASSERT_*` 致命断言**：断言失败时，**立即终止当前函数**，后续代码不会执行，仅用于校验必须成立的前置条件（如指针非空，否则后续会触发空指针崩溃）。
- **`EXPECT_*` 非致命断言**：断言失败时，**仅记录失败结果，继续执行当前测试用例**，不会终止函数，是日常开发最常用的断言类型。

所有断言都支持自定义错误信息输出，格式：`EXPECT_EQ(a, b) << "自定义错误描述，a=" << a << ",b=" << b;`

#### 常用断言分类与示例
##### 1. 布尔值断言
| 断言 | 含义 |
|------|------|
| `ASSERT_TRUE(cond)` / `EXPECT_TRUE(cond)` | 验证cond表达式结果为true |
| `ASSERT_FALSE(cond)` / `EXPECT_FALSE(cond)` | 验证cond表达式结果为false |

示例：
```cpp
EXPECT_TRUE(3 > 2);
ASSERT_FALSE(ptr == nullptr); // 指针为空则终止，避免后续空指针访问
```

##### 2. 数值二元比较断言
优先使用专用比较断言，而非`EXPECT_TRUE(a==b)`，前者失败时会打印a、b的实际值，调试更友好。

| 断言 | 含义 |
|------|------|
| `ASSERT_EQ(val1, val2)` / `EXPECT_EQ(val1, val2)` | 验证 val1 == val2 相等 |
| `ASSERT_NE(val1, val2)` / `EXPECT_NE(val1, val2)` | 验证 val1 != val2 不等 |
| `ASSERT_LT(val1, val2)` / `EXPECT_LT(val1, val2)` | 验证 val1 < val2 小于 |
| `ASSERT_LE(val1, val2)` / `EXPECT_LE(val1, val2)` | 验证 val1 <= val2 小于等于 |
| `ASSERT_GT(val1, val2)` / `EXPECT_GT(val1, val2)` | 验证 val1 > val2 大于 |
| `ASSERT_GE(val1, val2)` / `EXPECT_GE(val1, val2)` | 验证 val1 >= val2 大于等于 |

示例：
```cpp
int add(int a, int b) { return a + b; }
TEST(AddTest, PositiveNum) {
    EXPECT_EQ(add(1,2), 3);
    EXPECT_NE(add(1,2), 4);
    EXPECT_GT(add(5,3), 7);
}
```

##### 3. C风格字符串断言
针对`char*`类型字符串，`std::string`直接使用`EXPECT_EQ`即可。

| 断言 | 含义 |
|------|------|
| `ASSERT_STREQ(str1, str2)` / `EXPECT_STREQ(str1, str2)` | 字符串内容相等（区分大小写） |
| `ASSERT_STRNE(str1, str2)` / `EXPECT_STRNE(str1, str2)` | 字符串内容不等（区分大小写） |
| `ASSERT_STRCASEEQ(str1, str2)` / `EXPECT_STRCASEEQ(str1, str2)` | 字符串内容相等（忽略大小写） |
| `ASSERT_STRCASENE(str1, str2)` / `EXPECT_STRCASENE(str1, str2)` | 字符串内容不等（忽略大小写） |

##### 4. 浮点数断言
浮点数存在精度误差，禁止直接使用`EXPECT_EQ`比较，必须使用专用断言。

| 断言 | 含义 |
|------|------|
| `ASSERT_FLOAT_EQ(val1, val2)` / `EXPECT_FLOAT_EQ(val1, val2)` | 两个float值相等，误差在4个ULP内 |
| `ASSERT_DOUBLE_EQ(val1, val2)` / `EXPECT_DOUBLE_EQ(val1, val2)` | 两个double值相等，误差在4个ULP内 |
| `ASSERT_NEAR(val1, val2, abs_error)` / `EXPECT_NEAR(val1, val2, abs_error)` | 两个值的差值绝对值不超过abs_error |

示例：
```cpp
TEST(FloatTest, Compare) {
    EXPECT_FLOAT_EQ(3.14f, 3.1400001f);
    EXPECT_NEAR(3.1415926, 3.14, 0.01);
}
```

##### 5. 异常断言
验证代码是否抛出预期类型的异常。

| 断言 | 含义 |
|------|------|
| `ASSERT_THROW(statement, exception_type)` / `EXPECT_THROW(...)` | statement执行时抛出exception_type类型的异常 |
| `ASSERT_ANY_THROW(statement)` / `EXPECT_ANY_THROW(...)` | statement执行时抛出任意类型的异常 |
| `ASSERT_NO_THROW(statement)` / `EXPECT_NO_THROW(...)` | statement执行时不抛出任何异常 |

示例：
```cpp
int divide(int a, int b) {
    if (b == 0) throw std::invalid_argument("divisor is zero");
    return a / b;
}
TEST(DivideTest, Exception) {
    EXPECT_NO_THROW(divide(4,2));
    EXPECT_THROW(divide(4,0), std::invalid_argument);
}
```

### 3.2 基础测试用例编写（TEST宏）
`TEST`宏是gtest最基础的测试用例定义宏，用于无共享环境的简单测试场景，语法如下：
```cpp
TEST(测试套件名, 测试用例名) {
    // 测试代码 + 断言
}
```

完整可运行示例：
```cpp
// math.h
#ifndef MATH_H
#define MATH_H
int add(int a, int b);
int multiply(int a, int b);
#endif

// math.cpp
#include "math.h"
int add(int a, int b) { return a + b; }
int multiply(int a, int b) { return a * b; }

// math_test.cpp
#include <gtest/gtest.h>
#include "math.h"

// 加法测试套件
TEST(AddTest, PositiveNumber) {
    EXPECT_EQ(add(1, 2), 3);
    EXPECT_EQ(add(100, 200), 300);
}

TEST(AddTest, NegativeNumber) {
    EXPECT_EQ(add(-1, -2), -3);
    EXPECT_EQ(add(-1, 2), 1);
}

// 乘法测试套件
TEST(MultiplyTest, ZeroInput) {
    EXPECT_EQ(multiply(0, 5), 0);
    EXPECT_EQ(multiply(5, 0), 0);
}

TEST(MultiplyTest, PositiveNumber) {
    EXPECT_EQ(multiply(3, 4), 12);
}
```

### 3.3 测试固件（Test Fixture）与TEST_F宏
当多个测试用例需要复用相同的初始化、清理逻辑，或共享相同测试数据时，使用测试固件，避免代码冗余，同时保证每个用例的测试环境独立。

#### 实现步骤
1. 定义类，**public继承自`::testing::Test`**；
2. 用`protected`修饰成员，方便测试用例访问；
3. 重写`SetUp()`函数：**每个测试用例执行前都会调用**，用于初始化测试环境；
4. 重写`TearDown()`函数：**每个测试用例执行后都会调用**，用于清理测试资源；
5. 可选：重写静态函数`SetUpTestSuite()`/`TearDownTestSuite()`：**整个测试套件执行前后仅调用一次**，用于套件级别的全局资源初始化/清理。

#### 完整示例
```cpp
#include <gtest/gtest.h>
#include <queue>

// 定义测试固件
class QueueTest : public ::testing::Test {
protected:
    // 每个测试用例都会拥有独立的queue实例
    std::queue<int> q;

    // 每个测试用例执行前调用
    void SetUp() override {
        // 初始化队列：每个用例启动时都有3个初始元素
        q.push(1);
        q.push(2);
        q.push(3);
    }

    // 每个测试用例执行后调用
    void TearDown() override {
        // 清理队列，释放资源
        while (!q.empty()) q.pop();
    }

    // 整个测试套件执行前调用一次
    static void SetUpTestSuite() {
        // 套件级初始化：如打开文件、连接数据库、加载全局配置等
    }

    // 整个测试套件执行后调用一次
    static void TearDownTestSuite() {
        // 套件级清理：关闭文件、断开数据库连接等
    }
};

// 使用TEST_F宏，第一个参数必须是测试固件类名
TEST_F(QueueTest, PushElement) {
    // 直接访问固件的protected成员q
    q.push(4);
    EXPECT_EQ(q.size(), 4);
    EXPECT_EQ(q.back(), 4);
}

TEST_F(QueueTest, PopElement) {
    q.pop();
    EXPECT_EQ(q.size(), 2);
    EXPECT_EQ(q.front(), 2);
}

TEST_F(QueueTest, GetFrontElement) {
    EXPECT_EQ(q.front(), 1);
}
```

> 关键特性：每个`TEST_F`测试用例执行时，都会创建一个**独立的固件类实例**，不同用例之间的成员变量互不影响，彻底避免用例间的状态污染。

### 3.4 main函数的两种实现
#### 1. 内置main函数（推荐，90%场景适用）
无需自己编写main函数，只需在CMake中链接`GTest::gtest_main`库，gtest内置的main函数会自动处理命令行参数、初始化框架、执行所有测试用例。

#### 2. 自定义main函数
如果需要在测试执行前后做全局初始化/清理，可自定义main函数，格式固定如下：
```cpp
#include <gtest/gtest.h>

int main(int argc, char **argv) {
    // 必须调用：初始化gtest，解析命令行参数
    testing::InitGoogleTest(&argc, argv);

    // 此处可添加全局初始化代码：如初始化日志、加载全局配置、初始化第三方库等

    // 运行所有测试用例，返回值：0=全部通过，非0=存在失败用例
    int ret = RUN_ALL_TESTS();

    // 此处可添加全局清理代码

    return ret;
}
```

> 注意：`RUN_ALL_TESTS()` 在整个程序生命周期内**只能调用一次**，多次调用会导致未定义行为。

## 四、进阶高级用法
### 4.1 参数化测试
当同一个测试逻辑需要验证多组输入输出时，参数化测试可避免重复编写多个测试用例，大幅简化代码，同时方便批量覆盖边界条件。

#### 实现步骤
1. 定义参数化固件类，继承自`::testing::TestWithParam<T>`，`T`为参数类型（基础类型、结构体、tuple等）；
2. 使用`TEST_P`宏定义参数化测试逻辑，通过`GetParam()`获取当前参数值；
3. 使用`INSTANTIATE_TEST_SUITE_P`宏实例化测试套件，传入多组测试参数。

#### 示例1：tuple多参数场景
```cpp
#include <gtest/gtest.h>
#include <tuple>

int add(int a, int b) { return a + b; }

// 定义参数化固件：参数为tuple<a, b, 预期结果>
class AddParamTest : public ::testing::TestWithParam<std::tuple<int, int, int>> {
};

// 定义参数化测试逻辑
TEST_P(AddParamTest, MultiInputVerify) {
    // 获取当前测试参数
    int a = std::get<0>(GetParam());
    int b = std::get<1>(GetParam());
    int expect = std::get<2>(GetParam());
    // 执行断言
    EXPECT_EQ(add(a, b), expect);
}

// 实例化测试套件，批量传入多组参数
INSTANTIATE_TEST_SUITE_P(
    AddParamGroup,  // 实例名，自定义即可
    AddParamTest,   // 参数化固件类名
    ::testing::Values( // 多组测试参数
        std::make_tuple(1, 2, 3),
        std::make_tuple(-1, -2, -3),
        std::make_tuple(-1, 2, 1),
        std::make_tuple(0, 5, 5),
        std::make_tuple(100, 200, 300)
    )
);
```

#### 示例2：结构体参数化（可读性更强）
```cpp
#include <gtest/gtest.h>
#include <vector>
#include <string>

// 定义测试用例结构体
struct AddTestCase {
    int a;
    int b;
    int expect;
    std::string case_name; // 用例名，用于测试报告展示
};

class AddStructParamTest : public ::testing::TestWithParam<AddTestCase> {
};

TEST_P(AddStructParamTest, MultiInputVerify) {
    const auto& param = GetParam();
    EXPECT_EQ(add(param.a, param.b), param.expect);
}

// 实例化，同时自定义用例名，方便调试和报告查看
INSTANTIATE_TEST_SUITE_P(
    AddStructGroup,
    AddStructParamTest,
    ::testing::ValuesIn(std::vector<AddTestCase>{
        {1, 2, 3, "positive_number"},
        {-1, -2, -3, "negative_number"},
        {0, 5, 5, "zero_input"},
        {INT_MAX, 1, INT_MAX + 1, "int_overflow"}
    }),
    // 自定义用例名生成函数
    [](sslocal://flow/file_open?url=const+%3A%3Atesting%3A%3ATestParamInfo%3CAddTestCase%3E%26+info&flow_extra=eyJsaW5rX3R5cGUiOiJjb2RlX2ludGVycHJldGVyIn0=) {
        return info.param.case_name;
    }
);
```

### 4.2 死亡测试
死亡测试用于验证代码在异常场景下的行为，比如断言失败、程序崩溃、异常退出等，校验代码的容错性和异常处理逻辑，核心是验证“代码是否在预期场景下按预期方式终止进程”。

#### 核心宏与用法
| 宏 | 含义 |
|------|------|
| `ASSERT_DEATH(statement, regex)` / `EXPECT_DEATH(...)` | statement执行后进程退出，且stderr输出匹配regex正则表达式 |
| `ASSERT_EXIT(statement, predicate, regex)` / `EXPECT_EXIT(...)` | statement执行后进程退出，退出码符合predicate，且stderr匹配regex |

常用退出码校验predicate：
- `::testing::ExitedWithCode(code)`：进程正常退出，退出码等于code
- `::testing::KilledBySignal(signal)`：进程被信号杀死（Linux/Mac）

#### 完整示例
```cpp
#include <gtest/gtest.h>
#include <cstdio>
#include <cstdlib>
#include <cassert>

void check_null_ptr(int* ptr) {
    if (ptr == nullptr) {
        fprintf(stderr, "fatal error: ptr is null\n");
        abort(); // 终止进程
    }
}

void exit_with_code(int code) {
    exit(code);
}

// 死亡测试套件名建议以DeathTest结尾，gtest会优先执行，避免父进程状态影响
TEST(CheckPtrDeathTest, NullPtrCrash) {
    // 验证传入空指针时，进程终止，且stderr输出匹配正则
    ASSERT_DEATH(check_null_ptr(nullptr), "fatal error: ptr is null");
}

TEST(ExitCodeTest, VerifyExitCode) {
    // 验证进程正常退出，退出码为0
    ASSERT_EXIT(exit_with_code(0), ::testing::ExitedWithCode(0), "");
    // 验证进程正常退出，退出码为1
    ASSERT_EXIT(exit_with_code(1), ::testing::ExitedWithCode(1), "");
}
```

> 注意事项：
> 1. 死亡测试通过fork子进程执行测试代码，性能开销较大，仅用于异常场景测试；
> 2. Windows平台对死亡测试支持有限，优先在Linux/Mac平台使用。

### 4.3 测试用例过滤与执行控制
gtest支持通过命令行参数动态控制测试用例的执行，无需重新编译代码，适配调试、CI/CD等多种场景。

#### 常用命令行参数
1. **测试用例过滤**：`--gtest_filter=匹配模式`
支持通配符`*`（匹配任意字符）、`?`（匹配单个字符）、`:`（多个模式并列）、`-`（排除模式）。
```bash
# 只执行AddTest套件下的所有用例
./math_test --gtest_filter=AddTest.*

# 执行AddTest和MultiplyTest套件的所有用例
./math_test --gtest_filter=AddTest.*:MultiplyTest.*

# 执行所有用例，排除Negative相关的用例
./math_test --gtest_filter=*-*Negative*

# 只执行单个用例
./math_test --gtest_filter=AddTest.PositiveNumber
```

2. **重复执行测试**：`--gtest_repeat=N`
重复执行所有测试N次，`N=-1`则无限循环，用于排查偶现的不稳定bug。
```bash
# 重复执行100次，排查偶现问题
./math_test --gtest_repeat=100
```

3. **随机打乱执行顺序**：`--gtest_shuffle`
默认按代码定义顺序执行，该参数会随机打乱用例执行顺序，快速排查用例间的隐式依赖问题；配合`--gtest_random_seed=SEED`可固定随机种子，复现问题。

4. **执行禁用的用例**：`--gtest_also_run_disabled_tests`
默认跳过禁用的测试用例，该参数可强制执行所有禁用用例。

5. **输出测试报告**：`--gtest_output=xml:report.xml`
将测试结果输出为XML格式，方便集成到Jenkins、GitLab CI等CI/CD系统，生成可视化测试报告。

### 4.4 禁用测试用例
临时禁用某个测试用例/整个测试套件，无需注释代码，只需在**测试套件名或测试用例名前添加`DISABLED_`前缀**即可。

示例：
```cpp
// 禁用整个测试套件
TEST(DISABLED_AddTest, PositiveNumber) {
    EXPECT_EQ(add(1,2), 3);
}

// 禁用单个测试用例
TEST(AddTest, DISABLED_NegativeNumber) {
    EXPECT_EQ(add(-1,-2), -3);
}
```

> 特性：禁用的用例默认不会执行，gtest会在运行结果中提示“有多少个禁用的用例”，避免遗漏；配合`--gtest_also_run_disabled_tests`参数可强制执行。

### 4.5 事件监听器（EventListener）
gtest提供了事件监听机制，可自定义钩子函数，在测试执行的各个生命周期节点执行自定义逻辑，比如自定义日志格式、统计用例执行时间、上报测试结果、对接第三方监控系统等。

#### 实现步骤
1. 继承`::testing::EmptyTestEventListener`类（空实现的监听器基类，无需重写所有钩子）；
2. 重写需要的生命周期钩子函数；
3. 在main函数中，将自定义监听器注册到gtest的事件广播器中。

#### 示例：统计用例执行时间
```cpp
#include <gtest/gtest.h>
#include <chrono>
#include <cstdio>

// 自定义监听器：统计测试用例执行时间
class TimeStatListener : public ::testing::EmptyTestEventListener {
private:
    std::chrono::steady_clock::time_point case_start_time;

    // 钩子：测试套件开始前调用
    void OnTestSuiteStart(const ::testing::TestSuite& test_suite) override {
        printf("===== Test Suite [%s] start =====\n", test_suite.name());
    }

    // 钩子：测试套件结束后调用
    void OnTestSuiteEnd(const ::testing::TestSuite& test_suite) override {
        printf("===== Test Suite [%s] end, total tests: %d, passed: %d =====\n\n",
               test_suite.name(),
               test_suite.total_test_count(),
               test_suite.successful_test_count());
    }

    // 钩子：单个测试用例开始前调用
    void OnTestStart(const ::testing::TestInfo& test_info) override {
        printf("Test [%s.%s] start\n", test_info.test_suite_name(), test_info.name());
        case_start_time = std::chrono::steady_clock::now();
    }

    // 钩子：单个测试用例结束后调用
    void OnTestEnd(const ::testing::TestInfo& test_info) override {
        auto end_time = std::chrono::steady_clock::now();
        auto duration_ms = std::chrono::duration_cast<std::chrono::milliseconds>(end_time - case_start_time).count();
        printf("Test [%s.%s] end, result: %s, duration: %ld ms\n",
               test_info.test_suite_name(),
               test_info.name(),
               test_info.result()->Passed() ? "PASS" : "FAIL",
               duration_ms);
    }
};

int main(int argc, char **argv) {
    testing::InitGoogleTest(&argc, argv);

    // 注册自定义监听器
    auto& listeners = testing::UnitTest::GetInstance()->listeners();
    listeners.Append(new TimeStatListener);

    return RUN_ALL_TESTS();
}
```

### 4.6 与GoogleMock（gmock）配合使用
gmock是gtest配套的Mock框架，用于模拟依赖的接口，实现单元测试的完全隔离，避免测试依赖数据库、网络、第三方接口等外部模块，是面向接口编程的单元测试核心工具。

#### 基础使用步骤
1. 定义接口类（纯虚函数基类）；
2. 定义Mock类，继承接口类，使用`MOCK_METHOD`宏声明要模拟的函数；
3. 在测试用例中，创建Mock对象，通过`EXPECT_CALL`设置函数的预期行为；
4. 将Mock对象注入到待测试的业务代码中，执行测试断言。

#### 完整示例
```cpp
#include <gtest/gtest.h>
#include <gmock/gmock.h>
#include <string>

// 1. 定义依赖接口
class Database {
public:
    virtual ~Database() = default;
    virtual bool queryUser(int id, std::string& name) = 0;
    virtual int getUserCount() = 0;
};

// 2. 定义Mock类，实现接口
class MockDatabase : public Database {
public:
    // MOCK_METHOD宏格式：MOCK_METHOD(返回值类型, 函数名, (参数列表), (修饰符))
    MOCK_METHOD(bool, queryUser, (int id, std::string& name), (override));
    MOCK_METHOD(int, getUserCount, (), (override));
};

// 3. 待测试的业务类：依赖Database接口
class UserService {
public:
    explicit UserService(Database* db) : db_(db) {}

    std::string getUserName(int id) {
        std::string name;
        if (db_->queryUser(id, name)) {
            return name;
        }
        return "unknown";
    }

    int getUserTotal() {
        return db_->getUserCount();
    }

private:
    Database* db_;
};

// 4. 测试用例
TEST(UserServiceTest, GetUserName_Success) {
    MockDatabase mock_db;
    UserService service(&mock_db);

    // 设置预期：调用queryUser(1, _)，返回true，并给第二个参数赋值"Alice"
    EXPECT_CALL(mock_db, queryUser(1, testing::_))
        .WillOnce(testing::DoAll(
            testing::SetArgReferee<1>("Alice"),
            testing::Return(true)
        ));

    // 执行测试并断言
    EXPECT_EQ(service.getUserName(1), "Alice");
}

TEST(UserServiceTest, GetUserName_UserNotExist) {
    MockDatabase mock_db;
    UserService service(&mock_db);

    // 设置预期：调用queryUser(2, _)，返回false
    EXPECT_CALL(mock_db, queryUser(2, testing::_))
        .WillOnce(testing::Return(false));

    EXPECT_EQ(service.getUserName(2), "unknown");
}

TEST(UserServiceTest, GetUserTotal_ReturnCount) {
    MockDatabase mock_db;
    UserService service(&mock_db);

    // 设置预期：调用getUserCount()，返回100
    EXPECT_CALL(mock_db, getUserCount())
        .WillOnce(testing::Return(100));

    EXPECT_EQ(service.getUserTotal(), 100);
}
```

> 注意：使用gmock时，CMake中需要链接`GTest::gmock`和`GTest::gmock_main`库，替换原有的gtest链接配置。

## 五、官方最佳实践
1. **命名规范**
   - 测试套件名：大驼峰命名，对应待测试的类/模块名，如`UserServiceTest`、`MathUtilTest`；
   - 测试用例名：大驼峰命名，清晰描述测试场景+预期结果，如`GetUserNameSuccess`、`DivideByZeroThrowException`，禁止使用`Test1`、`Case1`等无意义命名；
   - 死亡测试套件名以`DeathTest`结尾，gtest会优先执行，避免状态污染。

2. **断言使用规范**
   - 优先使用`EXPECT_*`系列，仅当断言失败后后续代码无法安全执行时，才使用`ASSERT_*`；
   - 优先使用专用比较断言（如`EXPECT_EQ(a,b)`），而非`EXPECT_TRUE(a==b)`，前者失败时会打印具体值，大幅提升调试效率；
   - 一个测试用例中，多个断言尽量保持独立，避免前一个断言失败导致后续所有断言无法执行，无法定位全部问题。

3. **测试用例设计原则**
   - 单一职责：一个测试用例只验证一个场景、一个行为，避免一个用例覆盖多个功能，失败后无法定位问题；
   - 完全独立：测试用例之间互不依赖，执行顺序不影响测试结果，每个用例都有独立的初始化和清理；
   - 覆盖全面：覆盖正常场景、边界场景、异常场景，如输入为0、空指针、负数、极值等；
   - 可重复性：测试用例每次执行结果一致，避免依赖随机数、时间、外部环境等不可控因素。

4. **测试固件使用规范**
   - 仅当多个测试用例需要复用相同的初始化/清理逻辑时，才使用测试固件，避免过度使用；
   - 用例级别的初始化放在`SetUp()`/`TearDown()`，套件级别的全局初始化放在`SetUpTestSuite()`/`TearDownTestSuite()`；
   - 避免在测试固件中编写复杂业务逻辑，固件代码需保证简洁、可维护。

5. **代码可测试性规范**
   - 面向接口编程，使用依赖注入，避免硬编码依赖，方便Mock替换；
   - 函数功能单一，避免超大函数，方便单元测试覆盖；
   - 避免使用全局变量、静态变量，会导致测试用例之间的状态污染；
   - 避免在构造函数/析构函数中编写复杂逻辑，难以进行单元测试。

## 六、常见问题与解决方案
### 1. 链接错误：undefined reference to `testing::xxx`
- 原因1：CMake中未正确链接gtest库，或链接顺序错误；
- 原因2：项目的C++标准与gtest编译的C++标准不一致；
- 原因3：Windows平台Debug/Release版本、运行时库不匹配；
- 解决方案：
  1. 确保`target_link_libraries`中添加了`GTest::gtest`和`GTest::gtest_main`；
  2. 项目和gtest的`CMAKE_CXX_STANDARD`保持一致；
  3. Windows平台添加`set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)`配置。

### 2. 测试用例不执行，输出显示0 tests run
- 原因1：测试用例的cpp文件未被添加到`add_executable`的源文件列表中，未被编译；
- 原因2：自定义main函数中未调用`RUN_ALL_TESTS()`，或调用了多次；
- 原因3：测试用例被添加了`DISABLED_`前缀，默认不执行；
- 解决方案：检查CMakeLists.txt，确保测试cpp被编译；检查main函数，确保`RUN_ALL_TESTS()`被调用且仅调用一次。

### 3. ASSERT_EQ比较自定义类型时报编译错误
- 原因：自定义类型未重载`==`运算符，或未重载`<<`运算符（用于断言失败时打印对象值）；
- 解决方案：给自定义类型重载`==`运算符和`<<`运算符，示例：
```cpp
struct Point {
    int x;
    int y;
    // 重载==运算符，用于ASSERT_EQ比较
    bool operator==(const Point& other) const {
        return x == other.x && y == other.y;
    }
};

// 重载<<运算符，用于断言失败时打印值
std::ostream& operator<<(std::ostream& os, const Point& p) {
    os << "Point(" << p.x << ", " << p.y << ")";
    return os;
}
```

### 4. 测试用例单独执行通过，批量执行失败
- 原因：测试用例之间共享了全局变量、静态变量，或未正确清理资源，前一个用例的执行状态污染了后一个用例；
- 解决方案：
  1. 避免使用全局变量、静态变量，或在`SetUpTestSuite()`中正确重置状态；
  2. 确保每个测试用例的初始化和清理逻辑完整，使用测试固件保证环境独立；
  3. 使用`--gtest_shuffle`打乱执行顺序，快速定位依赖问题。

### 5. 死亡测试执行失败，提示"process didn't die"
- 原因1：测试的statement未导致进程退出，未调用`abort()`/`exit()`，或断言未触发；
- 原因2：正则表达式与stderr的输出不匹配；
- 原因3：Windows平台不支持该死亡测试特性；
- 解决方案：确保statement会导致进程终止；调试stderr输出内容，调整正则表达式；切换到Linux/Mac平台执行死亡测试。

## 七、官方参考资料
- 官方文档：https://google.github.io/googletest/
- 源码仓库：https://github.com/google/googletest
- 官方最佳实践：https://google.github.io/googletest/gtest_best_practices.html
- gmock官方指南：https://google.github.io/googletest/gmock_for_dummies.html


