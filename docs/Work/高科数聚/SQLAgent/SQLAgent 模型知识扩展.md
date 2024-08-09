---
title: SQLAgent 模型知识扩展
tags:
  - in-progress
date created: 2024/8/5 10:19
date modified: 2024/8/9 18:22
---
## [[Supersonic]]

**SuperSonic** 是一个基于大语言模型 (LLM) 的系统，旨在通过配套的工程化组件提升语义解析和 SQL 生成的可靠性和效率。其实现原理和工作流程如下：

1. **Semantic Layer**: 充当“翻译官”和“大管家”的角色，将技术名词翻译成业务术语，并统一化、精细化地管理技术口径。这一层增强了业务用户和 LLM 对数据的理解，简化了 LLM 的 SQL 生成难度。
2. **Schema Mapper**: 通过映射机制，将用户输入文本与语义模型中的字段名称、别名等进行匹配，减少不必要的 token 使用，提升推理效率。Schema Mapper 使用 NLP 框架，如 HanLP，来构建词典和进行分词匹配。
3. **Semantic Corrector**: 对 LLM 输出的 SQL 进行合法性检查和修正，避免错误或幻觉的生成。它通过类似 schema mapping 的方式，尝试找到正确的字段映射，并对 SQL 进行改写。
4. **Rule-based Parser**: 提供基于规则的语义解析方法，对于简单的查询问题，可以直接跳过 LLM，使用预定义的规则解析，从而提升查询效率。这种方法灵感来自 Google Analyza，利用 slot filling 思路映射查询意图。
5. **Chat Plugin**: 允许三方插件以 WebPage 或 WebService 方式注册，兼容其他场景如文档知识库或数据看板。SuperSonic 通过向量召回或函数调用方式选择合适的插件工具，扩展其应用场景。

### **工作流程**:

1. 用户输入问题后，首先由 **Schema Mapper** 进行字段匹配，生成候选的 schema 映射。
2. **Rule-based Parser** 根据规则进行初步解析，若规则匹配度高，直接生成查询意图；否则交由 LLM 解析。
3. LLM 生成逻辑 SQL 后，由 **Semantic Corrector** 检查并修正可能的错误。
4. 最终 SQL 被提交至 **Semantic Layer**，转换为物理 SQL 以执行数据查询。
5. 如果涉及多场景查询，**Chat Plugin** 会根据向量召回或函数调用选择适合的工具或插件。

这个系统通过上述工程化组件的协同工作，显著增强了语义解析的精确性、SQL 生成的可靠性以及整体查询的效率。

## 阿里云实验方式

[Copilot生成SQL_数据管理(DMS)-阿里云帮助中心 (aliyun.com)](https://help.aliyun.com/zh/dms/use-copilot-to-generate-sql-statements)

### **管理 SQL 知识**

在使用 DMS Data Copilot 时，为避免 AI 出错，DMS 引入了**知识库**和**相似查询**机制。如下图所示：
![](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/4106989071/p776687.png)

#### **管理业务知识**

- **生成业务知识**

	知识来源目前有两个：

	- DMS 根据库、表、列元数据和用户的查询历史主动挖掘积累的知识。
	- 用户在与 Copilot 进行交互的过程中，DMS 可向知识库补充用户反馈的信息。

	生成业务知识后，在 Copilot 生成 SQL 时会引用相关的业务知识，并标注出对业务知识的引用。

![](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/4106989071/p776710.png)

- **验证业务知识**

	在**表详情页**的**业务知识**页签下，您可审核生成的业务知识是否正确，如果正确，您可以将**待审核**或**待验证**的**知识等级**调整为**已验证**。

- **调整业务知识**

	如果业务知识不正确，您可在 SQL 引用的知识库区域，单击**编辑**。
	![](https://help-static-aliyun-doc.aliyuncs.com/assets/img/zh-CN/4106989071/p776345.png)

#### **管理相似 SQL**

Copilot 可以使用历史问题作为参考生成 SQL；Agent 只会使用已验证的知识。

- **生成相似的问题**

	如果您对 Copilot 生成的 SQL 很满意，可以在右下角点赞，点赞后即可保存本次查询记录。后续如果提问类似的问题，Copilot 会参考相似问题生成 SQL。

- **删除已保存的相似问题**

	双击表名称，进入**表详情**页面，在历史问题页签下删除问题。

## **常见问题**

- Q：在对 Copilot 生成的 SQL 较为满意的情况下，为什么需要给 SQL 点赞？
- A：点赞操作可以触发 Copilot 保存 SQL。后续提出相似的问题，能够大大提高回复的准确率。
- Q：当 Copilot 生成的 SQL 与提问不符时，为什么需要补充用户反馈？
- A：补充用户反馈可以提高 Copilot 回复问题的准确率，后续提出的相似问题，基本不会出错。

## Agent with RAG

- RAG 可以作为 LangChain（如果它是一个语言处理工具）的一部分，用于提供更加丰富和准确的语言生成能力。
- Agent 可以使用 LangChain 来处理自然语言的任务，比如理解用户输入和生成响应。
- 同时，Agent 也可以利用 RAG 技术来提高其在特定任务（如问答或对话系统）中的性能，尤其是在需要外部知识来支持决策时。

### 向量数据库的崛起

[向量数据库｜一文全面了解向量数据库的基本概念、原理、算法、选型-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/2312534)

在 GPT 模型的限制下，开发者们不得不寻找其他的解决方案，而向量数据库就是其中之一。向量数据库的核心思想是将文本转换成向量，然后将向量存储在数据库中，当用户输入问题时，将问题转换成向量，然后在数据库中搜索最相似的向量和上下文，最后将文本返回给用户。

当我们有一份文档需要 GPT 处理时，例如这份文档是客服培训资料或者操作手册，我们可以先将这份文档的所有内容转化成向量（这个过程称之为 Vector Embedding），然后当用户提出相关问题时，我们将用户的搜索内容转换成向量，然后在数据库中搜索最相似的向量，匹配最相似的几个上下文，最后将上下文返回给 GPT。这样不仅可以大大减少 GPT 的计算量，从而提高响应速度，更重要的是降低成本，并绕过 GPT 的 tokens 限制。

## Enhancing SQL Agents with Retrieval Augmented Generation (RAG)

[Enhancing SQL Agents with Retrieval Augmented Generation (RAG) | by Luc Nguyen | Medium](https://medium.com/@lucnguyen_61589/enhancing-sql-agents-with-retrieval-augmented-generation-rag-e20dbd8bb685)

![[RAG 增强 SQLAgent]]