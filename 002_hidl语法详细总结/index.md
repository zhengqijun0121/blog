# HIDL 语法详细总结


# HIDL语法详细总结
## 一、HIDL基础概述
HIDL（HAL Interface Definition Language）是Android 8.0（Oreo）随Treble项目引入的硬件抽象层接口定义语言，核心目标是实现Android Framework与HAL的彻底解耦，解决Android碎片化OTA升级难题，让厂商无需重新编译HAL即可完成Framework升级。

- **运行模式**：分为**绑定式模式（Binderized）**（基于Binder IPC跨进程通信，主流模式）和**直通模式（Passthrough）**（同进程dlopen加载，仅C++支持，用于兼容旧版HAL）。
- **生命周期**：Android 10起官方宣布逐步弃用，推荐使用AIDL替代，但仍是Android 8-13版本HAL开发的核心规范，存量设备仍大量使用。
- **核心设计原则**：接口发布后即冻结，仅可通过版本升级扩展，严格保证前向兼容；RPC仅使用in参数，避免内存所有权歧义；数据结构采用C++标准布局，无需解包即可直接用于IPC传输。

## 二、核心语法规则与文件结构
### 1. 文件规范
- 后缀名：统一为`.hal`
- 两类核心文件：
  - **接口文件**：如`IFoo.hal`，每个文件仅能定义一个接口，接口名必须与文件名一致，以`I`开头。
  - **类型定义文件**：`types.hal`，仅能定义用户自定义类型（结构体、枚举、联合体等），不能定义接口方法，包内所有接口自动导入该文件，无需显式import。
- 目录结构：`[根目录]/[模块名]/[版本号]/`，例如`hardware/interfaces/nfc/1.0/`；核心系统包根目录为`hardware/interfaces`，厂商包根目录为`vendor/[厂商名]/interfaces`。

### 2. 基础语法规则
- 语句终止：所有语句必须以分号`;`结尾，包括最后一个元素。
- 空白符：换行符与空格、制表符等效，除单行注释外，换行不影响语法解析。
- 预处理器：不支持C/C++预处理器（无#define、#include等）。
- 大小写敏感：标识符、关键字、类型名均大小写敏感。
- 标识符规则：遵循C标准标识符规则，由字母、数字、下划线组成，不能以数字开头。

### 3. 注释规范
- **文档注释**：`/** */`，仅可用于类型、方法、字段、枚举值声明，会被编译到生成的代码文档中。
- **多行注释**：`/* */`，用于普通说明，不会生成到文档中。
- **单行注释**：`//`，行尾注释，不推荐用于文档说明，不会生成到输出代码中。

## 三、包声明与导入
### 1. 包声明（Package）
- 语法：`package 包名@主版本号.次版本号;`
- 命名规范：
  - 核心系统包：`android.hardware.子系统名[.子模块]@x.y`，全小写，多单词用snake_case或子模块拆分。
  - 厂商包：`vendor.厂商名.hardware.子系统名@x.y`
- 位置：必须位于文件首行（许可证注释之后），每个.hal文件有且仅有一个包声明。
- 命名空间映射：包名会自动转换为对应语言的命名空间，例如`android.hardware.nfc@1.0`在C++中转为`android::hardware::nfc::V1_0`。

### 2. 导入声明（Import）
用于导入其他包或同包内的类型、接口，避免全限定名重复书写。
- 语法格式：
  1. 全量包导入：`import 包名@x.y;`，导入该包下所有类型和接口。
  2. 指定类型导入：`import 包名@x.y::类型名;`，仅导入指定的用户自定义类型或接口。
  3. 同包导入：`import 接口名/类型名;`，导入同包内的其他接口或类型。
  4. 仅导入类型：`import 包名@x.y::types;`，仅导入该包的types.hal中定义的类型。
- 核心规则：
  - 当前包的types.hal会被自动导入，无需显式import。
  - 导入语句必须位于包声明之后，接口/类型定义之前。
  - 导入分组按规范排序：系统包→厂商包→同包内导入，组内按字母序排列，组间空行分隔。

## 四、完整数据类型系统
HIDL数据类型分为基础类型、复合类型、特殊类型三大类，所有类型均有明确的跨语言映射规则，保证IPC传输的一致性。

### 1. 基础标量类型
| HIDL类型 | 说明 | C++映射 | Java映射 |
|----------|------|---------|----------|
| int8_t   | 8位有符号整数 | int8_t | byte |
| uint8_t  | 8位无符号整数 | uint8_t | byte（无符号语义需自行处理） |
| int16_t  | 16位有符号整数 | int16_t | short |
| uint16_t | 16位无符号整数 | uint16_t | short |
| int32_t  | 32位有符号整数 | int32_t | int |
| uint32_t | 32位无符号整数 | uint32_t | int |
| int64_t  | 64位有符号整数 | int64_t | long |
| uint64_t | 64位无符号整数 | uint64_t | long |
| float    | 32位单精度浮点数 | float | float |
| double   | 64位双精度浮点数 | double | double |
| bool     | 布尔值 | bool | boolean |
| string   | UTF-8编码字符串 | std::string | String |

### 2. 定长数组
- 语法：`类型[数组长度]`，数组长度必须是大于0的C风格常量表达式constexpr。
- 示例：`int32_t[4] position;`、`string[10] nameList;`
- 规则：长度固定，编译期确定，支持多维数组，如`uint8_t[3][3] matrix;`

### 3. 动态向量vec<T>
- 语法：`vec<元素类型>`，对应C++的std::vector，动态长度数组。
- 示例：`vec<uint8_t> dataBuffer;`、`vec<vec<int32_t>> multiDimVector;`
- 特殊规则：接口类型仅支持单层vec嵌套，即`vec<IFoo>`合法，`vec<vec<IFoo>>`非法。
- 复合用法：支持向量与数组嵌套，如`vec<bool[4]> arrayVector;`

### 4. 特殊类型
- `memory`：对应Ashmem共享内存句柄，用于跨进程大数据传输，映射为`hidl_memory`类型。
- `pointer`：无类型指针，仅用于直通模式，绑定式模式下不支持跨进程传输，映射为`void*`。
- `native_handle_t`：原生句柄，用于传递文件描述符、驱动句柄等，支持跨进程传输。
- `bitfield<T>`：位域类型，T必须是用户定义的枚举类型，用于按位存储枚举值，实现位掩码功能。
- `fmq_sync<T>` / `fmq_unsync<T>`：快速消息队列（Fast Message Queue），分别对应同步/无锁异步模式，用于高频率低延迟的跨进程数据传输，T必须是可序列化的固定大小类型。

### 5. 接口类型
- 所有自定义interface都是合法的类型，可作为方法参数、返回值、结构体字段。
- 通用接口类型`interface`，等价于`android.hidl.base@1.0::IBase`，是所有HIDL接口的基类。

## 五、用户自定义类型
HIDL支持丰富的用户自定义类型，所有类型均可用于方法参数、返回值和复合类型嵌套，**不支持前向声明**，类型必须在使用前定义（同文件内后续定义编译器会自动解析）。

### 1. 枚举（enum）
- 语法：
```hidl
enum 枚举名 : 底层标量类型 {
    枚举值1 [= 常量表达式],
    枚举值2 [= 常量表达式],
    ...
};
```
- 核心规则：
  - 底层类型必须是标量整数类型，必须显式声明。
  - 枚举值默认从0开始递增，也可手动赋值，支持位运算常量表达式（如`1 << 3`）。
  - 枚举值命名规范：全大写，下划线分隔。
  - 支持枚举嵌套，可定义在结构体、接口内部。
  - 全限定名规则：`包名@x.y::枚举名:枚举值`，例如`android.hardware.nfc@1.0::NfcStatus:STATUS_OK`。
- 示例：
```hidl
/** NFC操作状态码 */
enum NfcStatus : int32_t {
    STATUS_OK = 0,
    STATUS_ERR_ARG = 1,
    STATUS_ERR_TIMEOUT = 2,
    STATUS_ERR_UNKNOWN = -1,
};
```

### 2. 结构体（struct）
- 语法：
```hidl
struct 结构体名 {
    类型1 字段名1;
    类型2 字段名2;
    ...
    // 支持嵌套类型定义
    struct 嵌套结构体名 {
        ...
    };
};
```
- 核心规则：
  - 字段命名采用lowerCamelCase，类型名采用UpperCamelCase。
  - 支持嵌套结构体、枚举、联合体定义，嵌套类型通过`.`访问，例如`Foo.Bar`。
  - 生成的C++代码为标准布局结构体，内存布局固定，可直接用于IPC传输。
  - 不支持前向声明，结构体不能直接或间接引用自身。
- 示例：
```hidl
struct Point {
    int32_t x;
    int32_t y;
};

struct Rect {
    Point topLeft;
    Point bottomRight;
    vec<uint8_t> extraData;
};
```

### 3. 联合体（union）与安全联合体（safe_union）
- 普通union语法：
```hidl
union 联合体名 {
    类型1 字段名1;
    类型2 字段名2;
    ...
};
```
- safe_union：HIDL扩展的安全联合体，自动包含tag字段标识当前有效的成员，避免内存越界，Java仅支持safe_union，推荐优先使用。
- 核心规则：
  - 联合体所有成员共享同一块内存，大小为最大成员的大小。
  - 普通union不支持包含接口类型、string、vec等动态大小类型，safe_union无此限制。
- 示例：
```hidl
union DataUnion {
    int32_t intValue;
    float floatValue;
    int64_t longValue;
};

safe_union SafeDataUnion {
    int32_t intValue;
    float floatValue;
    vec<uint8_t> buffer;
};
```

### 4. 类型别名（typedef）
- 语法：`typedef 原类型 别名;`
- 作用：为复杂类型定义简短别名，提升代码可读性。
- 示例：
```hidl
typedef vec<uint8_t> ByteBuffer;
typedef int32_t[4] FourIntArray;
typedef android.hardware.nfc@1.0::NfcStatus StatusCode;
```

## 六、接口与方法定义
接口是HIDL的核心，定义了HAL提供的服务能力，每个接口对应一个独立的.hal文件，编译后生成对应的C++/Java客户端与服务端代码。

### 1. 接口声明
- 基础语法：
```hidl
[注解]
interface 接口名 [extends 父接口全限定名] {
    // 类型定义（枚举、结构体、typedef等）
    // 方法定义
};
```
- 核心规则：
  - 接口名必须以`I`开头，采用UpperCamelCase，必须与文件名一致。
  - 支持单继承，仅能继承一个其他HIDL接口，父接口必须是全限定名导入的接口。
  - 所有HIDL接口都隐式继承自`android.hidl.base@1.0::IBase`，无需显式声明。
  - 接口内部可定义局部类型，仅在该接口内部可见。
- 最简示例：
```hidl
package android.hardware.foo@1.0;

/** 示例服务接口 */
interface IFoo {
    /** 无返回值方法 */
    close();

    /** 带返回值方法 */
    open() generates (int32_t status);
};
```

### 2. 方法定义
- 基础语法：
```hidl
[注解] [oneway] 方法名(参数列表) [generates (返回值列表)];
```
- 核心组成与规则：
  1. **方法名**：采用lowerCamelCase，同一个接口内不支持方法重载。
  2. **参数列表**：`类型 参数名`格式，多个参数用逗号分隔，所有参数均为in方向，无out/inout参数，避免内存所有权歧义。
  3. **oneway关键字**：标记方法为异步单向调用，客户端调用后立即返回，不等待服务端执行，**不能有generates返回值**，服务端异常客户端无法感知。
  4. **generates关键字**：用于定义返回值，支持多个返回值，是HIDL与AIDL的核心区别之一。

- 方法示例：
```hidl
/** 无返回值同步方法 */
reset();

/** 单返回值同步方法 */
getVersion() generates (string versionName);

/** 多返回值同步方法 */
readData(int32_t address, uint32_t length) generates (int32_t status, vec<uint8_t> data);

/** oneway异步方法 */
oneway reportHeartbeat(int64_t timestamp);
```

### 3. 回调接口设计
HIDL通过接口类型参数实现回调，服务端可反向调用客户端实现的接口方法，实现异步事件通知。核心设计模式：
1. 定义回调接口，声明服务端需要调用的回调方法。
2. 主接口提供注册方法，接收回调接口实例作为参数。
3. 客户端实现回调接口，传递给服务端，服务端在事件触发时调用回调方法。

- 示例：
```hidl
// IDataCallback.hal
package android.hardware.foo@1.0;

interface IDataCallback {
    /** 数据就绪回调 */
    oneway onDataReady(vec<uint8_t> data);
};

// IFoo.hal
package android.hardware.foo@1.0;

import IDataCallback;

interface IFoo {
    /** 注册数据回调 */
    registerDataCallback(IDataCallback callback) generates (int32_t status);
};
```

## 七、注解系统
HIDL支持Java风格的注解，用于为类型、方法、参数添加额外的语义信息，编译器和工具链会根据注解做对应的校验和代码生成。

### 1. 注解基础语法
- 无参数注解：`@注解名`
- 单参数注解：`@注解名(值)`
- 多参数注解：`@注解名(键1=值1, 键2=值2, ...)`
- 数组参数：`@注解名(键={值1, 值2, ...})`
- 位置：注解必须位于目标元素之前，单独占一行，多个注解按字母序排列。

### 2. 常用核心注解
| 注解名 | 适用目标 | 核心作用 |
|--------|----------|----------|
| @entry | 方法 | 标记为接口核心入口方法，VTS测试优先覆盖 |
| @exit | 方法 | 标记为资源释放、注销类退出方法 |
| @callback | 方法 | 标记为回调方法，提示由服务端反向调用客户端实现 |
| @nullable | 接口类型参数/字段 | 标记该接口类型参数可以为null，默认不允许为null |
| @noreturn | 方法 | 标记该方法不会返回，编译器做死代码检查 |
| @deprecated | 接口/方法/类型 | 标记已废弃，编译器生成警告 |
| @async | 方法 | 标记为异步方法，结果通过回调传递 |

### 3. 注解使用示例
```hidl
@deprecated
interface IOldFoo {
    @entry
    open() generates (int32_t status);

    @exit
    close();

    registerCallback(@nullable ICallback callback);
};
```

## 八、版本控制规则
HIDL的核心设计之一是版本化管理，接口发布后即冻结，所有修改必须通过版本升级实现，严格遵循语义化版本规范。

### 1. 版本号格式
- 格式：`主版本号.次版本号`，无补丁版本号。
- **次版本号升级（Minor）**：必须保证前向兼容，仅可新增类型、新增接口、为已有接口新增方法，不能修改/删除已有方法、不能修改已有类型的定义。
- **主版本号升级（Major）**：不保证前向兼容，可修改、删除、重构接口和类型，与旧版本完全独立。

### 2. 核心版本规则
- 包级版本控制：版本号作用于整个包，包内所有接口和类型共享同一个版本号。
- 接口冻结：发布后的包不可修改，任何修改必须创建新版本的包。
- 继承扩展：新版本接口可继承旧版本接口，复用已有能力，新增扩展方法。
- 导入规则：次版本升级的包必须导入同主版本的上一个次版本包，保证兼容。

### 3. 版本升级示例
- 兼容升级（1.0→1.1）：
```hidl
package android.hardware.nfc@1.1;

import android.hardware.nfc@1.0::INfc;

interface INfc extends @1.0::INfc {
    // 仅新增方法，不修改原有定义
    getNfcVersion() generates (string version);
};
```
- 不兼容升级（1.1→2.0）：
```hidl
package android.hardware.nfc@2.0;

// 完全重构，无需兼容旧版本
interface INfc {
    init(NfcConfig config) generates (NfcStatus status);
    deinit();
};
```

## 九、代码风格与命名规范
基于AOSP官方HIDL代码风格指南，核心规范如下：

| 元素类型 | 命名规范 | 示例 |
|----------|----------|------|
| 包名 | 全小写，多单词用snake_case或子模块拆分 | android.hardware.nfc@1.0 |
| 接口名 | I开头，UpperCamelCase大驼峰 | INfc、IPower |
| 方法名 | lowerCamelCase小驼峰 | openDevice、readData |
| 结构体/枚举/typedef名 | UpperCamelCase大驼峰 | NfcConfig、NfcStatus |
| 枚举值 | 全大写，下划线分隔 | STATUS_OK、ERROR_TIMEOUT |
| 字段/参数名 | lowerCamelCase小驼峰 | dataBuffer、maxLength |
| 文件名 | 与接口名一致，UpperCamelCase，.hal后缀 | INfc.hal、types.hal |

- 格式规范：使用4个空格缩进，单行代码最大长度100列；左大括号与声明位于同一行，右大括号单独占一行；文档注释多行内容每行以`*`开头，对齐排列。

## 十、高级特性
### 1. 直通模式（Passthrough）
仅支持C++，用于将旧版HAL快速适配到HIDL框架，无需跨进程Binder通信。编译时生成`BsFoo.h`头文件，实现`dlopen`加载的入口函数`HIDL_FETCH_IFoo`；客户端通过`getService(name, true)`获取直通模式服务，同进程内直接函数调用。

### 2. 快速消息队列FMQ
基于共享内存实现的无Binder IPC开销的通信机制，分为`fmq_sync<T>`（同步模式，多写单读）和`fmq_unsync<T>`（无锁模式，单写单读，性能最高），用于高频率低延迟的跨进程数据传输（如传感器、音视频数据流）。

### 3. 共享内存memory
基于Ashmem匿名共享内存，用于传输大体积数据，突破Binder IPC的4MB大小限制。通过`hidl_memory`句柄在进程间传递，对方映射到进程地址空间，实现零拷贝数据共享。

### 4. 嵌套类型访问
嵌套定义的类型通过`.`访问，例如接口内的结构体`IFoo.InnerStruct`，全限定名示例：`android.hardware.foo@1.0::IFoo.InnerStruct`。

### 5. 权限控制
每个HIDL方法对应一个Binder事务，支持SELinux权限控制，可通过SELinux策略限制进程的服务访问权限和方法调用权限。


