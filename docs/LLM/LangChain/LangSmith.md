# LangSmith：评估 LLM 应用能力的工具

> [如何使用 LangChain 和 LangSmith 优化链-CSDN博客](https://blog.csdn.net/wangjiansui/article/details/140329613)
> [LangSmith 教程 | Langchain v0.1](https://langchain114.com/docs/langsmith/walkthrough/)
> [Evaluation | 🦜️🛠️ LangSmith ~ 评估 | 🦜️🛠️ LangSmith (langchain.com)](https://docs.smith.langchain.com/concepts/evaluation#agents)

- [构建 LangChain 和 LangSmith 优化链的示例代码](LangSmith.md#构建%20LangChain%20和%20LangSmith%20优化链的示例代码)
	- [评估 LLM 应用](LangSmith.md#构建%20LangChain%20和%20LangSmith%20优化链的示例代码)
	- [创建数据集](LangSmith.md#构建%20LangChain%20和%20LangSmith%20优化链的示例代码)
	- [定义衡量标准](LangSmith.md#构建%20LangChain%20和%20LangSmith%20优化链的示例代码)
	- [运行评估](LangSmith.md#构建%20LangChain%20和%20LangSmith%20优化链的示例代码)
- [评估 SQLAgent based LangGraph](LangSmith.md#评估%20SQLAgent%20based%20LangGraph)
	- [评估 Agent 最终响应](LangSmith.md#评估%20SQLAgent%20based%20LangGraph)
		- [创建数据集](LangSmith.md#评估%20Agent%20最终响应)
		- [定义衡量标准](LangSmith.md#评估%20Agent%20最终响应)
		- [运行评估](LangSmith.md#评估%20Agent%20最终响应)
	- [评估 Agent 的单个步骤](LangSmith.md#评估%20SQLAgent%20based%20LangGraph)
	- [评估一个 Agent 的轨迹](LangSmith.md#评估%20SQLAgent%20based%20LangGraph)

Langsmith 是一种记录和评估通过 LangChain 构建的 LLM 应用的工具。它可以帮助我们更好地调整提示词等中间过程，从而优化应用效果。

## 构建 LangChain 和 LangSmith 优化链的示例代码

以下是使用 LangChain 和 LangSmith 优化链的示例代码：

### 评估 LLM 应用

根据您或用户的重要标准来衡量应用程序的性能可能非常困难，但这是至关重要的。通过评估，您可以在迭代应用程序时使用一组固定的数据来衡量其性能。快速、可靠地获得这些洞察力，可以让您更加自信地进行改进。以下是一些评估步骤：

1. **创建用于衡量性能的初始黄金数据集**
2. **定义用于衡量性能的指标**
3. **对一些不同的提示或模型进行评估**
4. **手动比较结果**
5. **随时间跟踪结果**
6. **设置自动测试，以便在 CI/CD 中运行**

### 创建数据集

第一步是定义要评估的数据点：

- **模式**：每个数据点至少应包括应用程序的输入。如果可能，定义预期输出也非常有帮助。这代表您期望正常运行的应用程序输出内容。
- **数量**：没有硬性规定，但要确保适当覆盖可能需要防范的边缘案例。即使 10-50 个示例也能提供很大价值。
- **获取方法**：可以从手工收集最初的 10-20 个数据点开始，这些数据集一般会随着时间的推移而增长。

```python
from langsmith import Client

# 创建一个与LangSmith交互的客户端对象
client = Client()

# 定义测试用例数据集的名称
dataset_name = "QA Example Dataset"

try:
    # 使用指定的名称创建一个新的数据集
    dataset = client.create_dataset(dataset_name)
    
    # 为数据集创建示例
    client.create_examples(
        inputs=[
            {"question": "什么是LangChain？"},
            {"question": "什么是LangSmith？"},
            {"question": "什么是OpenAI？"},
            {"question": "什么是Google？"},
            {"question": "什么是Mistral？"},
        ],
        outputs=[
            {"answer": "一个用于构建LLM应用程序的框架"},
            {"answer": "一个用于观察和评估LLM应用程序的平台"},
            {"answer": "一个创建大型语言模型的公司"},
            {"answer": "一家以搜索闻名的技术公司"},
            {"answer": "一个创建大型语言模型的公司"},
        ],
        dataset_id=dataset.id,
    )
except Exception as e:
    print(e)

```

创建后，可以在 LangSmith UI 中找到并查看 QA 示例数据集中的五个新示例。
![QA Example Dataset]()

### 定义衡量标准

创建数据集后，可以定义一些指标来评估回答：

1. **正确性评估**：使用 LLM 来判断输出是否正确（相对于预期输出）。
2. **简洁度评估**：定义一个简单的 Python 函数来测量答案的长度。

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts.prompt import PromptTemplate
from langsmith.evaluation import LangChainStringEvaluator

# 创建一个ChatOpenAI对象，用于与OpenAI模型交互
eval_llm = ChatOpenAI(
    model_name="glm-4",
    openai_api_base="https://open.bigmodel.cn/api/paas/v4",
)

# 创建一个PromptTemplate对象，用于生成评分模板
_PROMPT_TEMPLATE = """You are an expert professor specialized in grading students' answers to questions.
You are grading the following question:
{query}
Here is the real answer:
{answer}
You are grading the following predicted answer:
{result}
Respond with CORRECT or INCORRECT:
Grade:
"""
PROMPT = PromptTemplate(
    input_variables=["query", "answer", "result"], template=_PROMPT_TEMPLATE
)

# 创建一个LangChainStringEvaluator对象，用于评估字符串
qa_evaluator = LangChainStringEvaluator("qa", config={"llm": eval_llm, "prompt": PROMPT})

```

对于响应长度的评估：

```python
from langsmith.schemas import Run, Example

def evaluate_length(run: Run, example: Example) -> dict:
    # 计算预测结果的长度
    prediction = run.outputs.get("output") or ""
    # 获取参考答案的长度
    required = example.outputs.get("answer") or ""
    # 判断预测结果的长度是否小于参考答案的两倍
    score = int(len(prediction) < 2 * len(required))
    # 返回评分结果
    return {"key":"length", "score": score}
```

### 运行评估

接下来，定义应用程序并运行评估：

```python
from langchain_core.messages import HumanMessage,SystemMessage

def my_app(question):
    # 创建消息列表，包括系统消息和用户消息
    message = [
        SystemMessage("Respond to the users question in a short, concise manner (one short sentence)."),  # 系统消息
        HumanMessage(question),  # 用户消息
    ]
    # 调用eval_llm对象的invoke方法，传入消息列表，获取响应结果
    response = eval_llm.invoke(message)
    # 返回响应结果的内容
    return response.content
```

定义封装器，将数据集的输入键映射到我们要调用的函数，然后将函数的输出映射到我们期望的输出键：

```python
def langsmith_app(inputs):
    # 调用my_app函数，传入问题参数，获取输出结果
    output = my_app(inputs["question"])
    # 返回包含输出结果的字典
    return {"output": output}
```

进行评估：

```python
from langsmith.evaluation import evaluate

# 使用evaluate函数对AI系统进行评估
# langsmith_app是你的AI系统
# data参数指定要预测和评分的数据集名称
# evaluators参数指定用于评分结果的评估器列表
# experiment_prefix参数是实验名称的前缀，用于方便识别
experiment_results = evaluate(
    langsmith_app,  # 你的AI系统
    data=dataset_name,  # 要预测和评分的数据集
    evaluators=[evaluate_length, qa_evaluator],  # 评估器列表
    experiment_prefix="GLM-4",  # 实验名称前缀
)
```

在工作台查看评估结果：

![Pasted image 20240718145736.png](../../01attachment/Pasted%20image%2020240718145736.png)

## 评估 SQLAgent based LangGraph

LangSmith 可以分别对 Agent 的最终响应、单个步骤和轨迹进行评估。

1. **评估 Agent 最终响应**：输入是用户输入和（可选）工具列表，输出是 Agent 的最终响应。
2. **评估 Agent 单个步骤**：输入是单个步骤的输入，输出是该步骤的输出，通常是 LLM 响应。
3. **评估 Agent 轨迹**：输入是整个 Agent 的输入，输出是工具调用列表，可以是“精确”的轨迹或预期的工具调用列表。

这种评估方式有助于更好地理解 Agent 的行为并进行相应的优化。

### 评估 Agent 最终响应

我们可以评估代理在某项任务中的整体表现。这基本上就是把代理当作一个黑盒子，评估它是否完成了任务。

#### 创建数据集

首先，创建一个数据集来评估代理的端到端性能。我们可以从这里提取一些与 Chinook 数据库相关的问题。

```python
from langsmith import Client

# 创建一个Langsmith客户端实例
client = Client()

# 创建一个数据集
examples = [
    ("Which country's customers spent the most? And how much did they spend?", "The country whose customers spent the most is the USA, with a total expenditure of $523.06"),
    ("What was the most purchased track of 2013?", "The most purchased track of 2013 was Hot Girl."),
    ("How many albums does the artist Led Zeppelin have?","Led Zeppelin has 14 albums"),
    ("What is the total price for the album “Big Ones”?","The total price for the album 'Big Ones' is 14.85"),
    ("Which sales agent made the most in sales in 2009?", "Steve Johnson made the most sales in 2009"),
]

# 指定数据集名称
dataset_name = "SQL Agent Response"

# 如果数据集不存在，则创建数据集
if not client.has_dataset(dataset_name=dataset_name):
    dataset = client.create_dataset(dataset_name=dataset_name)
    inputs, outputs = zip(
        *[({"input": text}, {"output": label}) for text, label in examples]
    )
    client.create_examples(inputs=inputs, outputs=outputs, dataset_id=dataset.id)
```

#### 定义衡量标准

将生成的答案与参考答案进行比较。

```python
from langchain import hub
from langchain_openai import ChatOpenAI

# 导入评估提示词
grade_prompt_answer_accuracy = prompt = hub.pull("langchain-ai/rag-answer-vs-reference")

def answer_evaluator(run, example) -> dict:
    """
    用于评估SQLAgent答案准确性的简单评估器
    """

    # 获取问题、真实答案和SQLAgent的答案
    input_question = example.inputs["input"]
    reference = example.outputs["output"]
    prediction = run.outputs["response"]
    
    # LLM评分器
    llm = ChatOpenAI(
        model_name="glm-4",
        openai_api_base="https://open.bigmodel.cn/api/paas/v4",
        temperature=0
    )

    # 结构化提示
    answer_grader = grade_prompt_answer_accuracy | llm

    # 运行评估器
    score = answer_grader.invoke({"question": input_question,
                                  "correct_answer": reference,
                                  "student_answer": prediction})
    print(score)
    try:
        score = score["Score"]
    except:
        score = 0

    return {"key": "answer_v_reference_score", "score": score}
```

#### 运行评估

定义运行 SQLAgent 的函数，此处省略的 SQLAgent 的创建步骤。

```python
import uuid

def predict_sql_agent_answer(example: dict):
    """用于答案评估的函数"""
    # 为每个线程生成一个唯一的ID
    thread_id = str(uuid.uuid4())
    config = {
    "configurable": {
        # 检查点通过线程ID访问
        "thread_id": thread_id,
        }
    }
    msg = {"messages": ("user", example["input"])}
    # 调用代理执行消息
    messages = agent_executor.invoke(msg, config)
    # 打印代理的最后一条消息内容
    print(messages['messages'][-1].content)
    return {"response": messages['messages'][-1].content}
```

创建评估

```python
from langsmith.evaluation import evaluate

# 使用evaluate函数对代理进行评估
# 参数说明：
# predict_sql_agent_answer: 预测SQL代理答案的函数
# data: 数据集名称
# evaluators: 评估器列表，用于评估答案准确性
# experiment_prefix: 实验前缀，用于标识实验
# num_repetitions: 重复次数
# metadata: 元数据，包含版本信息等

experiment_results = evaluate(
    predict_sql_agent_answer,
    data=dataset_name,
    evaluators=[answer_evaluator],
    experiment_prefix=experiment_prefix + "-response-v-reference",
    num_repetitions=1,
    metadata={"version": metadata},
)
```

在工作台查看评估结果：
![Pasted image 20240718161101.png](../../01attachment/Pasted%20image%2020240718161101.png)

### 评估 Agent 的单个步骤

[Evaluate an agent | 🦜️🛠️ LangSmith (langchain.com)](https://docs.smith.langchain.com/tutorials/Developers/agents#single-step-evaluation)

代理通常会执行多个操作。虽然对它们进行端到端评估很有用，但对这些单个操作进行评估也很有用。这通常涉及评估代理的一个步骤，即代理决定要做什么的 LLM 调用。

我们可以使用自定义评估器检查特定的工具调用：

输入应该是单个步骤的输入。根据测试内容的不同，这可能只是原始的用户输入（如提示和/或一组工具），也可能包括先前完成的步骤。

输出只是该步骤的输出，通常是 LLM 响应。LLM 响应通常包含工具调用，说明代理下一步应该采取什么行动。

其评估器通常是一些二进制分数，用于判断是否选择了正确的工具调用，以及一些启发式方法，用于判断对工具的输入是否正确。参考工具可以简单地指定为一个字符串。

这种评估方式有几个好处。它允许你对单个操作进行评估，从而找出应用程序可能失败的地方。运行速度也相对较快（因为只需调用一次 LLM），而且评估通常使用所选工具相对于参考工具的简单 heuristc 评估。它们的一个缺点是无法捕捉到整个代理过程，只能捕捉到一个特定的步骤。另一个缺点是数据集的创建具有挑战性，尤其是如果您想在代理输入中包含过去的历史记录。要为代理轨迹的早期步骤（例如，可能只包括输入提示）生成数据集非常容易，但要为轨迹的后期步骤（例如，包括代理之前的许多操作和响应）生成数据集却很困难。

### 评估一个 Agent 的轨迹

[Evaluate an agent | 🦜️🛠️ LangSmith (langchain.com)](https://docs.smith.langchain.com/tutorials/Developers/agents#trajectory)

评估一个代理的轨迹包括查看代理所采取的所有步骤，并对步骤序列进行评估。

输入同样是整个代理的输入（用户输入和可选的工具列表）。

输出则是工具调用列表，可以是“精确”的轨迹（如预期的工具调用序列），也可以是预期的工具调用列表（顺序不限）。

这里的评估者是所采取步骤的某个函数。评估“精确”轨迹可以使用单个二进制分数，确认序列中每个工具名称的精确匹配。这种方法很简单，但也存在一些缺陷。有时可能存在多个正确路径。这种评估方法也无法捕捉到轨迹偏差一步与完全错误之间的区别。

为了解决这些缺陷，评价指标可以集中在“错误”步骤的数量上，这样就能更好地说明轨迹接近与偏离较大的区别。评价指标还可以侧重于是否以任何顺序调用了所有预期工具。

然而，这些方法都无法评估工具的输入；它们只关注所选择的工具。为了考虑到这一点，另一种评估技术是将完整的代理轨迹（连同参考轨迹）作为一组信息（例如所有 LLM 响应和工具调用）传递给 LLM 即判断。这可以评估代理的完整行为，但它是最难编译的参考（幸运的是，使用 LangGraph 这样的框架可以帮助解决这个问题！）。另一个缺点是，评估指标的提出可能有些棘手。
