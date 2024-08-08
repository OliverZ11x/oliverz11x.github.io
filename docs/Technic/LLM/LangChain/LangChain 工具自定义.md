---
title: LangChain 工具自定义
date created: 2024年7月31日,星期三,上午,11:15:58
date modified: 2024年8月8日,星期四,下午,3:52:22
---
# LangChain 自定义工具

- [构建多参数的自定义 LangChain 工具](https://blog.csdn.net/Alex_StarSky/article/details/136574640?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0-136574640-blog-137740363.235^v43^pc_blog_bottom_relevance_base2&spm=1001.2101.3001.4242.1&utm_relevant_index=3)
- [Langchain 自定义 Tool 的三种方式](https://blog.csdn.net/weixin_48707135/article/details/137740363)

在构建你自己的 Agent 时，你需要提供一系列供它使用的 Tools。除了被调用的函数，Tool 还由以下部分组成：

- 名称 (name: str)：必填且需要唯一。
- 描述 (description: str)：可选但建议填写，可以供 agent 参考是否需要调用 Tool。
- 参数 schema (args_schema : Pydantic BaseModel)：可选但建议填写，可以用于提供额外信息，如少样本学习的例子或者对期望参数的校验。

LangChain 自定义 Tool 的方式一共有三种：使用 `@tool` 装饰器、继承 BaseTool 类、以及使用 StructuredTool。接下来我将用这三种方法自定义乘法工具，并对其进行测试。

## @tool 装饰器自定义工具

使用装饰器定义 Tool 是最简单的方式，会默认使用函数名作为 Tool 的名称。你也可以传递一个 string 类型的参数来覆盖名称。装饰器会使用函数的注释作为 Tool 的描述，所以函数必须有注释。

```python
@tool("Calculator_Decorator")
def multiply(a: int, b: int) -> int:
    """Multiply two numbers."""
    return a * b

print(multiply.name)
print(multiply.description)
print(multiply.args)
```

输出：

```json
Calculator_Decorator
Multiply two numbers.
{'a': {'title': 'A', 'type': 'integer'}, 'b': {'title': 'B', 'type': 'integer'}}
```

## 继承 BaseTool 类自定义工具

除了使用装饰器，还可以通过继承 BaseTool 类的方法来定义工具的参数说明。

```python
from pydantic import BaseModel, Field
from typing import Type, Optional

class CalculatorInput(BaseModel):
    a: int = Field(description="first number")
    b: int = Field(description="second number")

class CustomCalculatorTool(BaseTool):
    name = "Calculator_BaseTool"
    description = "Useful for when you need to answer questions about math"
    args_schema: Type[BaseModel] = CalculatorInput
    return_direct: bool = True

    def _run(
        self, a: int, b: int, run_manager: Optional[CallbackManagerForToolRun] = None
    ) -> str:
        """Use the tool."""
        return a * b

    async def _arun(
        self,
        a: int,
        b: int,
        run_manager: Optional[AsyncCallbackManagerForToolRun] = None,
    ) -> str:
        """Use the tool asynchronously."""
        raise NotImplementedError("Calculator does not support async")

calculator_tool = CustomCalculatorTool()
print(calculator_tool.name)
print(calculator_tool.description)
print(calculator_tool.args)
print(calculator_tool.return_direct)
```

输出：

```json
Calculator_BaseTool
Useful for when you need to answer questions about math 
{'a': {'title': 'A', 'description': 'first number', 'type': 'integer'}, 
 'b': {'title': 'B', 'description': 'second number', 'type': 'integer'}}
True
```

## StructuredTool 自定义工具

StructuredTool 结合了前两种方法的优点，比继承类更方便，比使用装饰器功能更多。

```python
from langchain.tools import StructuredTool

def multiply_structured(a: int, b: int) -> int:
    """Multiply two numbers."""
    return a * b

multiply_structuredTool = StructuredTool.from_function(
    func=multiply,
    name="Calculator_StructuredTool",
    description="Useful for when you need to answer questions about math"
    # coroutine= ... <- 如果需要，可以指定一个异步方法
)

print(multiply_tool.name)
print(multiply_tool.description)
print(multiply_tool.args)
```

输出：

```json
Calculator_StructuredTool
Useful for when you need to answer questions about math
{'a': {'title': 'A', 'type': 'integer'}, 'b': {'title': 'B', 'type': 'integer'}}
```

通过以上三种方式，你可以根据需求选择最适合的方式来自定义 Tool，以更好地构建你的 Agent。

## 使用 Tavily 搜索工具

LangChain 中内置了 Tavily 搜索引擎工具，可以通过 Tavily 搜索引擎进行网页查询。

```python
# Import relevant functionality
from langchain_community.tools.tavily_search import TavilySearchResults
from langchain_core.messages import HumanMessage
from langgraph.checkpoint.sqlite import SqliteSaver
from langgraph.prebuilt import create_react_agent

# Create the agent
memory = SqliteSaver.from_conn_string(":memory:")
zhipuai_api_key = "20a8e1c4c5348d62eaaa64beabbd2fa0.34ZWDCTkvM5o7PDp"
model = initialize_chat_model(zhipuai_api_key)
search = TavilySearchResults(max_results=2)
tools = [search]
agent_executor = create_react_agent(model, tools, checkpointer=memory)

# Use the agent
config = {"configurable": {"thread_id": "abc123"}}
for chunk in agent_executor.stream(
    {"messages": [HumanMessage(content="hi im bob! and i live in Chonqing")]}, config
):
    print(chunk)
    print("----")

for chunk in agent_executor.stream(
    {"messages": [HumanMessage(content="whats the weather where I live?")]}, config
):
    print(chunk)
    print("----")
```

## 全部代码

```python
from langchain_openai import ChatOpenAI
from langchain.pydantic_v1 import BaseModel, Field
from langchain_community.tools import BaseTool, StructuredTool, tool
from langchain_core.messages import AIMessage, HumanMessage, SystemMessage
from langgraph.prebuilt import create_react_agent
from langchain_community.tools.tavily_search import TavilySearchResults
from langchain.callbacks.manager import (
    AsyncCallbackManagerForToolRun,
    CallbackManagerForToolRun,
)
from typing import Optional, Type
import os

os.environ["TAVILY_API_KEY"] = "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"



def initialize_chat_model(api_key: str):
    """
    初始化聊天模型。
    参数：
        - api_key: str，API密钥。
    """
    chat_model = ChatOpenAI(
        model_name="glm-4",
        openai_api_base="https://open.bigmodel.cn/api/paas/v4",
        openai_api_key=api_key,
        streaming=True,
        verbose=True,
    )
    return chat_model


@tool("Calculator_Decorator")
def multiply_decorator(a: int, b: int) -> int:
    """Multiply two numbers."""
    return a * b


class CalculatorInput(BaseModel):
    a: int = Field(description="first number")
    b: int = Field(description="second number")


class CustomCalculatorTool(BaseTool):
    name = "Calculator_BaseTool"
    description = "Useful for when you need to answer questions about math"
    args_schema: Type[BaseModel] = CalculatorInput
    return_direct: bool = True

    def _run(
        self, a: int, b: int, run_manager: Optional[CallbackManagerForToolRun] = None
    ) -> str:
        """Use the tool."""
        return a * b

    async def _arun(
        self,
        a: int,
        b: int,
        run_manager: Optional[AsyncCallbackManagerForToolRun] = None,
    ) -> str:
        """Use the tool asynchronously."""
        raise NotImplementedError("Calculator does not support async")


def multiply_structured(a: int, b: int) -> int:
    """Multiply two numbers."""
    return a * b


def initialize_tavily_search():
    """
    初始化Tavily搜索工具。
    """
    search = TavilySearchResults(max_results=2)
    return search


def main():
    """
    主函数，用于测试上述功能。
    """
    # 初始化聊天模型
    zhipuai_api_key = "20a8e1c4c5348d62eaaa64beabbd2fa0.34ZWDCTkvM5o7PDp"
    model = initialize_chat_model(zhipuai_api_key)

    multiply_structuredTool = StructuredTool.from_function(
        func=multiply_structured,
        name="Calculator_StructuredTool",
        description="useful for when you need to answer questions about math",
    )

    # 定义工具
    tools = [
        multiply_decorator,
        # CustomCalculatorTool(),
        # multiply_structuredTool,
        initialize_tavily_search(),
    ]

    # 创建反应代理
    agent_executor = create_react_agent(model, tools)

    # 测试乘法工具
    for chunk in agent_executor.stream(
        {"messages": [HumanMessage(content="35*67等于多少？")]}
    ):
        print(chunk)
        print("----")
    print("----")
    print("----")
    print("----")

    # 测试Tavily搜索
    for chunk in agent_executor.stream(
        {"messages": [HumanMessage(content="重庆今天的天气怎么样？")]}
    ):
        print(chunk)
        print("----")


if __name__ == "__main__":
    main()
```

输出

```json
{'agent': {'messages': [AIMessage(content='', additional_kwargs={'tool_calls': [{'index': 0, 'id': 'call_202407161510514acf924ea38c4936b7ee47369684d623', 
'function': {'arguments': '{"a":35,"b":67}', 'name': 'Calculator_Decorator'}, 'type': 'function'}]}, response_metadata={'finish_reason': 'tool_callstool_calls', 'model_name': 'glm-4glm-4'}, id='run-70df4909-7c3b-4c5d-922d-6e3c5eb4ad07-0', tool_calls=[{'name': 'Calculator_Decorator', 'args': {'a': 35, 'b': 67}, 'id': 'call_202407161510514acf924ea38c4936b7ee47369684d623'}])]}}
----
{'tools': {'messages': [ToolMessage(content='2345', name='Calculator_Decorator', tool_call_id='call_202407161510514acf924ea38c4936b7ee47369684d623')]}}   
----
{'agent': {'messages': [AIMessage(content='根据您的要求，我们可以调用计算器API得到：35*67=2345', response_metadata={'finish_reason': 'stop', 'model_name': 'glm-4'}, id='run-b0bd04b5-5d46-4f8a-b4ea-5becca14a9ab-0')]}}
----
----
----
----
{'agent': {'messages': [AIMessage(content='这个问题涉及到一个非常敏感和严重的话题。我需要使用一个可靠的信息源来验证这个问题是否真实。', additional_kwargs={'tool_calls': [{'index': 0, 'id': 'call_20240716151057c425c924cf7641a8897241e58f1713bf', 'function': {'arguments': '{"query":"川普被枪杀"}', 'name': 'tavily_search_results_json'}, 'type': 'function'}]}, response_metadata={'finish_reason': 'tool_callstool_calls', 'model_name': 'glm-4glm-4'}, id='run-e899d06c-40a2-41cb-9f5c-506454bcdd30-0', tool_calls=[{'name': 'tavily_search_results_json', 'args': {'query': '川普被枪杀'}, 'id': 'call_20240716151057c425c924cf7641a8897241e58f1713bf'}])]}}
----
{'tools': {'messages': [ToolMessage(content='[{"url": "https://cn.nytimes.com/usa/20240714/trump-election-trump-rally-pennsylvania/", "content": "\\u7f8e\\u56fd\\u7279\\u52e4\\u5c40\\u5728\\u4e00\\u4efd\\u58f0\\u660e\\u4e2d\\u8868\\u793a\\uff0c\\u524d\\u603b\\u7edf\\u5510\\u7eb3\\u5fb7\\u00b7J\\u00b7\\u7279\\u6717\\u666e\\u5468\\u516d\\u5728\\u5bbe\\u5915\\u6cd5\\u5c3c\\u4e9a\\u5dde\\u5df4\\u7279\\u52d2\\u5e02\\u7684\\u7ade\\u9009\\u96c6\\u4f1a\\u5f00\\u59cb\\u51e0\\u5206\\u949f\\u540e\\u906d\\u9047\\u67aa\\u51fb\\uff0c\\u4e00\\u540d\\u96c6\\u4f1a\\u53c2\\u4e0e\\u8005\\u548c\\u4e00\\u540d\\u67aa\\u624b\\u5acc\\u7591\\u4eba\\u6b7b\\u4ea1\\uff0c\\u53e6\\u6709\\u4e24\\u540d\\u89c2\\u4f17\\u53d7\\u91cd\\u4f24\\u3002 \\u5b98\\u5458\\u8868\\u793a\\uff0c\\u8be5\\u4e8b\\u4ef6\\u6b63\\u5728\\u4f5c\\u4e3a\\u6697\\u6740\\u4f01\\u56fe\\u88ab ..."}, {"url": "https://zh.wikipedia.org/wiki/\\u5510\\u7d0d\\u00b7\\u5ddd\\u666e\\u9047\\u523a\\u6848", "content": "\\u5510\\u7d0d\\u00b7\\u5ddd\\u666e\\u9047\\u523a\\u6848\\u767c\\u751f\\u65bc\\u7f8e\\u6771\\u590f\\u4ee4\\u6642\\u95932024\\u5e747\\u670813\\u65e518\\u664211\\u5206\\uff0c\\u7f8e\\u570b\\u524d\\u7e3d\\u7d71\\u5510\\u7d0d\\u00b7\\u5ddd\\u666e\\u5728\\u70ba\\u5e74\\u5e95\\u603b\\u7edf\\u5927\\u9009\\u9020\\u52e2\\u7684\\u6d3b\\u52d5\\u96c6\\u6703\\u4e2d\\u906d\\u9047\\u4e00\\u540d\\u7f8e\\u570b\\u7537\\u5b50\\u69cd\\u64ca \\u3002 \\u4e8b\\u767c\\u5730\\u9ede\\u7232 \\u5bbe\\u5915\\u6cd5\\u5c3c\\u4e9a\\u5dde \\u5df4\\u7279\\u52d2 \\u7684\\u5df4\\u7279\\u52d2\\u8fb2\\u5834\\u5c55\\u89bd\\u5834\\u5730\\uff08\\u82f1\\u8a9e\\uff1a Butler Farm Show Grounds \\uff09\\uff0c\\u5ddd\\u666e\\u6c92\\u6709\\u5927\\u7919\\uff0c\\u4f46\\u5176\\u53f3\\u8033\\u53d7 ..."}]', name='tavily_search_results_json', tool_call_id='call_20240716151057c425c924cf7641a8897241e58f1713bf')]}}
----
{'agent': {'messages': [AIMessage(content='根据最新的消息，川普没有被枪杀。2024年7月14日，美国前总统川普在宾夕法尼亚州的一次集会上发表演讲，几分钟后，他遇到了一起枪击事件。一名持枪的男子在集会上开枪，导致一人死亡，两人受伤。川普本人没有受到伤害。这名枪手后来被警察制服。川普在事后表示，他感谢特勤局的保护，并称这次事件是“暗杀未遂”。', response_metadata={'finish_reason': 'stop', 'model_name': 'glm-4'}, id='run-1a595eed-f9b0-4808-923c-034d123d2ed9-0')]}}
----
```