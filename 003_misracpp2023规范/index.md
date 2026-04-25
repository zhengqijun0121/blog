# Misra C++ 2023 规范


# MISRA C++ 2023

> [https://ww2.mathworks.cn/help/bugfinder/misra-cpp-2023-rules-and-directives.html?lang=en](https://ww2.mathworks.cn/help/bugfinder/misra-cpp-2023-rules-and-directives.html?lang=en)

----

## 中文版本

### 语言无关问题

[规则 0.0.1 必需] 函数不得包含不可达语句。

[规则 0.0.2 建议] 控制表达式不应是不变的。

[规则 0.1.1 建议] 不应不必要地向局部对象写入值。

[规则 0.1.2 必需] 函数返回的值应当被使用。

[规则 0.2.1 建议] 具有有限可见性的变量应至少使用一次。

[规则 0.2.2 必需] 命名的函数参数应至少使用一次。

[规则 0.2.3 建议] 具有有限可见性的类型应至少使用一次。

[规则 0.2.4 建议] 具有有限可见性的函数应至少使用一次。

[指南 0.3.1 建议] 应适当地使用浮点运算。

[指南 0.3.2 必需] 函数调用不得违反函数的前置条件。

----

### 一般原则

[规则 4.1.1 建议] 程序应符合 `ISO/IEC 14882:2017` (C++17)。

[规则 4.1.2 必需] 不应使用已弃用的特性。

[规则 4.1.3 必需] 不得出现未定义或关键未指定行为。

[规则 4.6.1 必需] 对内存位置的操作应适当排序。

----

### 词法约定

[规则 5.0.1 建议] 不应使用类似三字符序列。

[规则 5.7.1 必需] `C` 风格注释内不得使用 `/*` 字符序列。

[指南 5.7.2 建议] 代码段不应被"注释掉"。

[规则 5.7.3 必需] 在 `//` 注释中不得使用行拼接。

[规则 5.10.1 必需] 用户定义标识符应具有适当的形式。

[规则 5.13.1 必需] 在字符字面量和非原始字符串字面量中，`\` 只能用于形成已定义的转义序列或通用字符名。

[规则 5.13.2 必需] 八进制转义序列、十六进制转义序列和通用字符名应被终止。

[规则 5.13.3 必需] 不得使用八进制常量。

[规则 5.13.4 必需] 无符号整数字面量应适当添加后缀。

[规则 5.13.5 必需] 小写形式的 `L` 不得用作字面量后缀的第一个字符。

[规则 5.13.6 必需] 类型为 `long long` 的整数字面量在任何后缀中不得仅使用单个 `L` 或 `l`。

[规则 5.13.7 必需] 具有不同编码前缀的字符串字面量不得连接。

----

### 基本概念

[规则 6.0.1 必需] 块作用域声明不得在视觉上产生歧义。

[规则 6.0.2 建议] 声明具有外部链接的数组时，应显式指定其大小。

[规则 6.0.3 建议] 全局命名空间中唯一的声明应为主函数 `main`、命名空间声明和 `extern "C"` 声明。

[规则 6.0.4 必需] 标识符 `main` 不得用于除全局函数 `main` 以外的其他函数。

[规则 6.2.1 必需] 不得违反单一定义规则。

[规则 6.2.2 必需] 变量或函数的所有声明应具有相同的类型。

[规则 6.2.3 必需] 实现实体的源代码应仅出现一次。

[规则 6.2.4 必需] 头文件不得包含具有外部链接的非内联函数或对象的定义。

[规则 6.4.1 必需] 内层作用域中声明的变量不得隐藏外层作用域中声明的变量。

[规则 6.4.2 必需] 派生类不得隐藏从基类继承的函数。

[规则 6.4.3 必需] 依赖基类中存在的名称不得通过非限定查找解析。

[规则 6.5.1 建议] 具有外部链接的函数或对象应在头文件中引入。

[规则 6.5.2 建议] 应适当地指定内部链接。

[规则 6.7.1 必需] 局部变量不得具有静态存储期限。

[规则 6.7.2 必需] 不得使用全局变量。

[规则 6.8.1 必需] 不得在对象生命周期之外访问该对象。

[规则 6.8.2 强制] 函数不得返回具有自动存储期限的局部变量的引用或指针。

[规则 6.8.3 必需] 赋值运算符不得将具有自动存储期限的对象的地址赋给具有更长生命周期的对象。

[规则 6.8.4 建议] 返回对其对象引用的成员函数应适当地使用引用限定符。

[规则 6.9.1 必需] 同一实体的所有声明应使用相同类型别名。

[规则 6.9.2 建议] 不应使用标准有符号整数类型和标准无符号整数类型的名称。

----

### 标准约定

[规则 7.0.1 必需] 不得从 `bool` 类型进行转换。

[规则 7.0.2 必需] 不得转换为 `bool` 类型。

[规则 7.0.3 必需] 不得使用字符的数值。

[规则 7.0.4 必需] 位运算符和移位运算符的操作数应适当。

[规则 7.0.5 必需] 整数提升和常规算术转换不得改变操作数的符号性或类型类别。

[规则 7.0.6 必需] 数值类型之间的赋值应适当。

[规则 7.11.1 必需] `nullptr` 应是空指针常量的唯一形式。

[规则 7.11.2 必需] 作为函数参数传递的数组不得退化为指针。

[规则 7.11.3 必需] 从函数类型到指向函数类型的指针的转换仅应在适当上下文中发生。

----

### 表达式

[规则 8.0.1 建议] 应使用括号使表达式的含义适当明确。

[规则 8.1.1 必需] 非临时 `lambda` 不得隐式捕获 `this`。

[规则 8.1.2 建议] 非临时 `lambda` 中应显式捕获变量。

[规则 8.2.1 必需] 虚基类只能通过 `dynamic_cast` 转换为派生类。

[规则 8.2.2 必需] 不得使用 `C` 风格强制转换和函数表示法强制转换。

[规则 8.2.3 必需] 强制转换不得从通过指针或引用访问的类型中移除任何 `const` 或 `volatile` 限定。

[规则 8.2.4 必需] 不得在指向函数的指针和其他任何类型之间执行强制转换。

[规则 8.2.5 必需] 不得使用 `reinterpret_cast`。

[规则 8.2.6 必需] 具有整数、枚举或指向 `void` 类型对象不得强制转换为指针类型。

[规则 8.2.7 建议] 强制转换不应将指针类型转换为整数类型。

[规则 8.2.8 必需] 对象指针类型不得强制转换为除 `std::uintptr_t` 或 `std::intptr_t` 以外的整数类型。

[规则 8.2.9 必需] `typeid` 的操作数不得是多态类类型的表达式。

[规则 8.2.10 必需] 函数不得直接或间接调用自身。

[规则 8.2.11 必需] 通过省略号传递的参数应具有适当类型。

[规则 8.3.1 建议] 内置一元 `-` 运算符不应应用于无符号类型的表达式。

[规则 8.3.2 建议] 内置一元 `+` 运算符不应使用。

[规则 8.7.1 必需] 指针运算不得形成无效指针。

[规则 8.7.2 必需] 指针之间的减法仅应应用于指向同一数组元素的指针。

[规则 8.9.1 必需] 内置关系运算符 `>`、`>=`、`<` 和 `<=` 不应应用于指针类型的对象，除非它们指向同一数组的元素。

[规则 8.14.1 建议] 逻辑 `&&` 或 `||` 运算符的右操作数不应包含持久副作用。

[规则 8.18.1 强制] 对象或子对象不得复制到重叠对象。

[规则 8.18.2 建议] 赋值运算符的结果不应被使用。

[规则 8.19.1 建议] 逗号运算符不应使用。

[规则 8.20.1 建议] 具有常量操作数的无符号算术运算不应溢出。

----

### 语句

[规则 9.2.1 必需] 显式类型转换不应是表达式语句。

[规则 9.3.1 必需] 迭代语句或选择语句的主体应是复合语句。

[规则 9.4.1 必需] 所有 `if ... else if` 结构应以 `else` 语句终止。

[规则 9.4.2 必需] `switch` 语句的结构应适当。

[规则 9.5.1 必需] 传统 `for` 语句应简单。

[规则 9.5.2 必需] `for-range-initializer` 应包含最多一个函数调用。

[规则 9.6.1 建议] 应不使用 `goto` 语句。

[规则 9.6.2 必需] `goto` 语句应引用周围块中的标签。

[规则 9.6.3 必需] `goto` 语句应跳转到函数体中稍后声明的标签。

[规则 9.6.4 必需] 声明了 `[[noreturn]]` 属性的函数不应返回。

[规则 9.6.5 必需] 非 `void` 返回类型的函数应在所有路径上返回值。

----

### 声明

[规则 10.0.1 建议] 声明不应声明多个变量或成员变量。

[规则 10.1.1 建议] 指针或左值引用参数的目标类型应适当 `const` 限定。

[规则 10.1.2 必需] 应适当使用 `volatile` 限定。

[规则 10.2.1 必需] 枚举应使用显式底层类型定义。

[规则 10.2.2 建议] 不应声明无作用域枚举。

[规则 10.2.3 必需] 不应使用没有固定底层类型的无作用域枚举的数值。

[规则 10.3.1 建议] 头文件中不应有未命名命名空间。

[规则 10.4.1 必需] 不应使用 `asm` 声明。

----

### 声明符

[规则 11.3.1 建议] 不应声明数组类型的变量。

[规则 11.3.2 建议] 对象声明不应包含超过两级指针间接。

[规则 11.6.1 建议] 所有变量应被初始化。

[规则 11.6.2 强制] 在对象被设置之前，其值不得被读取。

[规则 11.6.3 必需] 在枚举列表中，隐式指定的枚举常量的值应唯一。

----

### 类

[规则 12.2.1 建议] 不应声明位域。

[规则 12.2.2 必需] 位域应具有适当类型。

[规则 12.2.3 必需] 具有有符号整数类型的命名位域不应具有一个比特的长度。

[规则 12.3.1 必需] 不应使用 `union` 关键字。

[规则 12.3.1 必需] 不应使用 `union` 关键字。

----

### 派生类

[规则 13.1.1 建议] 类不应被虚继承。

[规则 13.1.2 必需] 可访问的基类不应在同一层次结构中既是 `virtual` 的又是非 `virtual` 的。

[规则 13.3.1 必需] 用户声明的成员函数应适当使用 `virtual`、`override` 和 `final` 限定符。

[规则 13.3.2 必需] 重写虚函数中的参数不应指定不同的默认参数。

[规则 13.3.3 必需] 函数的所有声明或重写中的参数应要么是未命名的，要么具有相同的名称。

[规则 13.3.4 必需] 潜在虚指针到成员函数的比较应仅与 `nullptr` 进行。

----

### 成员访问控制

[规则 14.1.1 建议] 非静态数据成员应全部为 `private` 或全部为 `public`。

----

### 特殊成员函数

[规则 15.0.1 必需] 应适当提供特殊成员函数。

[规则 15.0.2 建议] 类的用户提供的复制和移动成员函数应具有适当签名。

[规则 15.1.1 必需] 不应从对象的构造函数或析构函数中使用其动态类型。

[规则 15.1.2 建议] 类的所有构造函数应显式初始化其所有虚基类和直接基类。

[规则 15.1.3 必需] 可使用单个参数调用的转换运算符和构造函数应为显式。

[规则 15.1.4 建议] 类的所有直接、非静态数据成员应在类对象可访问之前被初始化。

[规则 15.1.5 必需] 类应仅在其唯一构造函数时定义初始化列表构造函数。

[指南 15.8.1] 用户提供的复制赋值运算符和移动赋值运算符应处理自赋值。

----

### 重载

[规则 16.5.1 必需] 不应重载逻辑与 `&&` 和逻辑或 `||` 运算符。

[规则 16.5.2 必需] 不应重载 `address-of` 运算符。

[规则 16.6.1 建议] 对称运算符应仅作为非成员函数实现。

----

### 模板

[规则 17.8.1 必需] 不应显式特化函数模板。

----

### 异常处理

[规则 18.1.1 必需] 异常对象不应具有指针类型。

[规则 18.1.2 必需] 空的 `throw` 仅应在 `catch` 处理程序的复合语句中发生。

[规则 18.3.1 建议] 应至少有一个异常处理程序来捕获所有其他未处理的异常。

[规则 18.3.2 必需] 类类型的异常应通过 `const` 引用或引用捕获。

[规则 18.3.3 必需] 构造函数或析构函数的 `function-try-block` 的处理程序不应引用其类或其基类的非静态成员。

[规则 18.4.1 必需] 异常不友好的函数应为 `noexcept`。

[规则 18.5.1 建议] 不应尝试将 `noexcept` 函数的异常传播到调用函数。

[规则 18.5.2 建议] 不应使用程序终止函数。

----

### 预处理指令

[规则 19.0.1 必需] 以 `#` 作为第一个标记的行应为有效的预处理指令。

[规则 19.0.2 必需] 不应定义函数式宏。

[规则 19.0.3 建议] `#include` 指令前面应仅由预处理指令或注释。

[规则 19.0.4 建议] `#undef` 应仅用于在相同文件中先前定义的宏。

[规则 19.1.1 必需] 应适当使用 `defined` 预处理器运算符。

[规则 19.1.2 必需] 所有 `#else`、`#elif` 和 `#endif` 预处理指令应与相关的 `#if`、`#ifdef` 或 `#ifndef` 指令位于同一文件中。

[规则 19.1.3 必需] `#if` 或 `#elif` 预处理指令的控制表达式中使用的所有标识符应在评估之前被定义。

[规则 19.2.1 必需] 应采取预防措施，以防止头文件的内容被包含多次。

[规则 19.2.2 必需] `#include` 指令后面应跟随 `<filename>` 或 `"filename"` 序列。

[规则 19.2.3 必需] `'` 或 `"` 或 `\` 字符和 `/*` 或 `//` 字符序列不应出现在头文件名中。

[规则 19.3.1 建议] 不应使用 `#` 和 `##` 预处理器运算符。

[规则 19.3.2 必需] 紧跟 `#` 运算符的宏参数不应立即跟随 `##` 运算符。

[规则 19.3.3 必需] 混合使用宏参数的参数不应被进一步展开。

[规则 19.3.4 必需] 应使用括号以确保宏参数被适当展开。

[规则 19.3.5 必需] 看起来像预处理指令的标记不应出现在宏参数中。

[规则 19.6.1 建议] 不应使用 `#pragma` 指令和 `_Pragma` 运算符。

----

### 语言支持库

[规则 21.2.1 必需] 不应使用 `<cstdlib>` 中的库函数 `atof`、`atoi`、`atol` 和 `atoll`。

[规则 21.2.2 必需] 不应使用 `<cstring>`、`<cstdlib>`、`<cwchar>` 和 `<cinttypes>` 中的字符串处理函数。

[规则 21.2.3 必需] 不应使用 `<cstdlib>` 中的库函数 `system`。

[规则 21.2.4 必需] 不应使用宏 `offsetof`。

[规则 21.6.1 建议] 不应使用动态内存。

[规则 21.6.2 必需] 动态内存应自动管理。

[规则 21.6.3 必需] 不应使用高级内存管理。

[规则 21.6.4 必需] 如果项目定义了全局运算符 `delete` 的有大小或无大小版本，则两者都应被定义。

[规则 21.6.5 必需] 不应删除不完整类类型的指针。

[规则 21.10.1 必需] 不应使用 `<cstdarg>` 的功能。

[规则 21.10.2 必需] 不应使用标准头文件 `<csetjmp>`。

[规则 21.10.3 必需] 不应使用标准头文件 `<csignal>` 提供的功能。

----

### 诊断库

[规则 22.3.1 必需] 不应使用常量表达式作为 `assert` 宏的参数。

[规则 22.4.1 必需] 字面值零应是赋给 `errno` 的唯一值。

----

### 通用工具库

[规则 23.11.1 必需] 不应使用 `std::shared_ptr` 和 `std::unique_ptr` 的原始指针构造函数。

----

### 字符串库

[规则 24.5.1 必需] 不应使用 `<cctype>` 和 `<cwctype>` 中的字符处理函数。

[规则 24.5.2 必需] 不应使用 `<cstring>` 中的 `C++` 标准库函数 `memcpy`、`memmove`、`memcmp`。

----

### 本地化库

[规则 25.5.1 必需] 不应调用 `setlocale` 和 `std::locale::global` 函数。

[规则 25.5.2 强制] 由 `C++` 标准库函数 `localeconv`、`getenv`、`setlocale` 或 `strerror` 返回的指针必须仅被用作具有 `const` 限定类型的指针。

[规则 25.5.3 强制] 由 `C++` 标准库函数 `asctime`、`ctime`、`gmtime`、`localtime`、`localeconv`、`getenv`、`setlocale` 或 `strerror` 返回的指针在后续调用相同函数后不得被使用。

----

### 容器库

[规则 26.3.1 建议] `std::vector` 不应使用 `bool` 进行特化。

----

### 算法库

[规则 28.3.1 必需] 谓词不应具有持久副作用。

[规则 28.6.1 必需] `std::move` 的参数应是非 `const` 左值。

[规则 28.6.2 必需] 转发引用和 `std::forward` 应一起使用。

[规则 28.6.3 必需] 对象不应在其可能被移动的状态下使用。

[规则 28.6.4 必需] 应使用 `std::remove`、`std::remove_if`、`std::unique` 和 `empty` 的结果。

----

### 输入/输出库

[规则 30.0.1 必需] 不应使用 `C` 库输入/输出函数。

[规则 30.0.2 必需] 同一文件流上的读取和写入应由定位操作分隔。

----

## English Version

### Language independent issues

[Rule 0.0.1 Required] A function shall not contain unreachable statements.

[Rule 0.0.2 Advisory] Controlling expressions should not be invariant.

[Rule 0.1.1 Advisory] A value should not be unnecessarily written to a local object.

[Rule 0.1.2 Required] The value returned by a function shall be used.

[Rule 0.2.1 Advisory] Variables with limited visibility should be used at least once.

[Rule 0.2.2 Required] A named function parameter shall be used at least once.

[Rule 0.2.3 Advisory] Types with limited visibility should be used at least once.

[Rule 0.2.4 Advisory] Functions with limited visibility should be used at least once.

[Dir 0.3.1 Advisory] Floating-point arithmetic should be used appropriately.

[Dir 0.3.2 Required] A function call shall not violate the function's preconditions.

----

### General principles

[Rule 4.1.1 Advisory] A program shall conform to `ISO/IEC 14882:2017` (C++17).

[Rule 4.1.2 Required] Deprecated features should not be used.

[Rule 4.1.3 Required] There shall be no occurrence of undefined or critical unspecified behaviour.

[Rule 4.6.1 Required] Operations on a memory location shall be sequenced appropriately.

----

### Lexical conventions

[Rule 5.0.1 Advisory] Trigraph-like sequences should not be used.

[Rule 5.7.1 Required] The character sequence `/*` shall not be used within a `C-style` comment.

[Dir 5.7.2 Advisory] Sections of code should not be "commented out".

[Rule 5.7.3 Required] Line-splicing shall not be used in `//` comments.

[Rule 5.10.1 Required] User-defined identifiers shall have an appropriate form.

[Rule 5.13.1 Required] Within character literals and non raw-string literals, `\` shall only be used to form a defined escape sequence or universal character name.

[Rule 5.13.2 Required] Octal escape sequences, hexadecimal escape sequences and universal character names shall be terminated.

[Rule 5.13.3 Required] Octal constants shall not be used.

[Rule 5.13.4 Required] Unsigned integer literals shall be appropriately suffixed.

[Rule 5.13.5 Required] The lowercase form of `L` shall not be used as the first character in a literal suffix.

[Rule 5.13.6 Required] An integer-literal of type `long long` shall not use a single `L` or `l` in any suffix.

[Rule 5.13.7 Required] String literals with different encoding prefixes shall not be concatenated.

----

### Basic concepts

[Rule 6.0.1 Required] Block scope declarations shall not be visually ambiguous.

[Rule 6.0.2 Advisory] When an array with external linkage is declared, its size should be explicitly specified.

[Rule 6.0.3 Advisory] The only declarations in the global namespace should be `main`, namespace declarations and `extern "C"` declarations.

[Rule 6.0.4 Required] The identifier `main` shall not be used for a function other than the global function `main`.

[Rule 6.2.1 Required] The one-definition rule shall not be violated.

[Rule 6.2.2 Required] All declarations of a variable or function shall have the same type.

[Rule 6.2.3 Required] The source code used to implement an entity shall appear only once.

[Rule 6.2.4 Required] A header file shall not contain definitions of functions or objects that are non-inline and have external linkage.

[Rule 6.4.1 Required] A variable declared in an inner scope shall not hide a variable declared in an outer scope.

[Rule 6.4.2 Required] Derived classes shall not conceal functions that are inherited from their bases.

[Rule 6.4.3 Required] A name that is present in a dependent base shall not be resolved by unqualified lookup.

[Rule 6.5.1 Advisory] A function or object with external linkage should be introduced in a header file.

[Rule 6.5.2 Advisory] Internal linkage should be specified appropriately.

[Rule 6.7.1 Required] Local variables shall not have static storage duration.

[Rule 6.7.2 Required] Global variables shall not be used.

[Rule 6.8.1 Required] An object shall not be accessed outside of its lifetime.

[Rule 6.8.2 Mandatory] A function must not return a reference or a pointer to a local variable with automatic storage duration.

[Rule 6.8.3 Required] An assignment operator shall not assign the address of an object with automatic storage duration to an object with a greater lifetime.

[Rule 6.8.4 Advisory] Member functions returning references to their object should be ref-qualified appropriately.

[Rule 6.9.1 Required] The same type aliases shall be used in all declarations of the same entity.

[Rule 6.9.2 Advisory] The names of the standard signed integer types and standard unsigned integer types should not be used.

----

### Standard conventions

[Rule 7.0.1 Required] There shall be no conversion from type `bool`.

[Rule 7.0.2 Required] There shall be no conversion to type `bool`.

[Rule 7.0.3 Required] The numerical value of a character shall not be used.

[Rule 7.0.4 Required] The operands of bitwise operators and shift operators shall be appropriate.

[Rule 7.0.5 Required] Integral promotion and the usual arithmetic conversions shall not change the signedness or the type category of an operand.

[Rule 7.0.6 Required] Assignment between numeric types shall be appropriate.

[Rule 7.11.1 Required] `nullptr` shall be the only form of the null-pointer-constant.

[Rule 7.11.2 Required] An array passed as a function argument shall not decay to a pointer.

[Rule 7.11.3 Required] A conversion from function type to pointer-to-function type shall only occur in appropriate contexts.

----

### Expressions

[Rule 8.0.1 Advisory] Parentheses should be used to make the meaning of an expression appropriately explicit.

[Rule 8.1.1 Required] A non-transient `lambda` shall not implicitly capture `this`.

[Rule 8.1.2 Advisory] Variables should be captured explicitly in a non-transient `lambda`.

[Rule 8.2.1 Required] A virtual base class shall only be cast to a derived class by means of `dynamic_cast`.

[Rule 8.2.2 Required] `C-style` casts and functional notation casts shall not be used.

[Rule 8.2.3 Required] A cast shall not remove any `const` or `volatile` qualification from the type accessed via a pointer or by reference.

[Rule 8.2.4 Required] Casts shall not be performed between a pointer to function and any other type.

[Rule 8.2.5 Required] `reinterpret_cast` shall not be used.

[Rule 8.2.6 Required] An object with integral, enumerated, or pointer to `void` type shall not be cast to a pointer type.

[Rule 8.2.7 Advisory] A cast should not convert a pointer type to an integral type.

[Rule 8.2.8 Required] An object pointer type shall not be cast to an integral type other than `std::uintptr_t` or `std::intptr_t`.

[Rule 8.2.9 Required] The operand to `typeid` shall not be an expression of polymorphic class type.

[Rule 8.2.10 Required] Functions shall not call themselves, either directly or indirectly.

[Rule 8.2.11 Required] An argument passed via ellipsis shall have an appropriate type.

[Rule 8.3.1 Advisory] The built-in unary `-` operator should not be applied to an expression of unsigned type.

[Rule 8.3.2 Advisory] The built-in unary `+` operator should not be used.

[Rule 8.7.1 Required] Pointer arithmetic shall not form an invalid pointer.

[Rule 8.7.2 Required] Subtraction between pointers shall only be applied to pointers that address elements of the same array.

[Rule 8.9.1 Required] The built-in relational operators `>`, `>=`, `<` and `<=` shall not be applied to objects of pointer type, except where they point to elements of the same array.

[Rule 8.14.1 Advisory] The right-hand operand of a logical `&&` or `||` operator should not contain persistent side effects.

[Rule 8.18.1 Mandatory] An object or subobject must not be copied to an overlapping object.

[Rule 8.18.2 Advisory] The result of an assignment operator should not be used.

[Rule 8.19.1 Advisory] The comma operator should not be used.

[Rule 8.20.1 Advisory] An unsigned arithmetic operation with constant operands should not wrap.

----

### Statements

[Rule 9.2.1 Required] An explicit type conversion shall not be an expression statement.

[Rule 9.3.1 Required] The body of an iteration-statement or a selection-statement shall be a compound-statement.

[Rule 9.4.1 Required] All `if ... else if` constructs shall be terminated with an `else` statement.

[Rule 9.4.2 Required] The structure of a `switch` statement shall be appropriate.

[Rule 9.5.1 Required] Legacy `for` statements should be simple.

[Rule 9.5.2 Required] A `for-range-initializer` shall contain at most one function call.

[Rule 9.6.1 Advisory] The `goto` statement should not be used.

[Rule 9.6.2 Required] A `goto` statement shall reference a label in a surrounding block.

[Rule 9.6.3 Required] The `goto` statement shall jump to a label declared later in the function body.

[Rule 9.6.4 Required] A function declared with the `[[noreturn]]` attribute shall not return.

[Rule 9.6.5 Required] A function with `non-void` return type shall return a value on all paths.

----

### Declarations

[Rule 10.0.1 Advisory] A declaration should not declare more than one variable or member variable.

[Rule 10.1.1 Advisory] The target type of a pointer or lvalue reference parameter should be `const-qualified` appropriately.

[Rule 10.1.2 Required] The `volatile` qualifier shall be used appropriately.

[Rule 10.2.1 Required] An enumeration shall be defined with an explicit underlying type.

[Rule 10.2.2 Advisory] Unscoped enumerations should not be declared.

[Rule 10.2.3 Required] The numeric value of an unscoped enumeration with no fixed underlying type shall not be used.

[Rule 10.3.1 Advisory] There should be no unnamed namespaces in header files.

[Rule 10.4.1 Required] The `asm` declaration shall not be used.

----

### Declarators

[Rule 11.3.1 Advisory] Variables of array type should not be declared.

[Rule 11.3.2 Advisory] The declaration of an object should contain no more than two levels of pointer indirection.

[Rule 11.6.1 Advisory] All variables should be initialized.

[Rule 11.6.2 Mandatory] The value of an object must not be read before it has been set.

[Rule 11.6.3 Required] Within an enumerator list, the value of an implicitly-specified enumeration constant shall be unique.

----

### Classes

[Rule 12.2.1 Advisory] Bit-fields should not be declared.

[Rule 12.2.2 Required] A bit-field shall have an appropriate type.

[Rule 12.2.3 Required] A named bit-field with signed integer type shall not have a length of one bit.

[Rule 12.3.1 Required] The `union` keyword shall not be used.

----

### Derived classes

[Rule 13.1.1 Advisory] Classes should not be inherited virtually.

[Rule 13.1.2 Required] An accessible base class shall not be both virtual and non-virtual in the same hierarchy.

[Rule 13.3.1 Required] User-declared member functions shall use the `virtual`, `override` and `final` specifiers appropriately.

[Rule 13.3.2 Required] Parameters in an overriding virtual function shall not specify different default arguments.

[Rule 13.3.3 Required] The parameters in all declarations or overrides of a function shall either be unnamed or have identical names.

[Rule 13.3.4 Required] A comparison of a potentially virtual pointer to member function shall only be with `nullptr`.

----

### Member access control

[Rule 14.1.1 Advisory] Non-static data members should be either all private or all public.

----

### Special member functions

[Rule 15.0.1 Required] Special member functions shall be provided appropriately.

[Rule 15.0.2 Advisory] User-provided copy and move member functions of a class should have appropriate signatures.

[Rule 15.1.1 Required] An object's dynamic type shall not be used from within its constructor or destructor.

[Rule 15.1.2 Advisory] All constructors of a class should explicitly initialize all of its virtual base classes and immediate base classes.

[Rule 15.1.3 Required] Conversion operators and constructors that are callable with a single argument shall be explicit.

[Rule 15.1.4 Advisory] All direct, non-static data members of a class should be initialized before the class object is accessible.

[Rule 15.1.5 Required] A class shall only define an initializer-list constructor when it is the only constructor.

[Dir 15.8.1 Advisory] User-provided copy assignment operators and move assignment operators shall handle self-assignment.

----

### Overriding

[Rule 16.5.1 Required] The logical `AND` and logical `OR` operators shall not be overloaded.

[Rule 16.5.2 Required] The `address-of` operator shall not be overloaded.

[Rule 16.6.1 Advisory] Symmetrical operators should only be implemented as non-member functions.

----

### Templates

[Rule 17.8.1 Required] Function templates shall not be explicitly specialized.

----

### Exception handling

[Rule 18.1.1 Required] An exception object shall not have pointer type.

[Rule 18.1.2 Required] An empty `throw` shall only occur within the compound-statement of a `catch` handler.

[Rule 18.3.1 Advisory] There should be at least one exception handler to catch all otherwise unhandled exceptions.

[Rule 18.3.2 Required] An exception of class type shall be caught by `const` reference or reference.

[Rule 18.3.3 Required] Handlers for a `function-try-block` of a constructor or destructor shall not refer to non-static members from their class or its bases.

[Rule 18.4.1 Required] Exception-unfriendly functions shall be `noexcept`.

[Rule 18.5.1 Advisory] A `noexcept` function should not attempt to propagate an exception to the calling function.

[Rule 18.5.2 Advisory] Program-terminating functions should not be used.

----

### Preprocessing directives

[Rule 19.0.1 Required] A line whose first token is `#` shall be a valid preprocessing directive.

[Rule 19.0.2 Required] Function-like macros shall not be defined.

[Rule 19.0.3 Advisory] `#include` directives should only be preceded by preprocessor directives or comments.

[Rule 19.0.4 Advisory] `#undef` should only be used for macros defined previously in the same file.

[Rule 19.1.1 Required] The `defined` preprocessor operator shall be used appropriately.

[Rule 19.1.2 Required] All `#else`, `#elif` and `#endif` preprocessor directives shall reside in the same file as the `#if`, `#ifdef` or `#ifndef` directive to which they are related.

[Rule 19.1.3 Required] All identifiers used in the controlling expression of `#if` or `#elif` preprocessing directives shall be defined prior to evaluation.

[Rule 19.2.1 Required] Precautions shall be taken in order to prevent the contents of a header file being included more than once.

[Rule 19.2.2 Required] The `#include` directive shall be followed by either a `<filename>` or `"filename"` sequence.

[Rule 19.2.3 Required] The `'` or `"` or `\` characters and the `/*` or `//` character sequences shall not occur in a header file name.

[Rule 19.3.1 Advisory] The `#` and `##` preprocessor operators should not be used.

[Rule 19.3.2 Required] A macro parameter immediately following a `#` operator shall not be immediately followed by a `##` operator.

[Rule 19.3.3 Required] The argument to a mixed-use macro parameter shall not be subject to further expansion.

[Rule 19.3.4 Required] Parentheses shall be used to ensure macro arguments are expanded appropriately.

[Rule 19.3.5 Required] Tokens that look like a preprocessing directive shall not occur within a macro argument.

[Rule 19.6.1 Advisory] The `#pragma` directive and the `_Pragma` operator should not be used.

----

### Language support library

[Rule 21.2.1 Required] The library functions `atof`, `atoi`, `atol` and `atell` from `<cstdlib>` shall not be used.

[Rule 21.2.2 Required] The string handling functions from `<cstring>`, `<cstdlib>`, `<cwchar>` and `<cinttypes>` shall not be used.

[Rule 21.2.3 Required] The library function `system` from `<cstdlib>` shall not be used.

[Rule 21.2.4 Required] The macro `offsetof` shall not be used.

[Rule 21.6.1 Advisory] Dynamic memory should not be used.

[Rule 21.6.2 Required] Dynamic memory shall be managed automatically.

[Rule 21.6.3 Required] Advanced memory management shall not be used.

[Rule 21.6.4 Required] If a project defines either a sized or unsized version of a global operator `delete`, then both shall be defined.

[Rule 21.6.5 Required] A pointer to an incomplete class type shall not be deleted.

[Rule 21.10.1 Required] The features of `<cstdarg>` shall not be used.

[Rule 21.10.2 Required] The standard header file `<csetjmp>` shall not be used.

[Rule 21.10.3 Required] The facilities provided by the standard header file `<csignal>` shall not be used.

----

### Diagnostics library

[Rule 22.3.1 Required] The `assert` macro shall not be used with a constant-expression.

[Rule 22.4.1 Required] The literal value zero shall be the only value assigned to `errno`.

----

### General utilities library

[Rule 23.11.1 Required] The raw pointer constructors of `std::shared_ptr` and `std::unique_ptr` should not be used.

----

### Strings library

[Rule 24.5.1 Required] The character handling functions from `<cctype>` and `<cwctype>` shall not be used.

[Rule 24.5.2 Required] The `C++` Standard Library functions `memcpy`, `memmove`, `memcmp` from `<cstring>` shall not be used.

----

### Localization library

[Rule 25.5.1 Required] The `setlocale` and `std::locale::global` functions shall not be called.

[Rule 25.5.2 Mandatory] The pointers returned by the `C++` Standard Library functions `localeconv`, `getenv`, `setlocale` or `strerror` must only be used as if they have pointer to `const-qualified` type.

[Rule 25.5.3 Mandatory] The pointer returned by the `C++` Standard Library functions `asctime`, `ctime`, `gmtime`, `localtime`, `localeconv`, `getenv`, `setlocale` or `strerror` must not be used following a subsequent call to the same function.

----

### Containers library

[Rule 26.3.1 Advisory] `std::vector` should not be specialized with `bool`.

----

### Algorithms library

[Rule 28.3.1 Required] Predicates shall not have persistent side effects.

[Rule 28.6.1 Required] The argument to `std::move` shall be a `non-const` lvalue.

[Rule 28.6.2 Required] Forwarding references and `std::forward` shall be used together.

[Rule 28.6.3 Required] An object shall not be used while in a potentially moved-from state.

[Rule 28.6.4 Required] The result of `std::remove`, `std::remove_if`, `std::unique` and `empty` shall be used.

----

### Input/output library

[Rule 30.0.1 Required] The `C` Library input/output functions shall not be used.

[Rule 30.0.2 Required] Reads and writes on the same file stream shall be separated by a positioning operation.

----

