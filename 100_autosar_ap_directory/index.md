# 

阅读 AUTOSAR Adaptive Platform (AP) 的官方文档，合适的方法和顺序至关重要。这份指南可以帮你理清头绪，高效入门。

<!-- more -->  

## 📚 第一站：整体框架和设计哲学

- AUTOSAR_EXP_PlatformDesign：阅读AP文档的极佳起点。它提供了AP设计的全局概览和所有关键概念，像一张“藏宝图”指引后续的深入方向。
- AUTOSAR_EXP_SWArchitecture：详细拆解AP的软件架构。
- AUTOSAR_TR_AdaptivePlatformReleaseOverview：对AP某个特定版本的“发行说明”，快速了解该版本包含的所有文档、功能和特性。
- AUTOSAR_RS_Main：AUTOSAR的顶层“总纲”文档，读起来会比较枯燥，建议在建立框架后略读，了解顶层需求。

## 🧭 第二站：熟悉高频概念与核心流程

- AUTOSAR_TR_FunctionalClusterShortnames：一份非常实用的“速查手册”，汇集了各个功能集群（FC）的缩写，便于快速查阅。
- AUTOSAR_TR_Glossary（术语表）：查阅本文档可帮你准确理解AUTOSAR体系中的专业术语。
- AUTOSAR_TR_AdaptiveMethodology：介绍AP的标准开发方法论、应用程序开发流程和工具链。

## ⚙️ 第三站：深入关键模块

- 执行管理（EM， Execution Management）
    - AUTOSAR_EXP_ExecutionManagement
 - AUTOSAR_SWS_ExecutionManagement、AUTOSAR_TPS_ManifestSpecification。EM相当于AP的“操作系统内核”，负责整个系统生命周期的管理，Manifest是EM等模块运行所必需的配置文件。
- 通信管理（CM， Communication Management）：推荐文档：AUTOSAR_EXP_ARAComAPI、AUTOSAR_SWS_CommunicationManagement。CM是AP实现SOA通信的核心，定义了用于服务发现和调用的API。
- 状态管理（SM， State Management）：推荐文档：AUTOSAR_RS_StateManagement、AUTOSAR_SWS_StateManagement。SM负责协调整个系统的运行模式切换。
- 持久性（PER， Persistency）：推荐文档：AUTOSAR_SWS_Persistency。PER为应用程序提供非易失性数据存储机制。
- 日志和追踪（LT， Log and Trace）：推荐文档：AUTOSAR_SWS_LogAndTrace。LT为AP应用程序提供了统一的日志记录接口。

## 📖 第四站：其他模块

- 应用开发者：阅读 AUTOSAR_SWS 开头的软件规范文档和 AUTOSAR_EXP_AdaptivePlatformInterfacesGuideline（接口使用指南）。
- 平台/服务开发者：重点关注 AUTOSAR_SWS、AUTOSAR_TPS（模板规范）及每个功能集群的详细设计说明（FC Design）。
- 工具链开发者：深入研读 AUTOSAR_TPS、MOD 和 MMOD 等元模型文档。
- 系统集成者：关注 AUTOSAR_TPS_ManifestSpecification及所有与配置、部署相关的文档。

## 📑 AUTOSAR 文档分类速查

AUTOSAR 文档通过文件名中的前缀来区分。理解这些前缀，能帮助你快速定位所需文档。

| 缩写 | 分类 | 说明 |
| -- | -- | -- |
| EXP | Explanatory Document | 解释说明性文档，提供入门介绍和概览，是初学者的首选。|
| SWS | Software Specification | 软件规范，提供详细、精确的设计与实现要求。 |
RS/SRS Requirement Specification 需求规范，描述系统或软件的顶层需求。
TR Technical Report 技术报告，提供一般性的技术信息或报告。
TPS Template Specification 模板规范，定义了用于配置的 XML 模板（ARXML）结构。
MOD/MMOD Model / Meta Model 元模型，定义了 AUTOSAR 的UML模型。

----

