---
title: RAG 增强 SQLAgent
tags:
  - in-progress
date created: 2024年8月7日,星期三,下午,3:34:17
date modified: 2024年8月8日,星期四,晚上,6:24:06
---
# RAG 增强 SQLAgent

>[Enhancing SQL Agents with Retrieval Augmented Generation (RAG) | by Luc Nguyen | Medium](https://medium.com/@lucnguyen_61589/enhancing-sql-agents-with-retrieval-augmented-generation-rag-e20dbd8bb685)

- [背景](#%E8%83%8C%E6%99%AF)
- [技术内容描述](#%E6%8A%80%E6%9C%AF%E5%86%85%E5%AE%B9%E6%8F%8F%E8%BF%B0)
	- [1. RAG 增强 SQLAgent 的概念](#1.%20RAG%20%E5%A2%9E%E5%BC%BA%20SQLAgent%20%E7%9A%84%E6%A6%82%E5%BF%B5)
	- [2. Langchain 框架和 FAISS Vector DB](#2.%20Langchain%20%E6%A1%86%E6%9E%B6%E5%92%8C%20FAISS%20Vector%20DB)
- [代码编写流程](#%E4%BB%A3%E7%A0%81%E7%BC%96%E5%86%99%E6%B5%81%E7%A8%8B)
	- [1. 环境设置](#1.%20%E7%8E%AF%E5%A2%83%E8%AE%BE%E7%BD%AE)
	- [2. 准备文档](#2.%20%E5%87%86%E5%A4%87%E6%96%87%E6%A1%A3)
	- [3. 创建向量数据库](#3.%20%E5%88%9B%E5%BB%BA%E5%90%91%E9%87%8F%E6%95%B0%E6%8D%AE%E5%BA%93)
	- [4. 创建 SQLAgentRAGRetriever 检索工具](#4.%20%E5%88%9B%E5%BB%BA%20SQLAgentRAGRetriever%20%E6%A3%80%E7%B4%A2%E5%B7%A5%E5%85%B7)
	- [5. 重定义 SQLAgent](#5.%20%E9%87%8D%E5%AE%9A%E4%B9%89%20SQLAgent)
	- [6. 初始化工具包](#6.%20%E5%88%9D%E5%A7%8B%E5%8C%96%E5%B7%A5%E5%85%B7%E5%8C%85)
	- [7. 连接数据库](#7.%20%E8%BF%9E%E6%8E%A5%E6%95%B0%E6%8D%AE%E5%BA%93)
	- [8. 测试](#8.%20%E6%B5%8B%E8%AF%95)
- [结论](#%E7%BB%93%E8%AE%BA)
- [问题](#%E9%97%AE%E9%A2%98)
	- [1. 智谱 embedding](#1.%20%E6%99%BA%E8%B0%B1%20embedding)
	- [2. RAG 技术与 SQLAgent 的集成](#2.%20RAG%20%E6%8A%80%E6%9C%AF%E4%B8%8E%20SQLAgent%20%E7%9A%84%E9%9B%86%E6%88%90)
- [完整代码](#%E5%AE%8C%E6%95%B4%E4%BB%A3%E7%A0%81)

## 背景

在处理复杂的数据库查询任务时，尽管现有的 SQLAgent 可以处理标准的数据检索需求，但它们通常缺乏处理更复杂、更具语境性问题的能力。特别是在面对模型不理解的业务知识时，传统 SQLAgent 的功能就显得力不从心。在这种情况下，引入了 RAG（Retrieval Augmented Generation）技术，旨在通过补充外部知识来增强 SQLAgent 的智能，从而使其能够理解并处理更加复杂的查询。

例如，Supersonic 项目尝试通过建立知识图谱和链接到外部数据源来实现类似的功能，尽管之前的测试效果并不理想，但其基本理念为我们提供了宝贵的参考。比如，通过将“BBA”这一简称与“宝马（BMW）、奔驰（Mercedes-Benz）、奥迪（Audi）”这些品牌关联起来，帮助模型理解行业特定的术语。

## 技术内容描述

### 1. RAG 增强 SQLAgent 的概念

Retrieval Augmented Generation (RAG) 是一种结合了检索和生成的机器学习技术，它可以通过从相关文档中检索信息来辅助生成任务。在 SQLAgent 中应用 RAG 技术，可以使得模型在构建 SQL 查询时，能够参考和利用存储在 Vector DB 中的业务知识，从而生成更准确的 SQL 语句。

### 2. Langchain 框架和 FAISS Vector DB

Langchain 是一个支持多种语言技术整合的框架，允许开发者构建和部署基于语言模型的应用。FAISS（Facebook AI Similarity Search）是一个高效的相似性搜索库，用于构建 Vector DB，存储文档的向量表示。在 langchain 框架下，通过 FAISS 创建的 Vector DB 可以存储大量的业务知识，供 SQLAgent 在构建查询时调用。

## 代码编写流程

### 1. 环境设置

```python
import os
from urllib.parse import quote_plus as urlquote
from typing import Optional

from langchain.agents import tool
from langchain_openai import OpenAIEmbeddings, AzureOpenAIEmbeddings, ChatOpenAI
from langchain_community.vectorstores import FAISS
from langchain.prompts.chat import ChatPromptTemplate
from langchain_core.messages import AIMessage, HumanMessage, SystemMessage
from langchain_community.utilities import SQLDatabase
from langchain_community.agent_toolkits import SQLDatabaseToolkit
from langchain_core.prompts import ChatPromptTemplate
from langchain.tools import BaseTool
from langchain.callbacks.manager import (
    AsyncCallbackManagerForToolRun,
    CallbackManagerForToolRun,
)
from langgraph.prebuilt import create_react_agent
from langgraph.checkpoint.sqlite import SqliteSaver

# Set API keys and environment variables
os.environ['OPENAI_API_KEY'] = "20a8e1c4c5348d62eaaa64beabbd2fa0.34ZWDCTkvM5o7PDp"
os.environ['AZURE_OPENAI_API_KEY'] = "1140131bf4f94e75b3927e82589058af"
os.environ['AZURE_OPENAI_ENDPOINT'] = "https://oliverz11ai.openai.azure.com/"
```

- **导入库和模块：** 从各种库中导入必要的模块，包括 `langchain_openai`, `langchain.vectorstores`, `langchain.prompts.chat`, `langchain.llms.openai`, 以及其他相关的工具包和模块。
- **设置环境变量：** 使用 `os.environ` 设置 API 密钥和其他配置参数，包括 OpenAI 和 Azure 的 API 密钥和端点。
- **准备工作环境：** 设置所需的环境变量，确保代码能够正确访问 API 服务，并导入所有必要的库以支持后续的操作。

### 2. 准备文档

```python
autosight_knowledge = """
### BBA定义
- BBA指的是德国三大豪华汽车品牌：宝马（BMW）、奔驰（Mercedes-Benz）和奥迪（Audi）。
- bba：常用于非正式或口语交流中，指同样的品牌组合
- 德系三强：指的是同样的三个品牌，强调其在德国及全球市场的领先地位
- 豪华品牌集团：可能用于描述这三个品牌在高端市场的集合
"""
```

- **定义知识库：** 定义一个名为 `autosight_knowledge` 的字符串，其中包含关于汽车品牌和市场的知识。
- **提供语义检索的基础：** 该知识库将用于语义检索，以增强后续的 SQL 查询操作，支持 RAG 过程。

### 3. 创建向量数据库

```python
# Create Vector DB
technical_list = [autosight_knowledge]

# AzureOpenAIEmbeddings class parameters
embedding_function = AzureOpenAIEmbeddings(
    azure_deployment="text-embedding-3-small",
    openai_api_version="2023-05-15",
)

# Load it into FAISS
db = FAISS.from_texts(technical_list, embedding_function)
```

- **初始化知识列表：** 将 `autosight_knowledge` 包装成 `technical_list`。
- **设置嵌入函数：** 使用 `AzureOpenAIEmbeddings` 创建文本嵌入。
- **创建 FAISS 向量数据库：** 将知识列表嵌入并加载到 FAISS 向量数据库中。
- **实现快速检索：** FAISS（Facebook AI Similarity Search）向量数据库用于实现快速的语义检索，使得在大规模数据中查找相关信息更加高效。

### 4. 创建 SQLAgentRAGRetriever 检索工具

```python
# Create Tearadata Search Tool
retriever = db.as_retriever()

class SQLAgentRAGRetriever(BaseTool):
    name = "sqlagent_rag_retriever"
    description = "Input to this tool is a SQL query or keyword. Output is a retrieval of related context and definitions to enhance the understanding of the query using RAG."

    def _run(
        self, query: str, run_manager: Optional[CallbackManagerForToolRun] = None
    ) -> str:
        """Use the tool to retrieve relevant documents based on the query."""
        global retriever
        relevant_docs = retriever.get_relevant_documents(query)
        if len(relevant_docs) == 0 or len(query) == 0:
            return "No relevant context found for the provided query."
        else:
            return relevant_docs[0].page_content

    async def _arun(
        self, query: str, run_manager: Optional[AsyncCallbackManagerForToolRun] = None
    ) -> str:
        """Use the tool asynchronously to retrieve context. Currently not supported."""
        raise NotImplementedError("Asynchronous retrieval is not supported for this tool.")

sqlagent_rag_retriever = SQLAgentRAGRetriever()
```

- **转换为检索器：** 使用 `db.as_retriever()` 将 FAISS 数据库转换为检索器。
- **定义 SQLAgentRAGRetriever 类：** 创建一个工具类，用于从向量数据库中检索相关文档。
  - **`_run` 方法：** 实现同步检索，根据输入查询从知识库中检索相关内容。
  - **`_arun` 方法：** 定义异步检索方法，目前未实现。
- **增强查询理解：** 提供一个工具用于检索与查询相关的背景知识和定义，以增强对查询的理解，并支持 RAG 方法。

### 5. 重定义 SQLAgent

```python
# Redefine SQLAgent

# Create system message
def create_system_message_cn():
    """
    Create system message for guiding the agent on how to respond.
    """
    SQL_PREFIX_TEMPLATE = """
    角色：
    您是一个设计用来与SQL数据库交互的代理。您可以查询数据库并直接访问结果。

    目标：
    针对一个问题，首先进行RAG检索收集完整信息，然后创建一个SQL查询来检索所需的数据，执行此查询，并直接返回查询的结果，而不是查询本身。如果遇到错误或初始结果为空，请诊断问题，调整查询，并再次尝试。

    指示：
    - 首先进行RAG检索收集完整信息优化问题
    - 然后识别相关表格，以了解可用数据的范围。
    - 使用'sql_db_schema'工具获取这些表的结构详细信息。
    - 根据表结构信息构建一个适当的SQL查询来回答问题。
    - 使用'sql_db_query'工具执行查询。
    - 直接返回执行结果。除非用户明确请求，否则不返回SQL查询文本。
    - 如果查询执行成功但没有找到结果，请重新评估假设并相应修改查询。
    - 如果发生错误（例如，找不到表或语法错误），通过以下方式排除错误：
        - 检查表名和表结构。
        - 调整查询以修正任何错误。
        - 重新执行更正后的查询。
    - 仅关注检索与问题答案相关的列。
    - 默认情况下将结果限制为最多5个条目，除非用户指定了更多。
    - 按相关列排序结果，以首先呈现最相关的信息。
    - 不断调整和重新运行查询，直到获得有效结果或确认数据库中不存在所需数据。
    - 避免对数据库的结构或内容进行任何修改（例如，不进行INSERT、UPDATE、DELETE、DROP语句）。
    - 确保最终响应直接基于查询结果，提供简洁准确的答案。
    """
    return SystemMessage(content=SQL_PREFIX_TEMPLATE)
```

- **创建系统消息：** 定义两个函数 `create_system_message_v 2` 和 `create_system_message`，为 SQLAgent 提供操作指导。
- **指导代理行为：** 通过系统消息向代理提供关于如何处理 SQL 查询的详细指导，包括如何诊断和修复错误、调整查询，以及如何返回结果。

### 6. 初始化工具包

```python
# Initialize Toolkit
def initialize_toolkit(db, llm):
    """
    Initialize the toolkit for interacting with the database.
    Args:
        - db: Database connection object.
        - llm: Chat model object.
    """
    toolkit = SQLDatabaseToolkit(db=db, llm=llm)
    tools = toolkit.get_tools()

    # Query checking
    query_check_system = """
    You are a SQL expert with a strong attention to detail.
    Double check the SQLite query for common mistakes, including:
    - Using NOT IN with NULL values
    - Using UNION when UNION ALL should have been used
    - Using BETWEEN for exclusive ranges
    - Data type mismatch in predicates
    - Properly quoting identifiers
    - Using the correct number of arguments for functions
    - Casting to the correct data type
    - Using the proper columns for joins

    If there are any of the above mistakes, rewrite the query. If there are no mistakes, just reproduce the original query.

    Execute the correct query with the appropriate tool.
    """
    query_check_prompt = ChatPromptTemplate.from_messages([("system", query_check_system),("user", "{query}")])
    query_check = query_check_prompt | llm

    @tool
    def check_query_tool(query: str) -> str:
        """
        Use this tool to double check if your query is correct before executing it.
        """
        return query_check.invoke({"query": query}).content

    # Query result checking
    query_result_check_system = """You are grading the result of a SQL query from a DB.
    - Check that the result is not empty.
    - If it is empty, instruct the system to re-try!"""
    query_result_check_prompt = ChatPromptTemplate.from_messages([("system", query_result_check_system),("user", "{query_result}")])
    query_result_check = query_result_check_prompt | llm

    @tool
    def check_result(query_result: str) -> str:
        """
        Use this tool to check the query result from the database to confirm it is not empty and is relevant.
        """
        return query_result_check.invoke({"query_result": query_result}).content

    tools.append(check_query_tool)
    tools.append(check_result)
    tools.append(sqlagent_rag_retriever)
    return tools
```

- **初始化工具包：** 创建一个函数 `initialize_toolkit`，用于初始化与数据库交互的工具包。
- **查询检查：** 使用 `query_check_prompt` 创建一个工具来检查 SQL 查询中的常见错误。
- **查询结果检查：** 使用 `query_result_check_prompt` 创建一个工具来验证查询结果是否为空。
- **确保查询准确性：** 通过自动化工具检查 SQL 查询和查询结果，确保查询的准确性和有效性，减少手动检查的负担。

### 7. 连接数据库

```python
# Connect to the database
def connectToTheDatabase(connect_name = 'autosight_model'):
    # ClickHouse database connection parameters
    db_config = {
        'user': "autosight_read_only",
        'password': 'DFb7734HBdfBB33',
        'host': 'cc-2ze66w4q0tsjnd3b6.public.clickhouse.ads.aliyuncs.com',
        'port': 8123,
        'db': 'autosight_model',
        'db_type': 'clickhouse'
    }
    db_type = db_config['db_type']
    db_user = db_config['user']
    db_password = db_config['password']
    db_host = db_config['host']
    db_port = db_config['port']
    db_name = db_config['db']
    # db_name = connect_name
    db = SQLDatabase.from_uri(
        f'{db_type}://{db_user}:{urlquote(db_password)}@{db_host}:{db_port}/{db_name}?charset=utf8', view_support=True)
    return db
```

- **定义数据库连接参数：** 使用 `db_config` 字典定义数据库连接参数，包括用户、密码、主机、端口和数据库名称。
- **创建数据库连接：** 使用 `SQLDatabase. From_uri` 创建一个数据库连接对象。
- **建立数据库连接：** 通过提供数据库连接参数，确保代码能够正确连接到指定的数据库。

### 8. 测试

```python
# Test
if __name__ == "__main__":
    db_test = connectToTheDatabase()

    # Define LLM model
    chat_model = ChatOpenAI(
        model_name="GLM-4",
        openai_api_base="https://open.bigmodel.cn/api/paas/v4",
        streaming=True,
        verbose=True,
    )

    # Setting connection to db and SQL Toolkits
    tools = initialize_toolkit(db_test, chat_model)
    system_message = create_system_message_v2()
    memory = SqliteSaver.from_conn_string(":memory:")
    agent_executor = create_react_agent(chat_model, tools, state_modifier=system_message, checkpointer=memory)

    question = "BBA五月份的平均库存深度是多少？"
    
    config = {
        "configurable": {
            # Checkpoints accessed by thread ID
            "thread_id": 'bob',
        }
    }
    
    for chunk in agent_executor.stream(
        {"messages": [HumanMessage(content=question)]}, config
    ):
        print(chunk)
        print("----")
```

- **连接数据库：** 使用 `connectToTheDatabase` 函数建立数据库连接。
- **定义 LLM 模型：** 使用 `ChatOpenAI` 初始化聊天模型。
- **初始化工具和代理：** 调用 `initialize_toolkit` 和 `create_react_agent` 函数，准备好工具和代理执行器。
- **执行测试查询：** 定义一个测试问题，并通过 `agent_executor. Stream` 执行查询，输出结果。
- **验证系统功能：** 通过实际测试，验证系统的各个部分是否正常工作，包括数据库连接、查询执行、结果检索和输出。

## 结论

通过在 SQLAgent 中集成 RAG 技术和 langchain 框架下的 FAISS Vector DB，能够显著提升模型对业务知识的理解和查询的相关性。这种方法不仅增强了 SQL 查询的准确性，也为处理复杂的业务逻辑提供了强大的支持。

## 问题

### 1. 智谱 embedding

目前的实现中未能集成智谱的 embedding 模型，而是用的 Azure 的 embedding 模型，尚未清楚如何在 langchain 中调用智谱的 embedding 模型，有待查阅。

### 2. RAG 技术与 SQLAgent 的集成

尽管已经实现了使用 RAG 技术检索相关内容的功能，但似乎 SQLAgent 并未能有效记忆或利用这些检索到的知识来优化用户的查询问题。这表明在 RAG 技术与 SQLAgent 的集成过程中可能存在信息传递或持久化的问题。

目前考虑以下改进：
- **改进工具调用逻辑：** 确保在 SQLAgent 中调用相关工具（如 `SQLAgentRAGRetriever`）时，能够将检索到的信息有效地用于查询优化或问题重新表述。可以在工具调用后添加一个步骤，用于分析和整合检索结果，进一步细化或修改即将执行的 SQL 查询。
- 工具传参调用

## 完整代码

```python
import os
from urllib.parse import quote_plus as urlquote
from typing import Optional

from langchain.agents import tool
from langchain_openai import OpenAIEmbeddings, AzureOpenAIEmbeddings, ChatOpenAI
from langchain_community.vectorstores import FAISS
from langchain.prompts.chat import ChatPromptTemplate
from langchain_core.messages import AIMessage, HumanMessage, SystemMessage
from langchain_community.utilities import SQLDatabase
from langchain_community.agent_toolkits import SQLDatabaseToolkit
from langchain_core.prompts import ChatPromptTemplate
from langchain.tools import BaseTool
from langchain.callbacks.manager import (
    AsyncCallbackManagerForToolRun,
    CallbackManagerForToolRun,
)
from langgraph.prebuilt import create_react_agent
from langgraph.checkpoint.sqlite import SqliteSaver

# Set API keys and environment variables
os.environ['OPENAI_API_KEY'] = "20a8e1c4c5348d62eaaa64beabbd2fa0.34ZWDCTkvM5o7PDp"
os.environ['AZURE_OPENAI_API_KEY'] = "1140131bf4f94e75b3927e82589058af"
os.environ['AZURE_OPENAI_ENDPOINT'] = "https://oliverz11ai.openai.azure.com/"

# LangSmith
os.environ["LANGCHAIN_PROJECT"] = "Evaluate_SQLAgent_RAG"
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = "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"

# Prepare Documents
autosight_knowledge = """
### BBA定义
- BBA指的是德国三大豪华汽车品牌：宝马（BMW）、奔驰（Mercedes-Benz）和奥迪（Audi）。
- bba：常用于非正式或口语交流中，指同样的品牌组合
- 德系三强：指的是同样的三个品牌，强调其在德国及全球市场的领先地位
- 豪华品牌集团：可能用于描述这三个品牌在高端市场的集合
"""


# Create Vector DB
technical_list = [autosight_knowledge]

# AzureOpenAIEmbeddings class parameters
embedding_function = AzureOpenAIEmbeddings(
    azure_deployment="text-embedding-3-small",
    openai_api_version="2023-05-15",
)

# Load it into FAISS
db = FAISS.from_texts(technical_list, embedding_function)

# Create Tearadata Search Tool
retriever = db.as_retriever()

class SQLAgentRAGRetriever(BaseTool):
    name = "sqlagent_rag_retriever"
    description = "Input to this tool is a SQL query or keyword. Output is a retrieval of related context and definitions to enhance the understanding of the query using RAG."

    def _run(
        self, query: str, run_manager: Optional[CallbackManagerForToolRun] = None
    ) -> str:
        """Use the tool to retrieve relevant documents based on the query."""
        global retriever
        relevant_docs = retriever.invoke(query)
        if len(relevant_docs) == 0 or len(query) == 0:
            return "No relevant context found for the provided query."
        else:
            return relevant_docs[0].page_content

    async def _arun(
        self, query: str, run_manager: Optional[AsyncCallbackManagerForToolRun] = None
    ) -> str:
        """Use the tool asynchronously to retrieve context. Currently not supported."""
        raise NotImplementedError("Asynchronous retrieval is not supported for this tool.")

sqlagent_rag_retriever = SQLAgentRAGRetriever()

# Redefine SQLAgent

# Create system message
def create_system_message_v2():
    """
    Create system message for guiding the agent on how to respond.
    """
    SQL_PREFIX_TEMPLATE = """
    角色：
    您是一个设计用来与SQL数据库交互的代理。您可以查询数据库并直接访问结果。

    目标：
    针对一个问题，首先进行RAG检索收集完整信息，然后创建一个SQL查询来检索所需的数据，执行此查询，并直接返回查询的结果，而不是查询本身。如果遇到错误或初始结果为空，请诊断问题，调整查询，并再次尝试。

    指示：
    - 首先进行RAG检索收集完整信息
    - 根据检索信息优化问题
    - 然后识别相关表格，以了解可用数据的范围。
    - 使用'sql_db_schema'工具获取这些表的结构详细信息。
    - 根据表结构信息构建一个适当的SQL查询来回答问题。
    - 使用'sql_db_query'工具执行查询。
    - 直接返回执行结果。除非用户明确请求，否则不返回SQL查询文本。
    - 如果查询执行成功但没有找到结果，请重新评估假设并相应修改查询。
    - 如果发生错误（例如，找不到表或语法错误），通过以下方式排除错误：
        - 检查表名和表结构。
        - 调整查询以修正任何错误。
        - 重新执行更正后的查询。
    - 仅关注检索与问题答案相关的列。
    - 默认情况下将结果限制为最多5个条目，除非用户指定了更多。
    - 按相关列排序结果，以首先呈现最相关的信息。
    - 不断调整和重新运行查询，直到获得有效结果或确认数据库中不存在所需数据。
    - 避免对数据库的结构或内容进行任何修改（例如，不进行INSERT、UPDATE、DELETE、DROP语句）。
    - 确保最终响应直接基于查询结果，提供简洁准确的答案。
    """
    return SystemMessage(content=SQL_PREFIX_TEMPLATE)

def create_system_message():
    SQL_PREFIX_TEMPLATE = """
    ROLE:
    You are an agent designed to interact with a SQL database. You can query the database and directly access the results.

    GOAL:
    Given a question, create a SQL query to retrieve the needed data, execute this query, and directly return the results of the query, not the query itself. If errors are encountered or the initial results are empty, diagnose the issues, adjust the query, and try again.

    INSTRUCTIONS:
    - Begin by identifying the relevant tables to understand the scope of available data.
    - Use the 'sql_db_schema' tool to obtain detailed information about the structure of these tables.
    - Construct an appropriate SQL query to answer the question based on the schema information.
    - Execute the query using the 'sql_db_query' tool.
    - Directly return the results from the execution. Do not return the SQL query text unless requested specifically by the user.
    - If the query execution is successful but no results are found, reassess the assumptions and revise the query accordingly.
    - If an error occurs (e.g., table not found or syntax error), troubleshoot the error by:
        - Reviewing the table names and schema.
        - Adjusting the query to fix any errors.
        - Re-executing the corrected query.
    - Focus only on retrieving relevant columns to answer the question.
    - Limit the results to a maximum of 5 entries by default, unless more is specified by the user.
    - Sort the results by relevant columns to present the most pertinent information first.
    - Continuously adjust and re-run the query until valid results are obtained or confirm that the data needed does not exist in the database.
    - Avoid making any modifications to the database's structure or content (e.g., no INSERT, UPDATE, DELETE, DROP statements).
    - Ensure the final response is based directly on the query results, providing concise and accurate answers.
"""
    return SystemMessage(content=SQL_PREFIX_TEMPLATE)

# Initialize Toolkit
def initialize_toolkit(db, llm):
    """
    Initialize the toolkit for interacting with the database.
    Args:
        - db: Database connection object.
        - llm: Chat model object.
    """
    toolkit = SQLDatabaseToolkit(db=db, llm=llm)
    tools = toolkit.get_tools()

    # Query checking
    query_check_system = """
    You are a SQL expert with a strong attention to detail.
    Double check the SQLite query for common mistakes, including:
    - Using NOT IN with NULL values
    - Using UNION when UNION ALL should have been used
    - Using BETWEEN for exclusive ranges
    - Data type mismatch in predicates
    - Properly quoting identifiers
    - Using the correct number of arguments for functions
    - Casting to the correct data type
    - Using the proper columns for joins

    If there are any of the above mistakes, rewrite the query. If there are no mistakes, just reproduce the original query.

    Execute the correct query with the appropriate tool.
    """
    query_check_prompt = ChatPromptTemplate.from_messages([("system", query_check_system),("user", "{query}")])
    query_check = query_check_prompt | llm

    @tool
    def check_query_tool(query: str) -> str:
        """
        Use this tool to double check if your query is correct before executing it.
        """
        return query_check.invoke({"query": query}).content

    # Query result checking
    query_result_check_system = """You are grading the result of a SQL query from a DB.
    - Check that the result is not empty.
    - If it is empty, instruct the system to re-try!"""
    query_result_check_prompt = ChatPromptTemplate.from_messages([("system", query_result_check_system),("user", "{query_result}")])
    query_result_check = query_result_check_prompt | llm

    @tool
    def check_result(query_result: str) -> str:
        """
        Use this tool to check the query result from the database to confirm it is not empty and is relevant.
        """
        return query_result_check.invoke({"query_result": query_result}).content

    tools.append(check_query_tool)
    tools.append(check_result)
    tools.append(sqlagent_rag_retriever)
    return tools

# Connect to the database
def connectToTheDatabase(connect_name = 'autosight_model'):
    # ClickHouse database connection parameters
    db_config = {
        'user': "autosight_read_only",
        'password': 'DFb7734HBdfBB33',
        'host': 'cc-2ze66w4q0tsjnd3b6.public.clickhouse.ads.aliyuncs.com',
        'port': 8123,
        'db': 'autosight_model',
        'db_type': 'clickhouse'
    }
    db_type = db_config['db_type']
    db_user = db_config['user']
    db_password = db_config['password']
    db_host = db_config['host']
    db_port = db_config['port']
    db_name = db_config['db']
    # db_name = connect_name
    db = SQLDatabase.from_uri(
        f'{db_type}://{db_user}:{urlquote(db_password)}@{db_host}:{db_port}/{db_name}?charset=utf8', view_support=True)
    return db

# Test
if __name__ == "__main__":
    db_test = connectToTheDatabase()

    # Define LLM model
    chat_model = ChatOpenAI(
        model_name="GLM-4",
        openai_api_base="https://open.bigmodel.cn/api/paas/v4",
        streaming=True,
        verbose=True,
    )

    # Setting connection to db and SQL Toolkits
    tools = initialize_toolkit(db_test, chat_model)
    system_message = create_system_message_v2()
    memory = SqliteSaver.from_conn_string(":memory:")
    agent_executor = create_react_agent(chat_model, tools, state_modifier=system_message, checkpointer=memory)

    question = "BBA是什么？其五月份的平均库存深度是多少？"
    
    config = {
        "configurable": {
            # Checkpoints accessed by thread ID
            "thread_id": 'bob',
        }
    }
    
    for chunk in agent_executor.stream(
        {"messages": [HumanMessage(content=question)]}, config
    ):
        print(chunk)
        print("----")

```