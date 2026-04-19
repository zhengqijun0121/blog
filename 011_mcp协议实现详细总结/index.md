# MCP 协议详细总结


# MCP（Model Context Protocol）协议实现详细总结
## 一、MCP核心定义与定位
### 1. 基础定义
MCP（Model Context Protocol，模型上下文协议）是由Anthropic牵头、多家AI厂商联合推出的**开源标准化通信协议**，基于JSON-RPC 2.0构建，为大语言模型(LLM)应用与外部工具、数据源、系统服务之间提供统一、安全、可互操作的通信标准，彻底解决了传统Function Calling碎片化开发、多平台适配成本高、安全边界模糊、能力复用性差的行业痛点，实现了“一次开发，全生态兼容”的能力复用。

### 2. 核心价值（与传统Function Calling的核心区别）
| 维度 | MCP | 传统Function Calling |
|------|-----|------------------------|
| 核心目标 | 标准化LLM与外部系统的通信协议，实现跨平台互操作 | 解决单一场景下LLM与特定工具的调用逻辑 |
| 实现方式 | Client-Server解耦架构，基于统一协议规范 | 与特定模型/应用强绑定，无通用标准 |
| 开发成本 | 一次开发，多宿主（Claude/Cursor/Windsurf等）兼容 | 需为每个应用/模型单独开发适配 |
| 能力边界 | 支持工具调用、资源访问、提示模板、采样回调四大核心原语 | 仅支持基础函数调用 |
| 安全机制 | 内置权限隔离、用户同意、沙箱机制，明确安全边界 | 无统一安全规范，需开发者自行实现 |
| 复用性 | 高，Server可跨场景、跨宿主复用 | 低，函数通常为特定任务/场景定制 |

## 二、MCP核心架构与核心规范
### 1. 三层核心架构
MCP采用**Host-Client-Server三层解耦架构**，核心为Client-Server CS模式，每个Host可管理多个Client实例，每个Client与Server保持**1:1独占的有状态长连接**。

| 核心角色 | 定义与典型代表 | 核心职责 |
|----------|----------------|----------|
| MCP Host（宿主） | 承载LLM能力的AI应用，协议发起方与消费方<br>典型代表：Claude Desktop、Cursor IDE、自研AI平台 | 1. 管理多个Client生命周期，协调多Server连接<br>2. 对接LLM，将模型意图转化为MCP标准指令<br>3. 负责用户交互与权限审批，确保操作获得用户知情同意<br>4. 接收Server返回结果，注入模型上下文完成推理闭环 |
| MCP Client（客户端） | 嵌入Host内部的通信中间件，Host与Server的唯一桥梁 | 1. 与对应Server建立并维护长连接，处理保活与异常重连<br>2. 完成协议序列化/反序列化，实现Host与Server的消息双向路由<br>3. 处理初始化握手与能力协商，校验协议版本与能力集<br>4. 管理资源订阅、通知推送与会话状态，维护Server间安全隔离 |
| MCP Server（服务端） | 实现MCP协议、封装并暴露特定能力的轻量级程序<br>部署形态：本地子进程、远程HTTP服务、容器化服务 | 1. 通过标准原语对外暴露工具、资源、提示模板三类能力<br>2. 实现业务逻辑，对接数据库、API、文件系统等底层系统<br>3. 响应Client请求，处理能力发现与调用，遵守协议安全约束<br>4. 可主动发起采样请求，反向回调Host的LLM能力<br>5. 维护会话状态，处理资源变更通知与权限校验 |

### 2. 底层协议规范
MCP完全基于**JSON-RPC 2.0**标准构建，定义了三种核心消息类型，所有通信严格遵循该规范：
1.  **请求（Request）**：带唯一ID、方法名、参数的双向消息，接收方必须返回对应响应
2.  **响应（Response）**：匹配对应请求ID的消息，包含成功结果或标准错误对象
3.  **通知（Notification）**：无ID的单向消息，无需接收方响应，用于事件推送、状态变更

### 3. 能力协商机制
MCP采用**声明式能力协商**，在初始化握手阶段，Client与Server必须明确声明自身支持的能力集，会话期间仅可使用双方共同支持的协议功能，禁止使用未声明的能力：
- Server端声明：资源订阅、工具调用、提示模板、日志推送等能力
- Client端声明：采样回调、通知处理、根证书校验等能力

### 4. 四大核心原语
MCP定义了四个标准化能力单元，覆盖LLM与外部系统交互的全场景需求：
1.  **工具（Tool）**：可被LLM调用的执行函数，最常用的原语，支持入参JSON Schema校验，对应标准化的Function Calling能力
2.  **资源（Resource）**：可被LLM读取的上下文数据，支持URI寻址、内容更新通知、增量订阅，如文件内容、数据库表、实时数据
3.  **提示（Prompt）**：标准化的提示词模板，支持参数注入，可被LLM直接调用，实现复杂prompt的复用与标准化
4.  **采样（Sampling）**：Server端主动向Client发起的LLM调用请求，实现反向回调，让服务端可按需使用大模型能力

## 三、MCP完整通信生命周期
MCP的全流程通信分为4个核心阶段，严格遵循协议规范执行：

### 1. 初始化握手与能力协商（必选）
这是会话建立的前提，未完成握手前禁止发送业务请求，步骤如下：
1.  Client向Server发送`initialize`请求，携带协议版本、Client支持的能力集、客户端信息
2.  Server校验协议版本兼容性，返回`initialize`响应，携带Server能力集、服务端信息、版本确认
3.  Client验证能力集匹配性，向Server发送`initialized`通知，确认会话初始化完成
4.  握手完成，双方进入正常会话状态

### 2. 能力发现阶段（按需触发）
LLM需要获取Server可用能力时，Client发起标准化发现请求：
1.  发送`tools/list`请求，获取工具列表（含名称、描述、入参JSON Schema）
2.  发送`resources/list`请求，获取资源列表（含URI、名称、描述、MIME类型）
3.  发送`prompts/list`请求，获取提示模板列表（含名称、描述、参数定义）
4.  Host将所有能力合并到统一注册表，注入LLM系统提示，让模型理解可用操作

### 3. 核心能力调用流程（核心业务）
以最常用的工具调用为例，完整闭环流程如下：
1.  用户发起提问，LLM判断需要调用MCP工具，生成工具调用指令
2.  Host将指令下发给对应Client，Client封装为`tools/call`标准JSON-RPC请求，发送给Server
3.  Server接收请求后，先校验入参是否符合JSON Schema，再执行工具业务逻辑
4.  Server执行完成后，返回`tools/call`响应，携带执行结果或错误信息
5.  Client将响应回传给Host，Host将结果注入LLM上下文
6.  LLM基于工具返回结果，生成最终自然语言回答，完成调用闭环

### 4. 会话管理与销毁
- 会话保活：双方通过心跳通知维持长连接，检测连接健康状态
- 增量通知：订阅的资源内容变更时，Server主动发送`resources/updated`通知
- 会话销毁：连接断开时清理会话状态；Client可主动发送关闭通知优雅终止会话；异常断开时触发自动重连

## 四、MCP服务端完整实现
服务端是MCP能力的核心载体，也是开发者最常开发的部分，主流实现基于官方SDK，以Python FastMCP（最易用）和TypeScript SDK为主。

### 1. 实现前置准备
- 开发环境：Python 3.10+ / Node.js 18+
- 依赖安装：Python环境执行`pip install mcp`，Node.js环境执行`npm install @modelcontextprotocol/sdk`
- 需求明确：确定Server要暴露的能力、对接的底层系统，选择适配的传输方式（本地选stdio，远程选SSE+HTTP）

### 2. 标准实现步骤
1.  实例化MCP服务端对象，配置服务名称、描述、版本等基础信息
2.  定义并实现核心能力：通过装饰器/注册器声明工具、资源、提示模板
3.  补充完整元信息：名称、精准的描述、入参Schema、参数定义等，直接决定LLM调用准确率
4.  实现业务逻辑：对接底层系统、处理异常、格式化LLM友好的返回结果
5.  配置传输方式，启动服务端，监听Client连接
6.  测试验证：在支持MCP的Host中配置Server，验证能力调用的兼容性与准确性

### 3. 完整可运行代码示例（Python FastMCP，stdio传输）
```python
from mcp.server.fastmcp import FastMCP, Context
from typing import Optional
import os
import httpx

# 1. 实例化FastMCP服务端，配置基础信息
mcp = FastMCP(
    name="demo-mcp-server",
    description="演示MCP服务端，包含文件读取、天气查询、提示模板能力",
    version="1.0.0"
)

# 2. 定义工具：天气查询（对接第三方API）
@mcp.tool()
async def query_weather(city: str, days: Optional[int] = 3) -> str:
    """
    查询指定城市的未来几天的天气预报
    :param city: 要查询的城市名称，必填
    :param days: 预报天数，默认3天，最大7天
    :return: 格式化的天气预报文本
    """
    # 入参合法性校验
    if days < 1 or days > 7:
        return "错误：预报天数必须在1-7天之间"

    # 第三方API调用逻辑
    try:
        async with httpx.AsyncClient() as client:
            response = await client.get(
                f"https://api.open-meteo.com/v1/forecast?city={city}&forecast_days={days}",
                timeout=10
            )
            response.raise_for_status()
            data = response.json()
            return f"【{city}未来{days}天天气预报】\n{str(data)}"
    except Exception as e:
        return f"天气查询失败：{str(e)}"

# 3. 定义工具：本地文本文件读取
@mcp.tool()
def read_local_file(file_path: str, ctx: Context) -> str:
    """
    读取本地指定路径的文本文件内容
    :param file_path: 本地文件的绝对路径
    :param ctx: MCP上下文对象，用于日志输出
    :return: 文件内容文本
    """
    ctx.info(f"收到文件读取请求：{file_path}")
    # 安全校验：防止目录遍历攻击
    if not os.path.isabs(file_path):
        return "错误：必须提供文件的绝对路径"
    try:
        with open(file_path, "r", encoding="utf-8") as f:
            return f.read()
    except Exception as e:
        ctx.error(f"文件读取失败：{str(e)}")
        return f"文件读取失败：{str(e)}"

# 4. 定义资源：系统基础信息，支持LLM读取
@mcp.resource(uri="demo://system/info", name="系统信息", description="当前服务的基础系统信息")
def get_system_info() -> str:
    return f"""
    操作系统：{os.name}
    工作目录：{os.getcwd()}
    CPU核心数：{os.cpu_count()}
    服务版本：1.0.0
    """

# 5. 定义提示模板：标准化代码评审模板，支持参数注入
@mcp.prompt(
    name="code_review",
    description="标准化的代码评审提示模板",
    arguments=[
        {"name": "code", "description": "要评审的代码内容", "required": True},
        {"name": "language", "description": "代码的编程语言", "required": False, "default": "Python"}
    ]
)
def code_review_prompt(code: str, language: str = "Python") -> str:
    return f"""
    你是一名资深的{language}代码评审专家，请对以下代码进行全面评审，输出格式如下：
    1. 代码亮点
    2. 潜在问题与风险
    3. 优化建议
    4. 可直接使用的优化后代码

    待评审代码：
    ```{language}
    {code}
    ```
    """

# 6. 启动服务端，使用stdio传输（本地场景标准方案）
if __name__ == "__main__":
    mcp.run(transport="stdio")
```

### 4. 多语言实现支持
官方与社区提供了主流开发语言的SDK，覆盖不同业务场景：
- Python：`mcp` 包，提供FastMCP高阶封装，开发效率最高，适合快速原型与轻量服务
- TypeScript/JavaScript：`@modelcontextprotocol/sdk`，适配Node.js生态与前端相关服务
- Go/Java/Kotlin：社区官方兼容SDK，适合高性能、企业级后端服务开发

## 五、MCP客户端实现
客户端通常由AI应用（Host）厂商实现，普通开发者仅在自研AI平台需要集成MCP能力时，才需要开发客户端。

### 1. 极简客户端代码示例（Python）
```python
import asyncio
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client

async def main():
    # 1. 配置目标Server的启动参数
    server_params = StdioServerParameters(
        command="python",
        args=["demo_mcp_server.py"]  # 上述服务端代码的文件路径
    )

    # 2. 建立stdio连接，创建客户端会话
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(read, write) as session:
            # 3. 执行初始化握手
            await session.initialize()
            print("✅ 初始化握手完成，会话建立成功")

            # 4. 能力发现：获取工具列表
            tools = await session.list_tools()
            print(f"📋 可用工具：{[tool.name for tool in tools.tools]}")

            # 5. 调用工具：查询上海天气
            result = await session.call_tool(
                tool_name="query_weather",
                arguments={"city": "上海", "days": 3}
            )
            print(f"🔧 工具调用结果：{result.content}")

            # 6. 读取资源
            resource = await session.read_resource(uri="demo://system/info")
            print(f"📦 资源读取结果：{resource.contents}")

            # 7. 获取渲染后的提示模板
            prompt = await session.get_prompt(
                name="code_review",
                arguments={"code": "print('hello world')", "language": "Python"}
            )
            print(f"📝 提示模板结果：{prompt.messages}")

if __name__ == "__main__":
    asyncio.run(main())
```

## 六、传输层实现方案
MCP传输层与业务逻辑完全解耦，官方支持3种主流传输方式，适配不同部署场景：

| 传输方式 | 实现原理 | 适用场景 | 核心优势 | 注意事项 |
|----------|----------|----------|----------|----------|
| stdio（标准输入输出） | Client通过子进程启动Server，通过进程stdin/stdout进行二进制流通信 | 本地部署场景，Claude Desktop/Cursor等本地AI应用 | 开发简单、无端口暴露、安全性高、延迟极低 | 禁止向stdout打印业务日志（需用stderr），仅支持单客户端连接 |
| SSE+HTTP | Client通过HTTP POST发送请求，通过SSE长连接接收服务端响应与通知 | 远程部署、企业内网/云端服务 | 支持跨网络访问、多客户端并发、兼容HTTP网关/鉴权/负载均衡 | 必须配置CORS、HTTPS加密、身份认证，处理网络重连 |
| WebSocket | 基于WebSocket全双工长连接实现双向通信 | 实时性要求高、双向消息频繁的场景（如实时数据推送） | 全双工通信、低延迟、支持双向实时推送 | 官方原生支持度低于前两种，需自行实现心跳保活与消息顺序保证 |

## 七、安全机制实现
MCP内置多层安全机制，开发者必须严格落地，避免安全风险：
1.  **权限最小化**：Server仅暴露必要能力，工具/资源仅开放最小必要权限，禁止过度授权
2.  **用户知情同意**：Host必须在敏感操作前获得用户明确同意，禁止静默执行高危操作（如文件写入、命令执行）
3.  **输入校验与防注入**：所有工具入参必须做严格校验，防范SQL注入、命令注入、目录遍历等攻击
4.  **安全隔离**：每个Server连接相互隔离，本地Server使用沙箱限制资源访问范围
5.  **身份认证**：远程Server必须实现API Key、OAuth2.0、JWT等身份认证，禁止匿名访问
6.  **传输加密**：远程传输必须使用HTTPS/WSS加密，禁止明文传输敏感数据
7.  **审计日志**：所有能力调用必须记录完整审计日志，包含调用方、时间、参数、结果等，便于溯源
8.  **异常熔断**：针对高频调用、高失败率请求实现熔断机制，避免服务被滥用

## 八、最佳实践与避坑指南
### 最佳实践
1.  **元信息极致优化**：工具/资源的description必须精准完整，清晰说明用途、适用场景、入参含义，直接决定LLM调用准确率
2.  **单一职责原则**：每个Server专注于一个业务领域，避免集成过多无关能力，降低维护成本与安全风险
3.  **LLM友好的结果格式化**：返回结果结构化、简洁，避免冗余数据，减少上下文token消耗
4.  **完善的异常处理**：所有业务逻辑必须捕获异常，返回清晰的错误信息，帮助LLM理解失败原因并调整调用方式
5.  **语义化版本管理**：Server遵循语义化版本规范，能力变更时同步更新版本号，避免兼容性问题

### 常见避坑指南
1.  **初始化握手失败**：常见原因是协议版本不兼容、能力集声明格式错误、消息帧格式不符合规范
2.  **LLM不调用工具**：90%的原因是工具description不清晰、入参Schema定义不完整、参数描述模糊
3.  **stdio连接失败**：常见原因是Server启动命令错误、依赖缺失、stdout被业务日志占用
4.  **工具调用入参错误**：常见原因是JSON Schema定义不严格，Server端未做入参前置校验
5.  **远程Server访问失败**：常见原因是CORS配置错误、身份认证失败、HTTPS证书无效

## 九、典型应用场景
1.  **开发工具集成**：IDE（Cursor/Windsurf）通过MCP对接Git、Jira、数据库、云服务，实现AI全流程开发
2.  **企业数据集成**：通过MCP Server封装内部数据库、CRM、ERP系统，让AI助手安全访问内部数据，实现数据分析与报表生成
3.  **智能运维**：对接监控系统、云平台、K8s集群，实现AI故障排查、资源调度、自动化运维
4.  **生产力工具集成**：对接飞书、钉钉、Notion、Slack，实现AI自动化日程管理、文档处理、消息通知
5.  **专业领域能力封装**：封装金融行情、医疗数据、法律知识库等行业能力，打造专属行业AI助手
6.  **本地设备控制**：对接智能家居、物联网设备，实现AI语音控制与设备自动化


