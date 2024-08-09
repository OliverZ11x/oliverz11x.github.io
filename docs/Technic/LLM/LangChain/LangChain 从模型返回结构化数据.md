---
title: LangChain 结构化数据
date created: 2024/7/31 11:15
date modified: 2024/8/9 14:44
---
# LangChain从模型返回结构化数据

>[如何从模型返回结构化数据 | 🦜️🔗 LangChain](https://python.langchain.com/v0.2/docs/how_to/structured_output/#using-pydanticoutputparser)

让模型返回符合特定模式的输出通常很有用。一个常见的用例是从文本中提取数据以插入数据库或用于其他下游系统。本指南介绍了从模型获取结构化输出的几种策略。

## .with_structured_output() 方法

----------------------------

这是获取结构化输出的最简单、最可靠的方法。.with_structured_output() 是为那些提供了用于结构化输出的本地 API（如工具 / 函数调用或 JSON 模式）的模型而实现的，并在引擎盖下利用了这些功能。

该方法将模式作为输入，模式指定了所需输出属性的名称、类型和描述。该方法会返回一个类似模型的 Runnable，但输出的不是字符串或信息，而是与给定模式相对应的对象。模式可以指定为 JSON 模式或 Pydantic 类。如果使用 JSON 模式，Runnable 将返回一个字典；如果使用 Pydantic 类，则将返回 Pydantic 对象。

举个例子，让我们用一个模型来生成一个笑话，并将笑点和笑料分开：

```python
import getpass
import os

os.environ["OPENAI_API_KEY"] = getpass.getpass()
from langchain_openai import ChatOpenAI
llm = ChatOpenAI(model="gpt-3.5-turbo-0125")
```

如果我们希望模型返回一个 Pydantic 对象，只需传入所需的 Pydantic 类即可：

```python
from typing import Optional
from langchain_core.pydantic_v1 import BaseModel, Field

class Joke(BaseModel):
    """Joke to tell user."""
    setup: str = Field(description="The setup of the joke")
    punchline: str = Field(description="The punchline to the joke")
    rating: Optional[int] = Field(description="How funny the joke is, from 1 to 10")

structured_llm = llm.with_structured_output(Joke)
structured_llm.invoke("Tell me a joke about cats")
```

```python
Joke(setup='Why was the cat sitting on the computer?', punchline='Because it wanted to keep an eye on the mouse!', rating=8)
```

提示

除了 Pydantic 类的结构之外，Pydantic 类的名称、docstring 以及参数的名称和提供的描述也非常重要。大部分情况下，with_structured_output 使用的是模型的函数 / 工具调用 API，因此可以有效地将所有这些信息视为添加到了模型提示中。

如果您不想使用 Pydantic，我们也可以传入一个 JSON Schema dict。在这种情况下，响应也是一个 dict：

```python
json_schema = {
    "title": "joke",
    "description": "Joke to tell user.",
    "type": "object",
    "properties": {
        "setup": {
            "type": "string",
            "description": "The setup of the joke",
        },
        "punchline": {
            "type": "string",
            "description": "The punchline to the joke",
        },
        "rating": {
            "type": "integer",
            "description": "How funny the joke is, from 1 to 10",
        },
    },
    "required": ["setup", "punchline"],
}

structured_llm = llm.with_structured_output(json_schema)
structured_llm.invoke("Tell me a joke about cats")
```

```python
{'setup': 'Why was the cat sitting on the computer?',
 'punchline': 'Because it wanted to keep an eye on the mouse!',
 'rating': 8}


```

### 在多个模式之间进行选择

让模型从多个模式中进行选择的最简单方法是创建一个具有 Union 类型属性的父 Pydantic 类：

```python
from typing import Union

class ConversationalResponse(BaseModel):
    """Respond in a conversational manner. Be kind and helpful."""
    response: str = Field(description="A conversational response to the user's query")


class Response(BaseModel):
    output: Union[Joke, ConversationalResponse]

structured_llm = llm.with_structured_output(Response)
structured_llm.invoke("Tell me a joke about cats")
```

```python
Response(output=Joke(setup='Why was the cat sitting on the computer?', punchline='To keep an eye on the mouse!', rating=8))
```

```python
structured_llm.invoke("How are you today?")
```

```python
Response(output=ConversationalResponse(response="I'm just a digital assistant, so I don't have feelings, but I'm here and ready to help you. How can I assist you today?"))
```

另外，如果您选择的模型支持工具调用，您也可以直接使用工具调用，让模型在选项之间进行选择。这需要进行更多的解析和设置，但在某些情况下会带来更好的性能，因为您不必使用嵌套模式。更多详情，请参阅本操作指南。

### 流

当输出类型为 dict 时（即模式指定为 JSON Schema dict 时），我们可以对结构化模型的输出进行流式处理。

```python
structured_llm = llm.with_structured_output(json_schema)

for chunk in structured_llm.stream("Tell me a joke about cats"):
    print(chunk)
```

```python
{}
{'setup': ''}
{'setup': 'Why'}
{'setup': 'Why was'}
{'setup': 'Why was the'}
{'setup': 'Why was the cat'}
{'setup': 'Why was the cat sitting'}
{'setup': 'Why was the cat sitting on'}
{'setup': 'Why was the cat sitting on the'}
{'setup': 'Why was the cat sitting on the computer'}
{'setup': 'Why was the cat sitting on the computer?'}
{'setup': 'Why was the cat sitting on the computer?', 'punchline': ''}
{'setup': 'Why was the cat sitting on the computer?', 'punchline': 'Because'}
{'setup': 'Why was the cat sitting on the computer?', 'punchline': 'Because it'}
{'setup': 'Why was the cat sitting on the computer?', 'punchline': 'Because it wanted'}
{'setup': 'Why was the cat sitting on the computer?', 'punchline': 'Because it wanted to'}
{'setup': 'Why was the cat sitting on the computer?', 'punchline': 'Because it wanted to keep'}
{'setup': 'Why was the cat sitting on the computer?', 'punchline': 'Because it wanted to keep an'}
{'setup': 'Why was the cat sitting on the computer?', 'punchline': 'Because it wanted to keep an eye'}
{'setup': 'Why was the cat sitting on the computer?', 'punchline': 'Because it wanted to keep an eye on'}
{'setup': 'Why was the cat sitting on the computer?', 'punchline': 'Because it wanted to keep an eye on the'}
{'setup': 'Why was the cat sitting on the computer?', 'punchline': 'Because it wanted to keep an eye on the mouse'}
{'setup': 'Why was the cat sitting on the computer?', 'punchline': 'Because it wanted to keep an eye on the mouse!'}
{'setup': 'Why was the cat sitting on the computer?', 'punchline': 'Because it wanted to keep an eye on the mouse!', 'rating': 8}
```

### 少量提示

对于更复杂的模式，在提示中添加少量示例非常有用。有几种方法可以做到这一点。

最简单、最普遍的方法是在提示中的系统信息中添加示例：

```python
from langchain_core.prompts import ChatPromptTemplate

system = """You are a hilarious comedian. Your specialty is knock-knock jokes. \
Return a joke which has the setup (the response to "Who's there?") and the final punchline (the response to "<setup> who?").

Here are some examples of jokes:

example_user: Tell me a joke about planes
example_assistant: {{"setup": "Why don't planes ever get tired?", "punchline": "Because they have rest wings!", "rating": 2}}

example_user: Tell me another joke about planes
example_assistant: {{"setup": "Cargo", "punchline": "Cargo 'vroom vroom', but planes go 'zoom zoom'!", "rating": 10}}

example_user: Now about caterpillars
example_assistant: {{"setup": "Caterpillar", "punchline": "Caterpillar really slow, but watch me turn into a butterfly and steal the show!", "rating": 5}}"""

prompt = ChatPromptTemplate.from_messages([("system", system), ("human", "{input}")])

few_shot_structured_llm = prompt | structured_llm
few_shot_structured_llm.invoke("what's something funny about woodpeckers")
```

API 参考：聊天提示模板

```python
{'setup': 'Woodpecker',
 'punchline': "Woodpecker goes 'knock knock', but don't worry, they never expect you to answer the door!",
 'rating': 8}
```

当结构化输出的基本方法是工具调用时，我们可以将示例作为显式工具调用传入。您可以查看所使用的模型是否在其 API 参考中使用了工具调用。

```python
from langchain_core.messages import AIMessage, HumanMessage, ToolMessage

examples = [
    HumanMessage("Tell me a joke about planes", ),
    AIMessage(
        "",
        ,
        tool_calls=[
            {
                "name": "joke",
                "args": {
                    "setup": "Why don't planes ever get tired?",
                    "punchline": "Because they have rest wings!",
                    "rating": 2,
                },
                "id": "1",
            }
        ],
    ),
    
    ToolMessage("", tool_call_id="1"),
        
    HumanMessage("Tell me another joke about planes", ),
    AIMessage(
        "",
        ,
        tool_calls=[
            {
                "name": "joke",
                "args": {
                    "setup": "Cargo",
                    "punchline": "Cargo 'vroom vroom', but planes go 'zoom zoom'!",
                    "rating": 10,
                },
                "id": "2",
            }
        ],
    ),
    ToolMessage("", tool_call_id="2"),
    HumanMessage("Now about caterpillars", ),
    AIMessage(
        "",
        tool_calls=[
            {
                "name": "joke",
                "args": {
                    "setup": "Caterpillar",
                    "punchline": "Caterpillar really slow, but watch me turn into a butterfly and steal the show!",
                    "rating": 5,
                },
                "id": "3",
            }
        ],
    ),
    ToolMessage("", tool_call_id="3"),
]
system = """You are a hilarious comedian. Your specialty is knock-knock jokes. \
Return a joke which has the setup (the response to "Who's there?") \
and the final punchline (the response to "<setup> who?")."""

prompt = ChatPromptTemplate.from_messages(
    [("system", system), ("placeholder", "{examples}"), ("human", "{input}")]
)
few_shot_structured_llm = prompt | structured_llm
few_shot_structured_llm.invoke({"input": "crocodiles", "examples": examples})
```

API 参考：AIMessage | HumanMessage | ToolMessage

```python
{'setup': 'Crocodile',
 'punchline': "Crocodile 'see you later', but in a while, it becomes an alligator!",
 'rating': 7}
```

有关使用工具调用时少数镜头提示的更多信息，请参阅此处。

### (高级）指定结构化输出的方法

对于支持一种以上输出结构化方法的模型（即同时支持工具调用和 JSON 模式），您可以使用 method= 参数指定要使用的方法。

JSON 模式

如果使用 JSON 模式，仍需在模型提示中指定所需的模式。传递给 with_structured_output 的模式将仅用于解析模型输出，而不会像工具调用那样传递给模型。

要查看您使用的模型是否支持 JSON 模式，请查看 API 参考中的相关条目。

```python
structured_llm = llm.with_structured_output(Joke, method="json_mode")

structured_llm.invoke(
    "Tell me a joke about cats, respond in JSON with `setup` and `punchline` keys"
)
```

```python
Joke(setup='Why was the cat sitting on the computer?', punchline='Because it wanted to keep an eye on the mouse!', rating=None)
```

### (高级）原始输出

LLM 在生成结构化输出方面并不完美，尤其是当模式变得复杂时。通过 include_raw=True，可以避免引发异常并自行处理原始输出。这将改变输出格式，使其包含原始信息输出、解析值（如果成功）以及任何由此产生的错误：

```python
structured_llm = llm.with_structured_output(Joke, include_raw=True)

structured_llm.invoke(
    "Tell me a joke about cats, respond in JSON with `setup` and `punchline` keys"
)
```

```python
{'raw': AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_ASK4EmZeZ69Fi3p554Mb4rWy', 'function': {'arguments': '{"setup":"Why was the cat sitting on the computer?","punchline":"Because it wanted to keep an eye on the mouse!"}', 'name': 'Joke'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 36, 'prompt_tokens': 107, 'total_tokens': 143}, 'model_name': 'gpt-4-0125-preview', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-6491d35b-9164-4656-b75c-d7882cfb76cb-0', tool_calls=[{'name': 'Joke', 'args': {'setup': 'Why was the cat sitting on the computer?', 'punchline': 'Because it wanted to keep an eye on the mouse!'}, 'id': 'call_ASK4EmZeZ69Fi3p554Mb4rWy'}], usage_metadata={'input_tokens': 107, 'output_tokens': 36, 'total_tokens': 143}),
 'parsed': Joke(setup='Why was the cat sitting on the computer?', punchline='Because it wanted to keep an eye on the mouse!', rating=None),
 'parsing_error': None}
```

## 直接提示和解析模型输出

-----------

并非所有模型都支持 .with_structured_output()，因为并非所有模型都支持工具调用或 JSON 模式。对于这类模型，你需要直接提示模型使用特定格式，并使用输出解析器从原始模型输出中提取结构化响应。

### 使用 Pydantic 输出解析器

下面的示例使用内置的 PydanticOutputParser 来解析聊天模型的输出，提示其与给定的 Pydantic 模式相匹配。请注意，我们是从解析器的一个方法中直接向提示添加格式指令的：

```python
from typing import List

from langchain_core.output_parsers import PydanticOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.pydantic_v1 import BaseModel, Field


class Person(BaseModel):
    """Information about a person."""

    name: str = Field(..., description="The name of the person")
    height_in_meters: float = Field(
        ..., description="The height of the person expressed in meters."
    )


class People(BaseModel):
    """Identifying information about all people in a text."""

    people: List[Person]

parser = PydanticOutputParser(pydantic_object=People)

prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "Answer the user query. Wrap the output in `json` tags\n{format_instructions}",
        ),
        ("human", "{query}"),
    ]
).partial(format_instructions=parser.get_format_instructions())
```

API 参考：PydanticOutputParser | ChatPromptTemplate

让我们来看看有哪些信息会被发送到模型中：

```python
query = "Anna is 23 years old and she is 6 feet tall"

print(prompt.invoke(query).to_string())
```

```python
System: Answer the user query. Wrap the output in `json` tags
The output should be formatted as a JSON instance that conforms to the JSON schema below.

As an example, for the schema {"properties": {"foo": {"title": "Foo", "description": "a list of strings", "type": "array", "items": {"type": "string"}}}, "required": ["foo"]}
the object {"foo": ["bar", "baz"]} is a well-formatted instance of the schema. The object {"properties": {"foo": ["bar", "baz"]}} is not well-formatted.

Here is the output schema:
```

{"描述"： "识别文本中所有人物的信息"，"属性"： {"人物"： {"标题"： "人物", "type"： "array", "items"： {"$ref"： "#/definitions/Person"}}}, "required"： ["人员"]，"定义"： {"人"： {"标题"： "人"，"描述"： 关于一个人的信息 "，" 类型 "：" 对象 "，" 属性 "：" 属性 "：" 信息 "：" 对象 "，" 属性 "： {" 名称 "： {" 标题 "：" 姓名 "，" 描述 "：" 人的姓名 "，" 类型 "："string"（字符串）},"height_in_meters"： {" 标题 "：" 以米为单位的身高 "，" 描述 "：" 以米为单位的身高 "，" 类型 "：" 数字 "}}：" 数字 "}}，" 必填 "： ["name","height_in_meters"]}}}

```python
Human: Anna is 23 years old and she is 6 feet tall
```

现在让我们调用它：

```python
chain = prompt | llm | parser

chain.invoke({"query": query})
```

```python
People(people=[Person(name='Anna', height_in_meters=1.8288)])
```

如需深入了解如何使用输出解析器和提示技术进行结构化输出，请参阅本指南。

### 自定义解析

您还可以使用 LangChain 表达式语言（LCEL）创建自定义提示和解析器，使用普通函数解析模型输出：

```python
import json
import re
from typing import List

from langchain_core.messages import AIMessage
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.pydantic_v1 import BaseModel, Field

class Person(BaseModel):
    """Information about a person."""

    name: str = Field(..., description="The name of the person")
    height_in_meters: float = Field(
        ..., description="The height of the person expressed in meters."
    )


class People(BaseModel):
    """Identifying information about all people in a text."""

    people: List[Person]



prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "Answer the user query. Output your answer as JSON that  "
            "matches the given schema: ```json\n{schema}\n```. "
            "Make sure to wrap the answer in ```json and ``` tags",
        ),
        ("human", "{query}"),
    ]
).partial(schema=People.schema())



def extract_json(message: AIMessage) -> List[dict]:
    """Extracts JSON content from a string where JSON is embedded between ```json and ``` tags.

    Parameters:
        text (str): The text containing the JSON content.

    Returns:
        list: A list of extracted JSON strings.
    """
    text = message.content
    
    pattern = r"```json(.*?)```"

    
    matches = re.findall(pattern, text, re.DOTALL)

    
    try:
        return [json.loads(match.strip()) for match in matches]
    except Exception:
        raise ValueError(f"Failed to parse: {message}")
```

API 参考：AIMessage | ChatPromptTemplate

下面是发送给模型的提示：

```python
query = "Anna is 23 years old and she is 6 feet tall"

print(prompt.format_prompt(query=query).to_string())
```

```python
System: Answer the user query. Output your answer as JSON that  matches the given schema: ```json
{'title': 'People', 'description': 'Identifying information about all people in a text.', 'type': 'object', 'properties': {'people': {'title': 'People', 'type': 'array', 'items': {'$ref': '#/definitions/Person'}}}, 'required': ['people'], 'definitions': {'Person': {'title': 'Person', 'description': 'Information about a person.', 'type': 'object', 'properties': {'name': {'title': 'Name', 'description': 'The name of the person', 'type': 'string'}, 'height_in_meters': {'title': 'Height In Meters', 'description': 'The height of the person expressed in meters.', 'type': 'number'}}, 'required': ['name', 'height_in_meters']}}}
```. Make sure to wrap the answer in ```json and ``` tags
Human: Anna is 23 years old and she is 6 feet tall
```

下面是我们调用它时的样子：

```python
chain = prompt | llm | extract_json

chain.invoke({"query": query})
```

```python
[{'people': [{'name': 'Anna', 'height_in_meters': 1.8288}]}]
```

## 风险

当模型结构化输出方法应用于Agent类型数据时
- langchain_core库构建的模型结构化输出方法(with_structured_output)缺少bind_tools属性，无法适配Langgraph库构建的react_agent。这一问题限制了SQLAgent的功能和应用范围。
- 结构化输出模块的设计需要更高的灵活性，以应对复杂查询结果的多样性和不确定性。需要进一步扩展和优化pydantic模型，提升其适用性。
- 研究并解决langchain_core库与Langgraph库之间的兼容性问题，特别是bind_tools属性缺失的问题。尝试找到替代方案或进行库的调整，以确保SQLAgent的功能完整性。

---
