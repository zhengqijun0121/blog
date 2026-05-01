# C++ 编码规范


# C++ 编码规范

本文基于**Google C++ Style Guide**编写，完整覆盖规范的核心设计哲学、强制规则与工程最佳实践，是工业界最具影响力的C++工程化编码标准。

## 1. 头文件

头文件的正确使用直接决定代码的编译效率、可读性和依赖管理，核心规则如下：

### 1.1 基础要求

- 每个 `.cpp` 实现文件原则上必须对应一个关联的 `.h` 头文件，例外仅为单元测试、测试程序、仅含 `main()` 函数的小型文件。
- 头文件必须是**自包含的（Self-contained）**，用户无需额外引入其他头文件即可正常使用其提供的接口。
- 禁止在头文件中定义具名命名空间的 `static` 变量/函数，禁止使用 `using namespace xxx;`（尤其是 `using namespace std;`），避免全局命名空间污染。

### 1.2 头文件保护宏

必须使用 `#define` 头文件保护机制，防止头文件被重复包含，格式为：`<PROJECT>_<PATH>_<FILE>_H_`

```cpp
#ifndef FOO_BAR_BAZ_H_
#define FOO_BAR_BAZ_H_

// 头文件核心内容

#endif  // FOO_BAR_BAZ_H_
```

- 禁止使用 `#pragma once` 等非标准编译器扩展。

### 1.3 前置声明

**优先使用`#include`引入完整头文件，尽量避免前置声明**。仅在无依赖风险的极特殊场景可使用，核心原因：
- 前置声明会隐藏依赖关系，头文件修改时无法触发依赖代码的重编译，引发静默的语义变更。
- 前置声明`std`命名空间的符号会导致未定义行为。
- 函数/模板的前置声明会限制API的兼容变更（如参数类型加宽、新增默认模板参数）。

### 1.4 内联函数

- 仅当函数体极短（通常1-10行）、性能敏感且无复杂逻辑时，才允许在头文件中定义内联函数，必须显式标记 `inline` 保证ODR（单一定义规则）安全。
- 禁止在头文件内联定义析构函数、虚函数，其隐式生成的代码往往比表面更复杂，易导致代码膨胀。

### 1.5 头文件包含顺序与格式

包含顺序必须严格遵循以下分组，每组之间用空行分隔，同组内按字母序排序：
1. 关联头文件（当前`.cpp`对应的`.h`文件，优先保证头文件自包含性）
2. `C` 系统头文件（如`<unistd.h>`、`<stdlib.h>`）
3. `C++` 标准库头文件（如`<string>`、`<vector>`）
4. 第三方库头文件（如 `<boost/shared_ptr.hpp>`、`<gtest/gtest.h>`）
5. 项目内其他头文件（如 `"base/common.h"`）

- 标准库/系统头文件使用尖括号`<>`，项目内头文件使用双引号`""`，禁止使用`.`/`..`相对路径别名。

示例：
```cpp
#include "foo/server/fooserver.h"

#include <sys/types.h>
#include <unistd.h>

#include <string>
#include <vector>

#include <gtest/gtest.h>

#include "base/basictypes.h"
#include "foo/server/bar.h"
```

`.clang-format` 文件内容如下所示：

```
---
Language: Cpp
BasedOnStyle: Google
AccessModifierOffset: -4
ColumnLimit: 120
ContinuationIndentWidth: 8
IncludeCategories:
    - Regex: '^<ext/.*\.h>'
      Priority: 1
      SortPriority: 1
    - Regex: '^<[^/]*\.h>'
      Priority: 1
      SortPriority: 1
    - Regex: '^<[^/]*>'
      Priority: 2
      SortPriority: 2
    - Regex: '^<.*\.h>'
      Priority: 3
      SortPriority: 3
    - Regex: '^<.*\.hpp>'
      Priority: 3
      SortPriority: 3
    - Regex: '^".*'
      Priority: 4
      SortPriority: 4
IndentCaseLabels: false
IndentWidth: 4
Standard: c++17
```

## 2. 作用域

### 4.1 命名空间核心规则

- 除极少数例外，所有代码必须置于命名空间 `demo` 中，顶层命名空间必须基于项目名全局唯一，禁止污染全局命名空间。
- 命名空间名称使用**全小写+下划线（snake_case）** 命名。
- 禁止使用 `using namespace xxx;` 的 using 指令，仅允许在 `.cpp` 文件内使用 `using ::foo::bar;` 的有限using声明，禁止在头文件中使用。
- 禁止使用内联命名空间（inline namespaces）。
- `.cpp` 文件中，无需对外暴露的辅助函数/变量，优先使用**匿名命名空间**封装，实现内部链接，禁止在头文件中使用匿名命名空间。

```cpp
// .cpp文件示例
namespace {
const int kMaxRetryCount = 3;
void HelperFunc() { /* ... */ }
}  // namespace

namespace foo {
// 对外暴露的代码
}  // namespace foo
```

### 4.2 变量作用域规则

- 局部变量：在函数内尽可能晚声明，就近初始化，禁止在循环头外声明循环变量；允许在 `if`/`while`/`for` 语句中声明变量，限制其作用域。
- 静态/全局变量：禁止使用非POD类型的静态/全局变量，避免跨编译单元的初始化顺序未定义问题；静态变量必须保证线程安全。
- 禁止使用全局函数，优先将非成员函数置于命名空间中，禁止仅为了分组静态成员而创建类。
- 成员变量：类的非静态数据成员必须设为 `private`，仅常量成员可例外；测试夹具类的成员变量仅在 `.cpp` 文件内可设为 `protected`。

-----

## 4. 类

### 5.1 核心设计原则
- **优先组合，而非继承**：仅当满足「is-a」关系时使用继承，且必须使用`public`继承，禁止使用私有/保护继承，禁止虚继承（除非极特殊场景）。
- **单一职责**：一个类应只负责一件事，公有API数量尽量精简，不超过7个为宜。
- **struct与class的边界**：`struct`仅用于纯数据聚合（无私有成员、无自定义构造函数、无虚函数、无继承）；只要包含行为（方法），必须使用`class`。

### 5.2 构造函数规则
- 单参数构造函数必须加`explicit`关键字，禁止隐式类型转换。
- 禁止在构造函数中执行复杂、可能失败、会引发副作用的初始化逻辑；若初始化可能失败，使用工厂函数替代。
- 禁止在构造函数中调用虚函数，避免未定义行为。
- 委托构造函数、继承构造函数仅在简化代码、无歧义时使用。
- 对于可拷贝/移动的类，要么显式定义拷贝/移动构造函数和赋值运算符，要么显式用`=delete`禁用，要么完全不定义（使用编译器默认生成）。

### 5.3 成员声明顺序
类内成员必须严格按以下顺序声明，空的区段可省略：
1.  `public:`区段 → `protected:`区段 → `private:`区段（对外接口优先，隐藏实现细节）
2.  每个区段内的顺序：
    - 类型与类型别名（`typedef`/`using`、枚举、嵌套类/结构体）
    - （仅struct允许）非静态数据成员
    - 静态常量
    - 工厂函数
    - 构造函数与赋值运算符
    - 析构函数
    - 所有其他成员函数（静态/非静态、友元函数）
    - 所有其他数据成员（静态/非静态）

### 5.4 其他类规则
- 虚函数必须显式标记`override`或`final`，禁止重复写`virtual`关键字。
- 友元仅用于类与其紧密关联的类/函数，禁止滥用友元打破封装。
- 禁止将类的大方法内联定义在类声明中，仅极短、性能敏感的 trivial 方法可内联。

-----

## 6. 函数

### 6.1 函数基础规范

- **短小聚焦**：函数长度建议不超过 `40` 行，过长的函数必须拆分为更小的子函数，保证逻辑可理解、可测试。
- **参数顺序**：输入参数在前，输出参数在后；输入参数优先使用`const T&`常量引用，输出参数必须使用指针`T*`，明确标识可修改语义。
  示例：`void Parse(const std::string& input，int* output);`
- 禁止使用默认函数参数，避免重载决议歧义、API兼容问题。
- 函数重载仅当所有重载版本语义完全一致时使用，保证读者无需查看定义即可理解调用行为。
- 函数返回值：禁止忽略有状态的返回值（如`absl::Status`），必须做错误处理。

### 6.2 特殊函数规则

- 普通函数与静态成员函数：优先置于命名空间中，禁止创建仅包含静态成员的「工具类」。
- 虚函数：接口类的虚析构函数必须为`public`，基类的虚函数必须保证派生类的兼容性。
- lambda表达式：仅用于短平快的局部逻辑，禁止复杂嵌套，捕获列表必须显式，避免隐式捕获导致的生命周期问题。

-----

## 6. Google 奇技

----

## 7. 命名规则

命名规则是规范中最核心的一致性约束，**见名知意+无歧义**是核心原则，不同类型的实体有严格的命名区分。

| 实体类型 | 命名规则 | 正确示例 | 错误示例 |
| :---   | :--- | :--- | :--- |
| 文件名 | 全小写+下划线（优先）/短横线 | `my_class.h`、`http_server.cc` | `MyClass.h`、`myClass.cc` |
| 类型名（类、结构体、枚举、类型别名） | 大驼峰（PascalCase），首字母大写，无下划线 | `FooBar`、`UrlTable`、`Status` | `fooBar`、`foo_bar` |
| 函数名（普通函数、类成员函数） | 大驼峰（PascalCase） | `DoSomething()`、`GetValue()` | `doSomething()`、`do_something()` |
| 变量（局部变量、函数参数、全局变量） | 全小写+下划线（snake_case） | `user_name`、`buffer_size` | `userName`、`UserName` |
| 类私有/保护成员变量 | 全小写+下划线，**必须以下划线结尾** | `buffer_size_`、`user_name_` | `m_bufferSize`、`buffer_size` |
| 常量（const/constexpr，全局/类内） | `k`开头 + 大驼峰 | `kMaxBufferSize`、`kDaysInWeek` | `MAX_BUFFER_SIZE`、`max_buffer_size` |
| 枚举值 | 同常量规则，`k`开头 + 大驼峰 | `kErrorOk`、`kErrorNotFound` | `ERROR_OK`、`error_ok` |
| 宏定义 | 全大写+下划线，必须带项目前缀 | `PROJECT_MY_MACRO` | `my_macro`、`MY_MACRO` |
| 命名空间名 | 全小写+下划线 | `foo_bar`、`google_base` | `FooBar`、`FOO_BAR` |
| 模板参数 | 大驼峰（类型参数）/ 全小写（非类型参数） | `typename T`、`int MaxSize` | `typename t`、`int max_size` |

### 7.1 文件命名

- 文件名要全部小写，单词之间使用下划线连接。

### 7.2 类型命名

- 类型名称的每个单词首字母均大写，不包含下划线。
- 类型包括类、结构体、类型定义 (typedef 或 using)、枚举、类型模板参数。

### 7.3 变量命名

- 变量 (包括函数参数) 和数据成员名一律小写，单词之间用下划线连接。

**普通变量**

举例:

```cpp
string table_name;  // 好 - 用下划线.
string tablename;   // 好 - 全小写.

string tableName;  // 差 - 混合大小写
```

**类成员变量**

不管是静态的还是非静态的，类数据成员都可以和普通变量一样，但要接下划线.

```cpp
class TableInfo {
private:
    string table_name_;  // 好 - 后加下划线.
    string tablename_;   // 好.
    static Pool<TableInfo>* pool_;  // 好.
};
```

**结构体变量**

不管是静态的还是非静态的，结构体数据成员都可以和普通变量一样，不用像类那样接下划线:

```cpp
struct UrlTableProperties {
    string name;
    int num_entries;
    static Pool<UrlTableProperties>* pool;
};
```

**常量**

声明为 `constexpr` 或 `const` 的变量，或在程序运行期间其值始终保持不变的，命名时以 `k` 开头，大小写混合。例如:

```cpp
const int kDaysInAWeek = 7;
```

### 7.4 函数命名

- 常规函数使用大小写混合，取值和设值函数则要求与变量名匹配:

```cpp
void MyExcitingFunction();
void MyExcitingMethod();
void my_exciting_member_variable();
void set_my_exciting_member_variable();
```

- 对于首字母缩写的单词，更倾向于将它们视作一个单词进行首字母大写。写作 `StartRpc()` 而非 `StartRPC()`。

### 7.5 命名空间命名

- 命名空间使用小写字母，单词间用下划线分隔。
- 对于 `internal` 命名空间，一般为内部实现使用。

### 7.6 枚举命名

- 枚举的命名应当和 **常量** 或 **宏** 一致: `kEnumName` 或是 `ENUM_NAME`
- 单独的枚举值应该优先采用 **常量** 的命名方式. 但 **宏** 方式的命名也可以接受。

```cpp
enum UrlTableErrors {
    kOK = 0,
    kErrorOutOfMemory,
    kErrorMalformedInput,
};

enum AlternateUrlTableErrors {
    OK = 0,
    OUT_OF_MEMORY = 1,
    MALFORMED_INPUT = 2,
};
```

### 7.7 宏命名

- 宏命名采用全大写的方式，单词之间使用下划线分割。

### 7.8 特例命名

- 如果命名的实体与已有 `C/C++` 实体相似，可参考现有命名策略.

----

## 8. 注释

- 注释虽然写起来很痛苦，但对保证代码可读性至关重要。
- 注释固然很重要，但最好的代码应当本身就是文档. 有意义的类型名和变量名，要远胜过要用注释解释的含糊不清的名字。
- 可以使用 `//` 或 `/* */` 注释，但是需要统一。
- 在每个文件头添加一个版权声明。

----

## 9. 代码格式化规则

格式化规则的核心是保证视觉一致性，所有规则可通过`clang-format`工具自动化落地。

### 9.1 行长度

- 每一行代码字符数不超过 `120`。

### 9.2 字符编码

- 尽量不使用非 `ASCII` 字符，使用时必须使用 `UTF-8` 编码.

### 9.3 缩进

- 只使用空格，每次缩进 `4` 个空格。
- 禁止使用 `Tab` 字符。

### 9.4 函数声明与定义

- 返回类型和函数名在同一行，参数也尽量放在同一行，如果放不下就对形参分行，分行方式与 函数调用 一致.

函数看上去像这样:

```cpp
ReturnType ClassName::FunctionName(Type par_name1，Type par_name2) {
    DoSomething();
    ...
}
```

如果同一行文本太多，放不下所有参数:

```cpp
ReturnType ClassName::ReallyLongFunctionName(Type par_name1，Type par_name2,
                                             Type par_name3) {
    DoSomething();
    ...
}
```

甚至连第一个参数都放不下:

```cpp
ReturnType LongClassName::ReallyReallyReallyLongFunctionName(
        Type par_name1， // 8 space indent
        Type par_name2,
        Type par_name3) {
    DoSomething();  // 4 space indent
    ...
}
```

注意以下几点:
- 使用好的参数名。
- 只有在参数未被使用或者其用途非常明显时，才能省略参数名。
- 如果返回类型和函数名在一行放不下，分行。
- 如果返回类型与函数声明或定义分行了，不要缩进。
- 左圆括号总是和函数名在同一行。
- 函数名和左圆括号间永远没有空格。
- 圆括号与参数间没有空格。
- 左大括号总在最后一个参数同一行的末尾处，不另起新行。
- 右大括号总是单独位于函数最后一行，或者与左大括号同一行。
- 右圆括号和左大括号间总是有一个空格。
- 所有形参应尽可能对齐。
- 缺省缩进为 4 个空格。
- 换行后的参数保持 8 个空格的缩进。

### 9.5 Lambda 表达式

- Lambda 表达式对形参和函数体的格式化和其他函数一致; 捕获列表同理，表项用逗号隔开。
- 若用引用捕获，在变量名和 `&` 之间不留空格.

### 9.6 函数调用

- 要么一行写完函数调用，要么在圆括号里对参数分行，要么参数另起一行且缩进四格。如果没有其它顾虑的话，尽可能精简行数，比如把多个参数适当地放在同一行里。

函数调用遵循如下形式：

```cpp
bool retval = DoSomething(argument1，argument2，argument3);
```

如果同一行放不下，可断为多行，后面每一行都和第一个实参对齐，左圆括号后和右圆括号前不要留空格：

```cpp
bool retval = DoSomething(averyveryveryverylongargument1,
                          argument2，argument3);
```

参数也可以放在次行，缩进四格：

```cpp
if (...) {
    ...
    ...
    if (...) {
        DoSomething(
            argument1，argument2， // 4 空格缩进
            argument3，argument4);
    }
```

### 9.7 列表初始化



1.  **大括号**：
    - 所有控制流语句（`if`/`else`/`for`/`while`/`switch`）必须使用大括号，即使单行语句也不例外，杜绝Apple goto fail类漏洞。
    - 大括号使用「K&R风格」：左大括号不换行，与语句同行；右大括号单独一行；else与前一个if的右大括号同行。
    示例：
    ```cpp
    if (condition) {
        DoSomething();
    } else {
        DoOtherThing();
    }
    ```
2.  **空格规则**：
    - 条件语句的括号与条件之间留1个空格，函数名与参数括号之间无空格。
    - 二元运算符（`=`/`+`/`-`/`*`/`/`/`<`/`>`等）两侧各留1个空格。
    - 预处理指令`#include`/`#define`后留1个空格，无额外缩进。
3.  **指针与引用**：`*`和`&`紧贴类型名，而非变量名。示例：`const std::string& input`、`int* output`。
4.  **换行规则**：函数参数过长时，每个参数单独换行，与左括号对齐；表达式过长时，在运算符前换行，保证可读性。
5.  **编码**：源文件使用UTF-8编码，非ASCII字符必须极少使用，且必须使用UTF-8格式，禁止使用`wchar_t`/`char16_t`/`char32_t`（Windows API交互除外）。

## 九、注释规范
### 9.1 核心原则
注释必须解释**「为什么这么做」**，而非**「代码做了什么」**；代码本身应能清晰表达「做什么」，无需冗余注释复述代码逻辑。

### 9.2 注释分类与规则
1.  **文件注释**：每个头文件/实现文件顶部必须有版权声明，简要说明文件的功能、用途、作者信息，禁止冗余的文件描述。
2.  **类注释**：类声明前必须加注释，说明类的功能、使用场景、线程安全特性、生命周期约束，无需复述实现细节。
3.  **函数注释**：
    - 对外暴露的API函数必须加注释，说明函数的功能、输入输出参数的含义、返回值、副作用、线程安全、错误处理、调用前提。
    - 内部辅助函数若逻辑清晰，可省略注释；复杂逻辑必须加注释说明设计思路。
    - 函数重载集仅需一个统一的「伞形注释」，无需每个重载单独注释。
4.  **变量注释**：全局变量、类成员变量必须加注释说明用途、取值范围、生命周期约束；局部变量仅当逻辑不清晰时加注释。
5.  **实现注释**：函数内的复杂逻辑、非显而易见的分支、特殊处理、性能优化、临时方案，必须加行内注释。
6.  **TODO注释**：必须使用全大写`TODO`，后跟责任人/BUG ID/设计文档链接，以及明确的修复时间/触发事件，禁止无明确上下文的TODO。
    示例：`// TODO(bug 123456): 移除该兼容逻辑，2026Q4后所有客户端已支持新接口`
7.  **禁用注释**：禁止注释掉的代码（死代码），直接删除；禁止无意义的吐槽、梗、个人标记类注释。

## 十、核心特性管控与最佳实践
### 10.1 类型与类型转换
- 禁止使用C风格强制类型转换，必须使用C++的`static_cast`/`const_cast`/`reinterpret_cast`，且仅在必要时使用；`reinterpret_cast`必须极度谨慎，保证内存布局安全。
- 禁止隐式类型转换，尤其是整数、指针、bool之间的隐式转换。
- `auto`仅当类型显而易见、且不损害可读性时使用，禁止滥用`auto`隐藏类型信息，降低代码可读性。
- 优先使用`std::string_view`替代`const char*`/`const std::string&`作为只读字符串输入参数，避免拷贝。
- 优先使用`absl::Span`替代裸指针+长度的数组参数，保证边界安全。

### 10.2 内存管理
- 优先使用智能指针，禁止裸指针管理所有权：优先使用`std::unique_ptr`，仅当确需共享所有权时使用`std::shared_ptr`，禁止使用已废弃的`std::auto_ptr`。
- 裸指针仅用于不持有所有权的场景，必须保证指针的生命周期短于所指向的对象。
- 禁止手动调用`new`/`delete`，优先使用`std::make_unique`/`std::make_shared`创建对象。
- 禁止内存池之外的自定义`new`/`delete`重载。

### 10.3 现代C++特性使用边界
- 模板：仅当能显著提升代码复用性、无歧义时使用，禁止复杂的模板元编程、SFINAE等高级特性，除非收益可明确验证。
- 概念（Concepts）：仅用于简化模板约束、提升可读性，禁止复杂的约束组合。
- 协程：仅在官方许可的场景使用，禁止滥用。
- Lambda：仅用于局部回调、短逻辑，禁止长生命周期的lambda捕获，避免悬垂引用。
- 移动语义：合理使用右值引用、移动构造/赋值，避免不必要的拷贝，禁止对右值引用参数滥用`std::move`/`std::forward`。

### 10.4 多线程与并发
- 全局/静态变量必须保证线程安全，禁止线程不安全的懒初始化。
- 禁止使用线程局部存储`thread_local`，除非极特殊场景。
- 优先使用标准库的互斥量、条件变量，禁止无锁编程（除非性能收益可验证，且有充分的测试）。

## 十一、其他重要规则
1.  **包容性语言**：代码命名、注释中必须使用包容性语言，禁止使用带有歧视、冒犯性的术语（如master/slave、blacklist/whitelist），使用性别中立的语言。
2.  **预处理宏**：尽量避免使用宏，禁止使用宏定义C++ API、类结构、函数；必须使用时，名称必须全局唯一，带项目前缀。
3.  **可移植性**：代码需考虑编译器、平台的可移植性，避免依赖编译器未定义行为、平台专属特性。
4.  **测试友好**：代码设计需保证可测试性，核心逻辑必须可单元测试，禁止在核心逻辑中硬编码依赖。


