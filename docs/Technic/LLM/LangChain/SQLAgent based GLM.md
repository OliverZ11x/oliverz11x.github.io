---
title: SQLAgent based GLM
date created: 2024年7月31日,星期三,上午,11:15:58
date modified: 2024年8月7日,星期三,晚上,6:01:36
---
# 使用手册：与 SQL 数据库交互的 Agent

## 目录

- [简介](#简介)
- [环境配置](#环境配置)
- [代码详解](#代码详解)
	- [设置环境变量](#设置环境变量)
	- [初始化数据库](#初始化数据库)
	- [初始化聊天模型](#初始化聊天模型)
	- [初始化工具包](#初始化工具包)
	- [创建代理执行器](#创建代理执行器)
	- [创建提示模板](#创建提示模板)
- [运行主程序](#运行主程序)
- [结果测试](#结果测试)

## 简介

本文档旨在指导用户如何配置和使用一个与 SQL 数据库交互的 Agent。该 Agent 基于 LangChain 框架，通过自然语言与 SQLite 数据库进行交互。

## 环境配置

在开始使用该程序之前，请确保已正确设置环境变量并安装所需依赖项。

### 安装依赖

请使用以下命令安装所需的 Python 依赖项：

```bash
pip install --upgrade langchain langchain-community langchain-openai langgraph faiss-cpu

pip install PyJWT httpx-sse
```

### 设置环境变量

在运行程序之前，需要设置 API 密钥。您可以通过修改以下代码片段来设置环境变量：

```python
def set_environment_variables():
    """
    设置环境变量，包括API密钥。
    """
    os.environ["ZHIPUAI_API_KEY"] = "20a8e1c4c5348d62eaaa64beabbd2fa0.34ZWDCTkvM5o7PDp"
    # os.environ["OPENAI_API_KEY"] = "your_openai_api_key"
```

## 代码详解

### 初始化数据库

该函数用于初始化 SQLite 数据库连接，并打印数据库方言及可用表名。

```python
def initialize_database():
    """
    初始化数据库连接。
    """
    db = SQLDatabase.from_uri("sqlite:///Chinook.db")
    print(db.dialect)
    print(db.get_usable_table_names())
    db.run("SELECT * FROM Artist LIMIT 10;")
    return db
```

### 初始化聊天模型

该函数用于初始化聊天模型，这里使用的是 `ChatZhipuAI` 模型。

```python
def initialize_chat_model(api_key: str):
    """
    初始化聊天模型。
    参数：
        - api_key: str，API密钥。
    """
    chat_model = ChatOpenAI(
        model_name="glm-4",
        openai_api_base="https://open.bigmodel.cn/api/paas/v4",
        # openai_api_key=generate_token(api_key, 10),
        openai_api_key=api_key,
        streaming=True,
        verbose=True
    )
    return chat_model
```

### 初始化工具包

该函数用于初始化工具包，用于与数据库进行交互。

```python
def initialize_toolkit(db, llm):
    """
    初始化工具包，用于与数据库交互。
    """
    toolkit = SQLDatabaseToolkit(db=db, llm=llm)
    tools = toolkit.get_tools()
    return tools
```

### 创建代理执行器

该函数用于创建代理执行器，用于执行代理操作。

```python
def create_agent_executor(llm, tools):
    """
    创建代理执行器，用于执行代理操作。
    """
    prompt = create_prompt_template()
    agent = create_react_agent(llm, tools, prompt=prompt)
    agent_executor = AgentExecutor(agent=agent, tools=tools, handle_parsing_errors=True)
    return agent_executor
```

### 创建提示模板

该函数用于创建提示模板，用于指导代理如何响应。

```python
def create_system_message():
    """
    创建系统消息，用于指导代理如何响应。
    """
    SQL_PREFIX = """
    You are an agent designed to interact with a SQL database.
    Given an input question, create a syntactically correct SQLite query to run, then look at the results of the query and return the answer.
    Unless the user specifies a specific number of examples they wish to obtain, always limit your query to at most 5 results.
    You can order the results by a relevant column to return the most interesting examples in the database.
    Never query for all the columns from a specific table, only ask for the relevant columns given the question.
    You have access to tools for interacting with the database.
    Only use the below tools. Only use the information returned by the below tools to construct your final answer.
    You MUST double check your query before executing it. If you get an error while executing a query, rewrite the query and try again.
    DO NOT make any DML statements (INSERT, UPDATE, DELETE, DROP etc.) to the database.
    To start you should ALWAYS look at the tables in the database to see what you can query.
    Do NOT skip this step.
    Then you should query the schema of the most relevant tables.
    """
    return SystemMessage(content=SQL_PREFIX)
```

## 运行主程序

主函数用于测试上述功能。设置环境变量，初始化数据库和聊天模型，初始化工具包，创建代理执行器，然后通过异步函数调用代理执行器进行测试。

```python
def main():
    """
    主函数，用于测试上述功能。
    """
    zhipuai_api_key = "20a8e1c4c5348d62eaaa64beabbd2fa0.34ZWDCTkvM5o7PDp"
    db = initialize_database()
    chat_model = initialize_chat_model(zhipuai_api_key)
    tools = initialize_toolkit(db, chat_model)
    system_message = create_system_message()

    agent_executor = create_react_agent(chat_model, tools, messages_modifier=system_message)

    for s in agent_executor.stream(
        {"messages": [HumanMessage(content="哪个国家/地区的客户花费最多？")]}
    ):
        print(s)
        print("----")

```

## 结果测试

运行主程序后，程序将通过异步方式调用代理执行器，测试与数据库的交互，并打印结果。

```bash
python SQL_Agent_baseGLM.py
```

您可以根据实际需求修改输入问题，以测试不同的查询和结果。

## 注意事项

- 请确保已正确设置环境变量。
- 确保数据库文件 `Chinook.db` 存在并可访问。
- 根据实际需求修改 API 密钥和数据库连接信息。

## 全部代码

```python
import jwt
import time
from langchain_openai import ChatOpenAI
from langchain_core.messages import AIMessage, HumanMessage, SystemMessage
from langchain_community.agent_toolkits import SQLDatabaseToolkit
from langchain_community.utilities.sql_database import SQLDatabase
from langgraph.prebuilt import create_react_agent, chat_agent_executor

# 生成JWT令牌
def generate_token(apikey: str, exp_seconds: int):
    """
    生成JWT令牌的函数。
    参数：
        - apikey: str，API密钥。
        - exp_seconds: int，令牌过期时间（秒）。
    返回：
        - str，生成的JWT令牌。
    """
    id, secret = apikey.split(".")
    payload = {
        "api_key": id,
        "exp": int(round(time.time() * 10000)) + exp_seconds * 10000,
        "timestamp": int(round(time.time() * 10000)),
    }

    return jwt.encode(
        payload,
        secret,
        algorithm="HS256",
        headers={"alg": "HS256", "sign_type": "SIGN"},
    )

# 初始化数据库连接
def initialize_database():
    """
    初始化数据库连接。
    """
    db = SQLDatabase.from_uri("sqlite:///Chinook.db")
    print(db.dialect)
    print(db.get_usable_table_names())
    db.run("SELECT * FROM Artist LIMIT 10;")
    return db

# 初始化聊天模型
def initialize_chat_model(api_key: str):
    """
    初始化聊天模型。
    参数：
        - api_key: str，API密钥。
    """
    chat_model = ChatOpenAI(
        model_name="glm-4",
        openai_api_base="https://open.bigmodel.cn/api/paas/v4",
        # openai_api_key=generate_token(api_key, 10),
        openai_api_key=api_key,
        streaming=True,
        verbose=True
    )
    return chat_model

# 初始化工具包
def initialize_toolkit(db, llm):
    """
    初始化工具包，用于与数据库交互。
    参数：
        - db: 数据库连接对象。
        - llm: 聊天模型对象。
    """
    toolkit = SQLDatabaseToolkit(db=db, llm=llm)
    tools = toolkit.get_tools()
    return tools

# 创建系统消息
def create_system_message():
    """
    创建系统消息，用于指导代理如何响应。
    """
    SQL_PREFIX = """
    You are an agent designed to interact with a SQL database.
    Given an input question, create a syntactically correct SQLite query to run, then look at the results of the query and return the answer.
    Unless the user specifies a specific number of examples they wish to obtain, always limit your query to at most 5 results.
    You can order the results by a relevant column to return the most interesting examples in the database.
    Never query for all the columns from a specific table, only ask for the relevant columns given the question.
    You have access to tools for interacting with the database.
    Only use the below tools. Only use the information returned by the below tools to construct your final answer.
    You MUST double check your query before executing it. If you get an error while executing a query, rewrite the query and try again.
    DO NOT make any DML statements (INSERT, UPDATE, DELETE, DROP etc.) to the database.
    To start you should ALWAYS look at the tables in the database to see what you can query.
    Do NOT skip this step.
    Then you should query the schema of the most relevant tables.
    """
    return SystemMessage(content=SQL_PREFIX)

# 主函数，用于测试上述功能
def main():
    """
    主函数，用于测试上述功能。
    """
    zhipuai_api_key = "20a8e1c4c5348d62eaaa64beabbd2fa0.34ZWDCTkvM5o7PDp"
    db = initialize_database()
    chat_model = initialize_chat_model(zhipuai_api_key)
    tools = initialize_toolkit(db, chat_model)
    system_message = create_system_message()

    agent_executor = create_react_agent(chat_model, tools, messages_modifier=system_message)

    for s in agent_executor.stream(
        {"messages": [HumanMessage(content="哪个国家/地区的客户花费最多？")]}
    ):
        print(s)
        print("----")

    # answer = agent_executor.invoke({"messages": messages})
    # print(answer)

if __name__ == "__main__":
    main()
```

## 输出格式化

为了使用`pydantic`对输出的数据进行验证和解析，我们首先需要定义一个模型，该模型反映了数据的结构。在这个例子中，数据结构包含一个顶层的`agent`键，它又包含一个`messages`列表，列表中的每个元素都是一个包含`content`、`response_metadata`和`id`键的字典。

### 步骤 1：定义模型

1. **AIMessage 模型**：验证`messages`列表中的每个元素。
2. **Agent 模型**：验证顶层的结构，包含`messages`列表。

### 步骤 2：创建实例并验证

使用定义好的模型创建实例，并传入实际数据进行验证。

### 示例代码

```python
from pydantic import BaseModel
from typing import List, Dict, Any

class AIMessage(BaseModel):
    content: str
    response_metadata: Dict[str, Any]
    id: str

class Agent(BaseModel):
    messages: List[AIMessage]

# 示例数据
data = {
    'agent': {
        'messages': [
            {
                'content': '根据查询结果，美国（United States）的客户花费最多。',
                'response_metadata': {'finish_reason': 'stop', 'model_name': 'glm-4'},
                'id': 'run-d0a7eb19-1999-428d-b5fa-417d2260a926-0'
            }
        ]
    }
}

# 使用模型进行验证
try:
    agent_data = Agent(**data['agent'])
    print(agent_data)
except ValidationError as e:
    print(e.json())
print(agent_data.messages[0].content)
```

在这个示例中，我们首先定义了两个`pydantic`模型：`AIMessage`和`Agent`。`AIMessage`模型用于验证`messages`列表中的每个消息，包括它们的`content`、`response_metadata`和`id`。`Agent`模型则用于验证顶层的`agent`结构，它包含一个`messages`列表，该列表由`AIMessage`模型的实例组成。通过创建`Agent`模型的实例并传入实际数据，我们可以确保数据符合预期的结构和类型。如果数据不符合模型定义，`pydantic`会抛出`ValidationError`异常。

## 从模型返回结构化数据

当模型结构化输出方法应用于 Agent 类型数据时

- langchain_core 库构建的模型结构化输出方法 (with_structured_output) 缺少 bind_tools 属性，无法适配 Langgraph 库构建的 react_agent。这一问题限制了 SQLAgent 的功能和应用范围。
- 结构化输出模块的设计需要更高的灵活性，以应对复杂查询结果的多样性和不确定性。需要进一步扩展和优化 pydantic 模型，提升其适用性。
- 研究并解决 langchain_core 库与 Langgraph 库之间的兼容性问题，特别是 bind_tools 属性缺失的问题。尝试找到替代方案或进行库的调整，以确保 SQLAgent 的功能完整性。

### 示例代码

```python
from langchain_core.pydantic_v1 import BaseModel, Field
from typing import List, Optional, Dict, Any

class Function(BaseModel):
    arguments: str
    name: str

class ToolCall(BaseModel):
    index: Optional[int]
    id: str
    function: Optional[Function]
    name: Optional[str]
    args: Optional[Dict[str, Any]]
    type: Optional[str]

class AdditionalKwargs(BaseModel):
    tool_calls: List[ToolCall]

class ResponseMetadata(BaseModel):
    finish_reason: str
    model_name: str

class AIMessage(BaseModel):
    content: str
    additional_kwargs: Optional[AdditionalKwargs]
    response_metadata: Optional[ResponseMetadata]
    id: str
    tool_calls: Optional[List[ToolCall]]

class Agent(BaseModel):
    messages: List[AIMessage]

class ToolMessage(BaseModel):
    content: str
    name: str
    tool_call_id: str

class Tools(BaseModel):
    messages: List[ToolMessage]

class StructuredOutput(BaseModel):
    agent: Optional[Agent] = None
    tools: Optional[Tools] = None
    

structured_llm = llm.with_structured_output(
    StructuredOutput,
    method="json_mode",
    include_raw=True
)

answer = structured_llm.invoke('你好')

```

![[LangChain 从模型返回结构化数据]]

---
