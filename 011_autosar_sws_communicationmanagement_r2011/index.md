# AUTOSAR_SWS_CommunicationManagement


# AUTOSAR_SWS_CommunicationManagement

----

## 7 功能规范

### 7.1 总体描述

AUTOSAR自适应架构将自适应平台基础软件组织为多个功能集群。这些集群向应用程序提供通用功能服务。AUTOSAR自适应平台的 **通信管理（`CM`）** 正是这样一个功能集群，隶属于"AUTOSAR自适应应用运行时（`ARA`）"。它负责构建和监控应用程序之间的通信路径，支持本地和远程通信。

通信管理提供基础架构，实现同一机器内自适应 `AUTOSAR` 应用之间，以及与其他机器上软件实体（如其他自适应 `AUTOSAR` 应用或经典 `AUTOSAR` 软件组件）的通信。所有通信路径均可在设计阶段、启动阶段或运行阶段建立。

本规范包含API的语法、API 与模型的关系，并通过状态机、前置/后置条件假设以及API使用方式描述其语义。本规范不对平台实现的软件架构施加约束，因此未定义基础软件模块，也未规定通信管理的实现方式或内部技术架构。

#### 7.1.1 架构概念

`AUTOSAR` 自适应平台的通信管理在逻辑上可划分为以下子部分：
- 语言绑定
- 端到端通信保护
- 通信/网络绑定
- 通信管理软件

**图7.1：通信管理技术架构**

在通信管理范畴内，定义了以下类型的接口：
- **公共应用接口**：自适应 `AUTOSAR API` 的一部分，在本软件规范（`SWS`）中规定。即标准化的 `ara::com API`。
- **功能集群交互**：功能集群之间的交互。非规范性内容，旨在提高规范可读性并支持软件集成到演示系统中（图7.1中虚线箭头）。同时也包括功能集群内部元素之间的交互，未在规范中定义，属于非标准化接口，用于通信管理软件内部通信（图7.1中灰色箭头）。

请注意，语言绑定和通信绑定依赖于集成商的特定配置，但必须部署在应用程序二进制文件中。这意味着通信绑定的序列化过程将在自适应应用的执行上下文中运行。

`ARA API` 的设计遵循以下约束：
- 支持应用软件组件的独立性
- 采用面向服务的通信方式，不依赖特定通信协议
- 保持 `API` 尽可能精简，既不支持可在 `API` 之上实现的特殊用例，也不支持组件模型或更高层级概念。`API` 仅用于支持核心通信机制。
- 支持动态通信：
    - 应用中间件不执行发现：客户端知晓服务器，但服务器不知晓客户端。事件订阅是应用中唯一的动态通信模式。
    - 应用层全服务发现：配置阶段无需知晓任何通信路径。服务发现 `API` 允许应用代码自主选择服务实例。
- 同时支持事件/回调和轮询两种 `API` 使用方式，以兼容传统RTE范式。为满足基于回调/事件交互的高确定性要求，应提供避免非受控上下文切换的能力。
- 同时支持同步回调式通信和异步通信理念。
- 支持客户端/服务器通信。
- 支持发送/接收通信，包含"最新值优先" `last-is-best` 和队列两种语义。对于队列式通信，接收端缓存可配置。
- 支持任务激活触发条件的选择。
- 安全扩展。
- 服务质量（`QoS`）扩展。
- 实时系统可扩展性。
- 支持内置端到端通信保护，可在 `ARA API` 之上实现特定用例的行为。

#### 7.1.2 设计决策

ARA API的设计遵循以下原则：
- 采用 **代理/骨架（Proxy/Skeleton）** 模式：
    - **（服务）代理**：代表可能位于远程（即其他进程、其他内核、其他节点）的服务。它是使用该服务的应用/客户端本地的C++类实例。
    - **（服务）骨架**：将用户提供的服务实现连接到中间件传输基础架构。服务实现类继承自（服务）骨架。
    - 除代理/骨架外，可能存在所谓的"运行时（`Runtime`）"单例类，提供管理代理和骨架的基本功能。但这属于通信管理软件的实现细节，本规范未作规定，未来版本可能会补充。

关于代理/骨架设计模式的一般概念及其在中间件实现中的作用，参见文献[9][10]。
- 支持数据接收时的回调机制。
- `API` 具备零拷贝能力，包括中间件内存管理功能。
- 支持接收数据的过滤。
- 与 `AUTOSAR` 服务模型（服务、实例、事件、方法等）对齐，支持从该模型自动生成代理和骨架。
- `API` 层支持完整的服务发现和服务实例选择。
- 客户端/服务器通信采用 `C++11` 语言引入的概念（如 `std::future`、`std::promise`），全面支持不同上下文之间的方法调用。
- 抽象 `SOME/IP` 特定行为，但支持 `SOME/IP` 服务机制（方法、事件和字段）。
- 支持/实现文献[7]和[4]中规定的标准端到端保护协议。
- 支持服务契约版本化。
- 同等支持事件和轮询两种 `API` 使用方式，以兼容传统实时范式。
- 充分利用 `C++11/14` 特性设计 `API`，提升应用开发者的使用体验和便捷性。

关于 `ARA API` 设计的更多细节和说明，参见《`ARA Com API` 解释性文档》[1]。

#### 7.1.3 通信范式

**面向服务的通信（SoC）** 作为面向服务架构（`SOA`）[11]的一部分，是自适应 `AUTOSAR` 应用的主要通信模式。它支持在运行时建立通信路径，因此可用于构建参与者数量未知的动态通信。图7.2展示了面向服务通信的基本工作原理。

**图7.2：面向服务通信**

服务发现决定是否建立外部和内部面向服务通信。发现策略应支持返回特定服务实例，或请求时所有可用的服务实例（无论本地还是远程）。通信管理软件应根据服务提供者的位置，为服务发现和通信连接提供优化实现。关于服务发现的更多内容，参见《`SOME/IP` 服务发现协议规范》[12]。

服务类是自适应 `AUTOSAR` 中面向服务通信模式的核心元素。它通过收集实现具体服务功能的应用所提供或请求的方法和事件来表示服务。

#### 7.1.4 服务契约版本化

在面向服务架构（`SOA`）环境中，服务的客户端和提供者依赖于包含服务接口和行为的契约。服务的接口和行为可能随时间变化。因此，引入了**服务契约版本化**机制，以区分服务的不同版本。

**图7.3：服务契约随时间的版本化**

`AUTOSAR` 自适应平台支持服务契约版本化。服务契约版本化分为设计阶段和部署阶段。这意味着设计层面的任何服务都有自己的版本号，该版本号会映射到所使用网络绑定的版本号，反之亦然。映射过程由服务设计者或集成商手动完成。

**图7.4：服务契约版本化流程**

**注：**

1. 服务接口的 `ServiceInterface` 的契约版本由**主版本号** `majorVersion` 和**次版本号** `minorVersion` 组成：
    - 主版本号表示向后不兼容的服务变更
    - 次版本号表示向后兼容的服务变更
    - 对于向后不兼容的接口或行为变更，主版本号加 `1`，次版本号置 `0`
    - 对于向后兼容的接口或行为变更，主版本号不变，次版本号加 `1`
2. 服务接口的契约版本映射到服务接口部署版本。同一服务接口可进行多次映射，生成多个服务接口部署版本。根据文献[6]中的[constr_1723]要求，这种映射应确保在每个 `VLAN` 上的标识唯一性。

**[SWS_CM_99003]{草案}** ⌈服务发现应根据用于服务连接的网络绑定，评估服务接口部署版本的向后兼容性。⌋

----

### 7.2 事件的端到端通信保护

本节规定了 `ara::com` 中端到端（`E2E`）通信保护在事件处理中的集成方式。

**[SWS_CM_90402]{草案}** ⌈受 `E2E` 保护的事件，其选项应在 `End2EndEventProtectionProps` 和 `E2EProfileConfiguration` 中配置。⌋

**[SWS_CM_90433]{草案}** ⌈本节中提及的 `E2E` 函数 `E2E_protect` 和 `E2E_check` 应满足文献[7]中定义的E2E保护要求，并符合文献[4]中的E2E保护协议规范（特别是[PRS_E2E_00323]）。⌋

对于属于特定 `ServiceProxy`/`ServiceSkeleton` 类的每个具体事件类，都有对应的 `E2E Data ID`（例如，基于服务ID、服务实例ID和事件ID的组合生成）。

#### 7.2.1 局限性

本规范规定的事件E2E通信保护存在以下局限性：
- 不支持 `EndToEndTransformationComSpecProps`
- 关于E2E保护的一般局限性和可检测的故障模式，参见文献[4]

#### 7.2.2 发布者

**[SWS_CM_90453]{草案}** ⌈对于受 `E2E` 保护的事件，`E2E` 保护应在 `Send` 的上下文中执行。⌋

图7.5展示了发布者端E2E保护涉及的组件交互概览。

**图7.5：E2E发布者**

**[SWS_CM_90430]{草案}** ⌈对于受E2E保护的事件，`Send` 应序列化样本数据，并根据相应网络绑定的规则（例如，SOME/IP 网络绑定情况下的[SWS_CM_10291]）添加协议头，生成序列化数据。⌋

从 `E2E` 保护角度看，该序列化数据包含非保护部分和待保护部分（参见[PRS_E2E_USE_00236]和[PRS_E2E_USE_00741]）。

**[SWS_CM_90401]{草案}** ⌈对于受 `E2E` 保护的事件，应根据[PRS_E2E_00323]对待保护的序列化数据（作为 `serializedData` 参数传递给 `E2E_protect`）调用 `E2E_protect`。⌋

**[SWS_CM_90403]{草案}** ⌈对于受 `E2E` 保护的事件，应将 `End2EndEventProtectionProps.dataId`作为 `dataID` 参数传递给 `E2E_protect`。⌋

**[SWS_CM_90404]{草案}** ⌈对于受 `E2E` 保护的事件，在 `SOME/IP` 序列化情况下，应将 `E2E header` 添加到消息中。如果相应网络绑定的协议规范对 `E2E header` 的位置有约束（例如，`SOME/IP` 网络绑定情况下的[PRS_SOMEIP_00941]），则应遵守这些约束。⌋

#### 7.2.3 订阅者 - GetNewSamples

**[SWS_CM_90406]{草案}** ⌈对于受 `E2E` 保护的事件，`E2E` 校验应在 `GetNewSamples` 的上下文中执行。⌋

图7.6展示了订阅者端 E2E 校验涉及的组件交互概览。

**图7.6：E2E订阅者**

**[SWS_CM_90407]{草案}** ⌈对于受 `E2E` 保护的事件，`GetNewSamples` 应首先获取上次调用该函数后未被提取的所有序列化数据集合。⌋

从 `E2E` 保护角度看，该序列化数据包含非保护部分和待保护部分（参见[PRS_E2E_USE_00236]和[PRS_E2E_USE_00741]）。

##### 7.2.3.1 情况1 - 存在一个或多个序列化样本

对于受 `E2E` 保护的事件，如果接收到一个或多个样本的序列化数据，则对每个样本执行以下步骤：

**[SWS_CM_90408]{草案}** ⌈对于给定的受 `E2E` 保护样本，`GetNewSamples` 应处理样本序列化数据中的非 `E2E header`（如果存在）。⌋

**[SWS_CM_90410]{草案}** ⌈对于给定的受 `E2E` 保护样本，应根据[RS_E2E_08540]和[PRS_E2E_00323]对受保护的序列化数据（作为 `serializedData` 参数传递给 `E2E_check`）调用 `E2E_check`。⌋

**[SWS_CM_90454]{草案}** ⌈对于给定的受 `E2E` 保护样本，应将 `End2EndEventProtectionProps.dataId` 作为 `dataID` 参数传递给 `E2E_check`。⌋

**[SWS_CM_90411]{草案}** ⌈作为返回值，对于给定的受E2E保护样本，`E2E_check` 应提供一个结果（根据文献[4]的[PRS_E2E_00322]定义的 `e2eResult`），包含 `SMState`（根据文献[4]的[PRS_E2E_00322]定义的 `e2eState`）和 `ProfileCheckStatus`（根据文献[4]的[PRS_E2E_00322]定义的 `e2eStatus`）两个元素。⌋

**[SWS_CM_90455]{草案}** ⌈对于给定的受 `E2E` 保护样本，应从序列化数据中移除 `E2E header`。⌋

**[SWS_CM_90412]{草案}** ⌈对于给定的受 `E2E` 保护样本，`GetNewSamples` 应根据相应网络绑定的规则（例如，`SOME/IP` 网络绑定情况下的[SWS_CM_10294]）反序列化处理后的序列化数据，生成反序列化样本。⌋

**[SWS_CM_90413]{草案}** ⌈对于给定的受 `E2E` 保护样本，`GetNewSamples` 应将 `ProfileCheckStatus` 存储在 `SamplePtr` 中，并在该特定受 `E2E` 保护事件的事件类中更新/覆盖全局 `SMState`。⌋

##### 7.2.3.2 情况2 - 无序列化样本

对于受E2E保护的事件，如果未接收到任何序列化数据，处理步骤更简单，E2E保护将作为超时检测机制工作。

**[SWS_CM_90415]{草案}** ⌈应根据[RS_E2E_08540]和[PRS_E2E_00323]对空样本（即向`E2E_check`传递空指针作为`serializedData`参数）调用`E2E_check`。⌋

**[SWS_CM_90456]{草案}** ⌈应将`End2EndEventProtectionProps.dataId`作为`dataID`参数传递给`E2E_check`。⌋

**[SWS_CM_90416]{草案}** ⌈作为返回值，对于给定的空样本，`E2E_check`应提供一个结果（根据文献[4]的[PRS_E2E_00322]定义的`e2eResult`），包含`SMState`（根据文献[4]的[PRS_E2E_00322]定义的`e2eState`）和`ProfileCheckStatus`（根据文献[4]的[PRS_E2E_00322]定义的`e2eStatus`）两个元素。⌋

**[SWS_CM_90417]{草案}** ⌈`GetNewSamples`应在该特定受E2E保护事件的事件类中更新/覆盖全局`SMState`。⌋

#### 7.2.4 订阅者 - 可调用对象f

用户提供的可调用对象`f`会为每个接收到的样本调用。调用时将对应样本的`SamplePtr`作为参数传递给`f`。`SamplePtr`包含反序列化后的样本以及`ProfileCheckStatus`。

#### 7.2.5 订阅者 - E2E信息访问

**[SWS_CM_90457]{草案}** ⌈每个`SamplePtr`应提供`GetProfileCheckStatus`方法，用于访问每个样本的`ProfileCheckStatus`（参见[SWS_CM_90420]）。⌋

**[SWS_CM_10475]{草案}** ⌈应为特定`ServiceProxy`类的每个事件类提供`GetSMState`方法。⌋

**[SWS_CM_90431]{草案}** ⌈`GetSMState`方法应提供对特定事件类全局`SMState`的访问，该状态由上次调用`GetNewSamples`时最后一次执行的`E2E_check`函数确定（参见[SWS_CM_90417]）。⌋

```C++

ara::com::e2e::SMState GetSMState() const noexcept;
```

----

### 7.3 方法的端到端通信保护

本节规定了`ara::com`中端到端（E2E）通信保护在方法处理中的集成方式，包括方法请求的E2E通信保护，以及任何类型方法响应（正常响应或错误响应）的E2E通信保护。

**[SWS_CM_10460]{草案}** ⌈受E2E保护的方法，其选项应在`End2EndMethodProtectionProps`和`E2EProfileConfiguration`中配置。⌋

**[SWS_CM_90485]{草案}** ⌈本节中提及的E2E函数`E2E_protect`和`E2E_check`应满足文献[7]中定义的E2E保护要求，并符合文献[4]中的E2E保护协议规范（特别是[PRS_E2E_00828]）。⌋

对于属于特定`ServiceProxy`类的每个具体方法类（[SWS_CM_00196]），以及属于特定`ServiceSkeleton`类的每个提供方法（参见[SWS_CM_00191]），都有对应的E2E数据ID（例如，基于服务ID、服务实例ID和方法ID的组合生成）。

在本节范围内，**E2E校验失败**指调用`E2E_check`返回的`e2eStatus`为`REPEATED`、`WRONGSEQUENCE`、`NOTAVAILABLE`或`NONEWDATA`；**E2E校验成功**指调用`E2E_check`返回的`e2eStatus`不为上述值。

#### 7.3.1 局限性

本规范规定的方法E2E通信保护存在以下局限性：

- 受E2E保护的方法不支持`kEvent`处理模式（并发线程）

- 不支持`EndToEndTransformationComSpecProps`

- 关于E2E保护的一般局限性和可检测的故障模式，参见文献[4]

#### 7.3.2 服务方法请求的E2E保护（客户端）

**[SWS_CM_10462]{草案}** ⌈对于受E2E保护的方法，请求消息的E2E保护应在相应服务方法的方法类`operator()`的上下文中执行（参见[SWS_CM_00196]）。⌋

图7.7展示了客户端端方法请求E2E保护涉及的组件交互概览。

**图7.7：客户端端方法请求E2E保护的组件交互**

##### 7.3.2.1 负载序列化

**[SWS_CM_90458]{草案}** ⌈对于受E2E保护的方法请求，`operator()`应序列化方法的输入和输入输出参数，并根据相应网络绑定的规则（例如，SOME/IP网络绑定情况下的[SWS_CM_10301]）添加协议头，生成序列化数据。⌋

从E2E保护角度看，该序列化数据包含非保护部分和待保护部分（参见[PRS_E2E_USE_00236]和[PRS_E2E_USE_00741]）。

##### 7.3.2.2 负载的E2E保护

**[SWS_CM_90479]{草案}** ⌈对于受 `E2E` 保护的方法请求，应根据[RS_E2E_08541]、[PRS_E2E_00323]和[PRS_E2E_00828]对待保护的序列化数据（作为 `serializedData` 参数传递给 `E2E_protect`）调用 `E2E_protect`。⌋

**[SWS_CM_10463]{草案}** ⌈对于受 `E2E` 保护的方法请求，应将 `End2EndMethodProtectionProps.dataId` 作为 `dataID` 参数传递给 `E2E_protect`。⌋

**[SWS_CM_90486]{草案}** ⌈对于使用 `P04m` 或 `P07m` 配置文件的受 `E2E` 保护方法请求，应将`End2EndMethodProtectionProps.sourceId` 作为 `sourceID` 参数传递给 `E2E_protect`。⌋

**[SWS_CM_90487]{草案}** ⌈对于使用 `P04m` 或 `P07m` 配置文件的受 `E2E` 保护方法请求，应将 `STD_MESSAGETYPE_REQUEST(0)` 作为 `messageType` 参数传递给 `E2E_protect`。⌋

**[SWS_CM_90488]{草案}** ⌈对于使用 `P04m` 或 `P07m` 配置文件的受 `E2E` 保护方法请求，应将 `STD_MESSAGERESULT_OK(0)` 作为 `messageResult` 参数传递给 `E2E_protect`。⌋

**[SWS_CM_10464]{草案}** ⌈对于受 `E2E` 保护的方法请求，应将 `E2E header` 添加到消息中。如果相应网络绑定的协议规范对 `E2E header` 的位置有约束（例如，`SOME/IP` 网络绑定情况下的[PRS_SOMEIP_00941]），则应遵守这些约束。⌋

#### 7.3.3 服务方法请求的 E2E 校验（服务端）

**[SWS_CM_10466]{草案}** ⌈对于受 `E2E` 保护的方法请求，如果 `MethodCallProcessingMode` 设置为 `kEventSingleThread`，则 `E2E` 校验应在 `ServiceSkeleton` 的消息接收上下文中执行。⌋

**[SWS_CM_10468]{草案}** ⌈对于受 `E2E` 保护的方法请求，如果 `MethodCallProcessingMode` 设置为`kPoll`，则 `E2E` 校验应在 `ServiceSkeleton` 的 `ProcessNextMethodCall` 上下文中执行。⌋

**[SWS_CM_10467]{草案}** ⌈如果向使用 `E2E` 保护方法的服务的 `ServiceSkeleton` 命名构造函数传递了 `kEvent` 的 `MethodCallProcessingMode`（参见[SWS_CM_10436]或[SWS_CM_10435]），则在 `Create()` 命名构造函数的结果中应返回错误码 `kWrongMethodCallProcessingMode`。如果启用了日志记录，应记录该错误。⌋

**注：受E2E保护的方法不支持设置为 `kEvent` 的方法调用处理模式。**

图7.8和图7.9展示了服务器端方法请求 E2E 校验涉及的组件交互概览。

**图7.8：服务器端方法请求E2E校验的组件交互（轮询模式）**

**图7.9：服务器端方法请求E2E校验的组件交互（事件驱动模式）**

##### 7.3.3.1 负载的E2E校验

对于受E2E保护的方法请求，如果存在序列化数据，则执行以下步骤：

**[SWS_CM_90459]{草案}** ⌈对于给定的受 `E2E` 保护方法请求，应处理方法请求序列化数据中的非 `E2E header` （如果存在）。⌋

**[SWS_CM_90480]{草案}** ⌈对于给定的受 `E2E` 保护方法请求，应根据[RS_E2E_08541]、[PRS_E2E_00323]和[PRS_E2E_00828]对受保护的序列化数据（作为`serializedData`参数传递给`E2E_check`）调用`E2E_check`。⌋

**[SWS_CM_90460]{草案}** ⌈对于给定的受 `E2E` 保护方法请求，应将 `End2EndMethodProtectionProps.dataId` 作为 `dataID` 参数传递给 `E2E_check`。⌋

**[SWS_CM_90489]{草案}** ⌈对于使用 `P04m` 或 `P07m` 配置文件的受E2E保护方法请求，应将一个用于存储`sourceID`的变量引用作为`sourceID`参数传递给`E2E_check`。`E2E_check`应将 `E2E header` 中包含的E2E源ID提取到该变量中。提取的`sourceID`应被存储，以便在后续响应负载的 `E2E` 保护中使用（参见[SWS_CM_90492]）。⌋

**[SWS_CM_90490]{草案}** ⌈对于使用 `P04m` 或 `P07m` 配置文件的受E2E保护方法请求，应将`STD_MESSAGETYPE_REQUEST (0)`作为`messageType`参数传递给`E2E_protect`。⌋

**[SWS_CM_90491]{草案}** ⌈对于使用 `P04m` 或 `P07m` 配置文件的受 `E2E` 保护方法请求，应将`STD_MESSAGERESULT_OK (0)`作为`messageResult`参数传递给`E2E_protect`。⌋

**[SWS_CM_90461]{草案}** ⌈作为返回值，对于给定的受E2E保护方法请求，`E2E_check`应提供一个结果（根据文献[4]的[PRS_E2E_00322]定义的`e2eResult`），包含`SMState`（根据文献[4]的[PRS_E2E_00322]定义的`e2eState`）和`ProfileCheckStatus`（根据文献[4]的[PRS_E2E_00322]定义的`e2eStatus`）两个元素。⌋

**[SWS_CM_90462]{草案}** ⌈对于给定的受 `E2E` 保护方法请求，应从序列化数据中移除 `E2E header`。⌋

##### 7.3.3.2 负载反序列化

如果`E2E_check`调用（根据[SWS_CM_90459]）表明请求消息的E2E校验成功，则继续处理请求消息。

**[SWS_CM_90463]{草案}** ⌈对于给定的受E2E保护方法请求，应根据相应网络绑定的规则（例如，SOME/IP网络绑定情况下的[SWS_CM_10304]）反序列化处理后的序列化数据，生成方法调用的反序列化输入和输入输出参数。⌋

##### 7.3.3.3 E2E错误通知

如果`E2E_check`调用（根据[SWS_CM_90459]）表明请求消息的E2E校验失败，服务器应用可通过E2E错误处理程序获得通知。

**[SWS_CM_10470]{草案} E2E错误处理程序 - 存在性** ⌈`ServiceSkeleton`应提供一个虚拟的`E2EErrorHandler`方法，参数包括错误码、数据ID和消息计数器。该`E2EErrorHandler`函数应具有空实现，可由实际的`ServiceSkeleton`实现重写。`E2EErrorHandler`实现无需支持重入。⌋

```C++
virtual void E2EErrorHandler(
    ara::com::e2e::E2EErrorCode errorCode,
    ara::com::e2e::DataID dataID,
    ara::com::e2e::MessageCounter messageCounter
) {};
```

**[SWS_CM_90464]{草案} E2E错误处理程序 - 调用** ⌈当`E2E_check`报告E2E错误时，通信管理软件应在单独的线程中调用`E2EErrorHandler`。⌋

**[SWS_CM_10471]{草案} E2E错误处理程序 - 调用参数** ⌈当有新的请求消息可用时，调用`E2EErrorHandler`应使用以下参数：
- `errorCode`应设置为[SWS_CM_90411]中获得的`ProfileCheckStatus`
- `dataID`应设置为`End2EndMethodProtectionProps.dataId`
- `messageCounter`应设置为接收到的请求消息的E2E计数器

⌋

**[SWS_CM_90465]{草案} E2E错误处理程序 - 调用参数** ⌈当没有新的请求消息可用时，调用`E2EErrorHandler`应使用以下参数：
- `errorCode`应设置为`kNotAvailable`
- `dataID`应设置为0
- `messageCounter`应设置为0

⌋

#### 7.3.4 服务方法响应的E2E保护（服务器）

**[SWS_CM_90481]{草案}** ⌈对于受E2E保护的方法，响应消息的E2E保护应在服务方法执行完成后（根据[SWS_CM_90480]E2E校验成功的情况），或E2E错误处理程序执行完成后（根据[SWS_CM_90480]E2E校验失败的情况）执行。⌋

图7.10展示了服务器端方法响应E2E保护涉及的组件交互概览。

**图7.10：服务器端方法响应E2E保护的组件交互**

##### 7.3.4.1 E2E错误响应负载序列化

**[SWS_CM_10472]{草案} E2E错误响应** ⌈如果`E2E_check`（根据[SWS_CM_90480]）报告E2E错误，应根据所使用的网络绑定（例如，SOME/IP情况下的[SWS_CM_10312]）向客户端发送错误响应消息。⌋

**[SWS_CM_90466]{草案} E2E错误响应的负载** ⌈该错误响应消息的负载应包含一个错误域为`ara::com::e2e::E2EErrorDomain`的`ara::core::ErrorCode`。该`ara::core::ErrorCode`的值应设置为[SWS_CM_90421]中规定的`E2E_check`对应的错误值。该错误码的序列化以及可能的协议头添加应根据所使用的网络绑定执行（例如，SOME/IP情况下的[SWS_CM_10312]和[SWS_CM_10428]）。⌋

##### 7.3.4.2 响应负载序列化

**[SWS_CM_90467]{草案} 正常响应或应用错误响应的负载** ⌈对于受E2E保护的方法，应序列化方法的输入输出和输出参数或应用错误，并根据相应网络绑定的规则（例如，SOME/IP网络绑定情况下的[SWS_CM_10312]）添加协议头，生成序列化数据。⌋

从E2E通信保护角度看，该序列化数据包含非保护部分和待保护部分（参见[PRS_E2E_USE_00236]和[PRS_E2E_USE_00741]）。

##### 7.3.4.3 响应负载的E2E保护

**[SWS_CM_90468]{草案}** ⌈对于受E2E保护的方法响应，应根据[RS_E2E_08541]、[PRS_E2E_00323]和[PRS_E2E_00828]对待保护的序列化数据（作为`serializedData`参数传递给`E2E_protect`）调用`E2E_protect`。⌋

**[SWS_CM_10469]{草案}** ⌈对于受E2E保护的方法响应，应将`End2EndMethodProtectionProps.dataId`作为`dataID`参数传递给`E2E_protect`。⌋

**注：** 这与相应方法请求中包含的`dataID`相同。

**[SWS_CM_90492]{草案}** ⌈对于使用 `P04m` 或 `P07m` 配置文件的受E2E保护方法响应，应将存储的`sourceID`（根据[SWS_CM_90489]提取）作为`sourceID`参数传递给`E2E_protect`。⌋

**[SWS_CM_90493]{草案}** ⌈对于使用 `P04m` 或 `P07m` 配置文件的受E2E保护方法响应，应将`STD_MESSAGETYPE_RESPONSE (1)`作为`messageType`参数传递给`E2E_protect`。⌋

**[SWS_CM_90494]{草案}** ⌈对于使用 `P04m` 或 `P07m` 配置文件的受E2E保护方法响应，在正常响应情况下（即既不是应用错误响应消息也不是E2E错误响应消息），应将`STD_MESSAGERESULT_OK (0)`作为`messageResult`参数传递给`E2E_protect`。⌋

**[SWS_CM_90495]{草案}** ⌈对于使用 `P04m` 或 `P07m` 配置文件的受E2E保护方法响应，在错误响应情况下（即无论是应用错误响应消息还是E2E错误响应消息），应将`STD_MESSAGERESULT_ERROR (1)`作为`messageResult`参数传递给`E2E_protect`。⌋

**[SWS_CM_90469]{草案}** ⌈对于受E2E保护的方法响应，应将相应方法请求中包含的E2E计数器作为调用`E2E_protect`时的E2E计数器。⌋

**注：** 方法响应携带与相应方法请求相同的`dataID`和E2E计数器，以简化多客户端场景并允许客户端监控E2E计数器。

**[SWS_CM_90470]{草案}** ⌈对于受E2E保护的方法响应，应将E2E保护头添加到消息中。如果相应网络绑定的协议规范对E2E保护头的位置有约束（例如，SOME/IP网络绑定情况下的[PRS_SOMEIP_00941]），则应遵守这些约束。⌋

#### 7.3.5 服务方法响应的E2E校验（客户端）

**[SWS_CM_90471]{草案}** ⌈对于受E2E保护的方法响应，E2E校验应在`ServiceProxy`的消息接收上下文中执行。⌋

图7.11展示了客户端端方法响应E2E校验涉及的组件交互概览。

**图7.11：客户端端方法响应E2E校验的组件交互**

##### 7.3.5.1 负载的E2E校验

对于受E2E保护的方法响应，如果存在序列化数据，则执行以下步骤：

**[SWS_CM_90472]{草案}** ⌈对于给定的受E2E保护方法响应，应处理方法响应序列化数据中的非E2E保护头（如果存在）。⌋

**[SWS_CM_90473]{草案}** ⌈对于给定的受E2E保护方法响应，应根据[RS_E2E_08541]、[PRS_E2E_00323]和[PRS_E2E_00828]对受保护的序列化数据（作为`serializedData`参数传递给`E2E_check`）调用`E2E_check`。⌋

**[SWS_CM_90474]{草案}** ⌈对于给定的受E2E保护方法响应，应将`End2EndMethodProtectionProps.dataId`作为`dataID`参数传递给`E2E_check`。⌋

**[SWS_CM_10465]{草案}** ⌈受E2E保护的方法响应，其响应消息应携带与请求消息相同的E2E计数器值。如果E2E计数器不同，应丢弃该响应消息（不进行任何进一步处理）。⌋

**实现提示：** E2E计数器可从`E2E_Protect()`/`E2E_Check()`函数的结果状态中提取。

**[SWS_CM_90496]{草案}** ⌈对于使用 `P04m` 或 `P07m` 配置文件的受E2E保护方法响应，应将`End2EndMethodProtectionProps.sourceId`作为`sourceID`参数传递给`E2E_check`。⌋

**[SWS_CM_90497]{草案}** ⌈对于使用 `P04m` 或 `P07m` 配置文件的受E2E保护方法响应，应将`STD_MESSAGETYPE_RESPONSE (1)`作为`messageType`参数传递给`E2E_check`。⌋

**[SWS_CM_90498]{草案}** ⌈对于使用 `P04m` 或 `P07m` 配置文件的受E2E保护方法响应，在正常响应情况下（即既不是应用错误响应消息也不是E2E错误响应消息），应将`STD_MESSAGERESULT_OK (0)`作为`messageResult`参数传递给`E2E_check`。⌋

**[SWS_CM_90499]{草案}** ⌈对于使用 `P04m` 或 `P07m` 配置文件的受E2E保护方法响应，在错误响应情况下（即无论是应用错误响应消息还是E2E错误响应消息），应将`STD_MESSAGERESULT_ERROR (1)`作为`messageResult`参数传递给`E2E_check`。⌋

**[SWS_CM_90478]{草案}** ⌈作为返回值，对于给定的受E2E保护方法响应，`E2E_check`应提供一个结果（根据文献[4]的[PRS_E2E_00322]定义的`e2eResult`），包含`SMState`（根据文献[4]的[PRS_E2E_00322]定义的`e2eState`）和`ProfileCheckStatus`（根据文献[4]的[PRS_E2E_00322]定义的`e2eStatus`）两个元素。⌋

**[SWS_CM_90482]{草案}** ⌈应使用`E2E_check`根据[SWS_CM_90478]提供的结果中的`SMState`元素，更新/覆盖特定`ServiceProxy`类的特定方法类中的全局`SMState`。⌋

**[SWS_CM_90475]{草案}** ⌈对于给定的受E2E保护方法响应，应从序列化数据中移除E2E保护头。⌋

##### 7.3.5.2 负载反序列化

如果`E2E_check`调用（根据[SWS_CM_90473]）表明响应消息的E2E校验成功，则继续处理响应消息。

**[SWS_CM_90476]{草案}** ⌈对于给定的受E2E保护方法响应，应根据相应网络绑定的规则（例如，SOME/IP网络绑定情况下的[SWS_CM_10316]和[SWS_CM_10429]）反序列化处理后的序列化数据，生成方法调用的反序列化输入输出和输出参数，或反序列化的应用错误。⌋

**[SWS_CM_10473]{草案} E2E错误响应的处理** ⌈（因服务器端检测到请求中的E2E错误而发送的）E2E错误响应消息的处理方式，应与根据所使用网络绑定接收和处理任何其他错误响应消息的方式相同（例如，SOME/IP网络绑定情况下的[SWS_CM_10429]）。⌋

##### 7.3.5.3 E2E错误通知

如果`E2E_check`调用（根据[SWS_CM_90473]）表明响应消息的E2E校验失败，客户端应用应通过以下方式获得通知：

**[SWS_CM_90477]{草案} E2E错误返回码** ⌈对于给定的受E2E保护方法响应，如果E2E校验失败，应根据[SWS_CM_90421]构造一个错误域为`ara::com::e2e::E2EErrorDomain`、值为[SWS_CM_90478]中获得的`ProfileCheckStatus`的`ara::core::ErrorCode`。该`ara::core::ErrorCode`应作为参数传递给`Promise`的`SetError()`方法。⌋

正常响应和应用错误响应的处理（根据[SWS_CM_90476]），结合E2E错误响应的处理（根据[SWS_CM_10473]），以及响应消息中检测到的E2E错误的显式通知（根据[SWS_CM_90477]），最终将生成一个`ara::core::Result`，包含以下三种情况之一：
- 无任何错误时，包含服务器操作的正确输出
- 如果客户端-服务器操作抛出了其配置的可能应用错误之一，且请求消息和响应消息中均未检测到E2E错误，则包含错误域为`ApApplicationError.errorDomain`、值为所抛出`ApApplicationError`错误码的`ara::core::ErrorCode`
- 如果服务器端检测到请求消息中的E2E错误，且客户端端未检测到响应消息中的E2E错误，则包含错误域为`ara::com::e2e::E2EErrorDomain`、值为服务器端`E2E_check`调用结果的`ProfileCheckStatus`的`ara::core::ErrorCode`
- 如果客户端端检测到响应消息中的E2E错误，则包含错误域为`ara::com::e2e::E2EErrorDomain`、值为客户端端`E2E_check`调用结果的`ProfileCheckStatus`的`ara::core::ErrorCode`

**[SWS_CM_90483]{草案}** ⌈应为特定`ServiceProxy`类的每个方法类提供`GetSMState`方法。⌋

**[SWS_CM_90484]{草案}** ⌈`GetSMState`方法应提供对特定方法类全局`SMState`的访问，该状态由上次接收方法响应时最后一次执行的`E2E_check`函数确定（参见[SWS_CM_90482]）。⌋

```C++

ara::com::e2e::SMState GetSMState() const noexcept;
```

#### 7.3.6 超时监控

`ara::com`不支持方法调用的任何超时监控。丢失的响应消息可能会导致某些`ara::core::Future`方法（如`wait()`）永久阻塞。在E2E场景下，强烈建议自适应应用实现超时监控，例如使用`ara::phm::SupervisedEntity`的`ReportCheckpoint()`方法，或`ara::core::Future`的`wait_for()`、`wait_until()`或`is_ready()`方法。

---


