---
title: LangChain v0.2 学习记录
date created: 2024年7月31日,星期三,上午,11:15:58
date modified: 2024年8月7日,星期三,晚上,6:04:08
---
# LanguageChain v0.2 学习记录

- [[#LangChain 是什么|LangChain 是什么]]
- [[#LangChain Expression Language (LCEL)|LangChain Expression Language (LCEL)]]
	- [[#LangChain Expression Language (LCEL)#Runnable 接口|Runnable 接口]]
- [[#Model I/O|Model I/O]]
	- [[#Model I/O#Prompt Templates|Prompt Templates]]
	- [[#Model I/O#Language Model|Language Model]]
	- [[#Model I/O#Output Parsers|Output Parsers]]
		- [[#Output Parsers#PydanticOutputParser|PydanticOutputParser]]
		- [[#Output Parsers#JsonOutputParser：|JsonOutputParser：]]
		- [[#Output Parsers#StructuredOutputParser|StructuredOutputParser]]
		- [[#Output Parsers#模型的结构化输出|模型的结构化输出]]
- [[#Use case（Q&A with RAG）|Use case（Q&A with RAG）]]
- [[#Agent|Agent]]
	- [[#Agent#Tools（Function Calling）|Tools（Function Calling）]]
	- [[#Agent#Agent|Agent]]
	- [[#Agent#AgentExecutor|AgentExecutor]]
	- [[#Agent#SQLAgent|SQLAgent]]
- [[#Memory|Memory]]
	- [[#Memory#ConversationBufferMemory（对话缓存记忆）|ConversationBufferMemory（对话缓存记忆）]]
	- [[#Memory#ConversationBufferWindowMemory（对话缓存窗口记忆）|ConversationBufferWindowMemory（对话缓存窗口记忆）]]
	- [[#Memory#ConversationTokenBufferMemory（对话 token 缓存记忆）|ConversationTokenBufferMemory（对话 token 缓存记忆）]]
	- [[#Memory#ConversationSummaryMemory（对话摘要缓存记忆）|ConversationSummaryMemory（对话摘要缓存记忆）]]
- [[#LangChain 评估方法|LangChain 评估方法]]
- [[#LangGraph|LangGraph]]
- [[#LangSmith|LangSmith]]

[LangChain (github.com)](https://github.com/langchain-ai)

## LangChain 是什么

>[LangChain核心组成 - 掘金 (juejin.cn)](https://juejin.cn/post/7340093152862076928)

LangChain 是一个基于 `LLM` 开发应用程序的框架，把调用 LLM 的过程组成一条链的形式，具体要执行哪些函数是由 LLM 的推理结果决定的。（区别于传统程序是写死的）同时 LangChain 也是一个丰富的工具生态系统的一部分，我们可以在此框架集成并在其之上构建自己的 Agent。

![Agent 构成](https://img-blog.csdnimg.cn/direct/92395956fe6f4648b8acdb887c30d5f7.png)

LangChain 的模块组成：`Model I/O`（与语言模型进行接口）、`Retriever`（与特定于应用程序的数据进行接口）、`Memory`（在 Pipeline 运行期间保持记忆状态）、`Chain`（构建调用序列链条）、`Agent`（让管道根据高级指令选择使用哪些工具）、`Callback`（记录和流式传输任何管道的中间步骤）

![raw.githubusercontent.com/langchain-ai/langchain/c314222796798545f168f6ff7e750eb24e8edd51/docs/static/svg/langchain_stack.svg](https://raw.githubusercontent.com/langchain-ai/langchain/c314222796798545f168f6ff7e750eb24e8edd51/docs/static/svg/langchain_stack.svg)

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/3627f0b8fbf543ce969de0f7e142a18e.png)

快速安装：
![[LangChain 环境配置]]

## LangChain Expression Language (LCEL)

LangChain 应用程序的核心构建模块是 LLMChain。它结合了三个方面:

 * LLM: 语言模型是核心推理引擎。要使用 LangChain，您需要了解不同类型的语言模型以及如何使用它们。
 * Prompt Templates: 提供语言模型的指令。这控制了语言模型的输出，因此了解如何构建提示和不同的提示策略至关重要。
 * Output Parsers: 将 LLM 的原始响应转. 换为更易处理的格式，使得在下游使用输出变得容易。

每个 Langchain 组件都是 LCEL 对象，我们可以使用 `LangChain 表达式语句（LCEL）` 轻松的将各个组件链接在一起，如下实现 prompt + model + output parser 的 `chain = prompt | llm | output_parser`，其中 `|` 符号可以实现将数据从一个组件提供的输出，输入到下一个组件中：

```python
from langchain_community.llms import vllm   # this LLM class can be everyone
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are world class technical documentation writer."),
    ("user", "{input}")
])

llm = vllm.VLLM(model="/data1/huggingface/LLM/Mistral-7B-Instruct-v0.2")

output_parser = StrOutputParser()

chain = prompt | llm | output_parser

print(chain.invoke({
     "input": "how can langsmith help with testing?"}))
```

接下来仔细看一些下着三个组件：

`prompt` 是一个 `BasePromptTemplate`，这意味着它接受模板变量的字典并生成 `PromptValue`。`PromptValue` 是完整提示的包装器，可以传递给 `LLM` （将字符串作为输入）或 `ChatModel`（将一系列消息作为输入）。它可以与任何一种语言模型类型一起使用，因为它定义了生成 `BaseMessages` 和生成字符串的逻辑。

```python
prompt_value = prompt.invoke({
     "input": "how can langsmith help with testing?"})
```

打印出来可以看到，prompt\_value 是一个 `ChatPromptValue` 对象，里面的 message 是一个 list，包含不同角色 message 的对话信息。

```python
ChatPromptValue(messages=[SystemMessage(content='You are world class technical documentation writer.'), HumanMessage(content='how can langsmith help with testing?')])
```

如果 model 为 `ChatModel`，这意味着它将输出 `BaseMessage`。而如果我们的 model 是 `LLM`，它将输出一个 `字符串`。

最后，我们将 model 输出传递给 `output_parser`，这意味着 `BaseOutputParser` 它需要 `字符串或 BaseMessage` 作为输入。`StrOutputParser` 是将任何输入转换为字符串。

LCEL 可以轻松地从基本组件构建复杂的链条。它通过提供以下功能来实现此目的： 每个 LCEL 对象都实现该 Runnable 接口，该接口定义了一组通用的调用方法（`invoke`、`batch`、`stream`、`ainvoke`、 …）。这使得 LCEL 对象链也可以自动支持这些调用，大大简化了调用方式。也就是说，每个 LCEL 对象的 chain 本身就是一个 LCEL 对象。

而且每个组件都内置了与 LangSmith 的集成。如果我们设置以下两个环境变量，所有链跟踪都会记录到 LangSmith。

```python
import os
os.environ["LANGCHAIN_API_KEY"] = "..."
os.environ["LANGCHAIN_TRACING_V2"] = "true"
```

### Runnable 接口

标准接口包括：
`stream`：流回响应块（流式调用）
`invoke`：在输入上调用链（单次调用）
`batch`：在输入列表上调用链（批调用）

这些也有相应的异步方法：
`astream`：异步流回响应块
`ainvoke`：在输入异步上调用链
`abatch`：在输入列表上调用异步链
`astream_log`：除了最终响应之外，还实时流回发生的中间步骤
`astream_events`：链中发生的 betalangchain-core 流事件（ 0.1.14 中引入）

各种组件的输入输出格式：
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/98bcd9fe38a0457a9248265aac50cfd0.png)

## Model I/O

首先我们从最基本面的部分讲起，Model I/O 指的是和 LLM 直接进行交互的过程。

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/454a9a710e694a7f82347d4d91413b30.png)
在 langchain 的 Model I/O 这一流程中，LangChain 抽象的组件主要有三个：

 * Language models: 语言模型是核心推理引擎。要使用 LangChain，您需要了解不同类型的语言模型以及如何使用它们。
 * Prompt Templates: 提供语言模型的指令。这控制了语言模型的输出，因此了解如何构建提示和不同的提示策略至关重要。
 * Output Parsers: 将 LLM 的原始响应转换为更易处理的格式，使得在下游使用输出变得容易。

### Prompt Templates

![[Prompt Engineering]]

在 LangChain 中的相关组件主要有 `Prompt Template` 和 `Example selectors`，以及后面会提到的辅助/补充 Prompt 的一些其它组件

 * `Prompt Template`: 预定义的一系列指令和输入参数的 prompt 模版 (默认使用 `str.fromat` 格式化)，支持更加灵活的输入，如支持 `output instruction(输出格式指令)`, `partial input(提前指定部分输入参数)`, `examples(输入输出示例)` 等；LangChain 提供了大量方法来创建 Prompt Template，有了这一层组件就可以在不同 Language Model 和不同 Chain 下大量复用 Prompt Template 了，`Prompt Template` 中也会有下面将提到的 `Example selectors`, `Output Parser` 的参与
 * `Example selectors`: 在很多场景下，单纯的 `instruction + input` 的 prompt 不足以让 LLM 完成高质量的推理回答，这时候我们就还需要为 prompt 补充一些针对具体问题的示例（in-context learning），LangChain 将这一功能抽象为了 `Example selectors` 这一组件，我们可以基于关键字，相似度 (通常使用 `MMR` / `cosine` `similarity` / `ngram` 来计算相似度, 在后面的向量数据库章节中会提到)。为了让最终的 prompt 不超过 Language Model 的 token 上限（各个模型的 token 上限见下表），LangChain 还提供了 `LengthBasedExampleSelector`，根据长度来限制 example 数量，对于较长的输入，它会选择包含较少示例的提示，而对于较短的输入，它会选择包含更多示例。

PromptTemplate：`langchain.prompts` 中的 `PromptTemplate` 类使用 `from_template` 可以创建单个字符串类型的 prompt，需要 `str.format()` 格式化 prompt 的参数。

```python
from langchain.prompts import PromptTemplate
prompt_template = PromptTemplate.from_template(
    "Tell me a {adjective} joke about {content}."
)
prompt_template.format(adjective="funny", content="chickens")
# Tell me a funny joke about chickens.
```

ChatPromptTemplate：`langchain_core.prompts` 中的 `ChatPromptTemplate` 类，使用 `from_messages` 方法可以创建 chat message 类型的 prompt（包含 `role=system/human/ai` 和 `content=str_text` 两部分），这样就可以以 `list` 形式，构造 history 对话记录，让 llm 进行 `in-context learning`。需要 `format_messages` 格式化 prompt 的参数，得到的是 `AIMessage`, `HumanMessage`, `SystemMessage` 三者的 list 组合（当然我们也可以自己构建这个 list，并实例化对应的 Message 对象）。

```python
from langchain_core.prompts import ChatPromptTemplate
chat_template = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful AI bot. Your name is {name}."),
        ("human", "Hello, how are you doing?"),
        ("ai", "I'm doing well, thanks!"),
        ("human", "{user_input}"),
    ]
)
messages = chat_template.format_messages(name="Bob", user_input="What is your name?")
# [SystemMessage(content="You are a helpful assistant that re-writes the user's text to sound more upbeat."), HumanMessage(content="I don't like eating tasty things")]
```

其实 `PromptTemplate` 和 `ChatPromptTemplate` 都实现了 `Runnable` 接口, 支持 `LCEL` 的 invoke，ainvoke，stream，astream，batch，abatch，astream\_log 调用：

```python
from langchain.prompts import PromptTemplate
prompt_template = PromptTemplate.from_template(
    "Tell me a {adjective} joke about {subject}."
)
prompt_value = prompt_template.invoke({
            
   
     
     "adjective": "funny", "subject": "chickens"})
# prompt_value = StringPromptValue(text='Tell me a funny joke about chickens.')

from langchain_core.prompts import ChatPromptTemplate
chat_template = ChatPromptTemplate.from_messages(
    [
        ("system", "You are a helpful AI bot. Your name is {name}."),
        ("human", "Hello, how are you doing?"),
        ("ai", "I'm doing well, thanks!"),
        ("human", "{user_input}"),
    ])
prompt_value = chat_template.invoke({
            
   
     
     "name": "Bob", "user_input": "What is your name?"})
print(type(prompt_value))
# prompt_value = ChatPromptValue(messages=[SystemMessage(content='You are a helpful AI bot. Your name is Bob.'), HumanMessage(content='Hello, how are you doing?'), AIMessage(content="I'm doing well, thanks!"), HumanMessage(content='What is your name?')])
```

FewShotPromptTemplate：当然更加专业的 in-context learnding 方式是使用 `FewShotPromptTemplate` 显示的指定 Examples，构造 `few shot prompt`。使用字典来构造 examples，每个 example 都是一个 dict，多个 examples 存储在一个 list 中。当我们给定了很多 examples 时，这需要使用 `Example Selector` 利用 `MMR` / `cosine` `similarity` / `ngram` 来计算 input 和 examples 之间的语义相似度，来决定选择哪些 examples。

如下给出反义词的 task（使用基于 example 长度的 `LengthBasedExampleSelector`）：

```python
# 1. 定义example variables list
examples = [
    {
     
     "input": "happy", "output": "sad"},
    {

     "input": "tall", "output": "short"},
    {

     "input": "energetic", "output": "lethargic"},
    {

     "input": "sunny", "output": "gloomy"},
    {

     "input": "windy", "output": "calm"},
]
# 2. 为examples定义example_prompt模板
example_prompt = PromptTemplate(
    input_variables=["input", "output"],
    template="Input: {input}\nOutput: {output}",
)
# 3. 将example variables list和example_prompt输入到example_selector中
example_selector = LengthBasedExampleSelector(
    examples=examples,
    example_prompt=example_prompt,
    max_length=25)
# 4. 定义few_shot_prompt模板（prefix + examples + suffix）
dynamic_prompt = FewShotPromptTemplate(
    example_selector=example_selector,
    example_prompt=example_prompt,
    prefix="Give the antonym of every input",  # 前缀，定义task
    suffix="Input: {adjective}\nOutput:",  # 后缀
    input_variables=["adjective"],  # 后缀参数变量
)
print(dynamic_prompt.invoke({"adjective": "funny"}).text)
```

得到的 `dynamic_prompt = prefix + examples + suffix` 如下：

```python
Give the antonym of every input

Input: happy
Output: sad

Input: tall
Output: short

Input: energetic
Output: lethargic

Input: sunny
Output: gloomy

Input: windy
Output: calm

Input: funny
Output:
```

当然，我们的 examples 不仅可以使用 `PromptTemplate` 构造，还可以使用 `ChatPromptTemplate` 和 `FewShotChatMessagePromptTemplate` 构造 chat 的 few shot messages list：

```python
from langchain_core.prompts import (
    FewShotChatMessagePromptTemplate,
    ChatPromptTemplate)
# 1. ChatPromptTemplate和FewShotChatMessagePromptTemplate构造few_shot_prompt 
examples = [
    {
     "input": "2+2", "output": "4"},
    {
     "input": "2+3", "output": "5"},
]
example_prompt = ChatPromptTemplate.from_messages(
    [('human', '{input}'), ('ai', '{output}')]
)
few_shot_prompt = FewShotChatMessagePromptTemplate(
    examples=examples,
    # This is a prompt template used to format each individual example.
    example_prompt=example_prompt,
)
# 2. ChatPromptTemplate和few_shot_prompt构造final_prompt 
final_prompt = ChatPromptTemplate.from_messages(
    [
        ('system', 'You are a helpful AI Assistant'),
        few_shot_prompt,
        ('human', '{input}'),
    ]
)
final_prompt.format(input="What is 4+4?")
print(final_prompt)
```

```python
input_variables=['input'] messages=[SystemMessagePromptTemplate(prompt=PromptTemplate(input_variables=[], template='You are a helpful AI Assistant')), FewShotChatMessagePromptTemplate(examples=[{'input': '2+2', 'output': '4'}, {'input': '2+3', 'output': '5'}], example_prompt=ChatPromptTemplate(input_variables=['input', 'output'], messages=[HumanMessagePromptTemplate(prompt=PromptTemplate(input_variables=['input'], template='{input}')), AIMessagePromptTemplate(prompt=PromptTemplate(input_variables=['output'], template='{output}'))])), HumanMessagePromptTemplate(prompt=PromptTemplate(input_variables=['input'], template='{input}'))]
```

除基于长度的 `LengthBasedExampleSelector` 之外：
①基于文本余弦相似度的 `MaxMarginalRelevanceExampleSelector` 和 `SemanticSimilarityExampleSelector` 需要对 examples 进行 embedding 所以需要使用到 OpenAI 的 API 或其他本地 Embeddings 模型，选择最相似的 example。
② `NGramOverlapExampleSelector` 根据 ngram 重叠分数，根据与输入最相似的 example 来选择示例并对其进行排序。ngram 重叠分数是 0.0 到 1.0 之间的浮点数（含 0.0 和 1.0）。

### Language Model

当前 LLM 发展现状：`Online LLM` 全面领先于 `Opensource LLM`。

![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/8d939d5510624b30a77627c50944c383.png)

Language Model 是真正与 LLM / ChatModel 进行交互的组件，它可以直接被当作普通的 openai client 来使用，在 LangChain 中，主要使用到的是 `LLM`，`Chat Model` 两类 Language Model

 * LLM: 最基础的通过 `“text_str in ➡️ text_str out”` 模式来使用的 Language Model，另一方面，`LangChain也收录了大量的第三方LLM`。【注意：langchain v 0.2.0 以后使用 `langchain_community` 导入 `llms` 模块】

```python
from langchain.llms import vllm  # langchain v 0.1.0
from langchain_community.llms import vllm  # langchain v 0.2.0
llm = vllm.VLLM(model="/data1/huggingface/LLM/Mistral-7B-Instruct-v0.2")
llm.invoke({     
     "input": "how can langsmith help with testing?"})
```

 * Chat Model : 另一方面，LangChain 也收录了大量的第三方 Chat Model，是 LLM 的变体，抽象了 Chat 这一场景下的使用模式，由 `“text_str in ➡️ text_str out”` 变成了 `“chat_messages in ➡️ chat_message out”`。`chat_message` 由 2 个组件组成：`text` \+ `message type(System/Human/AI)`。【注意：langchain v 0.2.0 以后使用 `langchain_community` 导入 `chat_models` 模块】，3 种 message type 如下：
	 * `System message` \- 告诉 AI 要做什么的背景信息上下文，用于设定 llm 角色。
	 * `Human message` \- 标识用户传入的消息类型
	 * `AI message` \- 标识 AI 返回的消息类型

```python
from langchain_openai import ChatOpenAI
chat = ChatOpenAI(openai_api_key="...")
from langchain_core.messages import HumanMessage, SystemMessage
messages = [
    SystemMessage(content="You're a helpful assistant"),
    HumanMessage(content="What is the purpose of model regularization?"),
]
chat.invoke(messages)
```

### Output Parsers

`langchain.output_parsers` 中的各种解析器，通常我们希望 Language Model 的输出是固定的格式，以支持我们解析其输出为结构化数据，LangChain 将这一诉求所需的功能抽象成了 Output Parser 这一组件，并提供了一系列的预定义 Output Parser，如最常用的 `StructuredOutputParser`, `PydanticOutputParser`，`JsonOutputParser`，以及在 LLM 输出无法解析时发挥作用的 `Auto-fixing parser` 和 `Retry parser` 等。

#### PydanticOutputParser

`PydanticOutputParser` 可以使用 Pydantic 数据类作为输出格式，将 Pydantic 数据类导入解析器中。

所有数据类都要继承的 Pydantic 的 `BaseModel` 就像 Python 数据类，但具有实际的类型检查 + 强制。创建 prompt 时要用 `partial_variables` 部分填入变量 `parser.get_format_instructions()`。

如下面的例子，生成一问一答的冷笑话（输出的数据类包含 `问句setup` 和 `答句punchline`）：

```python
from typing import List
from langchain.output_parsers import PydanticOutputParser
from langchain.prompts import PromptTemplate
from langchain_core.pydantic_v1 import BaseModel, Field, validator

# Define your desired data structure.
class Joke(BaseModel):
    setup: str = Field(description="question to set up a joke")  # 笑话设置的部分
    punchline: str = Field(description="answer to resolve the joke")  # 笑话的冷笑话部分

    # 对笑话进行逻辑验证，保证setup的部分以？结尾
    @validator("setup")
    def question_ends_with_question_mark(cls, field):
        if field[-1] != "?":
            raise ValueError("Badly formed question!")
        return field

llm = vllm.VLLM(
    model="/data1/huggingface/LLM/Mistral-7B-Instruct-v0.2",
    trust_remote_code=False,  # mandatory for hf models
    max_new_tokens=128,
    top_k=10,
    top_p=0.95,
    temperature=0.8, # 生成结果的发散程度
)

# And a query intented to prompt a language model to populate the data structure.
joke_query = "Tell me a joke."

# Set up a parser + inject instructions into the prompt template.
parser = PydanticOutputParser(pydantic_object=Joke)

prompt = PromptTemplate(
    template="Asnwer the user query.\n{format_instructions}\n{query}\n",
    input_variables=["query"],
    partial_variables={
            
   
     
     "format_instructions": parser.get_format_instructions()},
)

chain = prompt | llm | parser

print(chain.invoke({
     "query": joke_query}))
```

```python
Joke(setup="Why don't scientists trust atoms?", punchline='Because they make up everything!')
```

#### JsonOutputParser：

`JsonOutputParser` 解析器允许用户指定任意 JSON 模式并查询符合该模式的输出的 LLM。

```python
from typing import List

from langchain.prompts import PromptTemplate
from langchain_core.output_parsers import JsonOutputParser
from langchain_core.pydantic_v1 import BaseModel, Field
from langchain_openai import ChatOpenAI
model = ChatOpenAI(temperature=0)
# Define your desired data structure.
class Joke(BaseModel):
    setup: str = Field(description="question to set up a joke")
    punchline: str = Field(description="answer to resolve the joke")
    # And a query intented to prompt a language model to populate the data structure.
joke_query = "Tell me a joke."

# Set up a parser + inject instructions into the prompt template.
parser = JsonOutputParser(pydantic_object=Joke)

prompt = PromptTemplate(
    template="Answer the user query.\n{format_instructions}\n{query}\n",
    input_variables=["query"],
    partial_variables={
    "format_instructions": parser.get_format_instructions()},
)

chain = prompt | model | parser

chain.invoke({
     "query": joke_query})
```

```python
{'setup': "Why don't scientists trust atoms?",
 'punchline': 'Because they make up everything!'}
```

#### StructuredOutputParser

当您想要返回多个字段时，可以使用此输出解析器。虽然 Pydantic/JSON 解析器更强大，但这个解析器对于生成功能较弱的 LLM 模型很有用。下面例子是生成问题的答案，并给出参考的 web 网站 url。

```python
from langchain.output_parsers import ResponseSchema, StructuredOutputParser
from langchain_core.prompts import PromptTemplate, load_prompt
from langchain_community.llms import vllm
# 响应模式指导LLM的输出解析格式，传入StructuredOutputParser
response_schemas = [
    ResponseSchema(name="answer", description="answer to the user's question"),
    ResponseSchema(name="source", description="source used to answer the user's question, should be a website."),
]
output_parser = StructuredOutputParser.from_response_schemas(response_schemas)
format_instructions = output_parser.get_format_instructions()
print(format_instructions)
prompt = PromptTemplate(
    template="answer the users question as best as possible.\n{format_instructions}\n{question}",
    input_variables=["question"],
    partial_variables={
    
     "format_instructions": format_instructions},
)
print(prompt)
llm = vllm.VLLM(
    model="/data1/huggingface/LLM/Mistral-7B-Instruct-v0.2",
    trust_remote_code=False,  # mandatory for hf models
    max_new_tokens=128,
    top_k=10,
    top_p=0.95,
    temperature=0.8, # 生成结果的发散程度
)

chain = prompt | llm | output_parser
print(chain.invoke({
     "question": "what's the capital of france?"}))
```

```python
{'answer': 'The capital of France is Paris.',
 'source': 'https://en.wikipedia.org/wiki/Paris'}
```

其中 `format_instructions` 如下：

```python
The output should be a markdown code snippet formatted in the following schema, including the leading and trailing "```json" and "```":
```

``` json
{     
        "answer": string  // answer to the user's question
        "source": string  // source used to answer the user's question, should be a website.
}
```

#### 模型的结构化输出

![[LangChain 从模型返回结构化数据#LangChain从模型返回结构化数据]]

## Use case（Q&A with RAG）

LLM 支持的最强大的应用程序之一是复杂的问答 （Q&A） 聊天机器人。这些应用程序可以回答有关特定源信息的问题。这些应用使用一种称为检索增强生成 （RAG） 的技术。（RAG 是一种利用 `外部数据库` 增强 LLM 知识的技术）

典型的 RAG 应用程序有两个主要组件：

 * `Indexing`：用于从源获取数据并为其建立索引的 pipeline。这通常发生在离线状态。
	![](https://img-blog.csdnimg.cn/direct/75ea877cd3134fb4b992140ebf287fff.png)

	1. `Load`：首先我们需要加载数据。这是通过 Document Loaders 完成的。
	2. `Split`：文本分割器将大文本分割 Documents 成更小的文本块。这对于索引数据和将其传递到模型都很有用，因为大块更难搜索并且不适合模型的有限上下文窗口。
	3. `Store`：我们需要某个地方来存储和索引我们的分割，以便以后可以搜索它们。这通常是使用 VectorStore 和 Embeddings 模型来完成的。
 * `Retrieval and generation`：实际的 RAG 链，它在运行时接受用户查询，并从 indexing 中检索相关数据，然后将其传递给模型。
	![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/553a89630ef34b549728e2eff98150b1.png)

	1. `Retrieve`：给定用户输入，使用 Retriever 从存储中检索相关的分割。
	2. `Generate`：ChatModel / LLM 使用包含问题和检索到的数据的提示生成答案

## Agent

### Tools（Function Calling）

工具是代理调用的函数。这里有两个重要的考虑因素：一是让代理能访问到正确的工具，二是以最有帮助的方式描述这些工具。如果你没有给代理提供正确的工具，它将无法完成任务。如果你没有正确地描述工具，代理将不知道如何使用它们。LangChain 提供了一系列的工具，同时你也可以定义自己的工具，或者将函数转换为 tool。

`Tools` 是 Agent 可以用来与世界交互的 `函数 API`。`Tools` 应该包含以下信息：

* 工具名称 `Tools Name (tool. name)`
* 该工具是什么的描述 `Tools Description (tool. description)`
* 工具输入内容的 JSON 架构 `Input Json Mode (tool. args)`
* 要调用的函数 `Function Calling`
* 工具的结果是否应直接返回给用户 `Return Result (tool. return_direct)`

以 `QuerySQLDataBaseTool` 为例：

``` python
    class BaseSQLDatabaseTool(BaseModel):
        """Base tool for interacting with a SQL database."""
        # SQLDatabase 类是一个对 SQLAlchemy 数据库操作的封装，它提供了一种结构化的方式来与数据库进行交互。
        db: SQLDatabase = Field(exclude=True)
        class Config(BaseTool.Config):
    	    pass
    
    class QuerySQLDataBaseTool(BaseSQLDatabaseTool, BaseTool):
    	"""Tool for querying a SQL database."""
    	name: str = "sql_db_query"
    	description: str = """
    	Execute a SQL query against the database and get back the result..
    	If the query is not correct, an error message will be returned.
    	If an error is returned, rewrite the query, check the query, and try again.
    	"""
    	args_schema: Type[BaseModel] = _QuerySQLDataBaseToolInput
    	
    	def _run(
    		self,
    		query: str,
    		"""
    		CallbackManagerForToolRun是一个为工具运行定义的回调管理器类
          
            初始化: 在工具开始运行之前，run_manager可能被初始化为CallbackManagerForToolRun的一个实例，或者保持为None，这取决于是否需要事件处理或回调管理功能。
    		
    		注册回调: 如果使用，run_manager可能提供方法来注册一系列回调函数，这些函数对应于工具运行期间可能发生的不同事件。
    		
    		事件触发: 在工具的执行过程中，当特定事件发生时（如工具启动、停止或遇到错误），run_manager会负责触发与这些事件相关联的回调函数。
    		
    		资源清理: 在工具运行完成后，run_manager可能还负责清理资源，注销回调函数，确保优雅地终止。
    		"""
    		run_manager: Optional[CallbackManagerForToolRun] = None,
    		) -> Union[str, Sequence[Dict[str, Any]], Result]:
    		"""Execute the query, return the results or an error message."""
    		return self.db.run_no_throw(query)

```

上述信息可以用于提示 LLM，以便它知道如何指定要采取的操作，然后要调用的函数相当于采取该操作。工具的输入越简单，LLM 就越容易使用它。许多 Agent 只能使用具有单个字符串输入的工具。重要的是，Name、Description 都在 Prompt 中使用。因此，清晰并准确地描述如何使用该工具非常重要。如果 LLM 不了解如何使用该工具，可能需要更改默认 Name、Description。

**工具包（Toolkits）**：是旨在一起用于特定任务并具有方便的加载方法的工具的集合。`from langchain. agents import load_tools` 中所有工具包都公开一个 `get_tools` 返回**工具列表的方法**，创建 Agent 的函数也和具体的 tools 有关，代码如下：

```python
    from langchain_community.agent_toolkits import SQLDatabaseToolkit
    toolkit = SQLDatabaseToolkit(db=db, llm=llm)
    tools = toolkit.get_tools()
    agent_executor = create_react_agent(llm, tools, messages_modifier=system_message)
```

**定义自定义工具函数（Tools Function）**：在构建自己的 Agent 时，您需要为其提供可以使用的**工具列表**，即上述的 `tools`。除了调用的**实际 tools 执行函数**之外，该工具还包含几个组件：

* `name (str)`，是必需的，并且在提供给代理的一组工具中必须是唯一的
* `description (str)`，是可选的，但建议使用，因为代理使用它来确定工具的使用
* `args_schema (Pydantic BaseModel)` 是可选的，但推荐使用，可用于提供更多信息（例如，少数样本示例）或对预期参数的验证。

**`@tool` 装饰器**：是定义 `自定义工具函数 Tools Function` 的最简单方法。装饰器默认使用**函数名**作为 `Tools Name`，但是可以通过给 `@tool` 传递字符串作为第一个参数来覆盖 `Tools Name`。此外，装饰器将使用**函数的文档字符串**作为 `Tools Description`，因此必须提供文档字符串。

```python 
    from langchain.tools import BaseTool, tool
    @tool
    def search(query: str) -> str:
        """Look up things online."""
        return "LangChain"
    
    print(search.name)
    search
    print(search.description)
    search(query: str) -> str - Look up things online.
    print(search.args)
    {'query': {'title': 'Query', 'type': 'string'}}
```

还可以通过将`Tools Name` 和 `JSON 参数`传递到**工具装饰器**`**@tool ("tool_name", args_schema=SearchInput, return_direct=True)**` 中来自定义它们。

```python 
    from langchain.pydantic_v1 import BaseModel, Field
    from langchain.tools import tool
    class SearchInput(BaseModel):
        query: str = Field(description="should be a search query")
        
    @tool("search-tool", args_schema=SearchInput, return_direct=True)
    def search(query: str) -> str:
        """Look up things online."""
        return "LangChain"
    # search-tool
    # search-tool(query: str) -> str - Look up things online.
    # {'query': {'title': 'Query', 'description': 'should be a search query', 'type': 'string'}}
    # True
```

**`StructuredTool`数据类**：也可以用来自定义`Tools Function`，相比于`**@tool**`**装饰器**更加规范：

```python
    from langchain.tools import StructuredTool
    def search_function(query: str):
        return "LangChain"
    search = StructuredTool.from_function(
        func=search_function,
        name="Search",
        description="useful for when you need to answer questions about current events",
        # args_schema=... <也可以自定义以提供有关输入class的更多信息>
        # coroutine= ... <- you can specify an async method if desired as well
    )
    print(search.name)
    # Search
    print(search.description)
    # Search(query: str) - useful for when you need to answer questions about current events
    print(search.args)
    # {'query': {'title': 'Query', 'type': 'string'}}
```

![[LangChain 工具自定义#继承 BaseTool 类自定义工具]]

### Agent

Agent 可以看做在 Chain 的基础上，进一步整合 Tool 的高级模块。与 Chain 相比，Agent 具有两个新增的能力：思考链和工具箱。**能根据环境变化更新计划，使决策更加健壮**。思考链释放了大模型的规划和调度潜能，是 Agent 的关键创新，Agent 定义了工具的标准接口，以实现无缝集成。与 Chain 直接调用模块 Runnable 相比，它只关心 tool 输入和输出，tool 内部实现对 Agent 透明，工具箱大大扩展了 Agent 的外部知识来源，使其离真正的通用智能更近一步。

常用的 Agent 类型如下：

* **Conversational Agent** 
	这类 Agent 可以根据 Language Model 的输出决定是否使用指定的 Tool，以及使用什么 Tool (这里的 Tool 也可以是一个 Chain)，以及时的为 Model I/O 的过程补充信息

* **OpenAI functions Agent** 
	类似 Conversational Agent，但它能够让 Agent 更进一步地帮忙提取指定 Tool 的参数等，甚至使用多个 Tools

* **Plan and execute Agent** 
	抽象 Agent“决定做什么”的过程为“planning what to do”和“executing the sub tasks”(这种方法来自"Plan-and-Solve"这一篇论文)

* **Self ask with search**
	这类 Agent 会基于 LLM 的输出，自行调用 Tools 以及 LLM 来进行额外的搜索和自查，以达到拓展和优化输出的目的

*    **ReAct Agent** 
	让 LLM 反复执行`Reasoning 推理`和`Action 执行`的方法，类似 OpenAI functions Agent，提供了一个更加明确的框架以及由论文支撑的方法

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLwL4WLGmqpz2/img/89749408-1fc5-4fe7-906b-666e8c2010c6.png)

LangChain 中的 SQL Agent 的类型为 ReAct Agent。

使用 `LLM`、`Prompts` 和 `Tools` 来初始化 Agent。Agent 负责接收输入并决定采取什么操作。至关重要的是，Agent 不执行这些操作，这是由 AgentExecutor 完成的（下一步）。

### AgentExecutor

**代理执行器（AgentExecutor）**：代理执行器是代理的运行环境，它调用代理并执行代理选择的操作。执行器也负责处理多种复杂情况，包括处理代理选择了不存在的工具的情况、处理工具出错的情况、处理代理产生的无法解析成工具调用的输出的情况，以及在代理决策和工具调用进行观察和日志记录。AgentExecuter 负责迭代运行 Agent，**直至满足设定的停止条件**，这使得 Agent 能够像生物一样循环处理信息和任务。

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLwL4WLGmqpz2/img/f2b46e7c-b479-4468-9bb1-25a1ed55db37.png)

AgentExecutor 由一个 Agent 和 Tool 的集合组成。AgentExecutor 负责调用 Agent，获取返回 call\_model 和 acall\_model，它们接收代理状态和配置作为参数，并调用 model\_runnable 的 invoke 或 ainvoke 方法来执行模型。如果是最后一步且有工具调用，则返回一条提示消息；否则返回模型的响应消息。

```python 
    def create_react_agent(
        model: LanguageModelLike,
        tools: Union[ToolExecutor, Sequence[BaseTool]],
        messages_modifier: Optional[Union[SystemMessage, str, Callable, Runnable]] = None,
        checkpointer: Optional[BaseCheckpointSaver] = None,
        interrupt_before: Optional[Sequence[str]] = None,
        interrupt_after: Optional[Sequence[str]] = None,
        debug: bool = False,
    ) -> CompiledGraph:
        """
        1. 检查tools的类型，如果是ToolExecutor类型，则获取其tools属性，否则直接使用tools。
    
        2. 将model与tool_classes绑定，以便在后续调用中使用。
    
        3. 定义了一个函数should_continue，用于判断是否继续执行工作流程。它检查最后一条消息是否有工具调用，如果没有则返回"end"，否则返回"continue"。
    
        4. 根据messages_modifier的类型，创建一个可运行对象model_runnable。如果messages_modifier为None，则直接使用model；如果为字符串，则创建一个SystemMessage对象，并将其作为第一条消息传入model；如果为SystemMessage对象，则将其作为第一条消息传入model；如果为可调用对象或可运行对象，则将其与model组合。
    
        5. 定义了两个调用模型的函数call_model和acall_model。它们接收代理状态和配置作为参数，并调用model_runnable的invoke或ainvoke方法来执行模型。如果是最后一步且有工具调用，则返回一条提示消息；否则返回模型的响应消息。
    
        6. 创建一个新的状态图workflow，使用AgentState作为状态的类型。
    
        7. 添加两个节点到状态图中，分别是代理节点和工具节点。
    
        8. 将入口点设置为代理节点，这意味着首先调用代理节点。
    
        9. 添加条件边，根据should_continue函数的返回值决定下一个调用的节点。如果返回"continue"，则调用工具节点；如果返回"end"，则结束。
    
        10. 添加从工具节点到代理节点的普通边，表示在调用工具节点后，下一个调用的是代理节点。
    
        11. 最后，将状态图编译成一个可运行对象，并返回。这个可运行对象可以像其他可运行对象一样使用。
    
        """
        if isinstance(tools, ToolExecutor):
            tool_classes = tools.tools
        else:
            tool_classes = tools
        model = model.bind_tools(tool_classes)
        
        # 定义判断是否继续的函数
        def should_continue(state: AgentState):
            messages = state["messages"]
            last_message = messages[-1]
            # 如果最后一条消息没有工具调用，则结束
            if not last_message.tool_calls:
                return "end"
            # 否则继续
            else:
                return "continue"
                
         # 添加消息修改器（如果存在）
        if messages_modifier is None:
            # 如果messages_modifier为None，则直接使用model作为可运行对象
            model_runnable = model
        elif isinstance(messages_modifier, str):
            # 如果messages_modifier是字符串，则创建一个SystemMessage对象，并将其作为第一条消息传入model
            _system_message: BaseMessage = SystemMessage(content=messages_modifier)
            model_runnable = (lambda messages: [_system_message] + messages) | model
        elif isinstance(messages_modifier, SystemMessage):
            # 如果messages_modifier是SystemMessage对象，则将其作为第一条消息传入model
            model_runnable = (lambda messages: [messages_modifier] + messages) | model
        elif isinstance(messages_modifier, (Callable, Runnable)):
            # 如果messages_modifier是可调用对象或可运行对象，则将其与model组合
            model_runnable = messages_modifier | model
        else:
            # 如果messages_modifier的类型不符合预期，则抛出ValueError异常
            raise ValueError(
                f"Got unexpected type for `messages_modifier`: {type(messages_modifier)}"
            )
      
    
        # 定义调用模型的函数
        def call_model(
            state: AgentState,
            config: RunnableConfig,
        ):
            messages = state["messages"]
            response = model_runnable.invoke(messages, config)
            # state["is_last_step"] 是一个表示当前步骤是否为最后一步的状态变量。如果它的值为真，意味着当前步骤是最后一步。
            # response.tool_calls 是一个表示工具调用的变量。如果它的值存在（非空），意味着有工具被调用。
            # 因此，这段代码的意思是：当当前步骤是最后一步，并且有工具被调用时，执行条件语句中的代码块。
            if state["is_last_step"] and response.tool_calls:
                return {
                    "messages": [
                        AIMessage(
                            id=response.id,
                            content="Sorry, need more steps to process this request.",
                        )
                    ]
                }
            # 返回一个列表，因为这将被添加到现有列表中
            return {"messages": [response]}
        # 异步函数是一种特殊类型的函数，它允许你使用 await 关键字来暂停函数的执行，等待异步操作完成，而不会阻塞整个程序的执行。这对于 I/O 密集型任务特别有用，比如网络请求、文件读写等。
        async def acall_model(
            state: AgentState,
            config: RunnableConfig
        ):
            messages = state["messages"]
            response = await model_runnable.ainvoke(messages, config)
            if state["is_last_step"] and response.tool_calls:
                return {
                    "messages": [
                        AIMessage(
                            id=response.id,
                            content="Sorry, need more steps to process this request.",
                        )
                    ]
                }
            # 返回一个列表，因为这将被添加到现有列表中
            return {"messages": [response]}
     
    
        # 定义一个新的图
        workflow = StateGraph(AgentState)
        # 定义两个节点
        workflow.add_node("agent", RunnableLambda(call_model, acall_model))
        workflow.add_node("tools", ToolNode(tools))
        # 将入口点设置为`agent`
        # 这意味着首先调用这个节点
        workflow.set_entry_point("agent")
        # 添加条件边
        workflow.add_conditional_edges(
            # 首先，定义起始节点，使用`agent`
            # 这意味着在调用`agent`节点后，将采取这些边
            "agent",
            # 接下来，传入决定下一个调用哪个节点的函数
            should_continue,
            # 最后，传入一个映射。
            # 键是字符串，值是其他节点。
            # END是一个特殊的节点，表示图应该结束。
            # 将调用`should_continue`函数，然后将其输出与此映射中的键进行匹配。
            # 根据匹配的结果，将调用相应的节点。
            {
                # 如果是`tools`，则调用工具节点。
                "continue": "tools",
                # 否则结束。
                "end": END,
            },
        )
        # 添加从`tools`到`agent`的普通边。
        # 这意味着在调用`tools`后，下一个调用的是`agent`节点。
        workflow.add_edge("tools", "agent")
        # 最后，编译它！
        # 这将把它编译成一个LangChain Runnable，
        # 这意味着您可以像使用其他可运行对象一样使用它
        return workflow.compile(
            checkpointer=checkpointer,
            interrupt_before=interrupt_before,
            interrupt_after=interrupt_after,
            debug=debug,
        )
```

### SQLAgent

![[SQLAgent based GLM]]

Agent 是无记忆的。这意味着它不记得以前的交互。为了给它记忆，我们需要传入 `history_message`。

## Memory

当我们在使用大型语言模型进行聊天对话时，**大型语言模型本身实际上是无状态的。语言模型本身并不记得到目前为止的历史对话**。每次调用 API 结点都是独立的。

聊天机器人似乎有记忆，只是因为通常有快速的代码可以向 LLM 提供迄今为止的完整对话以及上下文。因此，Memory 可以明确地存储到目前为止的所有术语或对话。这个 Memory 存储器被用作输入或附加上下文到 LLM 中，以便它可以生成一个输出，就好像它只有在进行下一轮对话的时候，才知道之前说过什么。

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/eYVOLwL4WLGmqpz2/img/8b3cff29-51b6-4d83-a3ba-fc218c8ee9a6.png)

常用的 Memory 类型如下：

* **Chat Messages**：最简单 Memory，将历史 Chat 记录作为补充信息放入 prompt 中
* **Vector store-based memory** ：基于向量数据库的 Memory，将 memory 存储在向量数据库中，并在每次调用时查询 TopK 的最“重要”的文档这与大多数其他 memory 类型的不同之处在于，它不显式跟踪交互的顺序，最多基于部分元数据筛选一下向量数据库的查询范围
* **Conversation buffer (window) memory**： 保存一段时间内的历史 Chat 记录，它只使用最后 K 个记录（仅保持最近交互的滑动窗口），这样 buffer 也就不会变得太大，避免超过 token 上限
* **Conversation summary memory**： 这种类型的 Memory 会随着 Chat 的进行创建对话的摘要，并将当前摘要存储在 Memory 中，用于后续对话的 history 提示；这种 memory 方案对长会话非常有用，但频繁的总结摘要会耗费大量的 token
* **Conversation Summary Buffer Memory**： 结合了 buffer memory 和 summary memory 的策略，依旧会在内存中保留最后的一些 Chat 记录作为 buffer，并在 buffer 的总 token 数达到预置的上限后，对所有 Chat 记录总结摘要作为 SystemMessage 并清理其它历史 Messages；这种 memory 方案结合了 buffer memory 和 summary memory 的优点，既不会频繁地总结摘要消耗 token，也不会让 buffer 缺失过多信息

### ConversationBufferMemory（对话缓存记忆）

ConversationBufferMemory 是一个最简单的记忆组件，他不对数据结构和获取算法做任何讲过，就是简单的原进原出。

``` python
OPENAI_API_KEY = os.environ['OPENAI_API_KEY']
llm = ChatOpenAI(temperature=0.0,openai_api_key=OPENAI_API_KEY)
memory  = ConversationBufferMemory()
conversation = ConversationChain(   #新建一个对话链（关于链后面会提到更多的细节）
    llm=llm, 
    memory = memory ,
    verbose=False
    #查看Langchain实际上在做什么，设为FALSE的话只给出回答，看到不到下面绿色的内容
)

conversation.predict(input="Hi there! I am Sam")
conversation.predict(input="How are you today?")
conversation.predict(input="Can you help me with some customer support?")
print(conversation.memory.buffer)
"""
Human: Hi there! I am Sam
AI: Hello Sam! It's nice to meet you. How can I assist you today?
Human: How are you today?
AI: I'm doing well, thank you for asking! I'm here and ready to help with any questions or tasks you have. What can I assist you with today, Sam?
Human: Can you help me with some customer support?
AI: Of course, I'd be happy to help with customer support! What specific issue are you facing or what question do you have? Let me know so I can assist you better.
"""
```

### ConversationBufferWindowMemory（对话缓存窗口记忆）

随着对话变得越来越长，所需的内存量也变得非常长。将大量的 tokens 发送到 LLM 的成本，也会变得更加昂贵, 这也就是为什么 API 的调用费用，通常是基于它需要处理的 tokens 数量而收费的。

针对以上问题，LangChain 也提供了几种方便的 memory 来保存历史对话。其中，对话缓存窗口记忆只保留一个窗口大小的对话缓存区窗口记忆。它只使用最近的 n 次交互。这可以用于保持最近交互的滑动窗口，以便缓冲区不会过大。

``` python
OPENAI_API_KEY = os.environ['OPENAI_API_KEY'] #专属的API key"
llm = ChatOpenAI(temperature=0.0,openai_api_key=OPENAI_API_KEY)
memory  = ConversationBufferWindowMemory(k=1)
conversation = ConversationChain(   #新建一个对话链（关于链后面会提到更多的细节）
    llm=llm, 
    memory = memory,
    verbose=False
    #查看Langchain实际上在做什么，设为FALSE的话只给出回答，看到不到下面绿色的内容
)

conversation.predict(input="Hi there! I am Sam")
conversation.predict(input="How are you today?")
conversation.predict(input="Can you help me with some customer support?")
print(conversation.memory.buffer)
"""
Human: Can you help me with some customer support?
AI: Of course, I can help with that! What specific issue are you experiencing with your customers? Let me know the details so I can provide you with the best possible solution.
"""
```

### ConversationTokenBufferMemory（对话 token 缓存记忆）

使用对话 token 缓存记忆，内存将限制保存的 token 数量。如果 token 数量超出指定数目，它会切掉这个对话的早期部分以保留与最近的交流相对应的 token 数量，但不超过 token 限制。

``` python
OPENAI_API_KEY = os.environ['OPENAI_API_KEY'] #专属的API key"
llm = ChatOpenAI(temperature=0.0,openai_api_key=OPENAI_API_KEY)
summary_memory  = ConversationSummaryMemory(llm=llm)
conversation = ConversationChain(   #新建一个对话链（关于链后面会提到更多的细节）
    llm=llm, 
    memory = summary_memory ,
    verbose=False
    #查看Langchain实际上在做什么，设为FALSE的话只给出回答，看到不到下面绿色的内容
)

conversation.predict(input="Hi there! I am Sam")
conversation.predict(input="How are you today?")
conversation.predict(input="Can you help me with some customer support?")
print(conversation.memory.buffer)
"""
The human introduces themselves as Sam, and the AI introduces itself as AI. 
The AI then asks what brings Sam here today, and Sam asks how the AI is doing. 
The AI responds that it is doing great, and when Sam asks for customer support,
the AI responds positively and asks what kind of customer support Sam needs.
"""
```

### ConversationSummaryMemory（对话摘要缓存记忆）

这种 Memory 的想法是，不是将内存限制为基于最近对话的固定数量的 token 或固定数量的对话次数窗口，而是使用 LLM 编写到目前为止历史对话的摘要，并将其保存

```python
OPENAI_API_KEY = os.environ['OPENAI_API_KEY'] #专属的API key"
llm = ChatOpenAI(temperature=0.0,openai_api_key=OPENAI_API_KEY)
summary_memory  = ConversationSummaryMemory(llm=llm)
conversation = ConversationChain(   #新建一个对话链（关于链后面会提到更多的细节）
    llm=llm, 
    memory = summary_memory ,
    verbose=False
    #查看Langchain实际上在做什么，设为FALSE的话只给出回答，看到不到下面绿色的内容
)

conversation.predict(input="Hi there! I am Sam")
conversation.predict(input="How are you today?")
conversation.predict(input="Can you help me with some customer support?")
print(conversation.memory.buffer)
"""
The human introduces themselves as Sam, and the AI introduces itself as AI. 
The AI then asks what brings Sam here today, and Sam asks how the AI is doing. 
The AI responds that it is doing great, and when Sam asks for customer support,
the AI responds positively and asks what kind of customer support Sam needs.
"""
```

## LangChain 评估方法

![[LangChain 评估方法#LangChain 评估的方法 Evaluation]]

## LangGraph

LangGraph 是 LangChain 的扩展，旨在通过将步骤建模为图中的边和节点，使用 LLM 构建健壮且有状态的多角色应用程序。

LangGraph 文档目前托管在一个单独的网站上。您可以在[此处阅读 LangGraph 教程](https://langchain-ai.github.io/langgraph/tutorials/)。

[langchain-ai / # 🦜🕸️LangGraph: Build resilient language agents as graphs. (github.com)](https://github.com/langchain-ai/langgraph)

LangGraph 原理解析

[LATS：比ToT和ReAct更强的大模型思维框架（LangGraph代码实现+拆解）_如何实现lats](https://blog.csdn.net/Attitude93/article/details/138491682)

## LangSmith

LangSmith 允许您密切跟踪、监控和评估 LLM 应用程序。它与 LangChain 无缝集成，您可以在构建过程中使用它来检查和调试链的各个步骤。

LangSmith 文档托管在一个单独的网站上。

[Get started with LangSmith | 🦜️🛠️ LangSmith (langchain.com)](https://docs.smith.langchain.com/)

---