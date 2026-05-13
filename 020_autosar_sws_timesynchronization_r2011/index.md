# AUTOSAR_SWS_TimeSynchronization_R2011


<!--more-->

# AUTOSAR_SWS_TimeSynchronization

----

## 7 功能规范

功能行为在以下特定上下文中描述：

- 启动行为
- 关闭行为
- 构造行为（初始化）
- 正常运行
- 错误处理
- 错误分类
- 版本检查

### 7.1 TS 概述

对于自适应平台，考虑了三种不同的技术来满足此类时间同步要求。这些技术是：

- 经典平台的 StbM
- 库 chrono - 即 std::chrono (C++11) 或 boost::chrono [8]
- Time posix 接口 [9]

下表显示了此 API 提供给应用程序的接口及其在 StbM 中的等效接口。


| 时间同步 API - AP | StbM - CP |
| --- | --- |
| GetCurrentTime | StbM_GetCurrentTime |
| SetTime | StbM_SetGlobalTime |
| updateTime | StbM_UpdateGlobalTime |
| setUserData | StbM_SetUserData |
| setOffset | StbM_SetOffset |
| getOffset | StbM_GetOffset |
| getRateDeviation | StbM_GetRateDeviation |
| setRateCorrection | StbM_SetRateCorrection |
| timeLeap (TimeBase Status 类的属性) | StbM_GetTimeLeap |
| getTimeBaseStatus | StbM_GetTimeBaseStatus |
| 不适用 | StbM_StartTimer |

表 7.1：TS 和 STBM 之间的接口比较

#### 7.1.1 每个时基的基本功能

每个时基必须提供一组最低限度的功能，如下所列：

- 提供获取当前时间戳并创建其参数快照的可能性

本章简要描述这些功能。有关如何使用这些核心方法以及其确切行为的详细信息在第 8 章中给出。

##### 7.1.1.1 时基状态

此 TimeBaseStatus 是其相关的时基资源的所有信息的快照，例如状态标志、TBR 已更新的次数、时间跳变信息（可能在时基资源的最后一次同步期间生成）等。

##### 7.1.1.2 速率偏差

应用程序对于可接受的时间漂移值会有不同的阈值。因此需要一种方式让应用程序能够访问此信息。

[SWS_TS_00202]{草案} [ara::tsync::SynchronizedTimeBaseConsumer::GetRateDeviation 应返回其 TBR 相对于其同步的时间源的已计算速率偏差。如果尚未计算速率偏差，则应返回初始速率偏差 1。]()

##### 7.1.1.3 时钟时间值

读取时钟的时间值很可能是与 TS 交互的应用程序执行的最常见的操作。

为了确保时间值的类型安全处理，时间点作为 std::chrono 结构提供。

关于如何实现这一点的更详细信息在后续章节和第 8 章中给出。

#### 7.1.2 TBR 的状态标志

时间同步定义了一组状态标志，用于表示 TBR 的特定状态条件。应用程序可以通过 GetTimeWithStatus 查询状态标志。

同步状态 GetSynchronizationStatus 包括：

- `kNotSynchronizedUntilStartup`：指示时基是否在启动前同步到其对应的 TBR（初始状态）
- `kTimeOut`：指示时基到其对应 TBR 的同步是否丢失或延迟。
- `kSynchronized`：指示对应 TBR 的时基是否已至少成功同步一次到其时间源。
- `kSynchronToGateway`：指示对应的 TBR 更新是否基于全局时间主机下的时间网关。

通过 GetTimeWithStatus 自上次状态请求以来是否发生时间跳变可以通过 GetLeapJump 获取：

- `kTimeLeapNone`：指示没有发生时间跳变
- `kTimeLeapFuture`：指示时间是否向未来跳变。
- `kTimeLeapPast`：指示时间是否向过去跳变。

#### 7.1.3 时间同步和协议

时间同步机制和协议（即 [10]）超出本文档范围，协议规范请参考 PRS（见 [1]）。

### 7.2 功能集群生命周期

#### 7.2.1 启动

本章描述了由控制时基资源的实体执行的必要初始化，以使 TS 模块为正常运行做好准备。初始化后，该模块预期向应用程序提供所有同步时间服务。

##### 7.2.1.1 默认值

当系统启动时，TBR 必须设置为已知的默认值，以便其行为被明确定义。

[SWS_TS_00007]{草案} [时基资源的特性应初始化如下：

- 活动状态标志应无效。
- 时钟更新计数器应设置为零。
- 用户数据应被删除。
- 时间跳变信息应重置。

]

##### 7.2.2 关闭

[SWS_TS_00212]{草案} [对于每个配置为时间主机的时基，如果配置参数要求持久存储，则全局时间的值应存储到持久内存中（通过 'TimeBaseProviderToPersistencyMapping.timeBaseProvider'）（见 [11]）。]

### 7.3 正常运行

#### 7.3.1 引言

一个全局时间网络由一个时间主机和至少一个时间从机组成。对于每个时间域，时间主机通过时间同步消息将全局时基分发给连接的时间从机。时间从机考虑到发送端的时间戳和自身生成的接收端时间戳，对接收到的全局时基进行校正。

从机时基的本地时间将被自主维护，并在从其关联的主机时基接收到新的时间值时进行更新。

图 7.1：全局时基分发。

##### 7.3.1.1 时基表现形式

从时间域的角度来看，时基分为同步时基和偏移时基。

同步时基和偏移时基的数量不受 TS 功能限制，而是由系统需要满足的功能需求决定（即 TS 未定义系统中偏移/同步时基标识符的数量限制）。

#### 7.3.2 时基资源的角色

##### 7.3.2.1 全局时间主机

一个 TBR 可以充当全局时间主机，在这种情况下，它是给定时间值的系统范围来源，该时间值随后通过网络分发到时间从机。

##### 7.3.2.2 时间从机

在时间从机的角色中，TBR 将其内部维护的本地时间更新为由相应 TSP 模块提供的全局时基的值。

#### 7.3.3 时基资源

##### 7.3.3.1 从机时基

[SWS_TS_00139]{草案} [仅当 timeLeapFutureThreshold 非零且 ara::tsync::SynchronizationStatus 不等于 kNotSynchronizedUntilStartup 时，才应启用对未来时间跳变的监视。]

[SWS_TS_00140]{草案} [仅当 timeLeapPastThreshold 非零且 ara::tsync::SynchronizationStatus 不等于 kNotSynchronizedUntilStartup 时，才应启用对过去时间跳变的监视。]

[SWS_TS_00141]{草案} [应在每次与主机时钟成功同步时执行时间跳变检查，但仅在时钟已同步一次之后（ara::tsync::SynchronizationStatus 不等于 kNotSynchronizedUntilStartup）。]

[SWS_TS_00027]{草案} [如果重新同步所做的调整超过了指定的阈值，则应设置相应的 ara::tsync::LeapJump 状态：
- `kTimeLeapNone`：如果没有发生时间跳变。
- `kTimeLeapFuture`：如果时间向未来的跳变大于 timeLeapFutureThreshold。
- `kTimeLeapPast`：如果时间向过去的跳变大于 timeLeapPastThreshold。]

[SWS_TS_00026]{草案} [（ara::tsync::LeapJump 的初始值应为 kTimeLeapNone。]

[SWS_TS_00028]{草案} [如果连续 timeLeapHealingCounter 次同步的时间跳变都低于时间跳变未来和过去阈值，则活动时间跳变状态（ara::tsync::LeapJump）应设置为 kTimeLeapNone。]

[SWS_TS_00030]{草案} [每个 ara::tsync::SynchronizedTimeBaseConsumer 实例应通过测量自上次更新以来的时间以及 syncLossTimeout 中指定的超时持续时间，独立监视同步超时。]

[SWS_TS_00032]{草案} [如果监视到超时（参考 [SWS_TS_00030]），则 ara::tsync::SynchronizationStatus 应设置为 kTimeOut。]

[SWS_TS_00011]{草案} [在成功更新时基且 SYNC_TO_GATEWAY 位被设置的情况下，ara::tsync::SynchronizationStatus 应设置为 kSynchToGateway。]

[SWS_TS_00033]{草案} [在成功更新时基且 SYNC_TO_GATEWAY 位未被设置的情况下，ara::tsync::SynchronizationStatus 应设置为 kSynchronized。]

#### 7.3.4 立即时间同步

所有 TSP 模块在处理总线特定的时间同步协议（即总线上 Timesync 消息的自主传输）方面独立于 TS 工作。

时间信息从 TSP 传递到 TBR。此类 TSP 与 TBR 交互的实现细节及范围不在本规范内，协议规范请参考 [1]）。

#### 7.3.5 用户数据

用户数据是每个时基的一部分。用户数据由每个时基的全局时间主机设置，并作为 Timesync 消息的一部分分发。

用户数据可用于表征时基，例如，关于底层时钟源的质量或时间的进程。

用户数据由一个字节向量组成。由于各种 Timesync 消息的帧格式，可能无法在每个总线系统上传输完整的向量。系统设计者有责任仅使用那些可以在车辆网络内分发的用户数据字节。

#### 7.3.6 时间校正

TS 为时间从机提供了对同步 TBR 执行速率和偏移校正以及对偏移时基执行速率校正的能力。

对于全局时间主机，TS 提供了对其时基执行速率校正的能力。

可以为每个时基单独配置时间校正。

##### 7.3.6.1 时间从机的速率校正

速率校正检测并消除时基本地实例和偏移时基的速率偏差。速率校正在一个测量范围内确定速率偏差。此速率偏差用作 TBR 在读取时基时间时（例如，在 ara::tsync::SynchronizedTimeBaseConsumer::GetCurrentTime 范围内）用于校正时间的校正因子。

[SWS_TS_00041]{草案} [如果 ara::tsync::SynchronizationStatus 设置为 kSynchronized，TBR 应执行速率校正测量以确定其速率偏差。]

[SWS_TS_00042]{草案} [TBR 应持续执行速率校正测量。一次测量的结束标志着下一次测量的开始。

测量的开始和结束总是由同步或偏移时基的时间值接收触发（并与之对齐）。]

<center>图 7.2：两次并行测量的可视化。</center>

[SWS_TS_00043]{草案} [在运行时，同步 TBR 应根据时钟 ara::core::SteadyClock 确定速率校正测量的时间跨度。]

[SWS_TS_00044]{草案} [TBR 应执行由参数 'TimeSyncCorrection.rateCorrectionsPerMeasurementDuration' 配置的尽可能多的同时速率校正测量。]

[SWS_TS_00045]{草案} [同时进行的速率校正测量应以定义的偏移量（to,）启动，以产生在测量持续时间内均匀分布的速率校正。该值将根据以下公式计算：\(\mathrm{t_{on} = n\star}\) (rateDeviationMeasurementDuration / rateCorrectionPerMeasurementDuration) （其中 'n' 是当前测量的从零开始的索引）]

[SWS_TS_00046]{草案} [在速率校正测量开始时，同步 TBR 应在 TSP 范围内拍摄时间快照 TGStart 和 TOStart。]

[SWS_TS_00047]{草案} [在速率校正测量开始时，偏移 TBR 应在 TSP 范围内拍摄以下时间快照：]

- TSStart
- TOSTart

[SWS_TS_00048]{草案} [在速率校正测量结束时，同步 TBR 应在 TSP 范围内拍摄时间快照 TGStop 和 TVStop。]

[SWS_TS_00049]{草案} [在速率校正测量结束时，偏移 TBR 应在 TSP 范围内拍摄以下时间快照：]

[SWS_TS_00050]{草案} [在速率校正测量结束时，同步 TBR 应根据以下公式计算结果校正速率 \((r_{rc})\)：

\(r_{rc} = (TG_{Stop} - TG_{Start}) / (TV_{Stop} - TV_{Start})\)]

注意：要确定结果速率偏差，必须从 \(r_{rc}\) 中减去值 1。

[SWS_TS_00051]{草案} [最后一个 \(r_{rc}\) 值将被使用，直到计算出新值。]

[SWS_TS_00052]{草案} [偏移 TBR 不应再执行另一次速率校正，因为这已经由底层 TBR 完成。]

[SWS_TS_00053]{草案} [在调用 ara::tsync::SynchronizedTimeBaseConsumer::GetRateDeviation 时，TBR 应返回计算的速率偏差（即 \(r_{rc} - 1\)）。]

[SWS_TS_00070]{草案} [如果尚未计算出速率偏差 \(r_{rc}\)，ara::tsync::SynchronizedTimeBaseConsumer::GetRateDeviation 应返回 0.0。]

[SWS_TS_00054]{草案} [如果已计算出有效的校正速率 \((r_{rc})\)，则同步 TBR 应应用速率校正。]

[SWS_TS_00071]{草案} [如果已计算出有效的校正速率 \((r_{oc})\)，则偏移 TBR 应应用速率校正。]

##### 7.3.6.2 时间消费者的偏移校正

偏移校正消除了同步时基本地实例的时间偏移。这种校正在读取当前时间时（例如，在 ara::tsync::SynchronizedTimeBaseConsumer::GetCurrentTime 范围内）发生。偏移是在 TSP 范围内同步时基本地实例时测量的。

[SWS_TS_00055]{草案} [对于同步 TBR，应在 TSP 函数范围内通过拍摄 TLSync 和 TVSync 的快照，在时基同步时测量其时基本地实例与全局时基之间的偏移。]

[SWS_TS_00056]{草案} [如果全局时基与时基本地实例之间的时间偏移的绝对值（abs(TG - TLsync)）大于或等于 'TimeSyncCorrection.offsetCorrectionJumpThreshold'，则 TBR 应根据以下公式计算其时基本地实例的校正时间 (TL)：

TL = TG + (TV - TVsync) \* rrc]

### 注：

此校正将在读取时间时进行，例如在函数 ara::tsync::SynchronizedTimeBaseConsumer::GetCurrentTime 的范围内。

### 注：

此校正将在 TBR 需要确定时基本地实例的时间时进行。

[SWS_TS_00057]{草案} [TBR 应通过临时应用一个附加速率 \((r_{oc})\) 给 \(r_{rc}\) 来校正小于 'TimeSyncCorrection.offsetCorrectionJumpThreshold' 值的全局时基与时基本地实例之间的绝对时间偏移（abs(TG - TLsync)）。该速率应持续由参数 'TimeSyncCorrection.offsetCorrectionAdaptionInterval' 定义的时间。\(r_{oc}\) 根据以下公式计算：\(r_{oc} = (TG - TLsync) / (TCorrInt) + 1\)]

[SWS_TS_00058]{草案} [如果全局时基与时基本地实例之间的绝对时间偏移（abs(TG - TLsync)）小于 'TimeSyncCorrection.offsetCorrectionJumpThreshold'，则 TBR 应在 'TimeSyncCorrection.offsetCorrectionAdaptionInterval' 期间内根据以下公式计算其时基本地实例的校正时间 (TL)：

TL = TLsync + (rc \* (TV - TVsync) \* rcc )]

### 注：

此校正将在读取时间时进行，例如在函数 ara::tsync::SynchronizedTimeBaseConsumer::GetCurrentTime 的范围内。

### 注：

此校正将在 TBR 需要确定时基本地实例的时间时进行。

[SWS_TS_00059]{草案} [如果全局时基与时基本地实例之间的绝对时间偏移（abs(TG - TL)）小于 TimeSyncCorrection.offsetCorrectionJumpThreshold，则 TBR 应在 TimeSyncCorrection.offsetCorrectionAdaptionInterval 时间段之后按照 [SWS_TS_00056] 中的规定计算其时基本地实例的校正时间 (TL)。]

[SWS_TS_00060]{草案} [如果 TimeSyncCorrection.offsetCorrectionJumpThreshold 设置为 0，则只能通过跳变校正来执行偏移校正。]

##### 7.3.6.3 全局时间主机的速率校正

全局时间主机中的速率校正可以应用于同步时基资源和偏移时基资源。

速率校正通过设置一个校正因子来应用，TBR 在通过网络传输时基时间时使用该校正因子。这独立于从机完成的速率校正。

[SWS_TS_00061]{草案} [如果 'TimeSyncCorrection.allowProviderRateCorrection' 等于 true，则调用 ara::tsync::SynchronizedTimeBaseProvider::SetRateCorrection 应设置速率校正值。否则 ara::tsync::SynchronizedTimeBaseProvider::SetRateCorrection 将不执行任何操作并返回错误 kLimitsExceeded。]

[SWS_TS_00062]{草案} [如果 allowProviderRateCorrection 等于 TRUE 并且已经通过 ara::tsync::SynchronizedTimeBaseProvider::SetRateCorrection 设置了有效的速率校正值，则 TBR 应应用速率校正。]

[SWS_TS_00063]{草案} [如果传递给 SetRateCorrection() 的速率校正参数 rateCorrection 的绝对值大于 MasterRateDeviationMax，则 SetRateCorrection() 应将实际应用的速率校正值设置为 (MasterRateDeviationMax) 或 (-MasterRateDeviationMax)（取决于 rateCorrection 的符号）。]

注：实际应用的结果速率将是传递的偏差值 + 1。如果将一时基的速率与另一时基的速率对齐，可以使用 GetRateDeviation() 并将该值作为参数传递给 ara::tsync::SynchronizedTimeBaseProvider::SetRateCorrection。

#### 7.3.7 时基消费者的通知

应用程序可以请求被通知特定 TBR 的专用事件。

##### 7.3.7.1 状态标志通知

可以通知 ara::tsync::SynchronizedTimeBaseStatus 中 StatusFlags 的更改。

[SWS_TS_00701]{草案} [如果以下任何内容发生更改，应调用通过 ara::tsync::SynchronizedTimeBaseConsumer::RegisterStatusChangeNotifier 注册的通知器：ara::tsync::SynchronizationStatus、ara::tsync::LeapJump 或用户数据。]()

##### 7.3.7.2 同步状态通知

可以通知 ara::tsync::SynchronizationStatus 中 StatusFlags 的更改（例如，如果时基处于超时状态）。

[SWS_TS_00702]{草案} [如果 ara::tsync::SynchronizationStatus 发生更改，应调用通过 ara::tsync::SynchronizedTimeBaseConsumer::RegisterSynchronizationStateChangeNotifier 注册的通知器。]()

##### 7.3.7.3 时间跳变通知

可以通知时间跳变。

[SWS_TS_00703]{草案} [如果 ara::tsync::LeapJump 发生更改，应调用通过 ara::tsync::SynchronizedTimeBaseConsumer::RegisterTimeLeapNotifier 注册的通知器。]()

#### 7.3.8 全局时间精度测量支持

为了验证每个本地时基与全局时基的精度，应为时间从机和时间网关可选地支持记录机制。原则上，在同步事件发生时，会拍摄所有所需数据的快照。通过主动推送的 API 函数在每个成功组装的数据块上提供对这些值的访问。外部测试仪收集每个数据块并随后计算精度，并相应地维护记录块及其元素的历史记录。数据将通过何种协议以及如何传输到外部测试仪将由应用程序指定。

[SWS_TS_00803]{草案} [仅当 isSystemWideGlobalTimeMaster 设置为 FALSE 时，才可能通过同步时基和偏移时基进行注册。]

[SWS_TS_00800]{草案} [对于同步时基，已注册的时间精度测量通知器（通过 ara::tsync::SynchronizedTimeBaseConsumer::RegisterTimePrecisionMeasurementNotifier）应在更新主时间元组后（即，用全局时基更新本地时基后）将以下块元素写入相关的测量记录表：
- glbSeconds
- glbNanoSeconds
- timeBaseStatus
- virtualLocalTimeLow
- rateDeviation
- locSeconds
- locNanoSeconds
- pathDelay

GlbSeconds、GlbNanoSeconds 是接收时间元组的全局时间部分元素（即 TGRx）；VirtualLocalTimeLow 是接收时间元组的虚拟本地时间部分的 nanosecondsLo 元素（即 TVRx）。]

[SWS_TS_00801]{草案} [对于偏移时基，已注册的时间精度测量通知器（通过 ara::tsync::SynchronizedTimeBaseConsumer::RegisterTimePrecisionMeasurementNotifier）仅应将块元素 GlbSeconds、GlbNanoSeconds 和 TimeBaseStatus 写入相关的测量记录表。]

#### 7.3.9 全局时间验证测量支持

图 7.3 概述了时间验证功能的基本概念。

时间从机收集时间同步过程中的信息，以基于其本地全局时间实例预测例如同步入口，并检查主机和从机是否就当前时间达成一致。预测本身将由一个单独的自适应应用程序在本地分析，以检测任何存在的缺陷。此外，来自时间主机和从机的关于时间同步过程的信息也与验证器自适应应用程序共享，该应用程序可能运行在网络中的任何位置，例如在全局时间的所有者上。

验证器使用通过用户定义的反馈通道从时间主机和时间从机实体接收到的关于时间同步过程的信息，来重建整个同步过程，并检查所有对等方之间是否建立了连贯的时基。

时间验证功能仅为自适应应用程序提供 API。反馈通道和由相应自适应应用程序执行的实际验证在 AUTOSAR 中未标准化。它是在应用程序级别以用户定义的方式完成的。

<center>图 7.3：时间验证机制</center>

为了可选地验证 Timesync 和 Pdelay 机制，时间同步功能集群提供了以下功能。

[SWS_TS_00424]{草案} [每次收到 Follow_Up 消息时，应更新 ara::tsync::TimeSlaveMeasurementType 定义的所有参数，并调用函数 ara::tsync::ConsumerTimeBaseValidationNotification::SetSlaveTimingData。]

[SWS_TS_00425]{草案} [每次发送 Sync 消息时，应更新 ara::tsync::TimeMasterMeasurementType 定义的所有参数，并调用函数 ara::tsync::ProviderTimeBaseValidationNotification::SetMasterTimingData。]

[SWS_TS_00426]{草案} [当前 Pdelay 测量完成后，即收到 Pdelay_Resp_Follow_Up 消息后，应更新 ara::tsync::PdelayInitiatorMeasurementType 定义的所有参数，并调用函数 ara::tsync::ConsumerTimeBaseValidationNotification::SetPdelayInitiatorData。]

[SWS_TS_00427]{草案} [当前 Pdelay 测量完成后，即发送 Pdelay_Resp_Follow_Up 后，应更新

由 ara::tsync::PdelayResponderMeasurementType 定义的所有参数，并调用函数 ara::tsync::ProviderTimeBaseValidationNotification::SetPdelayResponderData。]

注意：请注意，相应 PTP 事件消息的接收和发送与测量数据的转发之间存在解耦。

----

