# GoogleMock 使用详细总结


# GoogleMock 使用详细总结
Google Mock（简称gmock）是Google推出的C++开源Mock框架，与Google Test（gtest）深度集成，是现代C++单元测试的核心基础设施。它用于在单元测试中创建模拟对象，替代被测代码的外部依赖（数据库、网络、第三方接口、复杂组件等），实现**依赖隔离、行为精准控制、交互逻辑验证**，让测试更稳定、高效、可复现。

## 一、核心基础认知
### 1.1 核心定位与适用场景
- **核心能力**：实现测试替身（Test Double）模式，不仅能预定义依赖的返回值（Stub能力），更能精准校验依赖的调用次数、入参、调用顺序、执行动作（Mock核心能力）。
- **核心解决的问题**：
  1.  依赖不可用/不稳定（如数据库、网络服务）导致测试失败；
  2.  真实依赖执行慢，拖慢测试效率；
  3.  难以模拟异常场景（如接口报错、超时、返回异常值）；
  4.  无法验证被测代码与依赖的交互逻辑是否符合预期。
- **前置要求**：被测代码需遵循**面向接口编程+依赖注入**设计原则，这是gmock落地的核心前提。

### 1.2 与gtest的关系
gmock是gtest的配套框架，从1.8.0版本开始已完全整合到googletest仓库中，二者配合使用：
- gtest：负责测试用例管理、断言执行、测试结果统计；
- gmock：负责模拟依赖对象、定义行为、校验交互逻辑。

### 1.3 三种Mock对象模式
gmock提供三种Mock模式，控制对未设置期望的函数调用的处理行为：
| 模式 | 行为 | 适用场景 |
|------|------|----------|
| NaggyMock（默认） | 未设置期望的函数被调用时，打印警告日志，测试不失败 | 开发调试阶段，兼顾灵活性与提示性 |
| NiceMock | 未设置期望的函数被调用时，完全忽略，无警告无报错 | 只关注核心交互，非关键函数不影响测试结果 |
| StrictMock | 未设置期望的函数被调用时，直接触发测试失败 | 需严格校验协议，任何非预期调用都视为bug |

使用方式：`NiceMock<MockDB> mock_db;` `StrictMock<MockDB> mock_db;`

## 二、环境搭建与项目集成
gmock已整合到googletest仓库，推荐使用CMake进行项目集成，支持Linux/macOS/Windows全平台。

### 2.1 源码获取与编译
```bash
# 克隆googletest仓库（含gmock）
git clone https://github.com/google/googletest.git
cd googletest && mkdir build && cd build

# 编译配置（必须启用C++11及以上）
cmake .. -DCMAKE_CXX_STANDARD=11 -DBUILD_GMOCK=ON -DCMAKE_INSTALL_PREFIX=/usr/local
# 编译与安装
make -j4 && sudo make install
```

### 2.2 CMake项目集成
提供两种主流集成方式，推荐使用现代CMake写法。

#### 方式1：系统安装后集成（推荐）
```cmake
cmake_minimum_required(VERSION 3.10)
project(gmock_demo)
set(CMAKE_CXX_STANDARD 11)

# 查找gtest/gmock库
find_package(GTest REQUIRED)

# 添加测试可执行文件
add_executable(test_demo test.cpp user_service.cpp)
# 链接库（现代CMake写法，自动处理头文件与链接路径）
target_link_libraries(test_demo PRIVATE GTest::gtest GTest::gtest_main GTest::gmock GTest::gmock_main pthread)
```

#### 方式2：子模块嵌入项目
将googletest源码放入项目`third_party/googletest`目录，CMake配置如下：
```cmake
cmake_minimum_required(VERSION 3.10)
project(gmock_demo)
set(CMAKE_CXX_STANDARD 11)

# 添加gtest/gmock子目录
add_subdirectory(third_party/googletest)

add_executable(test_demo test.cpp user_service.cpp)
target_link_libraries(test_demo PRIVATE gtest gtest_main gmock gmock_main pthread)
```

### 2.3 头文件引入
所有gmock能力都通过核心头文件引入，测试文件中需包含：
```cpp
#include <gtest/gtest.h>   // gtest核心
#include <gmock/gmock.h>   // gmock核心
```

## 三、核心概念与语法
### 3.1 六大核心概念
| 概念 | 核心作用 |
|------|----------|
| Mock类 | 继承自待模拟的接口，通过gmock宏自动生成桩函数，替代真实依赖类 |
| 期望（EXPECT_CALL） | 核心宏，定义对mock函数的调用预期（是否调用、调用次数、入参要求） |
| 匹配器（Matcher） | 校验函数入参是否符合预期，支持精准匹配、模糊匹配、复合匹配 |
| 基数（Cardinality） | 定义函数的预期调用次数，如恰好1次、至少2次、任意次 |
| 动作（Action） | 定义mock函数被调用时执行的行为，如设置返回值、修改出参、调用回调 |
| 序列（Sequence） | 控制多个mock函数的调用顺序，确保执行符合预期时序 |

### 3.2 核心宏语法
#### 1. MOCK_METHOD：Mock函数声明
C++11及以上推荐使用统一的`MOCK_METHOD`宏，替代旧版本的`MOCK_METHOD0/MOCK_METHOD1`等宏。
**语法**：
```cpp
MOCK_METHOD(返回值类型, 函数名, (参数列表), (限定符, 可选));
```
**参数说明**：
- 第1个参数：原函数的返回值类型；
- 第2个参数：原函数的函数名；
- 第3个参数：原函数的参数列表，带类型，用括号包裹；
- 第4个参数：原函数的限定符，必须和原函数完全一致，包括`override`、`const`、`noexcept`等，多个限定符用逗号分隔。

**示例**：
```cpp
// 原接口
class DBInterface {
public:
    virtual ~DBInterface() = default;
    virtual bool Connect(const std::string& host, int port) = 0;
    virtual int Query(const std::string& sql, std::string& result) const = 0;
    virtual void Close() = 0;
};

// Mock类
class MockDB : public DBInterface {
public:
    MOCK_METHOD(bool, Connect, (const std::string& host, int port), (override));
    MOCK_METHOD(int, Query, (const std::string& sql, std::string& result), (const, override));
    MOCK_METHOD(void, Close, (), (override));
};
```
> 注意：无论原函数在基类中是public/protected/private，Mock函数必须声明在Mock类的public区域，否则`EXPECT_CALL`无法访问。

#### 2. EXPECT_CALL：期望设置核心语法
`EXPECT_CALL`是gmock的核心，用于定义函数的调用预期，完整语法如下：
```cpp
EXPECT_CALL(mock对象, 函数名(匹配器列表))
    .Times(基数)              // 可选：定义调用次数
    .With(多参数联合匹配)     // 可选：多参数联合校验
    .WillOnce(动作)           // 可选：单次调用执行的动作
    .WillRepeatedly(动作)     // 可选：重复调用执行的动作
    .InSequence(序列)         // 可选：加入调用序列控制
    .RetiresOnSaturation();   // 可选：匹配饱和后自动失效
```

#### 3. ON_CALL：默认行为设置
用于定义mock函数的默认行为，当函数被调用时，若没有匹配的`EXPECT_CALL`，则执行`ON_CALL`定义的动作，**不会校验函数是否被调用**，仅设置默认行为。
**语法**：
```cpp
ON_CALL(mock对象, 函数名(匹配器列表))
    .WillByDefault(默认动作);
```

## 四、基础使用全流程（完整可运行示例）
以用户登录服务为例，完整演示gmock从接口定义到测试用例编写的全流程。

### 步骤1：定义待模拟的依赖接口
`db_interface.h`
```cpp
#ifndef DB_INTERFACE_H
#define DB_INTERFACE_H
#include <string>

// 数据库依赖接口（纯虚函数）
class DBInterface {
public:
    virtual ~DBInterface() = default;
    virtual bool Connect(const std::string& host, int port) = 0;
    virtual int Query(const std::string& sql, std::string& result) = 0;
    virtual void Close() = 0;
};

#endif
```

### 步骤2：实现Mock类
`mock_db.h`
```cpp
#ifndef MOCK_DB_H
#define MOCK_DB_H
#include <gmock/gmock.h>
#include "db_interface.h"

class MockDB : public DBInterface {
public:
    MOCK_METHOD(bool, Connect, (const std::string& host, int port), (override));
    MOCK_METHOD(int, Query, (const std::string& sql, std::string& result), (override));
    MOCK_METHOD(void, Close, (), (override));
};

#endif
```

### 步骤3：编写被测代码（依赖注入）
`user_service.h`
```cpp
#ifndef USER_SERVICE_H
#define USER_SERVICE_H
#include "db_interface.h"
#include <string>

class UserService {
public:
    // 构造函数注入依赖，不直接new真实对象，便于替换为Mock
    explicit UserService(DBInterface* db) : db_(db) {}
    bool Login(const std::string& username, const std::string& password);
private:
    DBInterface* db_;
};

#endif
```

`user_service.cpp`
```cpp
#include "user_service.h"

bool UserService::Login(const std::string& username, const std::string& password) {
    // 1. 连接数据库
    if (!db_->Connect("127.0.0.1", 3306)) {
        return false;
    }
    // 2. 执行查询
    std::string result;
    std::string sql = "SELECT * FROM users WHERE username='" + username + "' AND password='" + password + "'";
    int row_count = db_->Query(sql, result);
    // 3. 关闭连接
    db_->Close();
    // 4. 结果判断
    return row_count > 0;
}
```

### 步骤4：编写单元测试用例
`test.cpp`
```cpp
#include <gtest/gtest.h>
#include <gmock/gmock.h>
#include "mock_db.h"
#include "user_service.h"

// 引入常用的匹配器与动作，简化代码
using ::testing::_;
using ::testing::Return;
using ::testing::SetArgReferee;
using ::testing::Sequence;
using ::testing::StartsWith;
using ::testing::AllOf;
using ::testing::Gt;
using ::testing::Lt;

// 测试场景1：登录成功，验证全流程交互
TEST(UserServiceTest, Login_Success) {
    // 1. 创建Mock对象
    MockDB mock_db;
    // 2. 设置期望：Connect被调用1次，参数匹配，返回true
    EXPECT_CALL(mock_db, Connect("127.0.0.1", 3306))
        .Times(1)
        .WillOnce(Return(true));

    // 3. 设置期望：Query被调用1次，sql匹配，设置出参，返回1
    std::string expect_sql = "SELECT * FROM users WHERE username='admin' AND password='123456'";
    EXPECT_CALL(mock_db, Query(expect_sql, _))
        .Times(1)
        .WillOnce(DoAll(
            SetArgReferee<1>("user_data"), // 给第2个参数（索引1，引用类型）赋值
            Return(1)
        ));

    // 4. 设置期望：Close被调用1次
    EXPECT_CALL(mock_db, Close()).Times(1);

    // 5. 注入Mock依赖，创建被测对象
    UserService service(&mock_db);
    // 6. 执行被测函数
    bool ret = service.Login("admin", "123456");
    // 7. 断言结果
    EXPECT_TRUE(ret);
}

// 测试场景2：数据库连接失败，登录失败，验证后续函数不被调用
TEST(UserServiceTest, Login_ConnectFailed) {
    MockDB mock_db;
    // 连接返回false，调用1次
    EXPECT_CALL(mock_db, Connect(_, _))
        .Times(1)
        .WillOnce(Return(false));
    // 连接失败时，Query和Close绝对不能被调用
    EXPECT_CALL(mock_db, Query(_, _)).Times(0);
    EXPECT_CALL(mock_db, Close()).Times(0);

    UserService service(&mock_db);
    EXPECT_FALSE(service.Login("admin", "123456"));
}

// 测试场景3：严格校验调用顺序
TEST(UserServiceTest, Login_CallOrderCheck) {
    MockDB mock_db;
    Sequence seq; // 创建序列对象

    // 所有期望加入序列，必须按声明顺序执行，否则测试失败
    EXPECT_CALL(mock_db, Connect(_, _))
        .InSequence(seq)
        .WillOnce(Return(true));
    EXPECT_CALL(mock_db, Query(_, _))
        .InSequence(seq)
        .WillOnce(Return(1));
    EXPECT_CALL(mock_db, Close())
        .InSequence(seq);

    UserService service(&mock_db);
    service.Login("admin", "123456");
}

// 测试入口
int main(int argc, char **argv) {
    testing::InitGoogleTest(&argc, argv);
    testing::InitGoogleMock(&argc, argv); // 必须初始化gmock
    return RUN_ALL_TESTS();
}
```

### 步骤5：编译与运行
```bash
# 编译
cmake -B build && cmake --build build
# 运行测试
./build/test_demo
```

## 五、核心能力详解
### 5.1 匹配器（Matcher）全解
匹配器用于校验函数入参是否符合预期，是gmock最常用的能力之一，支持丰富的匹配规则。

#### 1. 通用基础匹配器
| 匹配器 | 作用 |
|--------|------|
| `_` | 万能匹配，匹配任意类型、任意值的参数 |
| `A<T>()/An<T>()` | 匹配任意T类型的参数 |
| `Eq(value)` / `value` | 匹配等于value的参数，直接写value等价于Eq(value) |
| `Ne(value)` | 匹配不等于value的参数 |
| `Gt(value)` | 匹配大于value的参数 |
| `Ge(value)` | 匹配大于等于value的参数 |
| `Lt(value)` | 匹配小于value的参数 |
| `Le(value)` | 匹配小于等于value的参数 |
| `IsNull()` | 匹配空指针 |
| `NotNull()` | 匹配非空指针 |

#### 2. 字符串匹配器
支持`std::string`和C风格字符串，大小写敏感/不敏感可选：
| 匹配器 | 作用 |
|--------|------|
| `StrEq(str)` | 字符串内容完全相等 |
| `StrCaseEq(str)` | 忽略大小写相等 |
| `HasSubstr(substr)` | 包含指定子串 |
| `StartsWith(prefix)` | 以指定前缀开头 |
| `EndsWith(suffix)` | 以指定后缀结尾 |
| `MatchesRegex(regex)` | 匹配正则表达式 |

#### 3. 容器匹配器
支持vector、list、map等所有STL容器：
| 匹配器 | 作用 |
|--------|------|
| `ElementsAre(a,b,c...)` | 容器元素按顺序匹配指定值 |
| `ContainerEq(container)` | 两个容器完全相等 |
| `Contains(value)` | 容器包含指定值 |
| `Each(matcher)` | 容器中每个元素都匹配指定规则 |
| `SizeIs(matcher)` | 容器大小匹配指定规则 |

#### 4. 复合匹配器
支持多个匹配器组合，实现复杂逻辑校验：
| 匹配器 | 作用 |
|--------|------|
| `Not(matcher)` | 逻辑非，不匹配指定规则 |
| `AllOf(m1,m2,m3...)` | 逻辑与，同时匹配所有规则 |
| `AnyOf(m1,m2,m3...)` | 逻辑或，匹配任意一个规则 |

**示例**：
```cpp
// 匹配host以192.168开头，port在1024~65535之间
EXPECT_CALL(mock_db, Connect(StartsWith("192.168"), AllOf(Gt(1024), Lt(65535))))
    .WillOnce(Return(true));

// 匹配sql包含username，且不是空字符串
EXPECT_CALL(mock_db, Query(AllOf(HasSubstr("username"), Not(StrEq(""))), _));
```

### 5.2 基数（Cardinality）：调用次数控制
用于`Times()`子句，定义函数的预期调用次数，测试结束时gmock会自动校验，不符合则测试失败。

| 基数 | 含义 |
|------|------|
| `Times(n)` | 必须恰好被调用n次 |
| `AtLeast(n)` | 至少被调用n次 |
| `AtMost(n)` | 最多被调用n次 |
| `Between(m,n)` | 调用次数在m~n之间（包含边界） |
| `AnyNumber()` | 任意次数，不校验调用次数 |

> 自动推断规则：若不写`Times()`，gmock会根据动作自动推断：
> 1.  只有n个`WillOnce`，无`WillRepeatedly`：自动推断为`Times(n)`；
> 2.  有n个`WillOnce` + `WillRepeatedly`：自动推断为`Times(AtLeast(n))`。

### 5.3 动作（Action）：函数行为定义
定义mock函数被调用时执行的操作，支持返回值设置、出参修改、回调执行等丰富能力。

#### 1. 基础返回动作
| 动作 | 作用 |
|------|------|
| `Return(value)` | 返回指定value，必须与函数返回值类型匹配 |
| `Return()` | 用于void函数，无返回值 |
| `ReturnRef(var)` | 返回变量的引用 |
| `ReturnNew<T>(args)` | 返回`new T(args)`创建的指针 |

#### 2. 出参设置动作
| 动作 | 作用 |
|------|------|
| `SetArgPointee<N>(value)` | 给第N个参数（指针类型）指向的地址赋值 |
| `SetArgReferee<N>(value)` | 给第N个参数（引用类型）赋值 |
| `SaveArg<N>(ptr)` | 把第N个参数的值保存到ptr指向的变量 |
| `SaveArgPointee<N>(ptr)` | 把第N个指针参数指向的值保存到ptr |

> 注意：参数索引从0开始，第一个参数索引为0，第二个为1，以此类推。

#### 3. 复合与自定义动作
| 动作 | 作用 |
|------|------|
| `DoAll(a1,a2,a3...)` | 按顺序执行多个动作，最后一个动作的返回值作为函数返回值 |
| `Invoke(func)` | 调用全局函数/仿函数/lambda表达式，入参与mock函数一致 |
| `Invoke(obj, &Class::Method)` | 调用对象的成员函数 |
| `Throw(exception)` | 抛出指定异常 |
| `Assign(&var, value)` | 给指定变量赋值 |

**示例**：
```cpp
// 1. 指针出参设置
// 原函数：virtual int GetData(int* out_data) = 0;
EXPECT_CALL(mock_obj, GetData(_))
    .WillOnce(DoAll(
        SetArgPointee<0>(100), // 给指针参数赋值100
        Return(0)
    ));

// 2. lambda自定义动作
EXPECT_CALL(mock_db, Connect(_, _))
    .WillOnce(Invoke([](sslocal://flow/file_open?url=const+std%3A%3Astring%26+host%2C+int+port&flow_extra=eyJsaW5rX3R5cGUiOiJjb2RlX2ludGVycHJldGVyIn0=) {
        std::cout << "Connect to " << host << ":" << port << std::endl;
        return host == "127.0.0.1" && port == 3306;
    }));

// 3. 多动作组合
EXPECT_CALL(mock_db, Query(_, _))
    .WillOnce(DoAll(
        Assign(&call_count, 1), // 记录调用次数
        SetArgReferee<1>("result"), // 设置出参
        Return(2) // 返回值
    ));
```

### 5.4 调用顺序控制
通过`Sequence`实现严格的调用顺序校验，同一个序列中的期望，必须按声明的顺序执行，否则测试直接失败。

**进阶用法**：支持多个序列，实现多分支的顺序控制：
```cpp
TEST(SequenceTest, MultiSequence) {
    using ::testing::Sequence;
    MockDB mock_db;
    Sequence seq1, seq2;

    // seq1：Connect->Query1
    EXPECT_CALL(mock_db, Connect(_,_)).InSequence(seq1);
    EXPECT_CALL(mock_db, Query("sql1", _)).InSequence(seq1);

    // seq2：Connect->Query2
    EXPECT_CALL(mock_db, Connect(_,_)).InSequence(seq2);
    EXPECT_CALL(mock_db, Query("sql2", _)).InSequence(seq2);

    // 合法执行顺序：Connect->Query1->Query2 或 Connect->Query2->Query1
    mock_db.Connect("127.0.0.1", 3306);
    mock_db.Query("sql2", result);
    mock_db.Query("sql1", result);
}
```

## 六、进阶使用技巧
### 6.1 Mock重载函数
对于同名的重载函数，需通过**类型限定符**和**const限定符**区分不同版本，避免匹配模糊。

**示例**：
```cpp
// 带重载的接口
class FooInterface {
public:
    virtual ~FooInterface() = default;
    virtual void Bar(int x) = 0;
    virtual void Bar(const std::string& s) = 0;
    virtual int Bar(double d) const = 0;
};

// Mock类
class MockFoo : public FooInterface {
public:
    MOCK_METHOD(void, Bar, (int x), (override));
    MOCK_METHOD(void, Bar, (const std::string& s), (override));
    MOCK_METHOD(int, Bar, (double d), (const, override));
};

// 测试用例：区分重载版本
TEST(MockTest, OverloadFunc) {
    MockFoo mock_foo;
    // 匹配int版本
    EXPECT_CALL(mock_foo, Bar(5));
    // 匹配string版本，用TypedEq明确类型
    EXPECT_CALL(mock_foo, Bar(TypedEq<const std::string&>("hello")));
    // 匹配const版本，用Const()包裹mock对象
    EXPECT_CALL(Const(mock_foo), Bar(3.14)).WillOnce(Return(10));

    // 执行调用
    mock_foo.Bar(5);
    mock_foo.Bar("hello");
    const MockFoo& const_foo = mock_foo;
    const_foo.Bar(3.14);
}
```

### 6.2 Mock非虚函数
gmock默认只能mock虚函数，对于非虚函数，需通过**模板化+静态多态**实现，无任何运行时开销，也叫高性能依赖注入。

**示例**：
```cpp
// 真实类，无虚函数
class RealDB {
public:
    bool Connect(const std::string& host, int port) {
        // 真实连接逻辑
        return true;
    }
};

// 模板化被测代码，不依赖抽象类，依赖模板参数T
template <typename DBType>
class TemplateUserService {
public:
    explicit TemplateUserService(DBType* db) : db_(db) {}
    bool Login() {
        return db_->Connect("127.0.0.1", 3306);
    }
private:
    DBType* db_;
};

// Mock类，无需继承，只需实现同名函数
class MockDBNoVirtual {
public:
    MOCK_METHOD(bool, Connect, (const std::string& host, int port), ());
};

// 测试用例
TEST(TemplateTest, NonVirtualMock) {
    MockDBNoVirtual mock_db;
    EXPECT_CALL(mock_db, Connect("127.0.0.1", 3306)).WillOnce(Return(true));
    // 注入Mock类作为模板参数
    TemplateUserService<MockDBNoVirtual> service(&mock_db);
    EXPECT_TRUE(service.Login());
}
```

### 6.3 Mock静态/全局函数
gmock无法直接mock静态函数和全局函数，需通过**包装类+接口抽象**的方式，把函数调用封装到接口中，再mock该接口。

**示例**：
```cpp
// 待mock的全局函数
int GlobalCalc(int x) {
    return x * 2;
}

// 包装接口
class FuncWrapper {
public:
    virtual ~FuncWrapper() = default;
    virtual int Calc(int x) {
        return GlobalCalc(x); // 生产环境调用真实函数
    }
};

// Mock包装类
class MockFuncWrapper : public FuncWrapper {
public:
    MOCK_METHOD(int, Calc, (int x), (override));
};

// 被测代码，依赖包装接口
int BusinessLogic(FuncWrapper* wrapper, int x) {
    return wrapper->Calc(x) + 1;
}

// 测试用例
TEST(GlobalFuncTest, MockGlobal) {
    MockFuncWrapper mock_wrapper;
    EXPECT_CALL(mock_wrapper, Calc(5)).WillOnce(Return(10));
    EXPECT_EQ(BusinessLogic(&mock_wrapper, 5), 11);
}
```

### 6.4 部分Mock（委托真实对象）
当需要只mock类的部分函数，其他函数调用真实实现时，可通过`Invoke`委托给真实对象，实现部分Mock。

**示例**：
```cpp
class RealDB : public DBInterface {
public:
    bool Connect(const std::string& host, int port) override {
        std::cout << "Real Connect" << std::endl;
        return true;
    }
    int Query(const std::string& sql, std::string& result) override {
        result = "real_data";
        return 1;
    }
    void Close() override {
        std::cout << "Real Close" << std::endl;
    }
};

// 部分Mock类
class PartialMockDB : public DBInterface {
public:
    PartialMockDB() : real_db_(std::make_unique<RealDB>()) {}
    // 只mock Connect函数
    MOCK_METHOD(bool, Connect, (const std::string& host, int port), (override));
    // Query和Close调用真实实现
    int Query(const std::string& sql, std::string& result) override {
        return real_db_->Query(sql, result);
    }
    void Close() override {
        real_db_->Close();
    }
private:
    std::unique_ptr<RealDB> real_db_;
};

// 测试用例
TEST(PartialMockTest, DelegateReal) {
    PartialMockDB mock_db;
    // mock Connect返回false
    EXPECT_CALL(mock_db, Connect(_, _)).WillOnce(Return(false));
    // Query调用真实实现
    std::string result;
    EXPECT_EQ(mock_db.Query("sql", result), 1);
    EXPECT_EQ(result, "real_data");
}
```

### 6.5 ON_CALL与EXPECT_CALL的区别与配合
这是新手最容易混淆的两个核心宏，核心区别与最佳配合方式如下：
| 特性 | ON_CALL | EXPECT_CALL |
|------|---------|--------------|
| 核心作用 | 设置函数默认行为 | 设置调用期望，校验交互逻辑 |
| 调用校验 | 不校验函数是否被调用 | 严格校验调用次数、入参、顺序，不符合则测试失败 |
| 匹配优先级 | 低，无匹配的EXPECT_CALL时才生效 | 高，优先匹配EXPECT_CALL |
| 适用场景 | 设置通用默认行为，避免重复代码 | 校验当前测试用例关心的核心交互 |

**最佳实践**：
```cpp
TEST(OnCallTest, BestPractice) {
    MockDB mock_db;
    // 1. 用ON_CALL设置全局默认行为
    ON_CALL(mock_db, Connect(_, _)).WillByDefault(Return(true));
    ON_CALL(mock_db, Query(_, _)).WillByDefault(Return(0));
    ON_CALL(mock_db, Close()).WillByDefault(Return());

    // 2. 用EXPECT_CALL只校验当前用例关心的核心交互
    EXPECT_CALL(mock_db, Connect("127.0.0.1", 3306)).Times(1);
    EXPECT_CALL(mock_db, Close()).Times(1);

    // 执行逻辑，Query无EXPECT_CALL，执行ON_CALL默认行为
    UserService service(&mock_db);
    service.Login("admin", "123456");
}
```

## 七、最佳实践
1.  **面向接口编程，强制依赖注入**
    被测代码必须依赖抽象接口，而非具体实现类，通过构造函数/Setter注入依赖，禁止在被测代码中直接new真实依赖对象，这是可测试性的核心基础。

2.  **最小化期望设置，避免过度Mock**
    一个测试用例只测一个场景，仅设置与当前场景相关的期望，非核心函数用`ON_CALL`设置默认行为，避免测试用例过于脆弱，微小的代码改动就导致测试失败。

3.  **优先使用NiceMock，关键场景用StrictMock**
    常规测试使用`NiceMock`避免无关调用干扰测试结果，仅在需要严格校验协议交互（如RPC接口、硬件驱动）时使用`StrictMock`。

4.  **Mock类保持极简，不添加业务逻辑**
    Mock类的唯一作用是模拟依赖行为，禁止在Mock类中添加任何业务逻辑，否则会导致测试本身出现bug，失去测试意义。

5.  **明确区分Stub与Mock的使用场景**
    只需要控制依赖返回值时，用`ON_CALL`做Stub；需要验证被测代码与依赖的交互逻辑（调用次数、入参、顺序）时，用`EXPECT_CALL`做Mock。

6.  **遵循C++编码规范，避免未定义行为**
    接口类必须声明虚析构函数，Mock函数的限定符必须与原函数完全一致，避免编译错误或运行时未定义行为。

## 八、常见坑与避坑指南
1.  **接口类未声明虚析构函数**
    问题：基类析构函数非virtual，导致Mock对象销毁时，仅调用基类析构函数，造成内存泄漏、程序崩溃。
    解决：所有接口类的析构函数必须声明为`virtual ~Interface() = default;`。

2.  **MOCK_METHOD限定符与原函数不一致**
    问题：原函数是const的，Mock函数未加const限定符，导致未重写原函数，Mock不生效。
    解决：MOCK_METHOD的第四个参数必须与原函数的`override`、`const`、`noexcept`等限定符完全一致。

3.  **EXPECT_CALL声明顺序导致匹配失效**
    问题：gmock匹配期望时，按**声明的逆序**匹配，先声明的通用期望会覆盖后声明的精准期望。
    解决：先声明通用的、宽泛的期望，后声明特殊的、精准的期望；或给精准期望加上`RetiresOnSaturation()`。
    反例：
    ```cpp
    // 错误写法：Func(5)永远不会被匹配，先匹配到Func(_)
    EXPECT_CALL(mock, Func(_)).WillRepeatedly(Return(0));
    EXPECT_CALL(mock, Func(5)).WillOnce(Return(1));
    ```

4.  **出参设置索引错误**
    问题：参数索引从0开始，新手常从1开始，导致设置错参数。
    解决：严格遵循索引规则，第一个参数索引为0，依次递增。

5.  **void函数使用Return(value)**
    问题：void函数不能设置返回值，使用`Return(value)`会直接编译报错。
    解决：void函数使用`Return()`，或其他无返回值的动作。

6.  **忘记初始化gmock**
    问题：main函数中仅调用`InitGoogleTest`，未调用`InitGoogleMock`，导致gmock部分功能不生效。
    解决：main函数中必须同时初始化gtest和gmock。

7.  **Mock对象生命周期错误**
    问题：Mock对象提前销毁，被测代码持有野指针，导致程序崩溃、未定义行为。
    解决：Mock对象的生命周期必须长于被测对象，推荐在TEST中创建栈上的Mock对象，测试结束时才销毁。

8.  **重载函数匹配模糊**
    问题：重载函数的期望未明确类型，导致编译器无法匹配到正确的重载版本，编译报错。
    解决：使用`TypedEq<T>()`明确参数类型，用`Const()`区分const和非const版本。


