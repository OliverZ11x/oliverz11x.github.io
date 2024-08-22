---
date created: 2024/8/22 11:0
date modified: 2024/8/22 18:10
---
## 一、概述

使用 LangChain 框架构建一个支持 SQL 查询和交互的智能代理系统，并实现流式输出的功能。主要流程包括环境变量设置、数据库连接、工具包初始化、代理执行器的创建、以及通过 FastAPI 实现的流式输出 API。

## 二、流式输出实现

### 1. FastAPI 实现流式输出

流式输出是实现实时响应的关键。通过 `StreamingResponse` 返回生成的结果，确保数据在生成的同时能够逐步输出给客户端。

```python
# 定义处理聊天流的POST请求
@app.post("/chat_stream")
async def chat_stream(query: str = Form("你好")):  # 接收表单数据中的查询参数
    async def predict():
        msg = ''  # 初始化消息变量
        final = False  # 初始化最终消息标志
        async for event in agent_executor.astream_events({"messages": [HumanMessage(content=query)]}, version="v1"):
            kind = event["event"]  # 获取事件类型
            print(kind)  # 打印事件类型
            if kind == "on_chat_model_stream":
                msg = event['data']['chunk'].content  # 获取消息内容
                js_data = {"code": "200", "msg": "ok", "data": msg}  # 构建JSON数据
                yield f"data: {json.dumps(js_data, ensure_ascii=False)}\n\n"  # 通过流式传输发送数据
                if not final:
                    if "Final Answer:" in msg:
                        _msg = msg.split("Final Answer:")[1].lstrip()  # 提取最终答案
                        print(f"===Parser chunk: {_msg}", flush=True)  # 打印最终答案
                        final = True  # 设置最终消息标志
                else:
                    print(f"===Parser chunk: {event['data']['chunk'].content}", flush=True)  # 打印消息内容
            elif kind == "on_tool_start":
                tool_name = event['name']  # 获取工具名称
                tool_input = event['data']['input']  # 获取工具输入
                js_data = {"code": "200", "msg": "ok", "tool": {"name": tool_name, "input": tool_input}}  # 构建工具使用的JSON数据
                yield f"data: {json.dumps(js_data, ensure_ascii=False)}\n\n"  # 通过流式传输发送数据
                print(f"===Tool used: {tool_name} with input: {tool_input}", flush=True)  # 打印工具使用信息
                
           elif kind == "on_tool_end":
                print(event)
                tool_name = event['name']  # 获取工具名称
                tool_output = event['data']['output'].content  # 获取工具输出
                js_data = {"code": "200", "msg": "ok", "tool": {"name": tool_name, "output": tool_output}}  # 构建工具使用的JSON数据
                yield f"data: {json.dumps(js_data, ensure_ascii=False)}\n\n"  # 通过流式传输发送数据
                print(f"===Tool used: {tool_name} with output: {tool_output}", flush=True)  # 打印工具使用信息


    return StreamingResponse(predict(), media_type="text/event-stream")  # 返回流式响应

```

### 2. 使用 LangChain 事件流实现流式输出

通过 `astream_events` 实现事件驱动的流式响应。这种方式允许在生成时不断返回结果。有关 `stream_events` 的详细实现，请参考 [LangChain 官方文档](https://python.langchain.com/v0.2/docs/how_to/streaming/#using-stream-events)。

贴一份 `event` 数据的各种格式

```json
{'event': 'on_chain_start', 'data': {'input': 'output a list of the countries france, spain and japan and their populations in JSON format. Use a dict with an outer key of "countries" which contains a list of countries. Each country should have the key `name` and `population`'}, 'name': 'RunnableSequence', 'tags': ['my_chain'], 'run_id': 'fd68dd64-7a4d-4bdb-a0c2-ee592db0d024', 'metadata': {}}
{'event': 'on_chat_model_start', 'data': {'input': {'messages': [[HumanMessage(content='output a list of the countries france, spain and japan and their populations in JSON format. Use a dict with an outer key of "countries" which contains a list of countries. Each country should have the key `name` and `population`')]]}}, 'name': 'ChatAnthropic', 'tags': ['seq:step:1', 'my_chain'], 'run_id': 'efd3c8af-4be5-4f6c-9327-e3f9865dd1cd', 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='{', id='run-efd3c8af-4be5-4f6c-9327-e3f9865dd1cd')}, 'run_id': 'efd3c8af-4be5-4f6c-9327-e3f9865dd1cd', 'name': 'ChatAnthropic', 'tags': ['seq:step:1', 'my_chain'], 'metadata': {}}
{'event': 'on_parser_start', 'data': {}, 'name': 'JsonOutputParser', 'tags': ['seq:step:2', 'my_chain'], 'run_id': 'afde30b9-beac-4b36-b4c7-dbbe423ddcdb', 'metadata': {}}
{'event': 'on_parser_stream', 'data': {'chunk': {}}, 'run_id': 'afde30b9-beac-4b36-b4c7-dbbe423ddcdb', 'name': 'JsonOutputParser', 'tags': ['seq:step:2', 'my_chain'], 'metadata': {}}
{'event': 'on_chain_stream', 'data': {'chunk': {}}, 'run_id': 'fd68dd64-7a4d-4bdb-a0c2-ee592db0d024', 'name': 'RunnableSequence', 'tags': ['my_chain'], 'metadata': {}}
{'event': 'on_chat_model_stream', 'data': {'chunk': AIMessageChunk(content='\n  ', id='run-efd3c8af-4be5-4f6c-9327-e3f9865dd1cd')}, 'run_id': 'efd3c8af-4be5-4f6c-9327-e3f9865dd1cd', 'name': 'ChatAnthropic', 'tags': ['seq:step:1', 'my_chain'], 'metadata': {}}
...
{'event': 'on_tool_start', 'data': {'input': 'hello'}, 'name': 'correct_tool', 'tags': [], 'run_id': 'd5ea83b9-9278-49cc-9f1d-aa302d671040', 'metadata': {}}
{'event': 'on_chain_start', 'data': {'input': 'hello'}, 'name': 'reverse_word', 'tags': [], 'run_id': '44dafbf4-2f87-412b-ae0e-9f71713810df', 'metadata': {}}
{'event': 'on_chain_end', 'data': {'output': 'olleh', 'input': 'hello'}, 'run_id': '44dafbf4-2f87-412b-ae0e-9f71713810df', 'name': 'reverse_word', 'tags': [], 'metadata': {}}
{'event': 'on_tool_end', 'name': 'sql_db_list_tables', 'run_id': 'e4cb7aa5-60d9-490d-b376-64f571d65a5d', 'tags': ['seq:step:1'], 'metadata': {'langgraph_step': 2, 'langgraph_node': 'tools', 'langgraph_triggers': ['branch:agent:should_continue:tools'], 'langgraph_task_idx': 0, 'thread_ts': '1ef605aa-fbd9-6538-bffe-1b774554be0a'}, 'data': {'input': {}, 'output': ToolMessage(content='ads_ridehub_inventory_view', name='sql_db_list_tables', tool_call_id='call_202408221546479f989cd8396c4cd9')}, 'parent_ids': []}
...
```

### 3. 启动服务器

在终端中运行以下命令来启动服务器：

```shell
uvicorn SQLAgent_AutoSight_stream:app --host 127.0.0.1 --port 10088
```

### 4. 发送 POST 请求

在另一个终端中运行以下 `curl` 命令来发送 POST 请求：

```shell
curl -X POST "http://127.0.0.1:10088/chat_stream" -d "query=请你帮我上网搜索今天的热门资讯？"
```

## 四、总结

LangChain 官方虽然提供了流式输出的示例，但这些示例只能在控制台输出，无法直接获取生成器。通过 FastAPI 的 `StreamingResponse` 结合 LangChain 的事件流，真正实现了前端可见的实时流式输出，避免了伪流式输出的局限性，使代理系统在响应时更加灵活和高效。

## 五、代码汇总

```python
from typing import List
import os
import json
from dotenv import load_dotenv
from fastapi import FastAPI, HTTPException, Form
from fastapi.responses import StreamingResponse
import uvicorn
from langchain_community.utilities.sql_database import SQLDatabase
from urllib.parse import quote_plus as urlquote
from langchain_core.messages import AIMessage, HumanMessage, SystemMessage
from langchain_community.agent_toolkits import SQLDatabaseToolkit
from langchain_core.prompts import ChatPromptTemplate
from langchain.agents import tool
from langchain_openai import ChatOpenAI
from langgraph.prebuilt import create_react_agent
from langgraph.checkpoint.sqlite import SqliteSaver

# 设置环境变量
os.environ['OPENAI_API_KEY'] = "20a8e1c4c5348d62eaaa64beabbd2fa0.34ZWDCTkvM5o7PDp"
os.environ["LANGCHAIN_PROJECT"] = "Evaluate_SQLAgent_inventory"
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = "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"

# 连接数据库
def connectToTheDatabase(connect_name='autosight_model'):
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
    db = SQLDatabase.from_uri(
        f'{db_type}://{db_user}:{urlquote(db_password)}@{db_host}:{db_port}/{db_name}?charset=utf8', view_support=True)
    return db

db_test = connectToTheDatabase()

# 创建系统消息
def create_system_message_cn():
    SQL_PREFIX_TEMPLATE = """
    角色：
    您是一个设计用来与SQL数据库交互的代理。您可以查询数据库并直接访问结果。

    目标：
    针对一个问题，创建一个SQL查询来检索所需的数据，执行此查询，并直接返回查询的结果，而不是查询本身。如果遇到错误或初始结果为空，请诊断问题，调整查询，并再次尝试。

    指示：
    - 首先识别相关表格，以了解可用数据的范围。
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

# 初始化工具包
def initialize_toolkit(db, llm):
    toolkit = SQLDatabaseToolkit(db=db, llm=llm)
    tools = toolkit.get_tools()
    return tools

# 初始化代理执行器
def initialize_agent_executor():
    chat_model = ChatOpenAI(
        model_name="GLM-4",
        openai_api_base="https://open.bigmodel.cn/api/paas/v4",
        streaming=True,
        verbose=True,
    )
    tools = initialize_toolkit(db_test, chat_model)
    system_message = create_system_message_cn()
    # memory = SqliteSaver.from_conn_string(":memory:")
    global agent_executor
    agent_executor = create_react_agent(chat_model, tools, state_modifier=system_message, )
    return agent_executor

def get_final_answer(answer_list):
    return answer_list[-1].content

def chat_with_glm_non_stream(query, options):
    chat_model = ChatOpenAI(
        model_name="GLM-4",
        openai_api_base="https://open.bigmodel.cn/api/paas/v4",
        streaming=True,
        verbose=True,
    )
    response = chat_model.invoke(query)
    return response.content



db = 'autosight_model'  # 数据库名称
agent_executor = initialize_agent_executor()  # 初始化代理执行器

app = FastAPI()  # 创建FastAPI应用
load_dotenv()  # 加载环境变量

# 定义根路径处理函数
@app.get("/")
def read_root():
    return {"message": "Hello, World!"}  # 返回欢迎消息

# 定义处理聊天流的POST请求
@app.post("/chat_stream")
async def chat_stream(query: str = Form("你好")):  # 接收表单数据中的查询参数
    async def predict():
        msg = ''  # 初始化消息变量
        final = False  # 初始化最终消息标志
        async for event in agent_executor.astream_events({"messages": [HumanMessage(content=query)]}, version="v1"):
            kind = event["event"]  # 获取事件类型
            print(kind)  # 打印事件类型
            if kind == "on_chat_model_stream":
                msg = event['data']['chunk'].content  # 获取消息内容
                js_data = {"code": "200", "msg": "ok", "data": msg}  # 构建JSON数据
                yield f"data: {json.dumps(js_data, ensure_ascii=False)}\n\n"  # 通过流式传输发送数据
                if not final:
                    if "Final Answer:" in msg:
                        _msg = msg.split("Final Answer:")[1].lstrip()  # 提取最终答案
                        print(f"===Parser chunk: {_msg}", flush=True)  # 打印最终答案
                        final = True  # 设置最终消息标志
                else:
                    print(f"===Parser chunk: {event['data']['chunk'].content}", flush=True)  # 打印消息内容
            elif kind == "on_tool_start":
                tool_name = event['name']  # 获取工具名称
                tool_input = event['data']['input']  # 获取工具输入
                js_data = {"code": "200", "msg": "ok", "tool": {"name": tool_name, "input": tool_input}}  # 构建工具使用的JSON数据
                yield f"data: {json.dumps(js_data, ensure_ascii=False)}\n\n"  # 通过流式传输发送数据
                print(f"===Tool used: {tool_name} with input: {tool_input}", flush=True)  # 打印工具使用信息

            elif kind == "on_tool_end":
                print(event)
                tool_name = event['name']  # 获取工具名称
                tool_output = event['data']['output'].content  # 获取工具输出
                js_data = {"code": "200", "msg": "ok", "tool": {"name": tool_name, "output": tool_output}}  # 构建工具使用的JSON数据
                yield f"data: {json.dumps(js_data, ensure_ascii=False)}\n\n"  # 通过流式传输发送数据
                print(f"===Tool used: {tool_name} with output: {tool_output}", flush=True)  # 打印工具使用信息


    return StreamingResponse(predict(), media_type="text/event-stream")  # 返回流式响应

if __name__ == "__main__":
    uvicorn.run(app=app, host="127.0.0.1", port=10088)  # 运行FastAPI应用
```