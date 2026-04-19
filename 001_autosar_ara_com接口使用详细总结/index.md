# AUTOSAR ara::com 接口使用详细总结


# AUTOSAR ara::com 接口使用详细总结

ara::com 是 AUTOSAR Adaptive Platform（AP）自适应平台中**通信管理（Communication Management, CM）** 功能集群的标准化C++接口，是AP实现**面向服务通信（Service-Oriented Communication, SOC）** 的核心中枢，完全区别于Classic Platform（CP）基于信号的通信范式，为车载分布式自适应应用提供了跨进程、跨ECU的标准化、类型安全、可扩展的通信能力。

## 一、核心定位与设计目标

### 1. 核心定位

ara::com 是AP应用间通信的唯一标准入口，承担了类似CP平台RTE的通信调度职责，向上为应用层提供统一的通信API，向下屏蔽底层传输协议（SOME/IP、DDS、本地IPC共享内存等）的实现差异，实现应用逻辑与通信技术的解耦，支持车规级高实时、高可靠、功能安全的通信需求。

### 2. 核心设计目标

- **协议无关性**：标准化API与底层传输绑定解耦，支持无缝切换SOME/IP、DDS、本地IPC等传输协议；

- **双编程范式兼容**：同时支持**事件驱动（回调）** 和**轮询（Polling）** 两种编程模式，兼顾易用性与硬实时需求；

- **类型安全**：基于C++模板与代码生成机制，编译期完成类型校验，避免运行时通信数据类型错误；

- **功能安全兼容**：原生支持E2E端到端保护、错误处理与监控，满足ASIL等级的功能安全需求；

- **动态性**：支持运行时服务发布、发现、订阅与连接管理，适配车载高性能计算单元的动态软件架构。

## 二、核心架构与基础范式

### 1. 核心架构：Proxy/Skeleton 代理-骨架模式

ara::com 采用经典的Proxy/Skeleton架构，所有服务通信均基于该模式实现，核心逻辑如下：

- **服务接口定义**：通过AUTOSAR标准ARXML/ASIDL描述服务接口，包含服务的方法、事件、字段、触发器等通信原语；

- **代码生成**：工具链基于服务接口定义，自动生成对应的**Service Proxy（服务代理类）** 和**Service Skeleton（服务骨架类）** C++代码；

- **客户端（Consumer）**：通过Proxy类访问远程服务，所有通信调用均通过本地Proxy代理完成，无需关注底层通信细节；

- **服务端（Provider）**：继承并实现Skeleton类的纯虚接口，完成服务逻辑开发，通过Skeleton类向系统发布服务，响应客户端请求。

### 2. 分层架构

ara::com 采用分层设计，实现通信能力的标准化与可扩展：

|层级|核心职责|关键说明|
|---|---|---|
|应用层|服务逻辑实现|基于生成的Proxy/Skeleton类开发业务逻辑，仅依赖标准化API|
|语言绑定层|C++语言映射|定义ara::com标准化C++接口规范，实现服务接口到C++类的映射|
|核心中间件层|通信核心能力|实现服务发现、生命周期管理、数据序列化/反序列化、缓存管理、错误处理|
|网络绑定层|传输协议适配|适配底层传输协议（SOME/IP、DDS、共享内存IPC等），实现协议相关的通信逻辑|
|操作系统/驱动层|底层通信能力|基于OS套接字、共享内存、网络驱动提供基础数据收发能力|
## 三、核心概念详解

### 1. 服务与服务接口（Service & Service Interface）

服务是ara::com通信的最小功能单元，一个服务对应一组可对外提供的通信能力，由**服务接口**标准化定义，核心包含4类通信原语：

- **Methods（方法）**：请求-响应式通信，客户端发起调用，服务端执行并返回结果，支持同步/异步调用、Fire&Forget单向调用；

- **Events（事件）**：发布-订阅式通信，服务端发布事件数据，已订阅的客户端接收数据推送，适用于周期性数据、状态变更通知；

- **Fields（字段）**：表示服务的状态属性，是事件+方法的组合，包含Getter（读取）、Setter（写入）、Notifier（变更通知）三个可选能力；

- **Triggers（触发器）**：无数据的事件通知，仅用于告知客户端特定条件触发，不传输任何业务数据，复用事件的订阅-通知机制。

### 2. 实例标识相关核心类

ara::com 通过两级标识实现服务实例的精准寻址与部署解耦，是服务管理的核心：

- **ara::core::InstanceSpecifier**：服务实例的**逻辑名称**，是开发阶段的唯一标识，与部署配置解耦。开发者在代码中仅使用InstanceSpecifier，无需关注底层实例ID、网络地址等部署信息；

- **ara::com::InstanceIdentifier**：服务实例的**技术标识**，由Manifest部署配置在运行时解析生成，包含服务实例ID、协议、网络地址等底层信息，通过`ResolveInstanceIDs`接口完成InstanceSpecifier到InstanceIdentifier的映射；

- **核心价值**：实现同一份应用代码，通过修改Manifest配置即可实现多实例部署、跨平台移植，无需修改业务代码。

### 3. 句柄（Handle）相关概念

- **HandleType**：服务实例的句柄，唯一标识一个可用的远程服务实例，由服务发现机制返回，用于创建Proxy实例；

- **FindServiceHandle**：服务发现任务的句柄，用于管理持续型服务发现任务，可通过该句柄停止后台服务发现监听；

- **核心设计**：通过句柄实现服务实例生命周期与Proxy实例的解耦，开发者可自主控制是否共享Proxy实例状态，避免重复创建与资源浪费。

### 4. 数据管理核心类

- **SamplePtr**：智能指针类，用于管理事件/字段数据的内存，支持零拷贝传输，由中间件管理内存生命周期，避免应用层内存拷贝与泄漏；

- **SampleAllocateePtr**：服务端数据内存分配句柄，用于提前分配事件发送的内存，实现零拷贝发送，提升高频率数据发送的性能。

## 四、完整开发使用全流程

基于ara::com的服务通信开发，遵循**接口定义→代码生成→服务端开发→客户端开发→配置部署**的标准化流程，以下为详细步骤与核心实现。

### 步骤1：服务接口定义

这是开发的起点，通过AUTOSAR标准建模工具（Vector DaVinci Developer、EB tresos Studio等）定义服务接口ARXML文件，核心定义内容包括：

1. 服务基本信息：服务名称、服务ID、版本号；

2. 数据类型定义：服务通信使用的基础数据类型、复合数据类型（结构体、枚举、数组等）；

3. 通信原语定义：

    - Methods：定义方法名、入参、出参、是否为Fire&Forget、是否支持应用错误；

    - Events：定义事件名、数据类型、缓存深度、QoS配置；

    - Fields：定义字段名、数据类型、是否启用Getter/Setter/Notifier；

    - Triggers：定义触发器名称。

示例：简化的雷达服务接口定义（伪代码）

```Plain Text

Service Interface RadarService {
    Version 1.0.0
    // 方法：设置雷达工作模式
    Method SetRadarMode {
        In: uint8 mode
        Out: bool success
    }
    // 事件：周期性发布雷达点云数据
    Event PointCloudEvent {
        Type: PointCloudStruct
        CacheDepth: 10
    }
    // 字段：雷达当前扫描频率
    Field ScanRate {
        Type: uint32
        Getter: Enable
        Setter: Enable
        Notifier: Enable
    }
    // 触发器：障碍物告警触发
    Trigger ObstacleAlertTrigger
}
```

### 步骤2：代码生成

通过AUTOSAR AP工具链，基于服务接口ARXML文件，自动生成对应的Proxy和Skeleton代码，核心生成产物包括：

1. 服务数据类型的C++定义（头文件）；

2. **RadarServiceProxy** 类：客户端使用的服务代理类，包含服务所有通信原语的客户端调用接口；

3. **RadarServiceSkeleton** 类：服务端继承的服务骨架抽象类，包含方法的纯虚接口、事件/字段/触发器的发送接口。

生成的代码完全遵循ara::com标准，屏蔽了底层通信、序列化、服务发现的所有细节，开发者仅需基于生成的类实现业务逻辑。

### 步骤3：服务端（Skeleton）开发全流程

服务端核心职责是实现服务逻辑、发布服务、响应客户端请求、推送事件数据，完整开发流程如下：

#### 3.1 继承并实现Skeleton类

继承生成的RadarServiceSkeleton抽象类，实现所有Method的纯虚接口，完成业务逻辑开发。

```C++

#include "RadarServiceSkeleton.h"
#include "ara/core/instance_specifier.h"

// 继承生成的骨架类
class RadarServiceImpl : public radar::service::RadarServiceSkeleton {
public:
    // 构造函数：传入InstanceSpecifier与服务处理模式
    explicit RadarServiceImpl(const ara::core::InstanceSpecifier& instanceSpec)
        : radar::service::RadarServiceSkeleton(instanceSpec, ara::com::MethodCallProcessingMode::kEvent)
    {}

    // 实现Method：SetRadarMode
    ara::core::Future<radar::service::RadarService::SetRadarModeOutput>
    SetRadarMode(const uint8_t& mode) override {
        // 业务逻辑：设置雷达模式
        radar::service::RadarService::SetRadarModeOutput output;
        output.success = DoSetRadarMode(mode);

        // 封装结果并返回
        ara::core::Promise<radar::service::RadarService::SetRadarModeOutput> promise;
        promise.set_value(output);
        return promise.get_future();
    }

private:
    bool DoSetRadarMode(uint8_t mode) {
        // 实际硬件操作/业务逻辑实现
        return true;
    }
};
```

#### 3.2 服务实例化与发布

```C++

int main() {
    // 1. 定义服务实例的逻辑标识（与Manifest配置对应）
    const ara::core::InstanceSpecifier instanceSpec{"radar/service/RadarServiceInstance"};

    // 2. 创建服务实现实例
    auto radarService = std::make_shared<RadarServiceImpl>(instanceSpec);

    // 3. 向系统发布服务，启动服务发现广播，允许客户端连接
    ara::core::Result<void> offerResult = radarService->OfferService();
    if (!offerResult.HasValue()) {
        // 错误处理：服务发布失败
        return -1;
    }

    // 4. 业务运行：保持进程运行，推送事件数据
    while (true) {
        // 周期性采集并发布雷达点云事件
        auto pointCloudData = CollectRadarPointCloud();
        // 发送事件数据
        radarService->PointCloudEvent.Send(pointCloudData);

        // 检测障碍物，触发告警触发器
        if (CheckObstacle()) {
            radarService->ObstacleAlertTrigger.Send();
        }

        // 更新字段值，触发Notifier通知订阅客户端
        uint32_t currentRate = GetCurrentScanRate();
        radarService->ScanRate.Update(currentRate);

        std::this_thread::sleep_for(std::chrono::milliseconds(100));
    }

    // 5. 停止服务发布
    radarService->StopOfferService();
    return 0;
}
```

#### 3.3 服务端核心API说明

|API|核心作用|使用场景|
|---|---|---|
|`OfferService()`|发布服务实例，向服务注册中心注册服务，对外广播服务可用性|服务初始化完成后，对外提供服务前调用|
|`StopOfferService()`|停止服务发布，注销服务，断开所有客户端连接|服务退出、资源释放前调用|
|`Event::Send()`|发布事件数据，推送给所有已订阅的客户端|周期性数据、状态变更通知|
|`Field::Update()`|更新字段值，若启用Notifier，会自动向订阅客户端推送变更通知|服务内部状态更新时调用|
|`ProcessNextMethodCall()`|处理下一个客户端方法调用请求，仅在kPoll轮询模式下使用|硬实时场景，自主控制方法调用处理时机|
|`SetReceiveHandler()`|设置方法调用的接收回调，仅在kEvent事件模式下使用|事件驱动场景，有请求时自动触发回调|
### 步骤4：客户端（Proxy）开发全流程

客户端核心职责是发现服务、创建Proxy实例、调用服务方法、订阅事件/字段通知，完整开发流程如下：

#### 4.1 服务发现

ara::com 提供两种服务发现模式，适配不同场景：

- **一次性发现（FindService）**：单次调用，返回调用时刻可用的服务实例列表，适用于服务实例固定、无需动态感知服务上下线的场景；

- **持续发现（StartFindService）**：启动后台持续监听，服务实例上下线时通过回调实时通知，适用于需要动态感知服务可用性的场景。

#### 4.2 客户端完整实现示例

```C++

#include "RadarServiceProxy.h"
#include "ara/core/instance_specifier.h"

// 服务发现回调：服务实例上下线时触发
void ServiceAvailabilityCallback(ara::com::ServiceHandleContainer<radar::service::RadarServiceProxy::HandleType> handles) {
    if (handles.empty()) {
        // 服务实例下线
        return;
    }
    // 服务实例上线，创建Proxy实例
    auto radarProxy = std::make_shared<radar::service::RadarServiceProxy>(handles[0]);
    // 后续业务逻辑：调用方法、订阅事件等
}

int main() {
    // 1. 定义服务实例的逻辑标识（与Manifest配置对应）
    const ara::core::InstanceSpecifier instanceSpec{"radar/service/RadarServiceInstance"};

    // 2. 方式1：一次性服务发现
    auto findResult = radar::service::RadarServiceProxy::FindService(instanceSpec);
    if (!findResult.HasValue() || findResult.Value().empty()) {
        // 服务未发现，错误处理
        return -1;
    }
    // 创建Proxy实例
    auto radarProxy = std::make_shared<radar::service::RadarServiceProxy>(findResult.Value()[0]);

    // 2. 方式2：持续服务发现（推荐动态场景）
    auto findHandle = radar::service::RadarServiceProxy::StartFindService(
        ServiceAvailabilityCallback,
        instanceSpec
    );

    // 3. 调用服务Method：同步调用
    auto setModeResult = radarProxy->SetRadarMode(2).Get();
    if (setModeResult.HasValue()) {
        bool success = setModeResult.Value().success;
        // 处理调用结果
    }

    // 4. 调用服务Method：异步调用（非阻塞，推荐实时场景）
    radarProxy->SetRadarMode(1).then(
        [](ara::core::Future<radar::service::RadarService::SetRadarModeOutput> future) {
            auto result = future.GetResult();
            if (result.HasValue()) {
                // 异步处理调用结果
            }
        }
    );

    // 5. 订阅事件：PointCloudEvent
    // 5.1 设置事件接收回调
    radarProxy->PointCloudEvent.SetReceiveHandler(
        []() {
            // 回调触发，获取新的事件数据
            radarProxy->PointCloudEvent.GetNewSamples(
                [](const ara::com::SamplePtr<const PointCloudStruct>& sample) {
                    // 处理接收到的点云数据
                    ProcessPointCloud(*sample);
                }
            );
        }
    );
    // 5.2 发起事件订阅
    radarProxy->PointCloudEvent.Subscribe(10); // 缓存深度10

    // 6. 字段操作
    // 6.1 读取字段值（Getter）
    auto scanRateResult = radarProxy->ScanRate.Get().Get();
    if (scanRateResult.HasValue()) {
        uint32_t rate = scanRateResult.Value();
    }
    // 6.2 写入字段值（Setter）
    radarProxy->ScanRate.Set(50).Get();
    // 6.3 订阅字段变更通知（Notifier）
    radarProxy->ScanRate.SetReceiveHandler(
        []() {
            // 处理字段变更
        }
    );
    radarProxy->ScanRate.Subscribe(1);

    // 7. 订阅触发器
    radarProxy->ObstacleAlertTrigger.SetReceiveHandler(
        []() {
            // 处理障碍物告警触发
            DoObstacleAlert();
        }
    );
    radarProxy->ObstacleAlertTrigger.Subscribe(1);

    // 业务运行，保持进程存活
    while (true) {
        std::this_thread::sleep_for(std::chrono::seconds(1));
    }

    // 8. 资源释放
    radarProxy->PointCloudEvent.Unsubscribe();
    radar::service::RadarServiceProxy::StopFindService(findHandle);
    return 0;
}
```

#### 4.3 客户端核心API说明

|API|核心作用|使用场景|
|---|---|---|
|`FindService()`|一次性查询可用的服务实例列表|静态部署、服务实例固定的场景|
|`StartFindService()`|启动持续服务发现，通过回调实时感知服务上下线|动态部署、需要高可用的场景|
|`StopFindService()`|停止持续服务发现任务，释放监听资源|客户端退出前调用|
|`Event::Subscribe()`|订阅服务端事件/字段/触发器，设置接收缓存深度|准备接收事件数据前调用|
|`Event::Unsubscribe()`|取消订阅，停止接收事件推送|资源释放、业务停止时调用|
|`Event::SetReceiveHandler()`|设置事件接收回调，有新数据时自动触发|事件驱动编程模式|
|`Event::GetNewSamples()`|获取接收缓存中的新事件数据，支持回调遍历|轮询模式/回调中处理数据|
|`Method::operator()()`|发起方法调用，返回Future对象，支持同步/异步获取结果|客户端调用服务端方法|
|`Field::Get()/Set()`|读取/写入服务端字段值，通过Method实现|访问服务端状态属性|
## 五、核心通信机制深度解析

### 1. Method（方法）：请求-响应通信

Method是ara::com中实现远程函数调用的核心机制，分为三类：

- **同步调用**：客户端调用后阻塞等待，直到服务端返回结果，代码简洁，适用于非实时、低耗时的调用场景；

- **异步调用**：客户端调用后立即返回Future对象，不阻塞当前线程，通过then()注册回调处理结果，适用于高实时、高并发场景；

- **Fire&Forget（单向调用）**：客户端仅发送请求，不等待服务端响应，无返回值，适用于无需确认的指令发送场景，如控制指令下发。

**关键注意事项**：

- 方法调用支持应用错误上报，服务端可通过ErrorCode返回业务层面的错误码，客户端可统一处理；

- 异步调用支持取消操作，通过`Cancel()`接口取消未完成的方法调用，释放资源；

- 硬实时场景推荐使用异步调用+轮询Future结果，避免回调带来的上下文切换。

### 2. Event（事件）：发布-订阅通信

Event是ara::com中实现周期性数据、状态通知的核心机制，采用发布-订阅范式，核心特性如下：

- **接收缓存**：客户端订阅时指定缓存深度（CacheDepth），中间件会缓存指定数量的历史数据，避免数据丢失；

- **双访问模式**：

    - 事件驱动模式：通过`SetReceiveHandler()`设置回调，有新数据时自动触发，易用性高；

    - 轮询模式：通过`GetFreeSampleCount()`查询可用数据数量，主动调用`GetNewSamples()`获取数据，无上下文切换，适配硬实时场景；

- **零拷贝支持**：通过SamplePtr智能指针管理内存，避免应用层数据拷贝，大幅提升大体积数据（如点云、图像）的传输性能；

- **订阅状态监控**：通过`SetSubscriptionStateChangeHandler()`设置回调，实时感知订阅状态（已订阅/未订阅/订阅失败）。

### 3. Field（字段）：属性状态管理

Field是Method和Event的组合体，用于表示服务的可访问状态属性，三个可选能力可灵活组合：

- **Getter**：客户端主动读取字段当前值，由服务端实现Get方法，支持同步/异步调用；

- **Setter**：客户端主动修改字段值，由服务端实现Set方法，支持权限控制与参数校验；

- **Notifier**：字段值变更时，服务端自动向订阅客户端推送新值，复用Event的发布-订阅机制。

**典型使用场景**：

- 仅Getter：只读的服务状态属性，如设备版本号、当前工作状态；

- Getter+Setter：可读写的配置属性，如设备工作参数、阈值配置；

- 全能力：需要实时感知变更的可配置属性，如雷达扫描频率、摄像头曝光参数。

### 4. Trigger（触发器）：无数据通知

Trigger是无数据负载的特殊Event，仅用于通知客户端特定事件触发，不传输任何业务数据，核心特性：

- 复用Event的订阅-通知机制，API与Event高度一致；

- 无数据序列化/反序列化开销，传输效率极高；

- 适用于仅需触发动作、无需传递数据的场景，如告警触发、指令同步、状态跳转通知。

## 六、错误处理机制

ara::com 完全遵循AUTOSAR AP的标准化错误处理规范，基于`ara::core::Result`和`ara::core::ErrorCode`实现无异常的错误处理（适配车规级功能安全场景，禁用C++异常）。

### 1. 核心错误处理类

- **ara::core::Result<T>**：函数返回值的封装，包含成功时的返回值，或失败时的ErrorCode，通过`HasValue()`判断是否调用成功；

- **ara::core::ErrorCode**：错误信息封装，包含错误码、错误域、错误消息，唯一标识一个具体错误；

- **ara::com::ComErrorDomain**：ara::com专属错误域，定义了所有通信相关的标准错误码，如服务发现失败、订阅失败、方法调用超时、连接断开等；

- **ara::com::e2e::E2EErrorDomain**：E2E端到端保护相关错误域，定义了数据校验失败、数据丢失、序列异常等E2E错误码。

### 2. 标准错误处理范式

```C++

// 1. 同步调用错误处理
auto offerResult = radarService->OfferService();
if (!offerResult.HasValue()) {
    // 获取错误信息
    ara::core::ErrorCode error = offerResult.Error();
    // 打印错误信息，执行降级处理
    DLT_LOG(radarContext, DLT_LOG_ERROR,
        DLT_STRING("Offer service failed, error code: "),
        DLT_INT32(error.Value()));
    // 执行错误处理逻辑
    return -1;
}

// 2. 异步调用错误处理
radarProxy->SetRadarMode(1).then(
    [](ara::core::Future<radar::service::RadarService::SetRadarModeOutput> future) {
        auto result = future.GetResult();
        if (!result.HasValue()) {
            ara::core::ErrorCode error = result.Error();
            // 异步错误处理
            return;
        }
        // 正常处理结果
    }
);
```

### 3. 关键错误监控建议

- 服务端：监控服务发布/停止结果、方法调用异常、事件发送失败；

- 客户端：监控服务发现状态、订阅状态、方法调用超时/失败、事件数据丢失；

- 功能安全场景：必须监控E2E错误，针对校验失败、数据重复/丢失等错误执行对应的安全降级机制。

## 七、配置与部署

ara::com 的部署配置完全基于AUTOSAR AP的Manifest文件实现，实现了代码与部署配置的完全解耦，核心配置分为三类：

### 1. 服务接口Manifest

定义服务接口的元数据，包括服务ID、接口版本、通信原语的ID、数据类型序列化规则，由代码生成工具自动生成，与服务接口ARXML一一对应。

### 2. 服务实例部署Manifest

核心配置内容：

- InstanceSpecifier到InstanceIdentifier的映射，配置服务实例ID；

- 网络绑定配置：指定底层传输协议（SOME/IP/IPC/DDS）、服务端IP地址、端口号、协议版本；

- QoS配置：事件传输可靠性、方法调用超时时间、缓存深度、优先级配置；

- E2E保护配置：E2E Profile类型、数据ID、校验窗口、错误处理策略。

### 3. 客户端发现Manifest

配置客户端的服务发现策略：

- 服务发现模式：静态配置（固定服务地址）/动态发现（SOME/IP-SD）；

- 服务发现超时时间、重试策略；

- 服务版本兼容性规则，指定支持的服务主版本/次版本范围。

### 关键部署特性：Multi-Binding（多绑定）

ara::com 支持同一服务接口同时绑定多个底层传输协议，例如同一服务同时提供本地IPC绑定（用于同机内进程通信）和SOME/IP绑定（用于跨ECU通信），应用代码无需任何修改，仅通过Manifest配置即可实现，大幅提升通信性能与灵活性。

## 八、最佳实践与性能优化

### 1. 功能安全最佳实践

- 禁用C++异常，严格使用`ara::core::Result`进行错误处理，覆盖所有可能的错误分支；

- 关键通信必须启用E2E端到端保护，针对E2E错误定义明确的安全降级策略；

- 监控服务生命周期、订阅状态、通信健康度，异常时触发安全状态；

- 硬实时场景使用轮询模式，避免回调带来的不确定上下文切换，控制WCET。

### 2. 性能优化最佳实践

- **零拷贝传输**：大体积数据（点云、图像）使用`Allocate()`提前分配Sample内存，直接写入内存后发送，避免数据拷贝；

- **缓存策略优化**：根据事件发送频率、数据处理延迟，合理设置接收缓存深度，避免数据溢出丢失；

- **异步调用优先**：高并发场景优先使用异步方法调用，避免同步调用阻塞业务线程；

- **批量数据处理**：高频率事件数据，使用批量处理模式，减少回调触发次数与上下文切换；

- **同机通信优先使用IPC绑定**：同一ECU内的进程通信，优先使用共享内存IPC绑定，相比SOME/IP减少协议栈开销，大幅降低延迟。

### 3. 工程化最佳实践

- 代码中仅使用InstanceSpecifier，禁止硬编码服务实例ID、IP地址、端口号等部署信息；

- 封装Proxy/Skeleton的基础操作，实现统一的错误处理、日志记录、状态监控，避免重复代码；

- 服务端合理控制事件发送频率，避免高频小数据发送带来的协议栈开销；

- 客户端合理管理服务发现与订阅生命周期，进程退出前必须停止服务发现、取消订阅、释放Proxy实例，避免资源泄漏。

## 九、常见问题与排查思路

1. **服务发现失败，客户端找不到服务实例**

    - 排查思路：检查InstanceSpecifier是否与Manifest配置一致；检查服务端是否成功执行`OfferService()`；检查SOME/IP-SD服务发现配置是否正确，多播地址、端口是否互通；检查Manifest中的服务ID、版本号是否匹配。

2. **方法调用无响应/超时**

    - 排查思路：检查服务端是否正常运行，方法处理逻辑是否阻塞；检查方法调用超时配置是否合理；检查网络连通性，SOME/IP端口是否可访问；检查服务端方法处理模式，轮询模式下是否定期调用`ProcessNextMethodCall()`。

3. **事件订阅成功，但接收不到数据**

    - 排查思路：检查服务端是否成功调用`Send()`发送数据；检查客户端订阅的缓存深度是否合理，是否数据溢出被覆盖；检查客户端是否设置了接收回调，或主动调用`GetNewSamples()`获取数据；检查事件的QoS可靠性配置，是否为不可靠传输导致丢包。

4. **内存占用持续升高，出现内存泄漏**

    - 排查思路：检查客户端是否及时处理事件数据，是否未调用`GetNewSamples()`导致接收缓存持续堆积；检查SamplePtr是否被长期持有，未释放内存；检查是否未停止持续服务发现、未取消订阅，导致回调资源未释放；检查服务端是否频繁分配Sample内存，未复用内存池。
> （注：文档部分内容可能由 AI 生成）
