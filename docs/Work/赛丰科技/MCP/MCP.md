---
date created: 2025/5/14 9:27
date modified: 2025/5/15 15:39
---
### 通用架构

MCP 核心采用客户端-服务器架构，主机应用可以连接多个服务器：

![[IMG-2025-05-15-15-10-46-1.png]]

- **MCP Hosts**: 如 Claude Desktop、IDE 或 AI 工具，希望通过 MCP 访问数据的程序
- **MCP Clients**: 维护与服务器一对一连接的协议客户端
- **MCP Servers**: 轻量级程序，通过标准的 Model Context Protocol 提供特定能力
- **本地数据源**: MCP 服务器可安全访问的计算机文件、数据库和服务
- **远程服务**: MCP 服务器可连接的互联网上的外部系统（如通过 APIs）

## 概述

MCP 遵循一个 client-server 架构，其中：

- **Hosts** 是 LLM 应用（如 Claude Desktop 或 IDEs），它们发起连接
- **Clients** 在 host 应用中与 servers 保持 1:1 的连接
- **Servers** 为 clients 提供上下文、tools 和 prompts

[[IMG-2025-05-15-15-10-46.png]]

![[IMG-2025-05-15-15-10-46-2.png]]

### MCP Client

MCP client 充当 LLM 和 MCP server 之间的桥梁，MCP client 的工作流程如下：

- MCP client 首先从 MCP server 获取可用的工具列表。
- 将用户的查询连同工具描述通过 function calling 一起发送给 LLM。
- LLM 决定是否需要使用工具以及使用哪些工具。
- 如果需要使用工具，MCP client 会通过 MCP server 执行相应的工具调用。
- 工具调用的结果会被发送回 LLM。
- LLM 基于所有信息生成自然语言响应。
- 最后将响应展示给用户。

Claude Desktop 和Cursor都支持了MCP Server接入能力，它们就是作为 MCP client来连接某个MCP Server感知和实现调用。

### MCP Server

MCP server 是 MCP 架构中的关键组件，它可以提供 3 种主要类型的功能：

- 资源（Resources）：类似文件的数据，可以被客户端读取，如 API 响应或文件内容。
- 工具（Tools）：可以被 LLM 调用的函数（需要用户批准）。
- 提示（Prompts）：预先编写的模板，帮助用户完成特定任务。

这些功能使 MCP server 能够为 AI 应用提供丰富的上下文信息和操作能力，从而增强 LLM 的实用性和灵活性。

### MCP 主要解决的问题

#### 1. **模型与外部数据和工具的集成困难**

传统上，大模型在访问外部数据源和工具时，面临接口不统一、集成复杂等问题。MCP 提供了一套标准化的通信协议，使得不同模型能够以统一的方式调用各种外部服务和数据源，简化了集成过程。

#### 2. **开发者生态的碎片化**

在 MCP 出现之前，开发者需要针对不同模型和工具编写特定的集成代码，增加了开发成本。MCP 通过定义统一的接口规范，降低了开发者的集成门槛，促进了工具和模型之间的互操作性。

#### 3. **模型调用外部工具的能力有限**

虽然大模型具备强大的语言处理能力，但在调用 [**外部工具**](https://github.com/modelcontextprotocol/servers) 时，缺乏标准化的机制。MCP 引入了类似 Function Calling 的机制，使模型能够根据用户需求自动决定是否调用外部工具，并处理返回结果，从而增强了模型的实用性。

- **[Metricool MCP](https://github.com/metricool/mcp-metricool)** - 模型上下文协议服务器，与 Metricool 的社交媒体分析平台集成，可检索性能指标并安排 Instagram、Facebook、Twitter、LinkedIn、TikTok 和 YouTube 等网络的内容。
- **[HDW LinkedIn](https://github.com/horizondatawave/hdw-mcp-server)** - Access to profile data and management of user account with [HorizonDataWave.ai](https://horizondatawave.ai/).
- **[X (Twitter)](https://github.com/EnesCinr/twitter-mcp)** (by EnesCinr) - 与 Twitter API 互动。发布推文并根据查询搜索推文。
- **[X (Twitter)](https://github.com/vidhupv/x-mcp)** (by vidhupv) - 通过 Claude 聊天创建、管理和发布 X/Twitter 状态。
- **[微软 Azure](https://github.com/Azure/azure-mcp)** - 微软 Azure MCP 服务器为 MCP 客户端提供 Azure 存储、Cosmos DB、Azure CLI 等关键 Azure 服务和工具的访问权限。

## 数据源集成与扩展

### **常见数据源类型**

在 MCP 体系中，**Resources** 作为模型的基础数据支撑，可以来自多种数据源，主要包括：

1. **结构化数据库**
	
	- **典型数据库**：PostgreSQL、MySQL、SQL Server
	- **应用场景**：存储高度规范化的数据，适用于财务记录、客户信息、库存管理等任务。
		
2. **非结构化数据源**
	
	- **常见系统**：知识管理系统（KM）、Elasticsearch、MongoDB
	- **应用场景**：支持全文检索和大规模非结构化数据存储，广泛用于文档管理和日志分析。
		
3. **文件系统**
	
	- **支持格式**：CSV、JSON、XML
	- **应用场景**：存储轻量级数据，便于导入导出和跨系统数据交换。
		
4. **RESTful API**
	
	- **数据来源**：第三方服务接口，如天气预报、地图服务、财务数据等。
	- **应用场景**：实时获取动态数据，实现系统间的信息交互。

### 2.2 MCP 中的数据源集成

- MCP 支持多种数据源的统一管理，可以通过 Resource 接口进行扩展。
- **示例**：

	```python
    from mcp.types import Resource
    
    app = Server("resource search")
    
    @app.list_resources()
    async def list_resources() -> list[Resource]:
        return [
            Resource(uri="postgres://database/customers/schema", name="单位测试数据库", mimeType="multipart/form-data"),
            Resource(uri="km://zhiku", name="智库", mimeType="application/json"),
            Resource(uri="km://zhishiku", name="知识库", mimeType="application/json")
        ]
    ```