# Misra C++ 2008 规范


# Misra Cpp 2008

## 中文版本

### 语言无关问题

#### 不必要的构造

规则 0-1-1 （必需）项目不应包含无法访问的代码。

规则 0-1-2 （必需）项目不应包含不可行的路径。

规则 0-1-3 （必需）项目不应包含未使用的变量。

规则 0-1-4 （必需）项目不应包含只有一种用途的非易失性 POD 变量。

规则 0-1-5 （必需）项目不应包含未使用的类型声明。

规则 0-1-6 （必需）项目不应包含非易失性变量的实例，这些变量被赋予了以后从未使用过的值。

规则 0-1-7 （必需）应始终使用具有非 void 返回类型且不是重载运算符的函数返回的值。

规则 0-1-8 （必需）所有返回类型为 void 的函数都应具有外部副作用。

规则 0-1-9 （必需）不得有死代码。

规则 0-1-10（必需）每个定义的函数应至少调用一次。

规则 0-1-11（必需）在非虚拟函数中不得有未使用的参数（命名或未命名）。

规则 0-1-12（必需）在虚拟函数和覆盖它的所有函数的参数集中不应有未使用的参数（命名或未命名）。

#### 贮存

规则 0-2-1（必需）不应将对象分配给重叠对象。

#### 运行时失败

规则 0-3-1（文档）应通过使用以下至少一项来确保运行时故障的最小化：

1. 静态分析工具/技术；

2. 动态分析工具/技术；

3. 显式编码检查以处理运行时错误。

规则 0-3-2（必需）如果函数生成错误信息，则应测试该错误信息。

#### 算术

规则 0-4-1（文档）应记录缩放整数或定点算术的使用。

规则 0-4-2（文档）应记录浮点运算的使用。

规则 0-4-3（文档）浮点实现应符合定义的浮点标准。

----

### 一般的

#### 语言

规则 1-0-1（必需）所有代码均应符合 ISO/IEC 14882:2003“包含技术勘误 1 的 C++ 标准”。

规则 1-0-2（必需）仅当多个编译器具有通用的、已定义的接口时，才能使用它们。

规则 1-0-3（文档）应确定并记录所选编译器中整数除法的实现。

----

### 词汇约定

#### 字符集

规则 2-2-1（文档）字符集和相应的编码应记录在案。

#### 三字序列

规则 2-3-1（必需）不得使用三合符。

#### 替代令牌

规则 2-5-1（建议）不应使用有向图。

#### 注释

规则 2-7-1（必需）字符序列 /* 不得在 C 样式注释中使用。

规则 2-7-2（必需）不得使用 C 样式注释“注释掉”代码部分。

规则 2-7-3（建议）不应使用 C++ 注释“注释掉”代码部分。

＃＃＃＃ 身份标识

规则 2-10-1（必需）不同的标识符应在印刷上明确。

规则 2-10-2（必需）在内部范围内声明的标识符不应隐藏在外部范围内声明的标识符。

规则 2-10-3（必需）typedef 名称（包括限定，如果有的话）应是唯一标识符。

规则 2-10-4（必需）类、联合或枚举名称（包括限定，如果有）应是唯一标识符。

规则 2-10-5（必需）不应重复使用具有静态存储持续时间的非成员对象或函数的标识符名称。

规则 2-10-6（必需）如果一个标识符指代一个类型，它不应该指代同一范围内的一个对象或一个函数。

#### 字面量

规则 2-13-1（必需）仅应使用 ISO/IEC 14882:2003 中定义的转义序列。

规则 2-13-2（必需）不得使用八进制常量（非零）和八进制转义序列（非“\0”）。

规则 2-13-3（必需）应将“U”后缀应用于所有无符号类型的八进制或十六进制整数文字。

规则 2-13-4（必需）文字后缀应为大写。

规则 2-13-5（必需）不应连接窄和宽字符串文字。

----

### 基本概念

#### 声明和定义

规则 3-1-1（必需）应该可以在多个翻译单元中包含任何头文件而不违反单一定义规则。

规则 3-1-2（必需）不应在块范围内声明函数。

规则 3-1-3（必需）当声明一个数组时，它的大小应显式声明或通过初始化隐式定义。

#### 一个定义规则

规则 3-2-1（必需）对象或函数的所有声明都应具有兼容的类型。

规则 3-2-2（必需）不得违反单一定义规则。

规则 3-2-3（必需）在多个翻译单元中使用的类型、对象或函数应在一个且仅一个文件中声明。

规则 3-2-4（必需）具有外部链接的标识符应具有准确的定义。

#### 声明性区域和范围

规则 3-3-1（必需） 具有外部链接的对象或函数应在头文件中声明。

规则 3-3-2（必需）如果函数具有内部链接，则所有重新声明都应包括静态存储类说明符。

#### 名称查询

规则 3-4-1（必需）声明为对象或类型的标识符应在块中定义，以使其可见性最小化。

#### 类型

规则 3-9-1（必需）在所有声明和重新声明中，用于对象、函数返回类型或函数参数的类型应逐个令牌相同。

规则 3-9-2（建议）指示大小和符号的 typedef 应该用来代替基本的数字类型。

规则 3-9-3（必需）不应使用浮点值的底层位表示。

----

### 标准转换

#### 整体促销

规则 4-5-1（必需）除了赋值运算符 =、逻辑运算符 &&、||、!、等式运算符 == 和 != 之外，不应将 bool 类型的表达式用作内置运算符的操作数，一元 & 运算符和条件运算符。

规则 4-5-2（必需）类型为 enum 的表达式不得用作除下标运算符 [ ]、赋值运算符 =、相等运算符 == 和 !=、一元 & 运算符之外的内置运算符的操作数，以及关系运算符 <、<=、>、>=。

规则 4-5-3（必需）类型（纯）char 和 wchar_t 的表达式不得用作除赋值运算符 =、相等运算符 == 和 != 以及一元 & 运算符之外的内置运算符的操作数.

#### 指针转换

规则 4-10-1（必需）不得将 NULL 用作整数值。

规则 4-10-2（必需）不应将文字零 (0) 用作空指针常量。

----

### 表达式

#### 主要表达式

规则 5-0-1 （必需）在标准允许的任何评估顺序下，表达式的值都应相同。

规则 5-0-2 （建议）有限的依赖应该放在表达式中的 C++ 运算符优先规则上。

规则 5-0-3 （必需）不得将 cvalue 表达式隐式转换为不同的基础类型。

规则 5-0-4 （必需）隐式整数转换不应改变底层类型的符号。

规则 5-0-5 （必需）不应有隐式浮点整数转换。

规则 5-0-6 （必需）隐式整数或浮点转换不应减少基础类型的大小。

规则 5-0-7 （必需）不应有 cvalue 表达式的显式浮点整数转换。

规则 5-0-8 （必需）显式整数或浮点转换不应增加 cvalue 表达式的基础类型的大小。

规则 5-0-9 （必需）显式整数转换不应更改 cvalue 表达式的基础类型的符号。

规则 5-0-10（必需）如果按位运算符 ~ 和 << 应用于基础类型为 unsigned char 或 unsigned short 的操作数，则结果应立即转换为操作数的基础类型。

规则 5-0-11（必需）plain char 类型只能用于存储和使用字符值。

规则 5-0-12（必需）有符号字符和无符号字符类型只能用于数值的存储和使用。

规则 5-0-13（必需）if 语句的条件和迭代语句的条件应具有 bool 类型。

规则 5-0-14（必需）条件运算符的第一个操作数应具有 bool 类型。

规则 5-0-15（必需）数组索引应是指针运算的唯一形式。

规则 5-0-16（必需）指针操作数和使用该操作数的指针运算产生的任何指针都应寻址同一数组的元素。

规则 5-0-17（必需）指针之间的减法仅适用于寻址同一数组元素的指针。

规则 5-0-18（必需）>、>=、<、<= 不应应用于指针类型的对象，除非它们指向同一个数组。

规则 5-0-19（必需）对象的声明应包含不超过两级指针间接。

规则 5-0-20（必需）二进制位运算符的非常量操作数应具有相同的基础类型。

规则 5-0-21（必需）位运算符只能应用于无符号基础类型的操作数。

#### 后缀表达式

规则 5-2-1 （必需）逻辑 && 或 || 的每个操作数 应该是一个后缀表达式。

规则 5-2-2 （必需）只能通过 dynamic_cast 将指向虚拟基类的指针转换为指向派生类的指针。

规则 5-2-3 （建议）不应在多态类型上执行从基类到派生类的强制转换。

规则 5-2-4 （必需）不应使用 C 样式转换（除了 void 转换）和功能符号转换（除了显式构造函数调用）。

规则 5-2-5 （必需）强制转换不得从指针或引用的类型中删除任何 const 或 volatile 限定。

规则 5-2-6 （必需）强制转换不应将指向函数的指针转换为任何其他指针类型，包括指向函数类型的指针。

规则 5-2-7 （必需）不得将具有指针类型的对象直接或间接转换为不相关的指针类型。

规则 5-2-8 （必需）不得将整数类型的对象或指向 void 类型的指针转​​换为指针类型的对象。

规则 5-2-9 （建议）强制转换不应将指针类型转换为整数类型。

规则 5-2-10（建议）增量 (++) 和减量 (--) 运算符不应与表达式中的其他运算符混合。

规则 5-2-11（必需）逗号运算符、&& 运算符和 || 运算符不应超载。

规则 5-2-12（必需）作为函数参数传递的数组类型的标识符不应衰减为指针。

#### 一元表达式

规则 5-3-1（必需） ! 的每个操作数 运算符，逻辑 && 或逻辑 || 运算符的类型应为 bool。

规则 5-3-2（必需）一元减号运算符不应应用于基础类型为无符号的表达式。

规则 5-3-3（必需）一元 & 运算符不应被重载。

规则 5-3-4（必需）对 sizeof 运算符的操作数的评估不应包含副作用。

#### 移位运算符

规则 5-8-1（必需）移位运算符的右手操作数应位于比左手操作数的基础类型的位宽度小 0 到 1 之间。

#### 逻辑与运算符

规则 5-14-1（必需）逻辑 && 或 || 的右手操作数 运算符不得包含副作用。

#### 赋值运算符

规则 5-17-1（必需）应保留二元运算符及其赋值运算符形式之间的语义等价性。

#### 逗号运算符

规则 5-18-1（必需）不得使用逗号运算符。

#### 常量表达式

规则 5-19-1（建议）对常量无符号整数表达式的求值不应导致回绕。

----

### 语句

#### 表达式语句

规则 6-2-1（必需）赋值运算符不得用于子表达式。

规则 6-2-2（必需）不应直接或间接测试浮点表达式是否相等或不相等。

规则 6-2-3（必需）在预处理之前，空语句只能出现在一行上；如果空语句后面的第一个字符是空白字符，则它后面可以跟注释。

#### 复合语句

规则 6-3-1（必需）构成 switch、while、do ... while 或 for 语句主体的语句应为复合语句。

#### 选择语句

规则 6-4-1（必需）if (condition) 构造后应跟复合语句。else 关键字后应跟复合语句或另一个 if 语句。

规则 6-4-2（必需）所有 if - else if 结构应以 else 子句终止。

规则 6-4-3（必需）switch 语句应是格式良好的 switch 语句。

规则 6-4-4（必需）仅当最封闭的复合语句是 switch 语句的主体时，才应使用 switch-label。

规则 6-4-5（必需）无条件的 throw 或 break 语句应终止每个非空 switch 子句。

规则 6-4-6（必需）switch 语句的最后一个子句应为默认子句。

规则 6-4-7（必需）switch 语句的条件不得为 bool 类型。

规则 6-4-8（必需）每个 switch 语句应至少有一个 case 子句。

----

### 迭代语句

#### for 语句

规则 6-5-1（必需）for 循环应包含一个不具有浮点类型的循环计数器。

规则 6-5-2（必需）如果循环计数器未被 -- 或 ++ 修改，则在条件内，循环计数器应仅用作 <=、<、> 或 >= 的操作数。

规则 6-5-3（必需）不得在条件或语句中修改循环计数器。

规则 6-5-4（必需）循环计数器应修改为以下之一：--、++、-=n 或 +=n；其中 n 在循环期间保持不变。

规则 6-5-5（必需）不得在条件或表达式内修改循环计数器以外的循环控制变量。

规则 6-5-6（必需）除了在语句中修改的循环计数器之外的循环控制变量应具有 bool 类型。

#### 跳转语句

规则 6-6-1（必需）goto 语句引用的任何标签都应在同一块中声明，或在包含 goto 语句的块中声明。

规则 6-6-2（必需）goto 语句应跳转到稍后在同一函数体中声明的标签。

规则 6-6-3（必需）continue 语句只能在格式良好的 for 循环中使用。

规则 6-6-4（必需）对于任何迭代语句，用于循环终止的 break 或 goto 语句不得超过一个。

规则 6-6-5（必需）函数应在函数末尾有一个退出点。

----

### 声明

#### 说明符

规则 7-1-1（必需）未修改的变量应为 const 限定的。

规则 7-1-2（必需）如果相应对象未修改，函数中的指针或引用参数应声明为指向 const 的指针或对 const 的引用。

规则 7-1-3（必需）具有枚举基础类型的表达式应仅具有与枚举的枚举数相对应的值。

#### 枚举声明

规则 7-2-1（必需）具有枚举基础类型的表达式应仅具有与枚举的枚举数相对应的值。

#### 命名空间

规则 7-3-1（必需）全局命名空间应仅包含 main、命名空间声明和 extern "C" 声明。

规则 7-3-2（必需）标识符 main 不得用于除全局函数 main 之外的函数。

规则 7-3-3（必需）头文件中不应有未命名的名称空间。

规则 7-3-4（必需）不得使用使用指令。

规则 7-3-5（必需）同一命名空间中标识符的多个声明不应跨越该标识符的 using 声明。

规则 7-3-6（必需）不应在头文件中使用 Using-directives 和 using-declarations（不包括类范围或函数范围 using-declarations）。

#### asm 声明

规则 7-4-1（文档） 汇编程序的所有使用都应记录在案。

规则 7-4-2（必需）只能使用 asm 声明引入汇编程序指令。

规则 7-4-3（必需）汇编语言应被封装和隔离。

#### 联动规格

规则 7-5-1（必需）函数不应返回指向函数内定义的自动变量（包括参数）的引用或指针。

规则 7-5-2（必需）不应将具有自动存储功能的对象的地址分配给在第一个对象不再存在后可能持续存在的另一个对象。

规则 7-5-3（必需）函数不应返回引用或指向通过引用或 const 引用传递的参数的指针。

规则 7-5-4（建议）函数不应直接或间接调用自身。

----

### 声明符

＃＃＃＃ 一般的

规则 8-0-1（必需）一个 init-declarator-list 或一个 member-declarator-list 应分别由一个单独的 init-declarator 或 member-declarator 组成。

#### 声明符的含义

规则 8-3-1（必需）覆盖虚函数中的参数应使用与它们覆盖的函数相同的默认参数，否则不应指定任何默认参数。

#### 函数定义

规则 8-4-1（必需）不应使用省略号来定义函数。

规则 8-4-2（必需）函数重新声明中用于参数的标识符应与声明中的标识符相同。

规则 8-4-3（必需）具有非 void 返回类型的函数的所有退出路径都应具有带有表达式的显式返回语句。

规则 8-4-4（必需）函数标识符应用于调用函数，或者应以 & 开头。

#### 初始化器

规则 8-5-1（必需）所有变量在使用前都应具有定义的值。

规则 8-5-2（必需）在数组和结构的非零初始化中，应使用大括号来指示和匹配结构。

规则 8-5-3（必需）在枚举器列表中，= 构造不应用于显式初始化除第一个之外的成员，除非所有项目都显式初始化。

----

### 类

#### 成员函数

规则 9-3-1（必需）Const 成员函数不应返回非常量指针或对类数据的引用。

规则 9-3-2（必需）成员函数不应将非常量句柄返回给类数据。

规则 9-3-3（必需）如果成员函数可以设为静态，则应设为静态，否则如果可设为 const，则应设为 const。

#### 工会

规则 9-5-1（必需）不得使用工会。

#### 位域

规则 9-6-1（必需）当需要对表示位域的位进行绝对定位时，应记录位域的行为和打包。

规则 9-6-2（必需）位域应为 bool 类型或显式无符号或有符号整数类型。

规则 9-6-3（必需）位域不应具有枚举类型。

规则 9-6-4（必需）带符号整数类型的命名位字段的长度应超过一位。

----

### 派生类

#### 多个基类

规则 10-1-1（建议） 类不应从虚拟库派生。

规则 10-1-2（必需）只有在菱形层次结构中使用基类时才应将其声明为虚拟的。

规则 10-1-3（必需）可访问的基类在同一层次结构中不应既是虚拟的又是非虚拟的。

#### 成员名称查找

规则 10-2-1（建议）多重继承层次结构中的所有可访问实体名称都应该是唯一的。

#### 虚函数

规则 10-3-1（必需）通过继承层次结构的每条路径上的每个虚函数的定义不得超过一个。

规则 10-3-2（必需）每个覆盖的虚函数都应使用 virtual 关键字声明。

规则 10-3-3（必需）只有当虚函数本身被声明为纯虚函数时，它才能被纯虚函数覆盖。

----

###会员访问控制

＃＃＃＃ 一般的

规则 11-0-1（必需）非 POD 类类型中的成员数据应是私有的。

----

### 特殊成员函数

#### 构造函数

规则 12-1-1（必需）不应从其构造函数或析构函数的主体中使用对象的动态类型。

规则 12-1-2（建议）一个类的所有构造函数都应该显式调用它的所有直接基类和所有虚拟基类的构造函数。

规则 12-1-3（必需）所有可使用基本类型的单个参数调用的构造函数都应显式声明。

#### 复制类对象

规则 12-8-1（必需）复制构造函数只能初始化它的基类和它所属的类的非静态成员。

规则 12-8-2（必需）复制赋值运算符应在抽象类中声明为受保护或私有。

----

### 模板

#### 模板声明

规则 14-5-1（必需）只能在非关联命名空间的命名空间中声明非成员泛型函数。

规则 14-5-2（必需）当存在具有单个参数的模板构造函数是泛型参数时，应声明复制构造函数。

规则 14-5-3（必需）当模板赋值运算符的参数是泛型参数时，应声明复制赋值运算符。

#### 名称解析

规则 14-6-1（必需）在具有依赖基的类模板中，可以在该依赖基中找到的任何名称都应使用限定 ID 或 this->

规则 14-6-2（必需）重载决议选择的函数应解析为先前在翻译单元中声明的函数。

#### 模板实例化和特化

规则 14-7-1（必需）所有类模板、函数模板、类模板成员函数和类模板静态成员应至少实例化一次。

规则 14-7-2（必需）对于任何给定的模板特化，使用特化中使用的模板参数的模板的显式实例化不应导致程序格式错误。

规则 14-7-3（必需）模板的所有部分和显式特化应在与其主模板的声明相同的文件中声明。

#### 函数模板特化

规则 14-8-1（必需）重载的函数模板不应明确专门化。

规则 14-8-2（建议）函数调用的可行函数集应该不包含函数特化，或者只包含函数特化。

----

### 异常处理

#### 一般的

规则 15-0-1（文档）异常只能用于错误处理。

规则 15-0-2（建议）异常对象不应具有指针类型。

规则 15-0-3（必需）不得使用 goto 或 switch 语句将控制转移到 try 或 catch 块中。

#### 抛出异常

规则 15-1-1（必需）throw 语句的赋值表达式本身不应导致抛出异常。

规则 15-1-2（必需）不应显式抛出 NULL。

规则 15-1-3（必需）空的 throw (throw;) 只能用在 catch 处理程序的复合语句中。

#### 处理异常

规则 15-3-1（必需）仅在程序启动后和程序终止前才应提出异常。

规则 15-3-2（建议）应该至少有一个异常处理程序来捕获所有未处理的异常

规则 15-3-3（必需）类构造函数或析构函数的函数尝试块实现的处理程序不得引用此类或其基类的非静态成员。

规则 15-3-4（必需）在代码中显式抛出的每个异常都应在可能导致该点的所有调用路径中具有兼容类型的处理程序。

规则 15-3-5（必需）类类型异常应始终通过引用来捕获。

规则 15-3-6（必需）如果在单个 try-catch 语句或函数try-block 中为派生类及其部分或全部基类提供多个处理程序，则处理程序应按最派生到基类的顺序排列.

规则 15-3-7（必需）如果在单个 try-catch 语句或函数尝试块中提供多个处理程序，则任何省略号（catch-all）处理程序应最后出现。

#### 异常规范

规则 15-4-1（必需）如果使用异常规范声明函数，那么（在其他翻译单元中）同一函数的所有声明都应使用相同的类型 ID 集进行声明。

#### 特殊功能

规则 15-5-1（必需）类析构函数不应异常退出。

规则 15-5-2（必需）如果函数声明包含异常规范，则函数只能抛出指定类型的异常。

规则 15-5-3（必需）不应隐式调用 terminate() 函数。

----

### 预处理指令

＃＃＃＃ 一般的

规则 16-0-1（必需）文件中的 #include 指令只能位于其他预处理器指令或注释之前。

规则 16-0-2（必需）宏在全局命名空间中只能是 #define-d 或 #undef-d。

规则 16-0-3（必需）不得使用#undef。

规则 16-0-4（必需）不应定义类似函数的宏。

规则 16-0-5（必需）类函数宏的参数不应包含看起来像预处理指令的标记。

规则 16-0-6（必需）在类函数宏的定义中，参数的每个实例都应括在括号中，除非它用作 # 或 ## 的操作数。

规则 16-0-7（必需）在 #if 或 #elif 预处理器指令中不得使用未定义的宏标识符，除非作为已定义运算符的操作数。

规则 16-0-8（必需）如果 # 标记作为一行中的第一个标记出现，那么它应该紧跟一个预处理标记。

#### 条件包含

规则 16-1-1（必需）定义的预处理器操作符只能以两种标准形式之一使用。

规则 16-1-2（必需）所有 #else、#elif 和 #endif 预处理器指令应与它们相关的 #if 或 #ifdef 指令位于同一文件中。

#### 源文件包含

规则 16-2-1（必需）预处理器只能用于文件包含和包含保护。

规则 16-2-2（必需）C++ 宏只能用于包含防护、类型限定符或存储类说明符。

规则 16-2-3（必需）应提供包括防护装置。

规则 16-2-4（必需）'、"、/* 或 // 字符不得出现在头文件名中。

规则 16-2-5（建议） \ 字符不应出现在头文件名中。

规则 16-2-6（必需）#include 指令后应跟随 <filename> 或 “filename” 序列。

#### 宏替换

规则 16-3-1（必需）在单个宏定义中最多应出现一次 # 或 ## 运算符。

规则 16-3-2（建议）不应使用 # 和 ## 运算符。

#### Pragma 指令

规则 16-6-1（文档）#pragma 指令的所有使用都应记录在案。

----

### 库介绍

#### 一般的

规则 17-0-1（必需）不得定义、重新定义或取消定义标准库中的保留标识符、宏和函数。

规则 17-0-2（必需）不得重用标准库宏和对象的名称。

规则 17-0-3（必需）标准库函数的名称不应被覆盖。

规则 17-0-4（文档）所有库代码应符合 MISRA C++。

规则 17-0-5（文档）不应使用 setjmp 宏和 longjmp 函数。

----

### 语言支持库

#### 一般的

规则 18-0-1（必需）不得使用 C 库。

规则 18-0-2（必需）不应使用库 <cstdlib> 中的库函数 atof、atoi 和 atol。

规则 18-0-3（必需）不得使用库 <cstdlib> 中的库函数 abort、exit、getenv 和 system。

规则 18-0-4（必需）不得使用库 <ctime> 的时间处理功能。

规则 18-0-5（必需）不应使用库 <cstring> 的无界函数。

#### 实现属性

规则 18-2-1（必需）不应使用宏 offsetof。

#### 动态内存管理

规则 18-4-1（必需）不得使用动态堆内存分配。

#### 其他运行时支持

规则 18-7-1（必需）不应使用 <csignal> 的信号处理设施。

----

### 诊断库

#### 错误编号

规则 19-3-1（必需）不应使用错误指示符 errno。

----

### 输入/输出库

#### 一般的

规则 27-0-1（必需）不应使用流输入/输出库 <cstdio>。

----

## English Version

### Language independent issues

#### Unnecessary constructs

Rule 0-1-1	(Required)	A project shall not contain unreachable code.

Rule 0-1-2	(Required)	A project shall not contain infeasible paths.

Rule 0-1-3	(Required)	A project shall not contain unused variables.

Rule 0-1-4	(Required)	A project shall not contain non-volatile POD variables having only one use.

Rule 0-1-5	(Required)	A project shall not contain unused type declarations.

Rule 0-1-6	(Required)	A project shall not contain instances of non- volatile variables being given values that are never subsequently used.

Rule 0-1-7	(Required)	The value returned by a function having a non-void return type that is not an overloaded operator shall always be used.

Rule 0-1-8	(Required)	All functions with void return type shall have external side effect(s).

Rule 0-1-9	(Required)	There shall be no dead code.

Rule 0-1-10	(Required)	Every defined function shall be called at least once.

Rule 0-1-11	(Required)	There shall be no unused parameters (named or unnamed) in non-virtual functions.

Rule 0-1-12	(Required)	There shall be no unused parameters (named or unnamed) in the set of parameters for a virtual function and all the functions that override it.

#### Storage

Rule 0-2-1	(Required)	An object shall not be assigned to an overlapping object.

#### Runtime failures

Rule 0-3-1	(Document)	Minimization of run-time failures shall be ensured by the use of at least one of:

1.	static analysis tools/techniques;

2.	dynamic analysis tools/techniques;

3.	explicit coding of checks to handle run-time faults.

Rule 0-3-2	(Required)	If a function generates error information, then that error information shall be tested.

#### Arithmetic

Rule 0-4-1	(Document)	Use of scaled-integer or fixed-point arithmetic shall be documented.

Rule 0-4-2	(Document)	Use of floating-point arithmetic shall be documented.

Rule 0-4-3	(Document)	Floating-point implementations shall comply with a defined floating-point standard.

----

### General

#### Language

Rule 1-0-1	(Required)	All code shall conform to ISO/IEC 14882:2003 "The C++ Standard Incorporating Technical Corrigendum 1".

Rule 1-0-2	(Required)	Multiple compilers shall only be used if they have a common, defined interface.

Rule 1-0-3	(Document)	The implementation of integer division in the chosen compiler shall be determined and documented.

----

### Lexical conventions

#### Character sets

Rule 2-2-1	(Document)	The character set and the corresponding encoding shall be documented.

#### Trigraph sequences

Rule 2-3-1	(Required)	Trigraphs shall not be used.

#### Alternative tokens

Rule 2-5-1	(Advisory)	Digraphs should not be used.

#### Comments

Rule 2-7-1	(Required)	The character sequence /* shall not be used within a C-style comment.

Rule 2-7-2	(Required)	Sections of code shall not be "commented out" using C-style comments.

Rule 2-7-3	(Advisory)	Sections of code should not be "commented out" using C++ comments.

#### Identifiers

Rule 2-10-1	(Required)	Different identifiers shall be typographically unambiguous.

Rule 2-10-2	(Required)	Identifiers declared in an inner scope shall not hide an identifier declared in an outer scope.

Rule 2-10-3	(Required)	A typedef name (including qualification, if any) shall be a unique identifier.

Rule 2-10-4	(Required)	A class, union or enum name (including qualification, if any) shall be a unique identifier.

Rule 2-10-5	(Required)	The identifier name of a non-member object or function with static storage duration should not be reused.

Rule 2-10-6	(Required)	If an identifier refers to a type, it shall not also refer to an object or a function in the same scope.

#### Literals

Rule 2-13-1	(Required)	Only those escape sequences that are defined in ISO/IEC 14882:2003 shall be used.

Rule 2-13-2	(Required)	Octal constants (other than zero) and octal escape sequences (other than "\0") shall not be used.

Rule 2-13-3	(Required)	A "U " suffix shall be applied to all octal or hexadecimal integer literals of unsigned type.

Rule 2-13-4	(Required)	Literal suffixes shall be upper case.

Rule 2-13-5	(Required)	Narrow and wide string literals shall not be concatenated.

----

### Basic concepts

#### Declarations and definitions

Rule 3-1-1	(Required)	It shall be possible to include any header file in multiple translation units without violating the One Definition Rule.

Rule 3-1-2	(Required)	Functions shall not be declared at block scope.

Rule 3-1-3	(Required)	When an array is declared, its size shall either be stated explicitly or defined implicitly by initialization.

#### One Definition Rule

Rule 3-2-1	(Required)	All declarations of an object or function shall have compatible types.

Rule 3-2-2	(Required)	The One Definition Rule shall not be violated.

Rule 3-2-3	(Required)	A type, object or function that is used in multiple translation units shall be declared in one and only one file.

Rule 3-2-4	(Required)	An identifier with external linkage shall have exactly one definition.

#### Declarative regions and scope

Rule 3-3-1	(Required)	Objects or functions with external linkage shall be declared in a header file.

Rule 3-3-2	(Required)	If a function has internal linkage then all re-declarations shall include the static storage class specifier.

#### Name lookup

Rule 3-4-1	(Required)	An identifier declared to be an object or type shall be defined in a block that minimizes its visibility.

#### Types

Rule 3-9-1	(Required)	The types used for an object, a function return type, or a function parameter shall be token-for-token identical in all declarations and re-declarations.

Rule 3-9-2	(Advisory)	typedefs that indicate size and signedness should be used in place of the basic numerical types.

Rule 3-9-3	(Required)	The underlying bit representations of floating-point values shall not be used.

----

### Standard conversions

#### Integral promotions

Rule 4-5-1	(Required)	Expressions with type bool shall not be used as operands to built-in operators other than the assignment operator =, the logical operators &&, ||, !, the equality operators == and !=, the unary & operator, and the conditional operator.

Rule 4-5-2	(Required)	Expressions with type enum shall not be used as operands to built-in operators other than the subscript operator [ ], the assignment operator =, the equality operators == and !=, the unary & operator, and the relational operators <, <=, >, >=.

Rule 4-5-3	(Required)	Expressions with type (plain) char and wchar_t shall not be used as operands to built-in operators other than the assignment operator =, the equality operators == and !=, and the unary & operator.

#### Pointer conversions

Rule 4-10-1	(Required)	NULL shall not be used as an integer value.

Rule 4-10-2	(Required)	Literal zero (0) shall not be used as the null-pointer-constant.

----

### Expressions

#### Primary expressions

Rule 5-0-1	(Required)	The value of an expression shall be the same under any order of evaluation that the standard permits.

Rule 5-0-2	(Advisory)	Limited dependence should be placed on C++ operator precedence rules in expressions.

Rule 5-0-3	(Required)	A cvalue expression shall not be implicitly converted to a different underlying type.

Rule 5-0-4	(Required)	An implicit integral conversion shall not change the signedness of the underlying type.

Rule 5-0-5	(Required)	There shall be no implicit floating-integral conversions.

Rule 5-0-6	(Required)	An implicit integral or floating-point conversion shall not reduce the size of the underlying type.

Rule 5-0-7	(Required)	There shall be no explicit floating-integral conversions of a cvalue expression.

Rule 5-0-8	(Required)	An explicit integral or floating-point conversion shall not increase the size of the underlying type of a cvalue expression.

Rule 5-0-9	(Required)	An explicit integral conversion shall not change the signedness of the underlying type of a cvalue expression.

Rule 5-0-10	(Required)	If the bitwise operators ~ and << are applied to an operand with an underlying type of unsigned char or unsigned short, the result shall be immediately cast to the underlying type of the operand.

Rule 5-0-11	(Required)	The plain char type shall only be used for the storage and use of character values.

Rule 5-0-12	(Required)	Signed char and unsigned char type shall only be used for the storage and use of numeric values.

Rule 5-0-13	(Required)	The condition of an if-statement and the condition of an iteration-statement shall have type bool.

Rule 5-0-14	(Required)	The first operand of a conditional-operator shall have type bool.

Rule 5-0-15	(Required)	Array indexing shall be the only form of pointer arithmetic.

Rule 5-0-16	(Required)	A pointer operand and any pointer resulting from pointer arithmetic using that operand shall both address elements of the same array.

Rule 5-0-17	(Required)	Subtraction between pointers shall only be applied to pointers that address elements of the same array.

Rule 5-0-18	(Required)	>, >=, <, <= shall not be applied to objects of pointer type, except where they point to the same array.

Rule 5-0-19	(Required)	The declaration of objects shall contain no more than two levels of pointer indirection.

Rule 5-0-20	(Required)	Non-constant operands to a binary bitwise operator shall have the same underlying type.

Rule 5-0-21	(Required)	Bitwise operators shall only be applied to operands of unsigned underlying type.

#### Postfix expressions

Rule 5-2-1	(Required)	Each operand of a logical && or || shall be a postfix-expression.

Rule 5-2-2	(Required)	A pointer to a virtual base class shall only be cast to a pointer to a derived class by means of dynamic_cast.

Rule 5-2-3	(Advisory)	Casts from a base class to a derived class should not be performed on polymorphic types.

Rule 5-2-4	(Required)	C-style casts (other than void casts) and functional notation casts (other than explicit constructor calls) shall not be used.

Rule 5-2-5	(Required)	A cast shall not remove any const or volatile qualification from the type of a pointer or reference.

Rule 5-2-6	(Required)	A cast shall not convert a pointer to a function to any other pointer type, including a pointer to function type.

Rule 5-2-7	(Required)	An object with pointer type shall not be converted to an unrelated pointer type, either directly or indirectly.

Rule 5-2-8	(Required)	An object with integer type or pointer to void type shall not be converted to an object with pointer type.

Rule 5-2-9	(Advisory)	A cast should not convert a pointer type to an integral type.

Rule 5-2-10	(Advisory)	The increment (++) and decrement (--) operators should not be mixed with other operators in an expression.

Rule 5-2-11	(Required)	The comma operator, && operator and the || operator shall not be overloaded.

Rule 5-2-12	(Required)	An identifier with array type passed as a function argument shall not decay to a pointer.

#### Unary expressions

Rule 5-3-1	(Required)	Each operand of the ! operator, the logical && or the logical || operators shall have type bool.

Rule 5-3-2	(Required)	The unary minus operator shall not be applied to an expression whose underlying type is unsigned.

Rule 5-3-3	(Required)	The unary & operator shall not be overloaded.

Rule 5-3-4	(Required)	Evaluation of the operand to the sizeof operator shall not contain side effects.

#### Shift operators

Rule 5-8-1	(Required)	The right hand operand of a shift operator shall lie between zero and one less than the width in bits of the underlying type of the left hand operand.

#### Logical AND operator

Rule 5-14-1	(Required)	The right hand operand of a logical && or || operator shall not contain side effects.

#### Assignment operators

Rule 5-17-1	(Required)	The semantic equivalence between a binary operator and its assignment operator form shall be preserved.

#### Comma operator

Rule 5-18-1	(Required)	The comma operator shall not be used.

#### Constant expressions

Rule 5-19-1	(Advisory)	Evaluation of constant unsigned integer expressions should not lead to wrap-around.

----

### Statements

#### Expression statement

Rule 6-2-1	(Required)	Assignment operators shall not be used in sub-expressions.

Rule 6-2-2	(Required)	Floating-point expressions shall not be directly or indirectly tested for equality or inequality.

Rule 6-2-3	(Required)	Before preprocessing, a null statement shall only occur on a line by itself; it may be followed by a comment, provided that the first character following the null statement is a white-space character.

#### Compound statement

Rule 6-3-1	(Required)	The statement forming the body of a switch, while, do ... while or for statement shall be a compound statement.

#### Selection statements

Rule 6-4-1	(Required)	An if ( condition ) construct shall be followed by a compound statement. The else keyword shall be followed by either a compound statement, or another if statement.

Rule 6-4-2	(Required)	All if - else if constructs shall be terminated with an else clause.

Rule 6-4-3	(Required)	A switch statement shall be a well-formed switch statement.

Rule 6-4-4	(Required)	A switch-label shall only be used when the most closely-enclosing compound statement is the body of a switch statement.

Rule 6-4-5	(Required)	An unconditional throw or break statement shall terminate every non-empty switch-clause.

Rule 6-4-6	(Required)	The final clause of a switch statement shall be the default-clause.

Rule 6-4-7	(Required)	The condition of a switch statement shall not have bool type.

Rule 6-4-8	(Required)	Every switch statement shall have at least one case-clause.

----

### Iteration statements

#### The for statement

Rule 6-5-1	(Required)	A for loop shall contain a single loop-counter which shall not have floating type.

Rule 6-5-2	(Required)	If loop-counter is not modified by -- or ++, then, within condition, the loop-counter shall only be used as an operand to <=, <, > or >=.

Rule 6-5-3	(Required)	The loop-counter shall not be modified within condition or statement.

Rule 6-5-4	(Required)	The loop-counter shall be modified by one of: --, ++, -=n, or +=n; where n remains constant for the duration of the loop.

Rule 6-5-5	(Required)	A loop-control-variable other than the loop-counter shall not be modified within condition or expression.

Rule 6-5-6	(Required)	A loop-control-variable other than the loop-counter which is modified in statement shall have type bool.

#### Jump statements

Rule 6-6-1	(Required)	Any label referenced by a goto statement shall be declared in the same block, or in a block enclosing the goto statement.

Rule 6-6-2	(Required)	The goto statement shall jump to a label declared later in the same function body.

Rule 6-6-3	(Required)	The continue statement shall only be used within a well-formed for loop.

Rule 6-6-4	(Required)	For any iteration statement there shall be no more than one break or goto statement used for loop termination.

Rule 6-6-5	(Required)	A function shall have a single point of exit at the end of the function.

----

### Declarations

#### Specifiers

Rule 7-1-1	(Required)	A variable which is not modified shall be const qualified.

Rule 7-1-2	(Required)	A pointer or reference parameter in a function shall be declared as pointer to const or reference to const if the corresponding object is not modified.

Rule 7-1-3	(Required)	An expression with enum underlying type shall only have values corresponding to the enumerators of the enumeration.

#### Enumeration declarations

Rule 7-2-1	(Required)	An expression with enum underlying type shall only have values corresponding to the enumerators of the enumeration.

#### Namespaces

Rule 7-3-1	(Required)	The global namespace shall only contain main, namespace declarations and extern "C" declarations.

Rule 7-3-2	(Required)	The identifier main shall not be used for a function other than the global function main.

Rule 7-3-3	(Required)	There shall be no unnamed namespaces in header files.

Rule 7-3-4	(Required)	Using-directives shall not be used.

Rule 7-3-5	(Required)	Multiple declarations for an identifier in the same namespace shall not straddle a using-declaration for that identifier.

Rule 7-3-6	(Required)	Using-directives and using-declarations (excluding class scope or function scope using-declarations) shall not be used in header files.

#### The asm declaration

Rule 7-4-1	(Document)	All usage of assembler shall be documented.

Rule 7-4-2	(Required)	Assembler instructions shall only be introduced using the asm declaration.

Rule 7-4-3	(Required)	Assembly language shall be encapsulated and isolated.

#### Linkage specifications

Rule 7-5-1	(Required)	A function shall not return a reference or a pointer to an automatic variable (including parameters), defined within the function.

Rule 7-5-2	(Required)	The address of an object with automatic storage shall not be assigned to another object that may persist after the first object has ceased to exist.

Rule 7-5-3	(Required)	A function shall not return a reference or a pointer to a parameter that is passed by reference or const reference.

Rule 7-5-4	(Advisory)	Functions should not call themselves, either directly or indirectly.

----

### Declarators

#### General

Rule 8-0-1	(Required)	An init-declarator-list or a member-declarator-list shall consist of a single init-declarator or member-declarator respectively.

#### Meaning of declarators

Rule 8-3-1	(Required)	Parameters in an overriding virtual function shall either use the same default arguments as the function they override, or else shall not specify any default arguments.

#### Function definitions

Rule 8-4-1	(Required)	Functions shall not be defined using the ellipsis notation.

Rule 8-4-2	(Required)	The identifiers used for the parameters in a re-declaration of a function shall be identical to those in the declaration.

Rule 8-4-3	(Required)	All exit paths from a function with non-void return type shall have an explicit return statement with an expression.

Rule 8-4-4	(Required)	A function identifier shall either be used to call the function or it shall be preceded by &.

#### Initializers

Rule 8-5-1	(Required)	All variables shall have a defined value before they are used.

Rule 8-5-2	(Required)	Braces shall be used to indicate and match the structure in the non-zero initialization of arrays and structures.

Rule 8-5-3	(Required)	In an enumerator list, the = construct shall not be used to explicitly initialize members other than the first, unless all items are explicitly initialized.

----

### Classes

#### Member functions

Rule 9-3-1	(Required)	Const member functions shall not return non-const pointers or references to class-data.

Rule 9-3-2	(Required)	Member functions shall not return non-const handles to class-data.

Rule 9-3-3	(Required)	If a member function can be made static then it shall be made static, otherwise if it can be made const then it shall be made const.

#### Unions

Rule 9-5-1	(Required)	Unions shall not be used.

#### Bit-fields

Rule 9-6-1	(Required)	When the absolute positioning of bits representing a bit-field is required, then the behaviour and packing of bit-fields shall be documented.

Rule 9-6-2	(Required)	Bit-fields shall be either bool type or an explicitly unsigned or signed integral type.

Rule 9-6-3	(Required)	Bit-fields shall not have enum type.

Rule 9-6-4	(Required)	Named bit-fields with signed integer type shall have a length of more than one bit.

----

### Derived classes

#### Multiple base classes

Rule 10-1-1	(Advisory)	Classes should not be derived from virtual bases.

Rule 10-1-2	(Required)	A base class shall only be declared virtual if it is used in a diamond hierarchy.

Rule 10-1-3	(Required)	An accessible base class shall not be both virtual and non-virtual in the same hierarchy.

#### Member name lookup

Rule 10-2-1	(Advisory)	All accessible entity names within a multiple inheritance hierarchy should be unique.

#### Virtual functions

Rule 10-3-1	(Required)	There shall be no more than one definition of each virtual function on each path through the inheritance hierarchy.

Rule 10-3-2	(Required)	Each overriding virtual function shall be declared with the virtual keyword.

Rule 10-3-3	(Required)	A virtual function shall only be overridden by a pure virtual function if it is itself declared as pure virtual.

----

### Member access control

#### General

Rule 11-0-1	(Required)	Member data in non-POD class types shall be private.

----

### Special member functions

#### Constructors

Rule 12-1-1	(Required)	An object-s dynamic type shall not be used from the body of its constructor or destructor.

Rule 12-1-2	(Advisory)	All constructors of a class should explicitly call a constructor for all of its immediate base classes and all virtual base classes.

Rule 12-1-3	(Required)	All constructors that are callable with a single argument of fundamental type shall be declared explicit.

#### Copying class objects

Rule 12-8-1	(Required)	A copy constructor shall only initialize its base classes and the non-static members of the class of which it is a member.

Rule 12-8-2	(Required)	The copy assignment operator shall be declared protected or private in an abstract class.

----

### Templates

#### Template declarations

Rule 14-5-1	(Required)	A non-member generic function shall only be declared in a namespace that is not an associated namespace.

Rule 14-5-2	(Required)	A copy constructor shall be declared when there is a template constructor with a single parameter that is a generic parameter.

Rule 14-5-3	(Required)	A copy assignment operator shall be declared when there is a template assignment operator with a parameter that is a generic parameter.

#### Name resolution

Rule 14-6-1	(Required)	In a class template with a dependent base, any name that may be found in that dependent base shall be referred to using a qualified-id or this->

Rule 14-6-2	(Required)	The function chosen by overload resolution shall resolve to a function declared previously in the translation unit.

#### Template instantiation and specialization

Rule 14-7-1	(Required)	All class templates, function templates, class template member functions and class template static members shall be instantiated at least once.

Rule 14-7-2	(Required)	For any given template specialization, an explicit instantiation of the template with the template- arguments used in the specialization shall not render the program ill-formed.

Rule 14-7-3	(Required)	All partial and explicit specializations for a template shall be declared in the same file as the declaration of their primary template.

#### Function template specialization

Rule 14-8-1	(Required)	Overloaded function templates shall not be explicitly specialized.

Rule 14-8-2	(Advisory)	The viable function set for a function call should either contain no function specializations, or only contain function specializations.

----

### Exception handling

#### General

Rule 15-0-1	(Document)	Exceptions shall only be used for error handling.

Rule 15-0-2	(Advisory)	An exception object should not have pointer type.

Rule 15-0-3	(Required)	Control shall not be transferred into a try or catch block using a goto or a switch statement.

#### Throwing an exception

Rule 15-1-1	(Required)	The assignment-expression of a throw statement shall not itself cause an exception to be thrown.

Rule 15-1-2	(Required)	NULL shall not be thrown explicitly.

Rule 15-1-3	(Required)	An empty throw (throw;) shall only be used in the compound-statement of a catch handler.

#### Handling an exception

Rule 15-3-1	(Required)	Exceptions shall be raised only after start-up and before termination of the program.

Rule 15-3-2	(Advisory)	There should be at least one exception handler to catch all otherwise unhandled exceptions

Rule 15-3-3	(Required)	Handlers of a function-try-block implementation of a class constructor or destructor shall not reference non- static members from this class or its bases.

Rule 15-3-4	(Required)	Each exception explicitly thrown in the code shall have a handler of a compatible type in all call paths that could lead to that point.

Rule 15-3-5	(Required)	A class type exception shall always be caught by reference.

Rule 15-3-6	(Required)	Where multiple handlers are provided in a single try-catch statement or function-try-block for a derived class and some or all of its bases, the handlers shall be ordered most-derived to base class.

Rule 15-3-7	(Required)	Where multiple handlers are provided in a single try-catch statement or function-try-block, any ellipsis (catch-all) handler shall occur last.

#### Exception specifications

Rule 15-4-1	(Required)	If a function is declared with an exception-specification, then all declarations of the same function (in other translation units) shall be declared with the same set of type-ids.

#### Special functions

Rule 15-5-1	(Required)	A class destructor shall not exit with an exception.

Rule 15-5-2	(Required)	Where a function-s declaration includes an exception- specification, the function shall only be capable of throwing exceptions of the indicated type(s).

Rule 15-5-3	(Required)	The terminate() function shall not be called implicitly.

----

### Preprocessing directives

#### General

Rule 16-0-1	(Required)	#include directives in a file shall only be preceded by other preprocessor directives or comments.

Rule 16-0-2	(Required)	Macros shall only be #define-d or #undef-d in the global namespace.

Rule 16-0-3	(Required)	#undef shall not be used.

Rule 16-0-4	(Required)	Function-like macros shall not be defined.

Rule 16-0-5	(Required)	Arguments to a function-like macro shall not contain tokens that look like preprocessing directives.

Rule 16-0-6	(Required)	In the definition of a function-like macro, each instance of a parameter shall be enclosed in parentheses, unless it is used as the operand of # or ##.

Rule 16-0-7	(Required)	Undefined macro identifiers shall not be used in #if or #elif preprocessor directives, except as operands to the defined operator.

Rule 16-0-8	(Required)	If the # token appears as the first token on a line, then it shall be immediately followed by a preprocessing token.

#### Conditional inclusion

Rule 16-1-1	(Required)	The defined preprocessor operator shall only be used in one of the two standard forms.

Rule 16-1-2	(Required)	All #else, #elif and #endif preprocessor directives shall reside in the same file as the #if or #ifdef directive to which they are related.

#### Source file inclusion

Rule 16-2-1	(Required)	The pre-processor shall only be used for file inclusion and include guards.

Rule 16-2-2	(Required)	C++ macros shall only be used for include guards, type qualifiers, or storage class specifiers.

Rule 16-2-3	(Required)	Include guards shall be provided.

Rule 16-2-4	(Required)	The ', ", /* or // characters shall not occur in a header file name.

Rule 16-2-5	(Advisory)	The \ character should not occur in a header file name.

Rule 16-2-6	(Required)	The #include directive shall be followed by either a <filename> or "filename" sequence.

#### Macro replacement

Rule 16-3-1	(Required)	There shall be at most one occurrence of the # or ## operators in a single macro definition.

Rule 16-3-2	(Advisory)	The # and ## operators should not be used.

#### Pragma directive

Rule 16-6-1	(Document)	All uses of the #pragma directive shall be documented.

----

### Library introduction

#### General

Rule 17-0-1	(Required)	Reserved identifiers, macros and functions in the standard library shall not be defined, redefined or undefined.

Rule 17-0-2	(Required)	The names of standard library macros and objects shall not be reused.

Rule 17-0-3	(Required)	The names of standard library functions shall not be overridden.

Rule 17-0-4	(Document)	All library code shall conform to MISRA C++.

Rule 17-0-5	(Document)	The setjmp macro and the longjmp function shall not be used.

----

### Language support library

#### General

Rule 18-0-1	(Required)	The C library shall not be used.

Rule 18-0-2	(Required)	The library functions atof, atoi and atol from library <cstdlib> shall not be used.

Rule 18-0-3	(Required)	The library functions abort, exit, getenv and system from library <cstdlib> shall not be used.

Rule 18-0-4	(Required)	The time handling functions of library <ctime> shall not be used.

Rule 18-0-5	(Required)	The unbounded functions of library <cstring> shall not be used.

#### Implementation properties

Rule 18-2-1	(Required)	The macro offsetof shall not be used.

#### Dynamic memory management

Rule 18-4-1	(Required)	Dynamic heap memory allocation shall not be used.

#### Other runtime support

Rule 18-7-1	(Required)	The signal handling facilities of <csignal> shall not be used.

----

### Diagnostics library

#### Error numbers

Rule 19-3-1	(Required)	The error indicator errno shall not be used.

----

### Input/output library

#### General

Rule 27-0-1	(Required)	The stream input/output library <cstdio> shall not be used.

----


