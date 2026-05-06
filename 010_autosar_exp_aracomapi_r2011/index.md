# AUTOSAR_EXP_ARAComAPI


# AUTOSAR_EXP_ARAComAPI

---

[SWS_CORE_90003]{草案} ⌈以 `ARA` 开头的 `C/C++` 符号保留给 `AUTOSAR` 使用。⌋

自适应平台通常避免使用 `C/C++` 预处理器宏。但如果未来某个时间点引入了宏，所有此类宏都将以 `ARA` 为前缀。因此，平台供应商不应定义任何以此为前缀的符号（包括宏和 `C/C++` 符号），以免与标准未来新增内容发生冲突。

## 7.2 功能规范

本节介绍本功能集群引入的核心概念，重点阐述错误处理机制。

### 7.2.1 错误处理

#### 7.2.1.1 操作失败的类型

在自适应平台API的实现执行过程中，可能会检测到多种异常情况，需要进行处理和/或上报。根据其性质，自适应平台将操作失败分为以下类型：

**错误(Error)**：指假定无bug的API函数无法完成其指定功能的情况，通常由无效和/或意外的输入数据导致（即数据可能本身有效，但在非预期的上下文中被接收）。错误被认为是可恢复的。

**违规(Violation)**：指应用框架内部状态的前置条件或后置条件检查失败的结果，相当于自适应平台中的断言失败。违规被认为是不可恢复的。

**损坏(Corruption)**：指系统资源损坏导致的结果，例如栈溢出、堆溢出或硬件内存故障（甚至包括检测到的位翻转）。损坏被认为是不可恢复的。

**默认分配失败**：指框架的默认内存分配机制无法满足分配请求的情况。

违规或损坏通常发生在框架开发阶段（新功能集成时），一般不会被应用开发者遇到。除非系统环境存在严重问题（如硬件故障导致损坏）、违反了资源需求的基本假设（导致违规），或者用户在供应商不支持的配置下运行框架（导致违规）。

#### 7.2.1.2 C和C++中的传统错误处理

`C` 语言主要依赖错误码进行各类错误处理。虽然 `C` 也提供了 `setjmp`/`longjmp` 机制用于实现"非局部跳转"，但这种方式在错误处理中并不常用，主要原因是难以可靠地避免资源泄漏。

C语言中的错误码有以下几种形式：
- 返回值
- 输出参数
- 错误单例（如 `errno`）

通常，`C` 语言中的错误码是普通的 `int` 类型变量，属于非常底层的机制，不具备类型安全性。

`C++` 从 `C` 语言继承了这些错误处理方式（很大程度上是因为 `C` 标准库是 `C++` 标准库的一部分），但同时引入了异常作为另一种错误传播方式。使用异常进行错误传播有诸多优势，因此 `C++` 标准库通常依赖异常来传播错误。

尽管异常有诸多优点，但错误码在 `C++` 中仍然被广泛使用，甚至在标准库内部也是如此。部分原因是为了保持与 `C` 语言的二进制兼容性，但许多新库仍然更倾向于使用错误码而非异常，原因包括：
- 使用异常时，难以推断程序的控制流
- 异常的运行时开销远高于错误码（无论是整体开销，还是仅在抛出异常时的开销）

第一个原因同时影响人工代码审查和静态代码分析工具。因为异常本质上是一种隐藏的控制流，一个看似只有一条 `return` 语句的 `C++` 函数，实际上可能因异常而产生多个额外的返回路径。这不仅增加了人工审查的难度，也给静态代码分析工具带来了挑战。

第二个原因在安全关键软件的开发中尤为关键。`C++` 异常的规范给希望获得安全关键软件开发认证的编译器厂商带来了重大问题。事实上，通过 `ASIL` 认证的 `C++` 编译器通常完全不支持异常。异常的一个特殊问题是，`C++` 标准规定的异常处理机制隐含了动态内存分配的使用，而动态内存分配通常具有不可预测甚至无界的执行时间。这使得异常目前不适用于汽车行业中某些安全关键软件的开发。

#### 7.2.1.3 自适应平台中操作失败的处理

第7.2.1.1节（"操作失败的类型"）中定义的各类操作失败，需采用不同的方式处理。

[SWS_CORE_00002] 错误的处理 ⌈错误应作为 `ara::core::Result` 或 `ara::core::Future` 的实例从函数返回。⌋

[SWS_CORE_00003] 违规的处理 ⌈如果检测到违规，应将其发生记录为 `FATAL` 级别的日志消息（如果自适应平台相应功能集群启用了日志功能），然后通过以下两种方式之一终止操作：
- 抛出一个非 `ara::core::Exception` 子类的异常
- 通过调用 `ara::core::Abort` 显式异常终止进程

⌋

[SWS_CORE_00004] 损坏的处理 ⌈如果检测到损坏，应以实现定义的方式导致进程异常终止。⌋

注：根据实现检测损坏并通过清理资源做出响应的能力，终止可以是异常终止，也可以是正常的失败终止。

[SWS_CORE_00005] 默认分配失败的处理 ⌈"默认分配失败"应与违规同等处理。⌋

注：自定义分配器的错误不适用于此定义。

为了处理错误，自适应平台定义了一系列辅助数据类型，将在以下小节中介绍。

##### 7.2.1.3.1 ErrorCode

顾名思义，`ara::core::ErrorCode` 是一种错误码形式；但它是一个类类型，大致以 `std::error_code` 为模型，因此比典型 `C API` 中使用的简单错误码支持更复杂的错误处理。它始终包含一个底层错误码值和一个对错误域的引用。

错误码值是一个枚举类型，通常是强类型枚举。当存储到 `ara::core::ErrorCode` 中时，它会被类型擦除为整数类型，因此处理方式与 `C` 风格错误码类似。错误域引用定义了该错误码值适用的上下文，从而提供了一定程度的类型安全性。

`ara::core::ErrorCode` 还包含一个支持数据值，自适应平台的实现可以定义该值，以提供关于错误的供应商特定附加信息。

`ara::core::ErrorCode` 实例通常不直接创建，而是通过 `ara::core::Result::FromError` 函数的转发形式创建。

`ara::core::ErrorCode` 不受限于任何已知的错误域集合。其内部对枚举的类型擦除确保了它是一个简单（即非模板化）类型，可以包含来自任意域的任意错误。

然而，两个 `ara::core::ErrorCode` 实例的比较仅考虑错误码值和错误域引用；支持数据值成员不参与相等性检查。这是因为 `ara::core::ErrorCode` 实例通常是与一组已知错误进行比较：

```C++
ErrorCode ec = ...
if (ec == MyEnum::some_error)
    // ...
else if (ec == AnotherEnum::another_error)
    // ...
```

上述每次比较都会为比较运算符的右侧创建一个临时的 `ara::core::ErrorCode` 对象，然后将 `ec` 与该临时对象进行比较。这种自动创建的实例自然不包含任何有意义的支持数据值。

这种频繁创建临时 `ara::core::ErrorCode` 实例的操作预计非常快速，不会产生明显的运行时开销。这通常通过将 `ara::core::ErrorCode` 设计为字面量类型来保证。

[SWS_CORE_10300]{草案} 错误码类型属性 ⌈ `ara::core::ErrorCode` 类应是 `C++11` 标准 [5] 第 3.9 - 10 节 [`basic.types`] 定义的字面量类型。⌋

##### 7.2.1.3.2 ErrorDomain

`ara::core::ErrorDomain` 是功能集群甚至自适应应用中定义的具体错误域的抽象基类。该类大致以 `std::error_category` 为模型，但与之存在显著差异。

每个错误域都关联一个错误码枚举和一个基础异常类型。这两者通常与 `ara::core::ErrorDomain` 子类定义在同一个命名空间中。为了标准化访问这些关联类型，`ara::core::ErrorDomain` 子类中定义了具有标准化名称的类型别名。这使得错误域子类成为所有错误相关数据的根。

错误域的身份由唯一标识符定义。`AUTOSAR` 定义的错误域被赋予标准化标识符；用户定义的错误域也必须定义唯一标识符。

`ara::core::ErrorDomain` 类定义要求该唯一标识符为64位无符号整数类型(`std::uint64_t`)。其取值范围足够大，即使典型的UUID是128位，也可以应用类似UUID的生成模式（用于64位UID）。创建新错误域时（无论是AUTOSAR定义还是用户定义），都应生成一个对应的唯一标识符来表示该错误域。每个错误域的标识符必须唯一且标准化，因为调用方和被调用方（可能位于不同ECU）之间的错误信息交换基于此标识符。

基于这种错误域身份的定义，每个 `ara::core::ErrorDomain` 子类通常只应存在一个实例。虽然可以通过调用构造函数创建这些子类的新实例，但推荐的访问方式是调用其全局访问器函数。例如，错误域类 `ara::core::FutureErrorDomain` 通过调用 `ara::core::GetFutureErrorDomain` 来引用；在任何进程空间中，该调用始终返回该类的同一个全局实例的引用。

对于在ARXML中建模的错误域（作为 `ApApplicationErrorDomain`），C++语言绑定将为每个此类 `ApApplicationErrorDomain` 创建一个 `C++` 类。该 `C++` 类将是 `ara::core::ErrorDomain` 的子类，其名称遵循标准命名方案。

`ara::core` 预定义了两个错误域：`CoreErrorDomain`（包含 `ara::core` 功能集群中非 `Future/Promise` 设施返回的错误集合）和 `FutureErrorDomain`（包含与 `std::future_errc` 定义的错误等效的错误）。

应用程序员通常不直接与 `ara::core::ErrorDomain` 类或其子类交互；大多数访问通过 `ara::core::ErrorCode` 进行。

由于 `ara::core::ErrorDomain` 子类预计会在常量（即编译时）表达式中被隐式引用（通常涉及 `ara::core::ErrorCode`），因此它们也应是字面量类型。

[SWS_CORE_10400]{草案} 错误域类型属性 ⌈`ara::core::ErrorDomain` 类及其所有子类应是 `C++11` 标准[5]第3.9-10节[basic.types]定义的字面量类型。⌋

##### 7.2.1.3.3 Result

`ara::core::Result` 类型遵循 `C++` 提案p0786[6]中的"值或错误"概念。它要么包含一个值（`ValueType` 类型），要么包含一个错误（`ErrorType` 类型）。`ValueType` 和 `ErrorType` 都是 `ara::core::Result` 的模板参数，由于其模板化特性，值和错误可以是任意类型。然而，`ErrorType` 默认设置为 `ara::core::ErrorCode`，并且预计在整个自适应平台中保持此默认设置。

ara::core::Result充当一种"包装类型"，将使用ara::core::ErrorCode的无异常API方法与C++异常连接起来。由于ara::core::ErrorCode与特定域的异常类型之间存在直接映射，ara::core::Result允许通过调用ara::core::Result::ValueOrThrow将其包含的ara::core::ErrorCode转换为相应的异常类型。

##### 7.2.1.3.4 Future and Promise

`ara::core::Future` 及其配套类 `ara::core::Promise` 大致以 `std::future` 和 `std::promise` 为模型，但已进行了适配以与 `ara::core::Result` 互操作。与第7.2.1.3.3节描述的ara::core::Result类似，ara::core::Future类要么包含一个值，要么包含一个错误（但Future必须首先处于"就绪"状态）。ara::core::Promise类在两个方面进行了适配：移除了Promise::set_exception，并引入了Promise::SetError作为替代。对于ara::core::Future，新增了成员函数Future::GetResult，其功能类似于Future::get，但永远不会抛出异常，而是返回一个ara::core::Result。

因此，作为返回类型的ara::core::Future允许与ara::core::Result相同的双重错误处理方法：既可以使用基于异常的方式（通过Future::get），也可以使用无异常的方式（通过Future::GetResult）。

ara::core::Result用于从同步函数调用返回值或错误，而ara::core::Future用于从异步函数调用返回值或错误。

#### 7.2.1.4 错误码与异常的二元性

通过使用上述类，自适应平台的所有API都可以与基于异常或无异常的错误处理工作流一起使用。然而，任何API函数都不会通过直接抛出异常来处理错误；它始终以ara::core::Result或ara::core::Future返回值的形式返回错误码。然后，调用方可以将错误转换为异常，通常通过ara::core::Result::ValueOrThrow成员函数实现。

当使用完全不支持异常的C++编译器（或已通过g++的-fnoexceptions等选项禁用异常的编译器）时，所有API函数仍表现出相同的行为。不同之处在于，此时ara::core::Result::ValueOrThrow未定义——该成员函数仅在编译器支持异常时才定义。

#### 7.2.1.5 异常层次结构

自适应平台为标准中定义的所有异常定义了一个基础异常类型ara::core::Exception。该异常以ara::core::ErrorCode对象作为强制构造函数参数，类似于std::system_error以std::error_code参数进行构造的方式。

在该基础异常类型之下，还有一层额外的基础异常类型，每个错误域对应一个。

对于在ARXML中建模的错误域，C++语言绑定除了生成错误域子类（见第7.2.1.3.2节）外，还将生成一个异常类。该异常类也遵循标准命名方案：ApApplicationErrorDomain的短名称加上"Exception"后缀（这使其与错误域子类本身区分开来）。它与相应的错误域子类位于同一个命名空间中。

#### 7.2.1.6 创建新错误域

任何与自适应平台所有现有模块具有显著逻辑分离的新软件模块，都应定义一个或多个自己的错误域。

一个错误域包含：
- 一个错误条件枚举
- 一个异常基类
- 一个 `ara::core::ErrorDomain` 子类
- 一个全局错误域子类访问器函数
- 一个全局 `MakeErrorCode` 函数重载

所有这些都不应定义在 `ara::core` 命名空间中，而应定义在"目标"命名空间中。

[SWS_CORE_10999]{草案} 自定义错误域作用域 ⌈错误域子类及其对应的枚举、异常基类、全局访问器函数和 `MakeErrorCode` 重载，应定义在为其指定的软件模块所在的同一个命名空间中。⌋

注：这有助于确保 `C++` 的 `ADL`（参数依赖查找）机制按照本标准其他部分的预期工作。

以本节指定方式定义的错误域适用于 `ApApplicationErrorDomain` 模型元素。

在本节中，字符序列 `<SN>` 是 `ApApplicationErrorDomain` 短名称的占位符。

###### 7.2.1.6.1 错误条件枚举

错误条件枚举描述了新软件模块的所有已知错误条件。它应具有足够细的粒度，以允许用户区分可能需要以不同方式处理的错误条件。

[SWS_CORE_10900]{草案} 错误条件枚举类型 ⌈每个错误域应定义一个基类型为 `ara::core::ErrorDomain::CodeType` 的强类型枚举类，包含该错误域的所有错误条件。⌋

[SWS_CORE_10901]{草案} 错误条件枚举命名 ⌈错误域的错误条件枚举应遵循命名方案 `<SN>Errc`，其中 `<SN>` 是 `ApApplicationErrorDomain` 的短名称。⌋

[SWS_CORE_10902]{草案} 错误条件枚举内容 ⌈错误域的错误条件枚举不应包含任何表示成功的值。⌋

[SWS_CORE_10903]{草案} 错误条件枚举编号 ⌈错误域的错误条件枚举应保留数字 `0` 未分配。⌋

###### 7.2.1.6.2 异常基类

作为错误条件枚举的补充，还需要为该错误域定义一个异常基类。该异常基类用于将 `ara::core::ErrorCode` 对象转换为异常。

软件模块可以定义其他异常类型，但所有这些类型都应派生自此基类型。

[SWS_CORE_10910]{草案} 错误域异常基类型 ⌈每个错误域应定义一个异常基类型，该类型是 `ara::core::Exception` 的子类。⌋

[SWS_CORE_10911]{草案} 错误域异常基类型命名 ⌈[SWS_CORE_10910]指定的所有错误域异常基类型应遵循命名方案 `<SN>Exception`，其中 `<SN>` 是 `ApApplicationErrorDomain` 的短名称。⌋

[SWS_CORE_10912]{草案} 错误域异常类型层次结构 ⌈软件模块定义的所有其他异常类型都应将[SWS_CORE_10910]指定的异常基类型作为基类。⌋

###### 7.2.1.6.3 错误域子类

然后，创建一个派生自ara::core::ErrorDomain的新类，并覆盖所有纯虚成员函数。除此之外，还需要在其作用域中定义一个名为Errc的类型别名（指向错误条件枚举），以及另一个名为Exception的类型别名（指向该新错误域的异常基类）。

[SWS_CORE_10930]{草案} 错误域子类类型 ⌈每个错误域应定义一个公共派生自ara::core::ErrorDomain的类类型。⌋

[SWS_CORE_10931]{草案} 错误域子类命名 ⌈ara::core::ErrorDomain的所有子类应遵循命名方案<SN>ErrorDomain，其中<SN>是ApApplicationErrorDomain的短名称。⌋

[SWS_CORE_10932]{草案} 错误域子类不可扩展性 ⌈ara::core::ErrorDomain的所有子类应声明为final。⌋

[SWS_CORE_10933]{草案} 错误域子类Errc符号 ⌈ara::core::ErrorDomain的所有子类应在其作用域中包含一个名为Errc的类型别名，指向[SWS_CORE_10900]定义的错误条件枚举。⌋

[SWS_CORE_10934]{草案} 错误域子类Exception符号 ⌈ara::core::ErrorDomain的所有子类应在其作用域中包含一个名为Exception的类型别名，指向[SWS_CORE_10910]定义的异常基类型。⌋

所有错误域子类都可以在常量表达式中使用，参见[SWS_CORE_10400]。特别是，这意味着错误域子类可以定义为constexpr全局变量。

为了进一步简化错误域的使用，要求错误域子类的所有成员函数都是noexcept的，但ErrorDomain::ThrowAsException是明显的例外。

[SWS_CORE_10950]{草案} 错误域子类成员函数属性 ⌈除了ara::core::ErrorDomain::ThrowAsException之外，所有错误域子类的所有公共成员函数都应是noexcept的。⌋

虚成员函数ErrorDomain::Name()返回ApApplicationErrorDomain的短名称，主要用于日志记录目的。

[SWS_CORE_10951]{草案} 错误域子类短名称获取 ⌈错误域的Name()成员函数的返回值应等于ApApplicationErrorDomain的短名称。⌋

每个错误域都有一个用于确定错误域相等性的标识符。自适应平台预定义的错误域具有标准化标识符。应用特定的错误域应确保其标识符在系统范围内唯一。

[SWS_CORE_10952]{草案} 错误域子类唯一标识符获取 ⌈错误域的Id()成员函数的返回值应是遵循[SWS_CORE_00010]定义规则的唯一标识符。⌋

错误域可以将ErrorCode转换为异常。

[SWS_CORE_10953]{草案} 将错误码作为异常抛出 ⌈错误域子类实现的ErrorDomain::ThrowAsException函数抛出的异常类型，应派生自该错误域子类中[SWS_CORE_10934]定义的Exception类型别名。⌋

###### 7.2.1.6.4 全局错误域子类访问器函数

需要为新的错误域类定义一个全局访问器函数。对于错误域类MyErrorDomain，访问器函数命名为GetMyErrorDomain。该访问器函数返回该类的单个全局实例的引用。该访问器函数应完全支持constexpr；这反过来意味着错误域子类也应支持constexpr构造（参见[SWS_CORE_10400]）。

[SWS_CORE_10980]{草案} 错误域子类访问器函数 ⌈对于ara::core::ErrorDomain的所有子类，都应存在一个全局constexpr函数，返回其单例实例的const引用。⌋

[SWS_CORE_10981]{草案} 错误域子类访问器函数命名 ⌈所有ara::core::ErrorDomain子类访问器函数应遵循命名方案Get<SN>ErrorDomain，其中<SN>是ApApplicationErrorDomain的短名称。⌋

[SWS_CORE_10982]{草案} 错误域子类访问器函数 ⌈所有ara::core::ErrorDomain子类访问器函数的返回类型应为const ErrorDomain&。⌋

###### 7.2.1.6.5 全局 `MakeErrorCode` 重载

最后，需要定义一个全局工厂函数MakeErrorCode，该函数由ara::core::ErrorCode类的便捷构造函数隐式使用。该工厂函数将使用错误域子类的全局访问器函数，并调用ara::core::ErrorCode类的类型擦除构造函数。

[SWS_CORE_10990]{草案} 新错误域的MakeErrorCode重载 ⌈对于ara::core::ErrorDomain的所有子类，都应存在一个全局函数MakeErrorCode的constexpr重载，用于为ara::core::ErrorDomain子类错误条件范围内的给定错误条件值创建ara::core::ErrorCode实例。⌋

[SWS_CORE_10991]{草案} MakeErrorCode重载签名 ⌈全局函数MakeErrorCode的所有重载应具有以下签名：

```C++
constexpr ErrorCode MakeErrorCode(<SN>Errc code, ErrorDomain::SupportDataType data) noexcept;
```

其中 `<SN>` 是ApApplicationErrorDomain的短名称。⌋

###### 7.2.1.6.6 C++伪代码示例

以下 C++ 伪代码说明了这些定义如何组合在一起：

```C++
namespace my
{

enum class <SN>Errc : ara::core::ErrorDomain::CodeType
{
    // ...
};

class <SN>Exception : public ara::core::Exception
{
public:
    <SN>Exception(ara::core::ErrorCode err) noexcept;
};

class <SN>ErrorDomain final : public ara::core::ErrorDomain
{
public:
    using Errc = <SN>Errc;
    using Exception = <SN>Exception;

    constexpr <SN>ErrorDomain() noexcept;

    const char* Name() const noexcept override;
    const char* Message(ara::core::ErrorDomain::CodeType errorCode) const noexcept override;
    void ThrowAsException(const ara::core::ErrorCode& errorCode) const noexcept(false) override;
};

constexpr const ara::core::ErrorDomain& Get<SN>ErrorDomain() noexcept;

constexpr ara::core::ErrorCode MakeErrorCode(<SN>Errc code, ara::core::ErrorDomain::SupportDataType data) noexcept;
} // namespace my
```

#### 7.2.1.7 AUTOSAR错误域

唯一错误域标识符的完整范围被划分为AUTOSAR指定ID范围、供应商指定ID范围和用户指定ID范围。

用户指定ID的最高位设置为0，可以使用剩余的63位来保证唯一性。最高位设置为1的ID保留给AUTOSAR和栈供应商使用。

[SWS_CORE_00010]{草案} 错误域标识符 ⌈所有错误域都应具有一个系统范围内唯一的标识符，该标识符表示为64位无符号整数值。⌋

[SWS_CORE_00011]{草案} AUTOSAR错误域范围 ⌈第63位设置为1且第62位设置为0的错误域标识符保留给AUTOSAR定义的错误域。⌋

[SWS_CORE_00016]{草案} 供应商定义错误域范围 ⌈高32位（即第63位至第32位）等于0xc000'0000的错误域标识符保留给供应商特定的错误域。第31位至第16位保存供应商的数字标识符，第15位至第0位可供每个供应商用于错误域标识符。⌋

[SWS_CORE_00013] Future错误域 ⌈应为所有源自ara::core::Future和ara::core::Promise类交互的错误定义一个错误域ara::core::FutureErrorDomain。其短名称应为Future，标识符为0x8000'0000'0000'0013。⌋

[SWS_CORE_00014] Core错误域 ⌈应为所有源自ara::core非Future/Promise设施的错误定义一个错误域ara::core::CoreErrorDomain。其短名称应为Core，标识符为0x8000'0000'0000'0014。⌋

### 7.2.2 异步信号安全

异步信号安全函数是指可以在POSIX信号处理程序内部安全调用的函数。

POSIX标准[7]定义了一组保证异步信号安全的函数；所有不在此列表中的函数都应被假定为不适合在信号处理程序中调用。这包括所有ARA API，因为未指定（且通常无法确定）这些API内部调用了哪些其他函数（无论是来自POSIX、其他标准还是实现）。

除非另有说明，否则在信号处理程序中使用任何ARA API都将导致应用程序出现未定义行为。

### 7.2.3 显式操作终止

如果API函数的实现检测到违规，[SWS_CORE_00003]要求立即终止该操作。它允许两种方式：要么抛出特定类型的异常（如果实现支持C++异常），要么调用ara::core::Abort。

调用ara::core::Abort将导致显式操作终止，这通常会导致文献[8]定义的意外终止。本节定义了该机制的行为。

与std::abort类似，调用ara::core::Abort旨在异常且立即终止当前进程，不执行栈展开，也不调用静态对象的析构函数。

[SWS_CORE_12402]{草案} Abort的"不返回"属性 ⌈ara::core::Abort函数不应返回给其调用者。⌋

[SWS_CORE_12403]{草案} 显式操作终止的日志记录 ⌈除非为此应用程序停用了日志功能，否则调用ara::core::Abort应导致通过ara::log输出一条FATAL级别的日志消息，该消息应包含传递给函数的字符串参数。⌋

[SWS_CORE_12407]{草案} 显式操作终止的线程安全性 ⌈当一个ara::core::Abort调用正在进行时，其他对该函数的调用应阻塞调用线程。⌋

ara::core::Abort提供了一种向系统添加"钩子"的方法，通过调用ara::core::SetAbortHandler实现，类似于std::atexit允许为std::exit机制安装回调的方式。然而，与std::atexit不同，ara::core::SetAbortHandler只能设置一个处理程序。

[SWS_CORE_12404]{草案} 终止处理程序的调用 ⌈调用ara::core::Abort时，如果已设置终止处理程序，则应在按照[SWS_CORE_12403]输出日志消息之后调用该处理程序。⌋

#### 7.2.3.1

该处理程序可以通过ara::core::SetAbortHandler安装。当调用ara::core::Abort时，它会被依次调用，并且可以执行任意操作，然后在其最终语句中有以下四种主要选择：
- 终止进程
- 从函数调用返回
- 进入无限循环以延迟函数返回
- 执行非局部跳转操作，如std::longjmp

强烈不建议使用非局部跳转操作（包括std::longjmp），MISRA、AUTOSAR C++14编码指南以及大多数其他编码指南也明确禁止这种做法。

同样，不建议通过进入无限循环来延迟函数返回；虽然这仍然可以达到终止导致违规的操作的预期结果，但代价是使调用线程成为"僵尸线程"，并可能使已经遇到违规的软件不稳定。

强烈建议终止进程的终止处理程序通过调用std::abort来实现。这将确保执行管理将意外终止正确识别为异常终止。

如果终止处理程序返回，或者根本没有定义终止处理程序，则ara::core::Abort的最终操作是调用std::abort。

[SWS_CORE_12405]{草案} 无终止处理程序时的最终操作 ⌈如果没有通过ara::core::SetAbortHandler安装自定义ara::core::AbortHandler，则ara::core::Abort的实现应调用std::abort()。⌋

[SWS_CORE_12406]{草案} 终止处理程序返回时的最终操作 ⌈如果已通过ara::core::SetAbortHandler安装了自定义ara::core::AbortHandler且该处理程序返回，则ara::core::Abort的实现应调用std::abort()。⌋

#### 7.2.3.2 SIGABRT信号处理程序

除了ara::core::AbortHandler之外，或者作为其替代方案，应用程序还可以通过为SIGABRT安装信号处理程序来影响此机制。

SIGABRT的信号处理程序与ara::core::AbortHandler具有相同的操作选择：可以终止进程、从函数调用返回、进入无限循环以延迟函数返回，或者执行非局部跳转操作。同样的注意事项也适用于SIGABRT信号处理程序：应避免非局部跳转操作和无限循环。

如果SIGABRT处理程序不返回，通常应使用SIGABRT异常终止。为了在不进入无限循环的情况下实现这一点，应使用std::signal(SIGABRT, SIG_DFL)恢复SIGABRT的默认处理方式，然后通过例如std::raise(SIGABORT)重新引发SIGABRT。

SIGABRT处理程序提供的这种"第二步"影响机制，允许已经处理其他同步信号（如SIGSEGV或SIGFPE）的应用程序以相同的方式处理SIGABRT。

----

### 7.2.4 高级数据类型

#### 7.2.4.1 AUTOSAR类型

##### 7.2.4.1.1 `InstanceSpecifier`

`ara::core::InstanceSpecifier` 的实例用于标识AUTOSAR元模型中的服务端口原型实例，因此在ara::com API和其他地方使用。详细描述和背景信息可参见文献[9]第6.1节（"实例标识符"）和第9.4.4节（"基于ara::com的应用代码中元模型标识符的使用"）。

从概念上讲，`ara::core::InstanceSpecifier` 可以理解为有效元模型路径的字符串表示的包装器。它设计为可以通过工厂方法ara::core::InstanceSpecifier::Create从字符串表示构造（提供无异常解决方案），也可以直接使用构造函数构造（如果字符串表示无效，可能会抛出异常）。

[SWS_CORE_10200] 有效的实例说明符表示 ⌈有效实例说明符的内容由以"/"分隔的模型元素名称列表组成，从可执行文件开始，到实例说明符所应用的相应PortPrototype结束。⌋

[SWS_CORE_10201] 元模型路径验证 ⌈InstanceSpecifier类的构造机制应拒绝根据[SWS_CORE_10200]定义的语法规则在语法上无效的元模型路径。⌋

[SWS_CORE_10202] 实例说明符对象的构造 ⌈应提供可能抛出和不抛出两种形式的InstanceSpecifier对象构造API。⌋

#### 7.2.4.2 派生自基础C++标准的类型

除了上一节提到的AUTOSAR设计的数据类型外，自适应平台还包含许多通用数据类型和辅助函数。

有些类型已经包含在C++11标准[5]中；然而，自适应平台在ara::core命名空间中重新定义了行为几乎相同的类型。这样做的原因是std类型的内存分配行为通常不适合汽车应用。因此，ara::core中的类型定义了自己的内存分配行为，并进行了其他必要的适配，包括关于异常抛出的适配。

[SWS_CORE_00040]{草案} 源自C++标准类的错误 ⌈对于下文根据C++标准相应类指定的ara::core中的类，C++11标准[5]、C++17标准[10]或C++20标准草案[11]中规定会抛出任何异常的所有函数，在抛出异常时都被规定为导致违规。⌋

此类数据类型的示例包括：数组、向量、映射和字符串。

##### 7.2.4.2.1 `Array`

本节介绍 `ara::core::Array` 类型，它表示封装固定大小数组的容器。

ara::core::Array几乎等同于std::array，std::array的大多数类型属性也适用于ara::core::Array。

以下是与std::array的预期差异：
- 省略了std::array::at（以避免强制异常处理）

[SWS_CORE_11200]{草案} 数组基础行为 ⌈ara::core::Array及其所有成员函数和支持构造的行为应与C++14标准[4]头文件<array>中的行为相同，本文档中指定的差异除外。⌋

##### 7.2.4.2.2 SteadyClock

###### 7.2.4.2.2.1 术语定义

C++ std::chrono库定义了许多用于处理时间和持续时间的概念和类型。其中一个概念是"时钟"，它能够创建特定"时间点"的快照。在讨论时钟和时间点时，本文档区分以下三个属性：分辨率、精度和准确度。

**分辨率**：指时钟的测量数据类型能够表示的最小增量。

对于POSIX clock_gettime API的时钟，其分辨率由API使用的struct timespec及其timespec::tv_nsec字段隐式定义为纳秒。

对于std::chrono API的C++时钟，分辨率是可变的。

**精度**：指时钟的计时器能够测量的最小时间间隔。精度由实现定义，取决于物理机器和操作系统的属性和能力。

**准确度**：指时钟报告值与真实值之间的关系。

除此之外，纪元也是时钟的一个重要属性，因为它定义了时钟可以生成的时间范围的基准。测量日历时间的时钟通常使用"Unix时间"，即自"Unix纪元"（1970-01-01 00:00:00 UTC）以来的秒数（不包括闰秒）。

更注重高精度的时钟通常与日历时间无关，而是生成相对于系统上电时间的时间戳。

###### 7.2.4.2.2.2 自适应平台中的时钟

C++ std::chrono库定义了多个标准时钟。其中包括std::chrono::steady_clock，它表示一个单调时钟，其时间点以固定间隔严格递增。

然而，C++标准并未对该时钟的分辨率、精度和准确度提出任何要求。其分辨率的未定义性可能给应用程序员带来一些困难，但这些问题通常可以通过约定一个通用的（或最小的）分辨率来解决。精度和准确度始终取决于机器和操作系统的物理属性。

自适应平台将ara::core::SteadyClock定义为与std::chrono兼容的时钟，具有纳秒分辨率和std::int64_t数据类型。其精度和准确度仍由实现定义，可以作为具体平台的特征值给出。其纪元是ECU的上电时间。凭借这些属性，ara::core::SteadyClock生成的时间戳在其纪元后292年内不会溢出。

它是自适应平台的标准时钟，应用于大多数计时目的。

ara::core::SteadyClock的属性意味着，如果std::chrono::steady_clock::period等同于std::nano，且std::chrono::steady_clock::rep是64位有符号整数类型（如std::int64_t），则std::chrono::steady_clock的类型别名是ara::core::SteadyClock的符合实现。

[SWS_CORE_11800]{草案} 稳态时钟类型要求 ⌈ara::core::SteadyClock类应满足C++11标准[5]中TrivialClock的要求。⌋

[SWS_CORE_11801]{草案} 稳态时钟的纪元 ⌈ara::core::SteadyClock的纪元应为系统启动时间。⌋

#### 7.2.4.3 派生自较新C++标准的类型

这些类型已在较新的C++标准中定义或提出，自适应平台将它们包含在ara::core命名空间中，通常是因为它们是清单某些构造所必需的。

此类数据类型的示例包括：可选值、字符串视图、跨度和变体。

##### 7.2.4.3.1 `ara::core::Byte`

`ara::core::Byte` 是一种能够保存机器"字节"的类型。它是一种与任何其他类型都不同的独立类型。

本节的定义经过精心设计，使得C++17标准[10]中的std::byte成为符合实现，同时也允许仅使用C++11手段的基于类的实现。

与C++17标准[10]中的std::byte不同，ara::core::Byte是否可以用于类型别名而不触发未定义行为由实现定义。

[SWS_CORE_10100] ara::core::Byte的类型属性 ⌈`ara::core::Byte` 类型不应是整数类型。特别是，`std::is_integral<ara::core::Byte>::value` 的值应为0。⌋

[SWS_CORE_10101] ara::core::Byte类型的大小 ⌈`ara::core::Byte` 类型实例的大小（通过 `sizeof(ara::core::Byte)` 确定）应为 `1` 字节。⌋

[SWS_CORE_10102] ara::core::Byte类型的值范围 ⌈`ara::core::Byte` 类型实例的值应限制在 `[0..std::numeric_limits<unsigned char>::max()]` 范围内。⌋

[SWS_CORE_10103] ara::core::Byte实例的创建 ⌈ara::core::Byte类型的实例应可以通过花括号初始化语法从整数类型创建。这种初始化也应可以在常量表达式中调用。如果初始化器值超出ara::core::Byte类型的值范围（参见[SWS_CORE_10102]），则行为未定义。⌋

[SWS_CORE_10104] 默认构造的ara::core::Byte实例 ⌈ara::core::Byte类型的实例应可以在不提供初始化器值的情况下构造。这样的变量定义不应产生运行时开销，且实例的值应具有不确定的内容。⌋

[SWS_CORE_10105] ara::core::Byte类型的析构函数 ⌈ara::core::Byte类型的析构函数应是平凡的。⌋

[SWS_CORE_10106] 从其他类型的隐式转换 ⌈ara::core::Byte类型不应允许从任何其他类型进行隐式转换。⌋

[SWS_CORE_10107] 到其他类型的隐式转换 ⌈ara::core::Byte类型不应允许到任何其他类型的隐式转换，包括bool。⌋

[SWS_CORE_10108] 到unsigned char的转换 ⌈ara::core::Byte类型应允许通过static_cast<>表达式转换为unsigned char。这种转换也应可以在常量表达式中调用。⌋

[SWS_CORE_10109] ara::core::Byte的相等比较 ⌈ara::core::Byte类型应可以与其他ara::core::Byte类型的实例进行相等比较。这种比较也应可以在常量表达式中调用。⌋

[SWS_CORE_10110] ara::core::Byte的不等比较 ⌈ara::core::Byte类型应可以与其他ara::core::Byte类型的实例进行不等比较。这种比较也应可以在常量表达式中调用。⌋

---

