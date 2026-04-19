# Protobuf 协议详细总结


# Protocol Buffers (Protobuf) 语法详细总结
Protocol Buffers（简称Protobuf/protobuf）是Google开源的**跨语言、跨平台、高性能的二进制序列化协议**，核心通过`.proto`文件定义数据结构与服务接口，再通过`protoc`编译器生成对应编程语言的代码，实现高效的数据序列化与RPC通信。

当前主流版本为**proto2**和**proto3**，其中proto3是官方推荐的新版本，本文以proto3为核心，同时明确标注proto2的专属特性与版本差异。

---

## 一、文件基础结构与版本声明
### 1. 版本声明
`.proto`文件的**第一行必须显式声明语法版本**，否则编译器默认按proto2处理。
```protobuf
// proto3 必须显式声明
syntax = "proto3";

// proto2 可省略，默认按proto2解析
syntax = "proto2";
```

### 2. 标准文件结构
一个完整的`.proto`文件遵循以下结构，按顺序定义：
```protobuf
// 1. 版本声明
syntax = "proto3";

// 2. 包声明：避免命名冲突，对应各语言的命名空间/包名
package user;

// 3. 导入其他.proto文件
import "google/protobuf/any.proto";
import "base.proto";
// 公共导入：导入的内容会传递给依赖当前文件的其他文件
import public "common.proto";

// 4. 全局选项：针对文件级别的配置
option java_package = "com.example.user";
option java_outer_classname = "UserProto";
option go_package = "./user";

// 5. 枚举定义
enum Gender {
  UNKNOWN = 0;
  MALE = 1;
  FEMALE = 2;
}

// 6. 消息定义：核心数据结构
message User {
  int64 id = 1;
  string name = 2;
  Gender gender = 3;
}

// 7. 服务定义：用于gRPC RPC接口
service UserService {
  rpc GetUser(GetUserRequest) returns (GetUserResponse);
}
```

### 3. 导入规则
- 编译器通过`-I/--proto_path`参数指定的目录查找导入的文件，未指定则使用当前工作目录
- `import`仅能使用当前文件导入的内容，无法传递依赖
- `import public`可实现依赖传递，适用于公共依赖的封装

---

## 二、标量数据类型
Protobuf提供了一套跨语言的标量类型，不同类型对应不同的编码效率与适用场景，核心标量类型如下：

| Proto类型 | 编码方式 | 核心说明与适用场景 |
|-----------|----------|--------------------|
| double    | 64-bit固定长度 | IEEE754双精度浮点数，适用于高精度小数场景 |
| float     | 32-bit固定长度 | IEEE754单精度浮点数，适用于精度要求不高的小数场景 |
| int32     | 可变长varint | 32位有符号整数，**负数编码效率极低**，仅适用于正整数 |
| int64     | 可变长varint | 64位有符号整数，负数编码效率极低，仅适用于正整数 |
| uint32    | 可变长varint | 32位无符号整数，适用于始终为正的数值 |
| uint64    | 可变长varint | 64位无符号整数，适用于始终为正的大数值 |
| sint32    | 可变长varint+ZigZag | 32位有符号整数，**负数编码效率远高于int32**，适用于可能为负的整数 |
| sint64    | 可变长varint+ZigZag | 64位有符号整数，负数场景优先使用 |
| fixed32   | 32-bit固定长度 | 32位无符号整数，**数值大于2^28时编码效率高于uint32** |
| fixed64   | 64-bit固定长度 | 64位无符号整数，数值大于2^56时编码效率高于uint64 |
| sfixed32  | 32-bit固定长度 | 32位有符号整数，固定长度，大数值场景适用 |
| sfixed64  | 64-bit固定长度 | 64位有符号整数，固定长度，大数值场景适用 |
| bool      | 可变长varint | 布尔类型，取值true/false |
| string    | 长度前缀型 | 字符串，**必须是UTF-8编码或7位ASCII文本**，不可包含非UTF-8字节 |
| bytes     | 长度前缀型 | 任意字节序列，可存储二进制数据，无编码限制 |

> 补充：所有标量类型都有默认值，proto3不支持自定义默认值，proto2可通过`default`选项自定义。

---

## 三、核心字段定义与规则
### 1. 字段定义基础格式
```protobuf
[字段规则] 数据类型 字段名 = 字段编号 [字段选项];
```
示例：
```protobuf
// proto3 基础字段，默认singular规则
string name = 1;
// 带规则与选项的字段
repeated string tags = 2 [deprecated = true];
// proto3 3.12+ 可选字段，支持字段存在性检测
optional int32 age = 3;
```

### 2. 字段编号（核心）
字段编号是Protobuf序列化的核心，决定了字段在二进制数据中的唯一标识，**一旦定义不可修改**，核心规则：
1. 编号范围：`1 ~ 536870911`，其中`19000 ~ 19999`为Protobuf官方保留编号，不可使用
2. 编码效率：`1~15`仅用1字节编码，`16~2047`用2字节编码，**高频字段优先使用1-15编号**
3. 唯一性：同一消息内，字段编号必须唯一，不可重复使用
4. 兼容性：修改字段编号会直接导致新旧版本数据不兼容，严禁修改已发布的字段编号

### 3. 字段规则
字段规则定义了字段的出现次数与行为，proto2和proto3有显著差异：

#### （1）proto3 核心字段规则
| 规则 | 说明 | 适用场景 |
|------|------|----------|
| singular | 默认规则，字段可出现0次或1次，不可多次出现 | 绝大多数单值字段 |
| optional | 3.12.0版本新增，与singular兼容，额外支持**字段存在性检测**（可区分「未赋值」和「赋值为默认值」） | 需要判断字段是否显式赋值的场景 |
| repeated | 字段可出现0次或N次，元素顺序保留，对应编程语言中的数组/列表 | 集合、列表类数据 |
| map | 键值对类型，对应编程语言中的字典/Map | 关联型数据 |

> 补充：proto3中`repeated`字段默认开启`packed=true`紧凑编码，无需手动指定。

#### （2）proto2 专属字段规则
| 规则 | 说明 | 注意事项 |
|------|------|----------|
| required | 必须字段，必须赋值，否则编解码会直接失败 | **官方强烈不推荐使用**，一旦定义无法修改，新旧版本必须包含该字段，兼容性极差 |
| optional | 可选字段，可出现0次或1次，支持自定义默认值 | 与proto3的optional语义一致，proto2默认无字段存在性检测，需显式判断 |
| repeated | 重复字段，默认不开启packed编码，需手动添加`[packed=true]`选项 | 仅对标量数值类型生效 |

---

## 四、枚举类型 enum
枚举用于定义字段的预定义取值集合，限制字段的合法值，核心语法规则如下：

### 1. 基础定义
```protobuf
enum 枚举名 {
  // 第一个枚举常量必须为0，作为枚举的默认值
  常量名1 = 0;
  常量名2 = 1;
  常量名3 = 2;
}
```
示例：
```protobuf
enum UserStatus {
  USER_STATUS_UNSPECIFIED = 0; // 规范：默认值用UNSPECIFIED命名
  USER_STATUS_NORMAL = 1;
  USER_STATUS_DISABLED = 2;
  USER_STATUS_DELETED = 3;
}
```

### 2. 核心规则
1. **默认值强制要求**：枚举的第一个常量编号必须为0，作为枚举的默认值
2. **别名支持**：如需定义同数值的别名，需开启`allow_alias = true`选项
   ```protobuf
   enum HttpStatus {
     option allow_alias = true;
     OK = 0;
     SUCCESS = 0; // 别名，与OK同数值
     NOT_FOUND = 404;
   }
   ```
3. **作用域**：枚举可定义在message内部（嵌套枚举）或文件全局，嵌套枚举需通过`消息名.枚举名`访问
4. **数值范围**：枚举值为int32类型，不推荐使用负数，会降低编码效率
5. **兼容性**：枚举新增常量时，旧版本会保留未知枚举值，不会报错

---

## 五、消息定义与高级组合
message是Protobuf的核心数据结构，对应编程语言中的类/结构体，支持嵌套、自引用、组合等高级特性。

### 1. 基础消息定义
```protobuf
message User {
  // 标量字段
  int64 id = 1;
  string name = 2;
  // 枚举字段
  UserStatus status = 3;
  // 嵌套消息字段
  UserProfile profile = 4;
  // 重复字段
  repeated string tags = 5;
  // 映射字段
  map<string, string> ext_info = 6;

  // 嵌套消息定义
  message UserProfile {
    string avatar = 1;
    string desc = 2;
  }
}
```

### 2. 嵌套与递归消息
1. **嵌套消息**：message内部可嵌套定义其他message，作用域仅限当前消息，外部访问需通过`父消息名.子消息名`
2. **递归消息**：消息可引用自身，实现树、链表等递归数据结构
   ```protobuf
   // 树形结构示例
   message TreeNode {
     int64 id = 1;
     string name = 2;
     // 自引用，子节点列表
     repeated TreeNode children = 3;
   }
   ```

### 3. oneof 关键字
oneof用于定义「互斥字段集合」，集合内的字段**同时最多只能有一个被赋值**，设置其中一个字段会自动清空其他字段，可节省内存，适用于多选一的业务场景。

#### （1）基础语法
```protobuf
message Result {
  oneof data {
    // oneof内的字段不能是repeated
    string success_msg = 1;
    int32 error_code = 2;
    bytes raw_data = 3;
  }
}
```

#### （2）核心规则
1. oneof内的字段共享内存，同一时间仅一个字段生效
2. 赋值oneof内的任意字段，会自动清空同集合内的其他字段
3. oneof内的字段**不能定义为repeated**
4. 支持标量类型、消息类型，不支持map类型
5. 兼容性：新增/删除oneof内的字段需谨慎，可能导致新旧版本数据丢失

### 4. map 键值对类型
map用于定义关联型数据，对应编程语言中的字典/Map，简化键值对的定义。

#### （1）基础语法
```protobuf
map<key_type, value_type> 字段名 = 字段编号;
```
示例：
```protobuf
message Goods {
  int64 id = 1;
  string name = 2;
  // 字符串映射
  map<string, string> specs = 3;
  // 数值key，消息value
  map<int64, GoodsSku> skus = 4;
}

message GoodsSku {
  int64 sku_id = 1;
  int64 price = 2;
}
```

#### （2）核心规则
1. **key类型限制**：仅支持标量整数类型、string类型，不支持浮点型、bytes、枚举、消息、map
2. **value类型限制**：支持除map外的所有类型，不可嵌套map
3. **不可重复**：map字段不能使用repeated规则
4. **无序性**：map的元素顺序不保证，编解码时不可依赖map的顺序
5. **底层实现**：map本质是`repeated 键值对消息`的语法糖，兼容性与repeated一致

---

## 六、proto2 与 proto3 核心差异对比
| 特性 | proto3 | proto2 |
|------|--------|--------|
| 版本声明 | 必须在文件首行显式声明`syntax = "proto3";` | 可省略，默认按proto2解析 |
| 字段规则 | 移除`required`，默认`singular`，3.12+恢复`optional` | 支持`required`、`optional`、`repeated` |
| 默认值 | 不支持自定义默认值，使用类型系统默认值 | 支持通过`default`选项自定义字段默认值 |
| 枚举规则 | 第一个枚举常量必须为0 | 无强制要求，推荐第一个为0 |
| packed编码 | repeated标量字段默认开启`packed=true` | 需手动添加`[packed=true]`开启 |
| 扩展特性 | 移除`extensions`扩展，使用`Any`替代 | 支持`extensions`消息扩展 |
| 废弃特性 | 彻底移除`groups` | 保留已废弃的`groups`，不推荐使用 |
| 语言支持 | 支持更多新兴语言（Go、Rust、Dart等），更新活跃 | 仅维护兼容，无新特性更新 |
| 未知字段 | 3.5+版本保留并序列化未知字段 | 支持保留未知字段 |

---

## 七、选项 Options
选项用于给`.proto`文件、消息、字段等元素添加额外的配置元数据，分为**内置选项**和**自定义选项**，按作用域可分为6类。

### 1. 按作用域分类与常用内置选项
| 作用域 | 常用选项 | 说明 |
|--------|----------|------|
| 文件级 | java_package | 指定生成Java代码的包名 |
| | java_outer_classname | 指定生成Java代码的外层类名 |
| | optimize_for | 优化模式，可选`SPEED`（默认，速度优先）、`CODE_SIZE`（代码体积优先）、`LITE_RUNTIME`（轻量运行时） |
| | go_package | 指定生成Go代码的包路径 |
| 消息级 | message_set_wire_format | 启用消息集格式（proto2专属） |
| | deprecated | 标记消息为废弃，生成代码时添加废弃注解 |
| 字段级 | deprecated | 标记字段为废弃 |
| | default | 自定义字段默认值（proto2专属） |
| | packed | 开启repeated字段的紧凑编码 |
| 枚举级 | allow_alias | 允许枚举常量同数值别名 |
| | deprecated | 标记枚举为废弃 |
| 服务/方法级 | deprecated | 标记服务/方法为废弃 |

示例：
```protobuf
// 文件级选项
option java_package = "com.example.demo";
option deprecated = false;

message Demo {
  // 字段级选项
  string old_field = 1 [deprecated = true];
  // proto2 自定义默认值
  optional int32 count = 2 [default = 10];
}

enum EnumDemo {
  option allow_alias = true;
  DEFAULT = 0;
  START = 0;
}

service DemoService {
  option deprecated = true;
  rpc OldMethod(DemoRequest) returns (DemoResponse) {
    option deprecated = true;
  }
}
```

### 2. 自定义选项
Protobuf支持通过`extend`扩展官方的选项类型，实现自定义元数据，属于高级特性，多用于框架扩展、代码生成定制等场景。
```protobuf
import "google/protobuf/descriptor.proto";

// 扩展字段级选项，自定义字段权限
extend google.protobuf.FieldOptions {
  bool required_permission = 50001; // 自定义选项编号，需使用10000+的扩展范围
}

// 使用自定义选项
message User {
  string id = 1 [(required_permission) = true];
}
```

---

## 八、服务定义 Service（gRPC核心）
Protobuf的`service`关键字用于定义RPC服务接口，配合gRPC框架实现跨服务的远程调用，支持一元RPC和流式RPC四种模式。

### 1. 基础服务定义
```protobuf
// 服务定义
service UserService {
  // 一元RPC：客户端单次请求，服务端单次响应
  rpc GetUser(GetUserRequest) returns (GetUserResponse);

  // 服务端流式RPC：客户端单次请求，服务端多次流式响应
  rpc ListUser(ListUserRequest) returns (stream ListUserResponse);

  // 客户端流式RPC：客户端多次流式请求，服务端单次响应
  rpc BatchCreateUser(stream CreateUserRequest) returns (BatchCreateUserResponse);

  // 双向流式RPC：客户端和服务端均可双向流式读写
  rpc Chat(stream ChatRequest) returns (stream ChatResponse);
}

// 请求/响应消息定义
message GetUserRequest {
  int64 user_id = 1;
}
message GetUserResponse {
  User user = 1;
}
```

### 2. 核心规则
1. 服务内的方法名必须唯一
2. 每个RPC方法必须定义**请求消息**和**响应消息**，无参场景可使用`google.protobuf.Empty`
3. `stream`关键字可修饰请求、响应，分别对应客户端流式、服务端流式
4. 服务和方法支持`deprecated`等选项，标记废弃状态

---

## 九、标准扩展类型（Well-Known Types）
Protobuf官方提供了一套通用的标准类型，封装了常见的业务场景，需导入对应的`.proto`文件后使用，核心常用类型如下：

| 类型 | 导入文件 | 核心作用 |
|------|----------|----------|
| Any | google/protobuf/any.proto | 包装任意消息类型，实现泛型能力，支持`PackFrom`打包、`UnpackTo`解包、`Is`类型判断 |
| Timestamp | google/protobuf/timestamp.proto | 标准时间戳，对应Unix时间，精度到纳秒 |
| Duration | google/protobuf/duration.proto | 时间间隔/持续时间，精度到纳秒 |
| Empty | google/protobuf/empty.proto | 空消息，用于无参的RPC请求/响应 |
| FieldMask | google/protobuf/field_mask.proto | 字段掩码，用于增量更新，指定需要操作的字段 |
| Struct/Value | google/protobuf/struct.proto | 对应JSON的结构体和值，支持动态数据结构 |
| Wrapper类型（Int32Value、StringValue等） | google/protobuf/wrappers.proto | 标量类型的包装类，可区分「0/空字符串」和「未赋值」 |

示例：
```protobuf
import "google/protobuf/any.proto";
import "google/protobuf/timestamp.proto";
import "google/protobuf/empty.proto";

message Event {
  int64 id = 1;
  // 时间戳
  google.protobuf.Timestamp create_time = 2;
  // 任意消息包装
  google.protobuf.Any data = 3;
}

service EventService {
  // 无参请求
  rpc Heartbeat(google.protobuf.Empty) returns (google.protobuf.Empty);
}
```

---

## 十、保留字段 Reserved
`reserved`关键字用于预留字段编号和字段名，防止后续版本误用已删除的字段，**是保证前后兼容性的核心手段**。

### 1. 基础语法
```protobuf
message User {
  // 预留字段编号，支持单个编号、范围
  reserved 2, 5, 10 to 15;
  // 预留字段名，防止新增同名字段
  reserved "old_name", "old_age", "old_status";

  // 正常字段，不可使用已reserved的编号和名称
  int64 id = 1;
  string name = 3;
}
```

### 2. 核心规则
1. 已删除的字段，必须将其编号和名称加入`reserved`，防止后续复用
2. 不可在同一个`reserved`语句中同时声明编号和名称，需分开定义
3. 不可使用已`reserved`的编号和名称定义新字段，编译器会直接报错
4. 枚举类型同样支持`reserved`，用于预留已删除的枚举常量编号和名称

---

## 十一、编码核心原理（语法相关）
Protobuf的高性能源于其高效的二进制编码格式，核心编码规则与语法强相关，核心要点如下：
1. **TLV编码结构**：消息整体采用`Tag-Length-Value`（标签-长度-值）的结构编码，每个字段独立编码，未知字段可直接跳过，保证兼容性
2. **Tag计算规则**：Tag由`字段编号 << 3 | wire_type`组成，低3位为wire_type，决定值的编码方式
3. **核心Wire Type**：
   | Wire Type | 编码类型 | 对应Proto类型 |
   |-----------|----------|--------------|
   | 0 | Varint | int32/int64/uint32/uint64/sint32/sint64/bool/enum |
   | 1 | 64-bit固定长度 | double/fixed64/sfixed64 |
   | 2 | 长度前缀型 | string/bytes/message/repeated/map |
   | 5 | 32-bit固定长度 | float/fixed32/sfixed32 |
4. **Varint编码**：可变长度编码，每个字节最高位为续位标志，小数字仅需1字节，编码效率极高；负数需通过ZigZag编码（sint类型）转换为正整数，否则会占用10字节，效率极低

---

## 十二、兼容性规则与最佳实践
### 1. 核心前后兼容性规则
Protobuf的兼容性核心是「字段编号不变，行为不变」，必须遵守以下规则：
1. **永不修改已发布的字段编号**，修改编号会直接导致新旧版本完全不兼容
2. 新增字段时，旧版本会自动忽略未知字段，新版本可兼容旧版本未赋值的新字段
3. 删除字段时，必须将其编号和名称加入`reserved`，严禁后续复用
4. **永远不要使用required字段**，一旦定义无法删除，新旧版本必须强制赋值，兼容性灾难
5. 字段类型修改需谨慎，仅支持同wire_type的类型修改，否则会导致编解码失败
6. 枚举新增常量时，需保证旧版本可正常处理未知枚举值

### 2. 命名规范
- 消息名、服务名：使用`PascalCase`大驼峰命名
- 字段名、rpc方法名：使用`snake_case`下划线命名
- 枚举名：`PascalCase`大驼峰，枚举常量：`UPPER_SNAKE_CASE`全大写下划线
- 默认枚举值：统一使用`枚举名_UNSPECIFIED = 0`命名，避免歧义

### 3. 工程最佳实践
1. 新项目优先使用proto3，仅兼容老系统时使用proto2
2. 高频字段优先使用1-15的字段编号，低频字段、预留字段使用16以上编号
3. 公共依赖使用`import public`封装，避免循环导入和重复导入
4. 复杂结构拆分多个.proto文件，按业务域划分，避免单文件过大
5. 废弃字段及时标记`deprecated = true`，并加入`reserved`，不要直接删除
6. 区分「未赋值」和「默认值」时，使用proto3的`optional`关键字或官方包装类型，不要用魔数（如-1代表未赋值）


