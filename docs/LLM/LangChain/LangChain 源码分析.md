# LangChain 源码分析

[chatmodels模块介绍](https://www.zhihu.com/zvideo/1661208710797172736)

## 各大语言模型继承类对比

各个LLM中类的差别用红框标注，其余base部分函数功能相同
![[../../01attachment/Pasted image 20240624104959.png|Pasted image 20240624104959.png]]

**ChatOpenAI**
![[../../01attachment/Pasted image 20240626095159.png|Pasted image 20240626095159.png]]

**azure_openai**:
![[../../01attachment/Pasted image 20240626095327.png|Pasted image 20240626095327.png]]

## 基类语言模型接口

![[../../01attachment/Pasted image 20240624105423.png|Pasted image 20240624105423.png]]
语言模型的接口或基类，`BaseLanguageModel`，它定义了一系列与语言模型交互的方法。下面是对每个方法功能的解释：

1. **generate_prompt(prompts: List[PromptValue], stop: Optional[List[str]], callbacks: Callbacks): LLMResult**
   - 功能：根据提供的提示（prompts）生成一个提示，并可选地定义停止符（stop）和回调（callbacks），以产生语言模型的结果（LLMResult）。
   - 参数：
	 - `prompts`: 提示列表，用于引导语言模型的生成。
	 - `stop`: 可选的停止符列表，指示生成何时停止。
	 - `callbacks`: 生成过程中调用的回调函数集。

2. **predict(text: str): str**
   - 功能：给定一段文本（text），使用语言模型预测并返回接下来的文本。
   - 参数：
	 - `text`: 要进行预测的文本。

3. **predict_messages(messages: List[BaseMessage]): BaseMessage**
   - 功能：给定一系列消息（messages），使用语言模型预测并返回接下来的消息。
   - 参数：
	 - [`messages`](command:_github.copilot.openSymbolFromReferences?%5B%7B%22%24mid%22%3A1%2C%22path%22%3A%22%2Ff%3A%2Fpractice%20code%2Fguasscode%2Flangchain-anal-main%2Flangchain%2Fschema%2Fmessages.py%22%2C%22scheme%22%3A%22file%22%7D%2C%7B%22line%22%3A0%2C%22character%22%3A0%7D%5D "langchain/schema/messages.py"): 要进行预测的消息列表，每个消息都是[`BaseMessage`](command:_github.copilot.openSymbolFromReferences?%5B%7B%22%24mid%22%3A1%2C%22path%22%3A%22%2Ff%3A%2Fpractice%20code%2Fguasscode%2Flangchain-anal-main%2Flangchain%2Fschema%2Fmessages.py%22%2C%22scheme%22%3A%22file%22%7D%2C%7B%22line%22%3A57%2C%22character%22%3A6%7D%5D "langchain/schema/messages.py")的实例。

4. **get_num_tokens(text: str): int**
   - 功能：计算给定文本（text）的令牌（token）数量。
   - 参数：
	 - `text`: 要计算令牌数量的文本。

5. **get_num_tokens_from_messages(messages: List[BaseMessage]): int**
   - 功能：计算给定消息列表（messages）中的总令牌数量。
   - 参数：
	 - [`messages`](command:_github.copilot.openSymbolFromReferences?%5B%7B%22%24mid%22%3A1%2C%22path%22%3A%22%2Ff%3A%2Fpractice%20code%2Fguasscode%2Flangchain-anal-main%2Flangchain%2Fschema%2Fmessages.py%22%2C%22scheme%22%3A%22file%22%7D%2C%7B%22line%22%3A0%2C%22character%22%3A0%7D%5D "langchain/schema/messages.py"): 要计算令牌数量的消息列表，每个消息都是[`BaseMessage`](command:_github.copilot.openSymbolFromReferences?%5B%7B%22%24mid%22%3A1%2C%22path%22%3A%22%2Ff%3A%2Fpractice%20code%2Fguasscode%2Flangchain-anal-main%2Flangchain%2Fschema%2Fmessages.py%22%2C%22scheme%22%3A%22file%22%7D%2C%7B%22line%22%3A57%2C%22character%22%3A6%7D%5D "langchain/schema/messages.py")的实例。

6. **get_token_ids(text: str): List[int]**
   - 功能：将给定文本（text）转换为令牌ID的列表。
   - 参数：
	 - `text`: 要转换为令牌ID的文本。

这些方法提供了与语言模型进行交互的基础，包括生成文本、预测接下来的文本或消息、计算令牌数量以及将文本转换为令牌ID等功能。

`class ChatOpenAI(BaseChatModel):`

对象实例()，-> __call()___.

总结:
1. 实例化chatpenAI对象，使用@root_validator()功能，调用validate_environment函数初始化: 获取参数配置，实例化openai接口对象
2. 对象实例()，-> __call()___，作为执行入口
3. 对用户的输入进行数据格式的封装
4. 重试调用chat model接口，获取结果(缓存获取中间结果和历史数据).
5. 对结果进行解析和封装
6. 结果返回

---

# [promptTemplate模块源码剖析](https://www.zhihu.com/zvideo/1663740505211957248)

Chat
![[../../01attachment/Pasted image 20240626104949.png|Pasted image 20240626104949.png]]

