# AUTOSAR_SWS_Persistency_R2011


<!--more-->

# AUTOSAR_SWS_Persistency

---

## 7 功能规范

### 7.1 持久性架构

功能集群 Persistency 提供两种不同的机制来访问持久内存：键值存储提供对一组键及关联值的访问（类似于数据库），而文件存储提供对一组文件的访问（类似于文件系统的目录）。

自适应应用程序中持久性的典型用法如图 7.1 所示。如图所示，自适应应用程序可以组合使用多个键值存储和多个文件存储。

<center>图 7.1：自适应应用程序中持久性的典型用法</center>

#### 7.1.1 Manifest 中的持久性

自适应应用程序的持久性用法在 Execution Manifest（以下简称“manifest”）中建模，作为可执行文件的 AdaptiveApplicationSwComponentTypes 的一部分。模型有两个主要部分：应用设计信息（由 PersistencyKeyValueStorageInterface 和 PersistencyFileStorageInterface 聚合）和部署信息（由 PersistencyKeyValueStorage 和 PersistencyFileStorage 聚合）。

API 规范提供了类 ara::per::KeyValueStorage 和 ara::per::FileStorage，分别用于访问键值存储或文件存储。这些类的全局函数接收一个标识符（PortPrototype 的完全限定 shortName 路径，该 PortPrototype 由 PersistencyInterface 类型化）作为 ara::core::InstanceSpecifier 输入参数（参见 8.2.1 和 8.3.1）。根据 PortPrototype 的性质，键值存储或文件存储将可访问为：
- 只读：如果 PortPrototype 实例化为 RPortPrototype，或
- 读写：如果 PortPrototype 实例化为 PRPortPrototype，或
- 只写：如果 PortPrototype 实例化为 PRPortPrototype。

manifest 包含引用可执行文件的每个进程的单独部署数据。通过 PersistencyPortPrototypeToDeploymentMapping 类的特化，将进程绑定到部署数据，该类引用由 PersistencyInterface 类型化的 PortPrototype、PersistencyDeployment 和进程。

**manifest 中基类的使用**

为简化，适用于键值存储和文件存储的信息被收集在 manifest 的基类中，即 PersistencyInterface（用于 PersistencyKeyValueStorageInterface 和 PersistencyFileStorageInterface）和 PersistencyDeployment（用于 PersistencyKeyValueStorage 和 PersistencyFileStorage）。

同样，关于键和文件的公共信息被收集在 PersistencyInterfaceElement（用于 PersistencyDataElement 和 PersistencyFileElement）和 PersistencyDeploymentElement（用于 PersistencyKeyValuePair 和 PersistencyFile）中。

应用设计与部署信息之间的链接（由 PersistencyPortPrototypeToDeploymentMapping 表示）被特化为 PersistencyPortPrototypeToKeyValueStorageMapping 和 PersistencyPortPrototypeToFileStorageMapping。

#### 7.1.2 Manifest 中的键值存储

每个键值存储由应用设计中相应 AdaptiveApplicationSwComponentType 的一个 PortPrototype（由 PersistencyKeyValueStorageInterface 类型化）表示，并由一个包含部署信息的 PersistencyKeyValueStorage 表示。每个键值存储可以包含多个键值对。自适应应用程序可以使用 Persistency API 在运行时添加和删除键值对（参见 8.2.5.7 和 8.2.5.8）。

可以在自适应应用程序安装或更新期间，使用预定义的键值对部署键值存储，并带有默认数据。此操作由 UCM 模块在安装或更新期间使用自适应应用程序软件包提供的部署信息和数据（间接）触发。参见第 7.6 节。

键值存储的应用设计与部署信息之间的链接由 PersistencyPortPrototypeToKeyValueStorageMapping 表示，它引用由 PersistencyKeyValueStorageInterface 类型化的 PortPrototype、相应的 PersistencyKeyValueStorage 和一个进程。

#### 7.1.3 Manifest 中的文件存储

每个文件存储由应用设计中相应 AdaptiveApplicationSwComponentType 的一个 PortPrototype（由 PersistencyFileStorageInterface 类型化）表示，并由一个包含部署信息的 PersistencyFileStorage 表示。每个文件存储可以包含多个文件，如 [3] 所述。与上述键值对类似，自适应应用程序可以使用 Persistency API 在运行时创建和删除文件（参见 8.3.11.11、8.3.11.13 和 8.3.11.5）。

可以在安装或更新期间部署带有初始内容的预定义文件的文件存储。此操作同样由 UCM 模块（间接）触发。所有需要的部署信息和文件均随自适应应用程序的软件包提供。参见第 7.6 节。

文件存储的应用设计与部署信息之间的链接由 PersistencyPortPrototypeToFileStorageMapping 表示，它引用由 PersistencyFileStorageInterface 类型化的 PortPrototype、相应的 PersistencyFileStorage 和一个进程。

### 7.2 功能集群生命周期

#### 7.2.1 持久性的初始化和关闭

使用 ara::core::Initialize 和 ara::core::Deinitialize，应用程序可以启动和关闭所有具有直接 ARA 接口的功能集群（即自适应平台基础）。

[SWS_PER_00408] [当调用 ara::core::Initialize 时，Persistency 应读取 manifest 信息，并为 manifest 中定义的所有键值存储和文件存储准备访问结构。]

[SWS_PER_00409] [当调用 ara::core::Deinitialize 时，Persistency 应隐式确保所有文件存储中所有打开的文件都被持久化（如同调用了 ara::per::ReadWriteAccessor::SyncToFile），并如同析构了 ara::per::UniqueHandle 那样关闭，并且所有键值存储中未持久化的值被丢弃（如同调用了 ara::per::KeyValueStorage::DiscardPendingChanges）。之后，应释放所有访问结构。]

应用程序不应在 ara::core::Initialize 之前或 ara::core::Deinitialize 之后调用 Persistency 的任何 API，但 Persistency 需要保护自身免受此类意外情况的影响。

[SWS_PER_00410] [草稿] [Persistency 的所有函数及其类的方法，若在静态初始化之后、ara::core::Initialize 调用之前，或在 ara::core::Deinitialize 调用之后被调用，应返回错误 kNotInitialized。]

### 7.3 对持久数据的并行访问

根据 [7]，持久数据仅对一个进程本地可见。因此，Persistency 永远不会在两个（或更多）进程之间共享持久数据，即使它们属于同一可执行文件。此决策的背景是：Persistency 不应为应用程序提供除通信管理功能集群（例如使用 ara::com）之外的额外通信路径。

[SWS_PER_00309] [持久数据应始终仅对一个进程本地可见。]

如果需要多个进程（同一应用程序或不同应用程序）访问持久数据，应用程序设计者有责任提供服务接口进行通信。

另一方面，Persistency 已准备好处理同一应用程序中多个线程（在同一进程上下文中运行）的并发访问。为了创建对键值存储或文件存储的共享访问，可以将 ara::per::OpenKeyValueStorage 和 ara::per::OpenFileStorage 返回的 ara::per::SharedHandle 传递（即复制）给另一个线程，或者可以在独立线程中为同一个键值存储或文件存储分别调用 ara::per::OpenKeyValueStorage 和 ara::per::OpenFileStorage。键值存储和文件存储的所有操作都支持来自多个线程的并发访问，尽管像 ara::per::RecoverKeyValueStorage 和 ara::per::ResetKeyValueStorage 或 ara::per::RecoverAllFiles 和 ara::per::ResetAllFiles 这样的操作仅当相应的键值存储或文件存储未打开时才能成功。

对键值存储中单个键的访问可以同时来自多个线程，因为 ara::per::KeyValueStorage::GetValue 和 ara::per::KeyValueStorage::SetValue 的操作是原子的，ara::per::KeyValueStorage::RemoveKey、ara::per::KeyValueStorage::RemoveAllKeys、ara::per::KeyValueStorage::SyncToStorage 和 ara::per::KeyValueStorage::DiscardPendingChanges 也是如此。

对文件存储中单个文件的访问不能在多个线程之间共享，因为无法同步读写访问以及文件中相应查找位置的变化。因此，由 OpenFile* API 返回的 ara::per::UniqueHandle 只能移动到另一个线程，而尝试打开已打开的文件将失败。同样，像 ara::per::FileStorage::DeleteFile、ara::per::FileStorage::RecoverFile 和 ara::per::FileStorage::ResetFile 这样的操作对于打开的文件也不可行。

当它们的 ara::per::UniqueHandle 超出作用域，或者它们所属的文件存储被关闭时，文件会被隐式关闭。

[SWS_PER_00425] [当文件存储因所有相关的 ara::per::SharedHandle 超出作用域而被关闭时，任何仍处于打开状态的文件也会被关闭。]

访问已关闭文件存储的 ara::per::UniqueHandle 将导致未定义行为。

### 7.4 安全概念

Persistency 支持对存储在键值存储或文件存储中的数据进行加密和认证。是否应用加密和/或认证，在部署时决定。应用程序对此无感知。

通常，键值存储、键值存储中的某个键、文件存储或文件存储中的某个文件在存储创建时以及存储保存时被加密，并在存储打开时被解密。用于存储认证的签名哈希同样在打开存储时验证，并在安装或保存键值存储或文件存储时计算。

对于只读的键值存储或文件存储，加密仅在安装期间执行一次。用于只读键值存储或文件存储（或其内部的键或文件）认证的签名哈希，要么作为 manifest 中的 PersistencyDeploymentToCryptoKeySlotMapping.verificationHash 或 PersistencyDeploymentElementToCryptoKeySlotMapping.verificationHash 提供，要么在安装期间计算。

[SWS_PER_00210]{草稿} [如果 manifest 中存在 PersistencyDeploymentToCryptoKeySlotMapping 或 PersistencyDeploymentElementToCryptoKeySlotMapping，并且 PersistencyDeploymentToCryptoKeySlotMapping.keySlotUsage 或 PersistencyDeploymentElementToCryptoKeySlotMapping.keySlotUsage 设置为加密，则 Persistency 集群应在将相关数据存储到持久内存之前对其进行加密。]

[SWS_PER_00211]{草稿} [如果 manifest 中存在 PersistencyDeploymentToCryptoKeySlotMapping 或 PersistencyDeploymentElementToCryptoKeySlotMapping，并且 PersistencyDeploymentToCryptoKeySlotMapping.keySlotUsage 或 PersistencyDeploymentElementToCryptoKeySlotMapping.keySlotUsage 设置为加密，则 Persistency 集群应在从持久内存读取相关数据后对其进行解密。]

[SWS_PER_00449]{草稿} [如果 manifest 中存在 PersistencyDeploymentToCryptoKeySlotMapping 或 PersistencyDeploymentElementToCryptoKeySlotMapping，并且 PersistencyDeploymentToCryptoKeySlotMapping.keySlotUsage 或 PersistencyDeploymentElementToCryptoKeySlotMapping.keySlotUsage 设置为验证，则 Persistency 集群应在将相关数据存储到持久内存之前对其进行签名。]

[SWS_PER_00450]{草稿} [如果 manifest 中存在 PersistencyDeploymentToCryptoKeySlotMapping 或 PersistencyDeploymentElementToCryptoKeySlotMapping，并且 PersistencyDeploymentToCryptoKeySlotMapping.keySlotUsage 或 PersistencyDeploymentElementToCryptoKeySlotMapping.keySlotUsage 设置为验证，则 Persistency 集群应在从持久内存读取相关数据后验证其签名。]

[SWS_PER_00451]{草稿} [如果 PersistencyDeploymentToCryptoKeySlotMapping.verificationHash 或 PersistencyDeploymentElementToCryptoKeySlotMapping.verificationHash 可用，则 Persistency 集群应使用此哈希验证相关数据。]

Persistency 功能集群应使用 Crypto API 的服务进行加密和解密，以及创建和验证签名哈希。它应从 PersistencyDeploymentToCryptoKeySlotMapping 或 PersistencyDeploymentElementToCryptoKeySlotMapping 引用的 CryptoKeySlot 派生要使用的算法和密钥，并用于访问 Crypto API（详情参见 [8]）。

### 7.5 冗余概念

Persistency 功能集群应负责存储数据的完整性。这可以通过计算存储数据的 CRC 或哈希值，以及创建冗余副本来实现。所有这些措施都为存储数据创建了某种冗余。具体采取的措施是可配置的：应用程序设计者可以使用 PersistencyInterface.redundancy 请求冗余，或使用 PersistencyInterface.redundancyHandling 预选实际采取的措施。在部署期间，集成者可以使用 PersistencyDeployment.redundancyHandling 定义实际采取的数据完整性保证措施。如果配置了 PersistencyInterface.redundancyHandling，集成者应将其作为指导，但也可以基于对最终系统的更深入了解选择其他更合适的措施。

[SWS_PER_00317] [Persistency 集群应为每个由 PersistencyInterface 类型化的 PortPrototype 所表示的键值存储和文件存储存储冗余信息，其中 PersistencyInterface.redundancy 设置为 redundant 或 redundantPerElement，或者配置了 PersistencyInterface.redundancyHandling（另见 [SWS_PER_00318]、[SWS_PER_00319] 和 [SWS_PER_00447]）。]

[SWS_PER_00221] [Persistency 集群应使用冗余信息检测持久内存中的数据损坏。]

[SWS_PER_00222] [Persistency 集群应尽可能使用冗余信息恢复损坏的数据。]

如果数据损坏且无法使用冗余信息恢复，Persistency 将失败并返回 kValidationFailed。

然后，应用程序可以选择使用 ara::per::RecoverKeyValueStorage、ara::per::KeyValueStorage::RecoverKey、ara::per::RecoverAllFiles 或 ara::per::FileStorage::RecoverFile 来尽可能多地恢复，并将相应的键值存储或文件存储再次设置为一致状态。当然，在这种情况下，应用程序必须验证恢复后的数据。或者，它可以使用 ara::per::ResetKeyValueStorage、ara::per::KeyValueStorage::ResetKey、ara::per::ResetAllFiles 或 ara::per::FileStorage::ResetFile 将损坏的项重置为当前 manifest 定义的初始状态。

#### 7.5.1 冗余类型

Persistency 功能集群应用的冗余类型由一组 PersistencyRedundancyHandling 类定义，这些类作为 PersistencyDeployment.redundancyHandling 聚合。应用冗余的级别由 PersistencyRedundancyHandlingScopeEnum 的可能值定义，这些值为 persistencyRedundancyHandlingScopeStorage（应用于存储级别）和 persistencyRedundancyHandlingScopeElement（应用于元素级别，即键值存储中的键或文件存储中的文件）。

[SWS_PER_00318] [如果作为 PersistencyDeployment.redundancyHandling 聚合的 PersistencyRedundancyHandling 派生自 PersistencyRedundancyCrc，则 Persistency 集群在持久化键值存储、键值存储中的键、文件存储或文件存储中的文件时（取决于 PersistencyDeployment.redundancyHandling.scope）应计算 CRC 值，并在回读时使用该 CRC 检查相应项。]

[SWS_PER_00439] [Persistency 应使用 PersistencyRedundancyCrc.algorithmFamily 定义的算法族以及 PersistencyRedundancyCrc.length 定义的位宽来计算 CRC 值。]

[SWS_PER_00319] [如果作为 PersistencyDeployment.redundancyHandling 聚合的 PersistencyRedundancyHandling 派生自 PersistencyRedundancyMOutOfN，则 Persistency 集群在持久化键值存储、键值存储中的键、文件存储或文件存储中的文件时（取决于 PersistencyDeployment.redundancyHandling.scope）应存储 N 个副本，并在回读时检查该键值存储、键值存储中的键、文件存储或文件存储中的文件的 N 个副本中至少有 M 个是相同的。N 由 n 定义，M 由 m 定义。]

[SWS_PER_00447]{草稿} [如果作为 PersistencyDeployment.redundancyHandling 聚合的 PersistencyRedundancyHandling 派生自 PersistencyRedundancyHash，则 Persistency 集群在持久化键值存储、键值存储中的键、文件存储或文件存储中的文件时（取决于 PersistencyDeployment.redundancyHandling.scope）应计算哈希值，并在回读时使用该哈希值检查相应项。]

[SWS_PER_00448]{草稿} [Persistency 应使用 PersistencyRedundancyHash.algorithmFamily 定义的算法族以及 PersistencyRedundancyHash.length 定义的位宽来计算哈希值。如果配置了 PersistencyRedundancyHash.initializationVectorLength，则应计算该长度的包含随机数据的初始化向量，并传递给哈希算法。]

计算哈希值和随机数据的一种可能方法是使用 Crypto API（参见 [8]）。集成时需注意，配置的 PersistencyRedundancyHash.length 和 PersistencyRedundancyHash.initializationVectorLength 需得到配置的 PersistencyRedundancyHash.algorithmFamily 的支持。

### 7.6 持久数据的安装和更新

更新和配置管理处理自适应应用程序的生命周期，包含以下阶段：
- 新软件的安装
- 已安装软件的更新
- 更新成功后对已更新软件的最终确认
- 更新失败后对已更新软件的回滚
- 已安装软件的移除

对于所有这些阶段，持久数据需要与应用程序一起处理。自适应应用程序可以通过在安装或更新之后的验证阶段显式调用 ara::per::UpdatePersistency 来触发此处理，或者依赖 Persistency 集群在访问持久数据时（ara::per::OpenKeyValueStorage/ara::per::OpenFileStorage）隐式执行。在这两种情况下，Persistency 集群都会将存储的 manifest 版本与当前 manifest 版本进行比较，并执行所需的操作。

[SWS_PER_00378] [Persistency 应从 manifest 中提取包含 Persistency 部署数据的 SoftwareCluster 的 Executable.version 和 SoftwareCluster.version，并将它们与键值存储和文件存储一起持久存储。]

Executable.version 被 Persistency 用于检测应用程序的变化（参见 [SWS_PER_00387]），而 SoftwareCluster.version 用于检测已部署持久数据的变化（参见 [SWS_PER_00386] 和 [SWS_PER_00396]）。

根据 [SWS_UCM_CONSTR_00001]，当 Executable.version 增加时，SoftwareCluster.version 总是增加。

SoftwareCluster.version 和 Executable.version 是 StrongRevisionLabelString 类型。这些字符串由 MajorVersion、MinorVersion、PatchVersion 以及用于预发布版本和构建元数据的附加标签组成。假设前三个在版本更改时会递增，而最后一个可能是任意的。

自适应应用程序安装后，Persistency 集群将安装 manifest 中预定义的持久数据。在 manifest 中定义这些持久数据有几种不同的可能性：
- 持久数据可以由应用程序设计者在 PersistencyKeyValueStorageInterface 或 PersistencyFileStorageInterface 中定义。
- 应用程序设计者定义的持久数据可以由集成者在 PersistencyKeyValueStorage 或 PersistencyFileStorage 中更改。
- 持久数据可以由集成者直接在 PersistencyKeyValueStorage 或 PersistencyFileStorage 中定义。

[SWS_PER_00379] [部署数据（PersistencyKeyValueStorage 和 PersistencyFileStorage 及相关类）中定义的元素应始终优先于应用设计（PersistencyKeyValueStorageInterface 和 PersistencyFileStorageInterface 及相关类）中定义的元素。仅当前者不存在时才应使用后者。]

在更新自适应应用程序或 manifest 之后，Persistency 集群将创建持久数据的备份，然后使用以下策略之一更新现有持久数据：
- 保留现有持久数据不变 (keepExisting)
- 替换现有持久数据 (overwrite)
- 删除现有持久数据 (delete)
- 添加新的持久数据 (keepExisting 和 overwrite)

更新策略可以在应用设计或部署时设置，并且可以为整个键值存储或文件存储（PersistencyCollectionLevelUpdateStrategyEnum - keepExisting 或 delete）以及单个键或文件（PersistencyElementLevelUpdateStrategyEnum - keepExisting、overwrite 或 delete）定义。

[SWS_PER_00251] [在部署数据中定义的更新策略（PersistencyDeployment.updateStrategy、PersistencyDeploymentElement.updateStrategy）应始终优先于在应用设计中定义的更新策略（PersistencyInterface.updateStrategy、PersistencyInterfaceElement.updateStrategy）。仅当前者不存在时才应使用后者。]

[SWS_PER_00380] [为单个键或文件定义的更新策略（PersistencyDeploymentElement.updateStrategy、PersistencyInterfaceElement.updateStrategy）应始终优先于为包含它们的键值存储或文件存储定义的更新策略（PersistencyDeployment.updateStrategy、PersistencyInterface.updateStrategy）。仅当前者不存在时才应使用后者。]

当更新成功时，更新和配置管理将对新的自适应应用程序进行最终确认。Persistency 集群不需要做任何事情，尽管它可以释放上次备份占用的资源。

当更新失败时，更新和配置管理将回退到旧的自适应应用程序和/或 manifest。Persistency 集群随后将用更新期间创建的备份替换当前使用的持久数据。

最后，为了在移除自适应应用程序之前移除持久数据，自适应应用程序需要调用 ara::per::ResetPersistency。

#### 7.6.1 持久数据的安装

[SWS_PER_00382] [当应用程序通过 ara::per::OpenKeyValueStorage 或 ara::per::OpenFileStorage 打开键值存储或文件存储时，或者当调用 ara::per::UpdatePersistency 时，Persistency 应检查存储数据是否存在。如果未找到持久数据，Persistency 应初始化持久数据。]

持久数据的初始化在第 7.6.1.1 和 7.6.1.2 节中描述。

#### 7.6.1.1 键值存储的安装

[SWS_PER_00383] [Persistency 应为在新安装的自适应应用程序的 manifest 中找到的每个由 PersistencyKeyValueStorageInterface 类型化的 PortPrototype 创建一个键值存储。在运行时，该键值存储应通过 PortPrototype 的 shortName 路径（作为 ara::core::InstanceSpecifier 传递给 ara::per::OpenKeyValueStorage）来标识。]

[SWS_PER_00252] [Persistency 应为在新安装或更新的自适应应用程序的 manifest 中找到的每个 PersistencyKeyValueStorageInterface.dataElement 和 PersistencyKeyValueStorage.keyValuePair 创建一个条目，并且其更新策略为 keepExisting 或 overwrite。]

键值存储条目通过键来标识。相同的键可能同时在 PersistencyKeyValueStorageInterface 和 PersistencyKeyValueStorage 中定义，此时适用 [SWS_PER_00379]。更新策略根据 [SWS_PER_00251] 和 [SWS_PER_00380] 确定。

[SWS_PER_00253] [键值存储中的条目应使用 PersistencyDataElement 和/或 PersistencyKeyValuePair 的 shortName 作为键。]

[SWS_PER_00254] [键值存储中的条目应使用类型化 PersistencyDataElement 的 CppImplementationDataType 和/或由 PersistencyKeyValuePair.valueDataType 引用的 CppImplementationDataType 所定义的数据类型来创建。]

[SWS_PER_00384] [键值存储中的条目应使用取自 PersistencyKeyValuePair.initValue 的值创建，如果该值不存在，则使用 PersistencyDataRequiredComSpec.initValue 的值。]

[SWS_PER_CONSTR_00003] [如果任何 PersistencyKeyValuePair 或 PersistencyDataElement 的值或数据类型无法确定，或者确定的数据类型存在冲突，则 manifest 无效。]

无效的 manifest 应被工具拒绝。

#### 7.6.1.2 文件存储的安装

[SWS_PER_00385] [Persistency 应为在新安装的自适应应用程序的 manifest 中找到的每个由 PersistencyFileStorageInterface 类型化的 PortPrototype 创建一个文件存储。在运行时，该文件存储应通过 PortPrototype 的 shortName 路径（作为 ara::core::InstanceSpecifier 传递给 ara::per::OpenFileStorage）来标识。]

[SWS_PER_00265] [Persistency 应为在新安装或更新的自适应应用程序的 manifest 中找到的每个 PersistencyFileStorageInterface.fileElement 和 PersistencyFileStorage.file 创建一个文件，并且其更新策略为 keepExisting 或 overwrite。]

文件存储中的文件通过其名称标识。相同的文件名可能同时在 PersistencyFileStorageInterface 和 PersistencyFileStorage 中定义，此时适用 [SWS_PER_00379]。更新策略根据 [SWS_PER_00251] 和 [SWS_PER_00380] 确定。

[SWS_PER_00266] [文件存储中的文件应使用由 PersistencyFileElement.fileName 和/或 PersistencyFile.fileName 标识的名称。]

[SWS_PER_00267] [文件存储中的文件应使用来自（已安装的 SoftwarePackage 内的）资源的初始内容创建，该资源由 PersistencyFile.contentUri 寻址，如果不存在，则由 PersistencyFileElement.contentUri 寻址。如果两者都不存在，则创建一个空文件。]

[SWS_PER_CONSTR_00004] [如果具有相同文件名的 PersistencyFileElement 和 PersistencyFile 的 shortName 不同，则 manifest 无效。]

无效的 manifest 应被工具拒绝。

#### 7.6.2 持久数据的更新

[SWS_PER_00386] [当应用程序通过 ara::per::OpenKeyValueStorage 或 ara::per::OpenFileStorage 打开键值存储或文件存储时，或者当调用 ara::per::UpdatePersistency 时，Persistency 应将 manifest 中的 SoftwareCluster.version 与存储的版本进行比较。如果 manifest 中的版本高于存储的版本，Persistency 应首先创建持久数据的备份，然后更新数据。]

任何时候只需要保留一组备份数据。执行新更新时，旧备份数据可能会被覆盖。持久数据的更新在第 7.6.2.1 和 7.6.2.2 节中描述。

[SWS_PER_00387] [当应用程序通过 ara::per::OpenKeyValueStorage 或 ara::per::OpenFileStorage 打开键值存储或文件存储时，或者当调用 ara::per::UpdatePersistency 时，Persistency 应将 manifest 中的 Executable.version 与存储的版本进行比较。如果 manifest 中的版本高于存储的版本，Persistency 应根据 [SWS_PER_00386] 为每个已更新的键值存储和文件存储调用由应用程序通过 ara::per::RegisterApplicationDataUpdateCallback 注册的函数。]

应用程序通过 ara::per::RegisterApplicationDataUpdateCallback 注册的函数可用于手动更新键值存储中的键值对或文件存储中的文件。提供给此函数的 ara::core::InstanceSpecifier 用于标识键值存储或文件存储。然后，应用程序可以根据作为第二个参数提供的存储数据的 Executable.version，读取旧格式或旧类型的存储数据，转换数据，并以当前版本所需的新格式或新类型重新存储。

示例：应用程序的第 1 版将最大速度以 mph 存储为 uint8，但第 2 版期望以 km/h 存储为 uint16。然后，更新回调函数将看到来自 Executable 第 1 版的键值存储已更新到当前版本，并可以通过 ara::per::KeyValueStorage::GetValue 读取旧的 uint8 最大速度，进行转换，然后在使用 ara::per::KeyValueStorage::RemoveKey 删除旧值后，通过 ara::per::KeyValueStorage::SetValue 将其存储为 uint16。

#### 7.6.2.1 键值存储的更新

[SWS_PER_00388] [如果在更新的 manifest 中检测到新的由 PersistencyKeyValueStorageInterface 类型化的 PortPrototype，Persistency 应按照 [SWS_PER_00383] 中的规定创建一个键值存储。]

[SWS_PER_00389] [如果在更新的 manifest 中缺少某个由 PersistencyKeyValueStorageInterface 类型化的 PortPrototype，Persistency 应移除相应的键值存储。]

[SWS_PER_00390] [如果在更新的 manifest 中检测到带有新键的 PersistencyKeyValueStorageInterface.dataElement 和/或 PersistencyKeyValueStorage.keyValuePair，Persistency 应按照 [SWS_PER_00252]、[SWS_PER_00253]、[SWS_PER_00254] 和 [SWS_PER_00384] 中的规定在键值存储中创建一个新条目。]

[SWS_PER_00391] [如果键值存储中的某个现有键无法与更新的 manifest 中的任何 PersistencyKeyValueStorageInterface.dataElement 或 PersistencyKeyValueStorage.keyValuePair 关联，并且该键值存储对应的 PersistencyKeyValueStorage 或 PersistencyKeyValueStorageInterface 的更新策略为 delete，则 Persistency 应从该键值存储中移除该键的条目。]

更新策略根据 [SWS_PER_00251] 确定。

[SWS_PER_00275] [如果键值存储中的某个现有键可以与更新的 manifest 中的 PersistencyKeyValueStorageInterface.dataElement 或 PersistencyKeyValueStorage.keyValuePair 关联，并且更新策略为 overwrite，则 Persistency 应按照 [SWS_PER_00254] 和 [SWS_PER_00384] 中的规定，用新的类型和值替换该键值存储中的条目。]

相同的键可能同时在 PersistencyKeyValueStorageInterface 和 PersistencyKeyValueStorage 中定义，此时适用 [SWS_PER_00379]。更新策略根据 [SWS_PER_00251] 和 [SWS_PER_00380] 确定。

[SWS_PER_00277] [如果键值存储中的某个现有键可以与更新的 manifest 中的 PersistencyKeyValueStorageInterface.dataElement 或 PersistencyKeyValueStorage.keyValuePair 关联，并且更新策略为 delete，则 Persistency 应从该键值存储中移除该键的条目。]

更新策略为 keepExisting 的已更新键在更新期间不会被触及。Persistency 既不会检查现有条目的值，也不会检查其类型。

#### 7.6.2.2 文件存储的更新

[SWS_PER_00392] [如果在更新的 manifest 中检测到新的由 PersistencyFileStorageInterface 类型化的 PortPrototype，Persistency 应按照 [SWS_PER_00385] 中的规定创建一个文件存储。]

[SWS_PER_00393] [如果在更新的 manifest 中缺少某个由 PersistencyFileStorageInterface 类型化的 PortPrototype，Persistency 应移除相应的文件存储。]

[SWS_PER_00394] [如果在更新的 manifest 中检测到带有新文件名的 PersistencyFileStorageInterface.fileElement 和/或 PersistencyFileStorage.file，Persistency 应按照 [SWS_PER_00265]、[SWS_PER_00266] 和 [SWS_PER_00267] 中的规定在文件存储中创建一个新文件。]

[SWS_PER_00395] [如果文件存储中的某个现有文件无法与更新的 manifest 中的任何 PersistencyFileStorageInterface.fileElement 或 PersistencyFileStorage.file 关联，并且该文件存储对应的 PersistencyFileStorage 或 PersistencyFileStorageInterface 的更新策略为 delete，则 Persistency 应从该文件存储中移除该文件。]

更新策略根据 [SWS_PER_00251] 确定。

[SWS_PER_00281] [如果文件存储中的某个现有文件可以与更新的 manifest 中的 PersistencyFileStorageInterface.fileElement 或 PersistencyFileStorage.file 关联，并且更新策略为 overwrite，则 Persistency 应按照 [SWS_PER_00267] 中的规定，用新内容替换该文件存储中的文件内容。]

相同的文件名可能同时在 PersistencyFileStorageInterface 和 PersistencyFileStorage 中定义，此时适用 [SWS_PER_00379]。更新策略根据 [SWS_PER_00251] 和 [SWS_PER_00380] 确定。

[SWS_PER_00283] [如果文件存储中的某个现有文件可以与更新的 manifest 中的 PersistencyFileStorageInterface.fileElement 或 PersistencyFileStorage.file 关联，并且更新策略为 delete，则 Persistency 应从该文件存储中移除该文件。]

更新策略为 keepExisting 的已更新文件在更新期间不会被触及。Persistency 不会检查现有文件的内容。

#### 7.6.3 成功更新后持久数据的最终确认

安装和更新后，通常会在应用程序的验证阶段调用 ara::per::UpdatePersistency。当此操作成功时，UCM 将对应用程序进行最终确认，然后应用程序以正常执行模式重新启动。在这种情况下，Persistency 应移除先前更新期间创建的任何备份。

[SWS_PER_00446]{草稿} [当应用程序通过 ara::per::OpenKeyValueStorage 或 ara::per::OpenFileStorage 打开键值存储或文件存储，并且自 Persistency 初始化以来尚未调用 ara::per::UpdatePersistency 时，Persistency 应将 manifest 中的 SoftwareCluster.version 与存储的版本进行比较。如果两个版本相同，Persistency 应移除所有备份数据。]

持久数据的更新在第 7.6.2 节中描述。

#### 7.6.4 更新失败后持久数据的回滚

[SWS_PER_00396] [当应用程序通过 ara::per::OpenKeyValueStorage 或 ara::per::OpenFileStorage 打开键值存储或文件存储时，或者当调用 ara::per::UpdatePersistency 时，Persistency 应将 manifest 中的 SoftwareCluster.version 与存储的版本进行比较。如果 manifest 中的版本低于存储的版本，Persistency 应将 manifest 中的版本与备份数据中存储的版本进行比较。如果版本匹配，Persistency 应恢复备份。否则，它应移除所有键值存储和文件存储，并从 manifest 重新安装持久数据。]

持久数据的初始化在第 7.6.1 节中描述。

#### 7.6.5 持久数据的移除

[SWS_PER_00397] [当调用 ara::per::ResetPersistency 时，Persistency 应移除所有键值存储和文件存储。]

### 7.7 资源管理概念

Persistency 集群支持为键值存储或文件存储配置资源的上限和下限。
下限可能已经由应用程序开发者使用 PersistencyInterface.minimumSustainedSize 定义。在部署期间，集成者可以使用 PersistencyDeployment.minimumSustainedSize 更新下限，并使用 PersistencyDeployment.maximumAllowedSize 添加上限。

[SWS_PER_00320] [Persistency 集群应确保 PersistencyDeployment.minimumSustainedSize 配置的空间始终可用于该键值存储或文件存储。]

实现这一点的一种方法是在部署期间初始分配最小大小，并在移除持久数据时永远不将大小减小到该值以下。但 Persistency 集群的实现可以自由选择其他合适的措施。

[SWS_PER_00321] [Persistency 集群应确保键值存储或文件存储实际分配的空间永远不会超过 PersistencyDeployment.maximumAllowedSize 配置的数量。]

这可以通过监督所有对持久数据的写入访问来实现。但同样，Persistency 集群的实现可以自由选择其他合适的措施。

应用程序还可以通过分别使用 ara::per::GetCurrentKeyValueStorageSize 或 ara::per::GetCurrentFileStorageSize 来轮询整个键值存储或文件存储当前占用的存储量。当然，返回的值不会低于配置的最小大小（PersistencyDeployment.minimumSustainedSize）或高于配置的最大大小（PersistencyDeployment.maximumAllowedSize）。此外，应用程序可以使用打开的文件存储的 ara::per::FileStorage::GetCurrentFileSize 来轮询单个文件当前占用的存储量。

### 7.8 键值存储支持的数据类型

Persistency 集群在键值存储的函数 ara::per::KeyValueStorage::GetValue（通过 T 模板化）和 ara::per::KeyValueStorage::SetValue（通过 T 模板化）中支持以下类别的数据类型。

[SWS_PER_00302] [Persistency 集群应能够将 [9] 中描述的所有数据类型存储在键值存储中。]

[SWS_PER_00303] [Persistency 集群应能够将序列化的二进制数据存储在键值存储中。序列化的二进制数据必须作为 ara::core::Byte 的 ara::core::Vector 呈现。]

这允许应用程序存储自定义数据类型。

[SWS_PER_00304] [Persistency 集群应能够将应用设计中通过 PersistencyKeyValueStorageInterface.dataTypeForSerialization 或 PersistencyKeyValueStorageInterface.dataElement 引用的所有 CppImplementationDataType 存储在相应键值存储中。参见 [3]。]

### 7.9 获取有关文件的其他信息

为了获取有关存储文件的信息，Persistency 集群提供了方法 ara::per::FileStorage::GetFileInfo。此方法返回有关文件创建时间（creationTime）、最后修改时间（modificationTime）和最后访问时间（accessTime），以及文件如何及由谁创建（fileCreationState）和最后修改（fileModificationState）的信息。

[SWS_PER_00440] [方法 ara::per::FileStorage::GetFileInfo 应将所需信息收集到 ara::per::FileInfo 结构中，并将其返回给应用程序。]

如果 Persistency 集群使用底层操作系统的文件系统，则部分信息（如创建或访问时间）可以从文件系统中获取。只有当文件当前未打开时，这些信息才会准确。

---

