# LangChain 评估的方法 Evaluation

这部分文档涵盖了我们在 LangChain 中对评估的处理方式和思考方式，包括对内部链式操作/代理的评估，以及我们建议构建在 LangChain 之上的人如何进行评估的方法。

## 问题

评估 LangChain 链式操作和代理可能非常困难。这主要有两个原因：

1. 缺乏数据

	在开始项目之前，通常很难获得用于评估链式操作和代理的大量数据。这通常是因为大型语言模型（大多数链式操作和代理的核心）具有出色的小样本和零样本学习能力，这意味着您几乎总是能够在特定任务（如文本到 SQL、问答等）上开始，而无需大量的示例数据集。这与传统的机器学习形成鲜明对比，传统机器学习在使用模型之前必须先收集大量数据点。

2. 缺乏评估指标

	大多数链式操作和代理执行的任务往往没有很好的评估指标来评估性能。例如，生成某种形式的文本比评估分类预测或数值预测要复杂得多。

## 解决方案

LangChain 试图解决这两个问题。目前提供的解决方案是初步的尝试，并不是完美的解决方案。

目前针对每个问题的解决方案如下：

1. 缺乏数据
	已经创建了 LangChainDatasets，这是 Hugging Face 上的一个社区空间。该社区空间是一个用于评估常见链式操作和代理的开源数据集的集合。我们已经贡献了五个自己的数据集作为起点，但我们非常希望这成为一个社区的努力。要贡献数据集，您只需要加入社区，然后您就可以上传数据集。

	我们还致力于尽可能简化人们创建自己数据集的过程。作为初步尝试，我们添加了一个名为 QAGenerationChain 的链式操作，它可以根据文档生成问题-答案对，用于以后评估问题回答任务。

2. 缺乏评估指标

	针对缺乏评估指标，我们有两个解决方案。

	第一个解决方案是不使用评估指标，而是依靠目测结果来对链式操作和代理的性能有所了解。为了辅助这一点，我们开发了（并将继续开发）Tracing，这是一个基于 UI 的可视化工具，可以查看链式操作和代理运行的结果。

	第二个解决方案是使用语言模型本身来评估输出结果。为此，我们提供了一些针对此问题的链式操作和提示。

请注意，这些解决方案仍然在不断发展和改进中。我们欢迎用户提供反馈和建议，以帮助我们改进 LangChain 的评估功能。我们希望通过不断的改进和开放性的合作，为 LangChain 用户提供更好的评估工具和方法。

---

## LangChain 0.2.5 evaluation 方案

[LangChainDatasets (LangChainDatasets) (huggingface.co)](https://huggingface.co/LangChainDatasets)

[langchain 0.2.5 — 🦜🔗 LangChain 0.2.5](https://api.python.langchain.com/en/latest/langchain_api_reference.html#module-langchain.evaluation)

[text_to_sql · Datasets at Hugging Face](https://huggingface.co/datasets/gretelai/synthetic_text_to_sql?row=0)

---

## Supersonic LangChain evaluation 方案

[supersonic evaluation at master · (github.com)](https://github.com/tencentmusic/supersonic/tree/master/evaluation)

### 评测流程

1. 正常启动项目 (必须包括 LLM 服务)
2. 执行 evalution. sh 脚本，主要包括构建表数据、数据建模、获取模型预测结果，执行对比逻辑。可以在命令行看到执行准确率，错误 case 会写到同目录的 error_case. json 文件中。

### 评测意义

制定评测工具方便 supersonic 快速对接其他大模型、更改参数配置，对于评估提示词、代码更改所带来的影响至关重要，可以帮助我们了解这些变化是否会提高或降低准确率、响应速度。

### 评估结果

![[./docs/01attachment/Pasted image 20240624164720.png|Pasted image 20240624164720.png]]

![[./docs/01attachment/Pasted image 20240624171839.png|Pasted image 20240624171839.png]]

[模型效果测试，返回为空 · Issue #1205 · tencentmusic/supersonic (github.com)](https://github.com/tencentmusic/supersonic/issues/1205)

---

## LangSmith

### 评估 Agent

LangSmith 可以分别对 Agent 的最终响应、单个步骤和轨迹进行评估。

1. **评估最终响应**：输入是用户输入和（可选）工具列表，输出是 Agent 的最终响应。
2. **评估单个步骤**：输入是单个步骤的输入，输出是该步骤的输出，通常是 LLM 响应。
3. **评估轨迹**：输入是整个 Agent 的输入，输出是工具调用列表，可以是“精确”的轨迹或预期的工具调用列表。

这种评估方式有助于更好地理解 Agent 的行为并进行相应的优化。

### Evaluating an agent's final response 评估 Agent 的最终响应

[Evaluate an agent | 🦜️🛠️ LangSmith (langchain.com)](https://docs.smith.langchain.com/tutorials/Developers/agents#response-evaluation)

评估代理的一种方法是评估它在某项任务中的整体表现。这基本上就是把代理当作一个黑盒子，简单地评估它是否完成了任务。

输入应该是用户输入和（可选）工具列表。在某些情况下，工具被硬编码为代理的一部分，因此无需输入。在其他情况下，代理比较通用，这意味着它没有固定的工具集，因此需要在运行时输入工具。

输出应该是代理的最终响应。

评价器因要求代理执行的任务而异。许多代理会执行一系列相对复杂的步骤，并输出最终的文本响应。与 RAG 类似，LLM-as-judge 评价器在这些情况下通常也能有效地进行评价，因为它们可以直接从文本回复中评估代理是否完成了任务。

不过，这种评估方式有几个缺点。首先，它通常需要运行一段时间。其次，你并没有对代理内部发生的任何事情进行评估，因此在发生故障时很难进行调试。第三，有时很难定义适当的评估指标。

### Evaluating a single step of an agent 评估 Agent 的单个步骤

[Evaluate an agent | 🦜️🛠️ LangSmith (langchain.com)](https://docs.smith.langchain.com/tutorials/Developers/agents#single-step-evaluation)

代理通常会执行多个操作。虽然对它们进行端到端评估很有用，但对这些单个操作进行评估也很有用。这通常涉及评估代理的一个步骤，即代理决定要做什么的 LLM 调用。

输入应该是单个步骤的输入。根据测试内容的不同，这可能只是原始的用户输入（如提示和/或一组工具），也可能包括先前完成的步骤。

输出只是该步骤的输出，通常是 LLM 响应。LLM 响应通常包含工具调用，说明代理下一步应该采取什么行动。

其评估器通常是一些二进制分数，用于判断是否选择了正确的工具调用，以及一些启发式方法，用于判断对工具的输入是否正确。参考工具可以简单地指定为一个字符串。

这种评估方式有几个好处。它允许你对单个操作进行评估，从而找出应用程序可能失败的地方。运行速度也相对较快（因为只需调用一次 LLM），而且评估通常使用所选工具相对于参考工具的简单 heuristc 评估。它们的一个缺点是无法捕捉到整个代理过程，只能捕捉到一个特定的步骤。另一个缺点是数据集的创建具有挑战性，尤其是如果您想在代理输入中包含过去的历史记录。要为代理轨迹的早期步骤（例如，可能只包括输入提示）生成数据集非常容易，但要为轨迹的后期步骤（例如，包括代理之前的许多操作和响应）生成数据集却很困难。

### Evaluating an agent's trajectory 评估一个 Agent 的轨迹

[Evaluate an agent | 🦜️🛠️ LangSmith (langchain.com)](https://docs.smith.langchain.com/tutorials/Developers/agents#trajectory)

评估一个代理的轨迹包括查看代理所采取的所有步骤，并对步骤序列进行评估。

输入同样是整个代理的输入（用户输入和可选的工具列表）。

输出则是工具调用列表，可以是“精确”的轨迹（如预期的工具调用序列），也可以是预期的工具调用列表（顺序不限）。

这里的评估者是所采取步骤的某个函数。评估“精确”轨迹可以使用单个二进制分数，确认序列中每个工具名称的精确匹配。这种方法很简单，但也存在一些缺陷。有时可能存在多个正确路径。这种评估方法也无法捕捉到轨迹偏差一步与完全错误之间的区别。

为了解决这些缺陷，评价指标可以集中在“错误”步骤的数量上，这样就能更好地说明轨迹接近与偏离较大的区别。评价指标还可以侧重于是否以任何顺序调用了所有预期工具。

然而，这些方法都无法评估工具的输入；它们只关注所选择的工具。为了考虑到这一点，另一种评估技术是将完整的代理轨迹（连同参考轨迹）作为一组信息（例如所有 LLM 响应和工具调用）传递给 LLM 即判断。这可以评估代理的完整行为，但它是最难编译的参考（幸运的是，使用 LangGraph 这样的框架可以帮助解决这个问题！）。另一个缺点是，评估指标的提出可能有些棘手。
