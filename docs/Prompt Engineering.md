# Prompt Engineering

Prompt指用户的一系列指令和输入，是决定Language Model输出内容的唯一输入，主要用于帮助模型理解上下文，并生成相关和连贯的输出，如回答问题、拓写句子和总结问题。

## Principles of Prompting:

1. Write clear and specific instructions. (clear ≠ short)
- Tactic 1:
	- Use delimiters
	- Triple quotes: """
	- Triple backticks: \`\`\`
	- Triple dashes: ---
	- Angle brackets:< >
	- XML tags: \<tag>\</tag>
- Tactic 2: Ask for structured output
	- HTML, JSON
- Tactic 3:
	- Check whether conditions are satisfied;
	- Check assumptions required to do the task.
- Tactic 4:
	- Few-shot promptingGive successful examples of completing tasks
	- Then ask model to perform the task
2. Give the model time to think
- Tactic 1: Specify the steps to complete a task
	- Step 1: …
	- Step 2: …
	- Step N: …
- Tactic 2:
	- Instruct the model to work out itsown solution before rushing to a conclusion

## Model Limitations

Hallucination:
Makes statements that sound plausible but are not true

Reducing hallucinations:
First find relevant information, then answer the question based on the relevant information.

## Iterativa Prompt Development

Prompt guidelines
- Be clear and specific
- Analyze why result does not give desired output.
- Refine the idea and the prompt
- Repeat

Iterative Process
- Try something
- Analyze where the result does not give what you want
- Clarify instructions, give more time to think
- Refine prompts with a batch of examples

## 学习视频资料

[第1集 引言_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Z14y1Z7LJ?p=1&vd_source=40b0488ec89d5fe7f6accdb468d377dc)
[第2集 指南_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Z14y1Z7LJ?p=2&vd_source=40b0488ec89d5fe7f6accdb468d377dc)
[第3集 迭代_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Z14y1Z7LJ?p=3&vd_source=40b0488ec89d5fe7f6accdb468d377dc)

---

## 提示词编写原则：

1. 编写清晰和具体的指示。（清晰 ≠ 简短）
- 策略1：
  - 使用分隔符
  - 三引号："""
  - 三个反引号：```
  - 三个短划线：---
  - 尖括号：<>
  - XML标签：\<tag>\</tag>
- 策略2：要求结构化输出
  - HTML, JSON
- 策略3：
  - 检查条件是否满足；
  - 检查执行任务所需的假设。
- 策略4：
  - 提供成功完成任务的示例（Few-shot prompting）
  - 然后让模型执行任务
2. 给模型思考时间
- 策略1：指定完成任务的步骤
  - 第一步：…
  - 第二步：…
  - 第N步：…
- 策略2：
  - 指示模型在匆忙得出结论之前自己解决问题

## 模型局限性

幻觉：
做出听起来合理但不真实的陈述

减少幻觉：
首先找到相关信息，然后根据相关信息回答问题。

## 迭代提示开发

提示指南
- 清晰和具体
- 分析为什么结果没有给出预期的输出。
- 提炼想法和提示
- 重复

迭代过程
- 尝试一些方法
- 分析结果为什么没有达到预期
- 澄清指示，给予更多思考时间
- 用一批示例改进提示

---