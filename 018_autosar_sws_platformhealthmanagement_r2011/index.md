# AUTOSAR_SWS_PlatformHealthManagement_R2011


<!--more-->

# AUTOSAR_SWS_PlatformHealthManagement

----

## 7 功能规范

### 7.1 概述

平台健康管理根据时序约束（存活监督和截止时间监督）、逻辑程序序列（逻辑监督）以及平台健康（健康通道监督）来监控应用程序。在检测到故障时，平台健康管理会通知状态管理。作为平台的协调者，状态管理可以决定如何处理错误并触发适当的恢复操作。

平台健康管理还具有与硬件看门狗的接口，在发生关键故障（仅通知状态管理不足以应对）时，可以触发看门狗反应。

平台健康管理的所有算法和过程在 AUTOSAR Foundation 文件 [4] 中均有描述，此处不再赘述：下面仅展示 AUTOSAR Adaptive 的具体特性，包括与其他功能集群的接口。

健康管理与其他功能集群的接口仅为参考性质，并未标准化。

### 7.2 受监督实体的监督

状态管理通过功能组 [10] 协调平台。在一个功能组内，可能有多个进程在运行。

平台健康管理监控受监督实体。每个受监督实体映射到一个进程的全部或部分。只要对应的进程处于活动状态，监控就处于活动状态。

监督的细节在 [4] 中描述。受监督实体实例的监督结果反映在本地监督状态中。

功能组内本地监督的状态汇总为对应的全局监督状态。

**[SWS_PHM_00100] (草案) 全局监督范围** —— 平台健康管理应为一个功能组支持一个或少数几个全局监督。

### 7.3 健康通道监督

通过健康通道监督，系统集成者可以将外部监督结果挂钩到平台健康管理。外部监督可以是 RAM 测试、ROM 测试、内核状态、电压监控等例程。外部监督执行监控和去抖动处理。确定的结果根据可能的健康状态值进行分类，并发送给平台健康管理。

#### 健康通道可以是：
- 受监督软件的全局监督状态。
- 环境监控算法的结果，例如电压监控、温度监控。
- 内存完整性测试例程的结果，例如 RAM 测试、ROM 测试。
- 操作系统或内核的状态，例如 OS 状态、内核状态。
- 另一个平台实例、虚拟机或 ECU 的状态。

各种外部监控例程应将其结果或状态以定义的健康状态形式报告给平台健康管理。健康通道的健康状态是健康通道向平台健康管理提供的信息的抽象格式。两个不同的健康通道可以使用相同的健康状态名称来表示其结果，例如高、低、正常。

如果需要对确定的健康状态做出反应，平台健康管理会将该状态报告给状态管理。

#### 7.3.1 初始化后的健康状态

初始化后的健康状态由配置容器 `HealthStatusInitValue` 控制。该参数可以为每个健康通道在配置中设置一次。

**[SWS_PHM_00010] (草案) 未初始化的健康通道** —— 如果容器 `HealthStatusInitValue` 不存在或者健康通道尚未具有初始值，则平台健康管理应将相应的健康状态视为未定义，并且在相应健康通道首次更新之前不使用它。

#### 7.3.2 健康通道的配置

一个健康通道具有以下配置选项：

1. 名称：全局唯一名称标识符，供应用程序使用。
2. ID：全局唯一标识符（数字）。
3. `HealthStatusInitValue`：对应健康状态的初始值。
4. 状态名称：应用程序使用，在健康通道内唯一。
5. 状态 ID：健康状态标识符，在健康通道内唯一。

<center>图 7.1：健康通道配置</center>

### 7.4 监督模式

监督模式表示机器或一组应用程序的整体状态。它由一个元组 <机器状态, 功能组状态> 标识。平台健康管理使用状态管理接口字段参数 `FunctionGroupState` 来获知任一状态的变化。

### 7.5 恢复操作

平台健康管理的职责是监控平台上的安全相关进程，并将检测到的故障报告给状态管理。如果在状态管理中检测到故障，平台健康管理可以通过硬件看门狗触发反应。

<center>图 7.2：平台健康管理及其环境</center>

#### 7.5.1 通知状态管理

平台健康管理对受监督实体的故障进行去抖动处理，参见 [4] 中的状态 "failed"。去抖动后，需要执行恢复操作。因此，平台健康管理会通知状态管理。状态管理作为平台的协调者，可以决定如何处理检测到的故障并触发相应的恢复操作。多数情况下，这可能包括将有故障的功能组切换到另一个状态。

根据 ISO 26262，必须确保在发生安全相关故障后触发反应。因此，平台健康管理必须确保状态管理接收到故障通知。平台健康管理会监控 `RecoveryHandler` 的返回，并设置一个可配置的超时时间。如果在可配置的重试次数后状态管理仍未从 `RecoveryHandler` 正常返回，则 PHM 将通过错误触发或停止触发受服务的看门狗来采取自身的应对措施。

**[SWS_PHM_00101] (草案) 因监督失败通知状态管理** —— 如果通过 `RecoveryNotificationToPPortPrototypeMapping` 映射的全局监督状态切换到 `GLOBAL_STATUS_STOPPED`，平台健康管理应通过方法 `RecoveryHandler` 通知状态管理。参数 `executionError` 应包含相应的功能组和当前的 `ProcessExecutionError`。参数 `supervision` 应包含导致状态转换为 `GLOBAL_STATUS_STOPPED` 的监督类型。

*注：全局监督对应于整个或部分功能组，即为每个全局监督始终报告相同的功能组。`ProcessExecutionError` 在 `StartupConfig` 中定义，因此 `executionError.executionError` 取决于当前使用的 `StartupConfig`。*

**[SWS_PHM_00102] (草案) 因健康状态通知状态管理** —— 如果健康通道的健康状态发生切换，并且需要状态管理做出反应（即对于相应的 `PhmHealthChannelStatus.statusId`，`PhmHealthChannelStatus.triggersRecoveryNotification` 等于 `true`），则平台健康管理应通过方法 `RecoveryHandler` 通知状态管理。参数 `healthStatusId` 应从 `ReportHealthStatus` 方法传递过来。

这意味着是否需要反应的信息必须为平台健康管理进行配置。

**[SWS_PHM_00103] (草案) 通知状态管理的超时监控** —— 如果在向状态管理发送故障通知后，在 `RecoveryNotification.recoveryNotificationTimeout` 之前未收到状态管理的确认，平台健康管理应重新发送通知。此过程最多重复 `RecoveryNotification.recoveryNotificationRetry` 次。

**[SWS_PHM_00104] (草案) 通知状态管理超时的反应** —— 如果在连续 `RecoveryNotification.recoveryNotificationRetry` 次通知后，在 `RecoveryNotification.recoveryNotificationTimeout` 之前仍未收到状态管理的确认，平台健康管理应错误触发或停止触发受服务的看门狗。

**[SWS_PHM_01147] (草案) 启用处理器** —— 当调用 `Offer` 时，平台健康管理应启用对 `RecoveryHandler` 的潜在调用。

**[SWS_PHM_01148] (草案) 禁用处理器** —— 当调用 `StopOffer` 时，平台健康管理应禁用对 `RecoveryHandler` 的调用。

#### 7.5.2 通过看门狗的恢复操作

平台健康管理拥有与硬件看门狗的唯一接口。因此，看门狗监督平台健康管理，而 PHM 可以通过停止触发或发送错误触发来启动看门狗的反应。由于这种反应通常意味着机器复位，因此会影响所有功能，应仅在必要时作为最后手段使用，以确保免于干扰。需要看门狗反应的故障是状态管理和执行管理中的监督故障，因为在这些情况下，第 7.5.1 节描述的通过状态管理的恢复操作不可行。

**[SWS_PHM_00105] (草案) 执行管理或状态管理故障的恢复操作** —— 如果与状态管理或执行管理相对应的全局监督状态切换到 `GLOBAL_STATUS_STOPPED`，平台健康管理应错误触发或停止触发受服务的看门狗。

#### 7.5.3 配置参数

平台健康管理内恢复操作的配置有两个参数：

1. `recoveryNotificationTimeout`：平台健康管理在发送通知后等待状态管理确认的最大可接受时间。
2. `recoveryNotificationRetry`：平台健康管理在触发看门狗反应之前尝试向状态管理重新发送通知的次数。

### 7.6 多进程与多实例

在应用程序部署阶段，单个受监督实体或单个健康通道可能会被实例化多次：例如，当代表受监督实体或健康通道的同一个 C++ 对象类在代码中被显式实例化时，或者当包含受监督实体或健康通道的同一个可执行文件被启动/运行多次时。在这种情况下，每个受监督实体的实例都会被单独监督，每个实例都会生成一个本地监督状态实例。

为了标识受监督实体或健康通道的相关实例，需要使用特定实例的元模型名称引用。这可以通过调用函数 `ara::core::InstanceSpecifier(StringView metaModelIdentifier);` 来实现，该函数在 AUTOSAR Adaptive Platform Core Types 规范 [8] 中定义。

参数 `metaModelIdentifier` 实际上是一个与元模型相关的字符串，语法为 `<shortname context1>/<shortname context2>/.../<shortname contextN>/<shortname of PortPrototype>`，其中 PortPrototype 的短名称对应于元模型中代表受监督实体或健康通道的端口。

<center>图 7.3：同一受监督实体的多实例示例</center>

图 7.3 显示了一个属于唯一 SW 组件（示例中的 SWComponent_X）的单个受监督实体（称为 SE_A）的示例。SWComponent_X 在同一进程（进程 1）中被显式实例化两次，并在另一个不同的进程/应用程序（进程 2）中被实例化一次。在这种情况下，会创建三个代表受监督实体的端口原型实例。

<center>图 7.4：实例说明符和上下文的示例</center>

图 7.4 展示了一个更复杂的示例，用于演示多个上下文层级。图中，所有显示的 RPortPrototype 都引用 PHM 受监督实体接口。`<shortname context1>` 是可执行文件的短名称。其他上下文，即 `<shortname context2>` 到 `<shortname contextN>`，代表不同的组合层级。有关实例说明符语法的详细信息，请参见 [8]。

使用图 7.4 中给出的命名和组合，可以得到以下可能的实例说明符字符串：

- `exe0/RootSWCP_0/Comp_Lvl1/Comp_Lvl2/SwComponentPrototype_0/RPort_0`
- `exe0/RootSWCP_0/Comp_Lvl1/Comp_Lvl1/SwComponentPrototype_0/RPort_1`
- `exe0/RootSWCP_0/Comp_Lvl1/Comp_Lvl2/SwComponentPrototype_1/Rport_y`
- `exe0/RootSWCP_0/Comp_Lvl1/Comp_Lvl1/SwComponentPrototype_2/RPort_0`
- `exe0/RootSWCP_0/Comp_Lvl1/Comp_Lvl2/SwComponentPrototype_2/RPort_1`
- `exe0/RootSWCP_0/Comp_Lvl1/SwComponentPrototype_3/RPort_0`
- `exe0/RootSWCP_0/Comp_Lvl1/SwComponentPrototypes_3/RPort_1`

### 7.7 功能集群生命周期

#### 7.7.1 启动

平台健康管理功能集群的启动行为将在未来版本中指定。

#### 7.7.2 关闭

使用 `ara::core::Deinitialize`，所有具有直接 ARA 接口的功能集群都可以关闭。当平台健康管理功能集群关闭时，所有已配置的监督都将终止。集成者有责任正确使用关闭机制。有关确保安全执行的详细信息，请参见 [11]。

----

