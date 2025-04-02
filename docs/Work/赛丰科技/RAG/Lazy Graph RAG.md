---
date created: 2025/3/21 15:25
date modified: 2025/3/26 13:54
---

[GraphRAG:中文文档教程，助力大模型LLM应用开发从入门到精通](https://www.graphrag.club/)

[LazyGraphRAG：为质量和成本设定新标准 - Microsoft Research](https://www.microsoft.com/en-us/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/)

[GraphRAG：通过动态社区选择改进全局搜索 - Microsoft Research](https://www.microsoft.com/en-us/research/blog/graphrag-improving-global-search-via-dynamic-community-selection/)

## GraphRAG 有哪些限制？用户在使用系统时如何最大限度地减少 GraphRAG 限制的影响？

### GraphRAG 与传统向量 RAG 的主要区别

GraphRAG 相比传统向量 RAG（或“语义搜索”）的主要优势在于，它能够处理**全局查询**，即涉及整个数据集的问题，例如：

- 「数据中的主要主题是什么？」
- 「对 X 最重要的影响是什么？」

相比之下，**向量 RAG 更擅长局部查询**，即答案与查询相似，并存在于特定文本区域中的情况，例如：

- 「谁」、「什么」、「何时」和「在哪里」等具体问题。

### LazyGraphRAG 的成本与质量优势

LazyGraphRAG 是一种完全不同的支持图的 RAG 方法，该方法不需要对源数据进行预摘要，从而避免了**昂贵的预索引成本**，更适合对成本敏感的用户和用例。

LazyGraphRAG 的一个关键优势是它在成本和质量方面固有的可扩展性。在一系列竞争方法（标准载体 RAG、[猛禽](https://github.com/profintegra/raptor-rag)和 GraphRAG[当地](https://microsoft.github.io/graphrag/query/local_search/),[全球](https://microsoft.github.io/graphrag/query/global_search/)和[漂移](https://microsoft.github.io/graphrag/query/drift_search/)搜索机制），LazyGraphRAG 在成本质量范围内显示出强大的性能，如下所示：

- LazyGraphRAG 数据索引成本与矢量 RAG 相同，是完整 GraphRAG 成本的 0.1%。
- 对于与矢量 RAG 相当的查询成本，LazyGraphRAG 在本地查询上优于所有竞争方法，包括长上下文向量 RAG 和 GraphRAG [漂移](https://www.microsoft.com/en-us/research/blog/introducing-drift-search-combining-global-and-local-search-methods-to-improve-quality-and-efficiency/)搜索（我们最近推出的 RAG 方法显示性能优于矢量 RAG）以及 GraphRAG 本地搜索。
- 对于全局查询，相同的 LazyGraphRAG 配置也显示出与 GraphRAG Global Search 相当的答案质量，但_查询成本降低了 700 倍_以上。
- 对于 GraphRAG 全局搜索 4% 的查询成本，LazyGraphRAG 在本地和全局查询类型上_都明显优于所有竞争方法_，包括 C2 级别的 GraphRAG 全局搜索（为大多数应用程序推荐的社区层次结构的第三级）。

LazyGraphRAG 开源[GraphRAG 库](https://github.com/microsoft/graphrag)，为轻量级数据索引的本地和全局查询提供统一的查询接口，其成本与标准向量 RAG 相当。

## 将向量 RAG 和图形 RAG 与延迟 LLM 使用混合

LazyGraphRAG 旨在融合矢量 RAG 和图形 RAG 的优点，同时克服它们各自的局限性：

| 方法                | 优势                               | 局限性                 |
| ----------------- | -------------------------------- | ------------------- |
| **向量 RAG**        | 使用与查询最相似的文本块进行匹配（最佳优先搜索）         | 无法考虑数据集的广度，无法处理全局查询 |
| **GraphRAG 全局搜索** | 基于文本实体的社区结构，确保查询能考虑整个数据集（广度优先搜索） | 无法选择最佳社区进行局部查询      |

**LazyGraphRAG 的创新之处**在于：

- 将**最佳优先搜索**（Vector RAG）与**广度优先搜索**（GraphRAG）相结合，通过**迭代加深搜索**的方式动态进行查询。
- 相比于完整 GraphRAG 的全局搜索机制，这种“懒惰”的方法可以**延迟调用 LLM**，显著提升生成答案的效率。

**GraphRAG 与 LazyGraphRAG 对比总结**

| **功能模块** | **GraphRAG**                                                                | **LazyGraphRAG**                                                                                                                                                                                                  |
| -------- | --------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **索引构建** | - 使用 LLM 提取并描述实体及其关系。  <br>- 总结每个实体和关系的所有观察结果。  <br>- 使用图统计优化实体图并提取层次化社区结构。 | - 使用 NLP 名词短语提取技术识别概念及其共现关系。  <br>- 使用图统计优化概念图并提取层次化社区结构。  <br>_无需在索引构建阶段使用 LLM，显著降低前期成本。_                                                                                                                        |
| **索引摘要** | 使用 LLM 对每个社区的实体和关系进行摘要。                                                     | _无需摘要 —— 采用“懒惰”策略，推迟所有 LLM 使用至查询阶段，从而进一步降低前期成本。_                                                                                                                                                                  |
| **查询优化** | 无优化 —— 查询阶段始终使用原始查询。                                                        | - 使用 LLM 对查询进行动态优化：  <br>a) 将原始查询拆分为相关子查询，并将其合并为一个扩展查询。  <br>b) 使用概念图匹配相关概念，优化子查询。  <br> _通过动态优化提升本地查询准确性。_                                                                                                       |
| **查询匹配** | - 广度优先搜索：查询时遍历所有社区摘要进行匹配。                                                   | 对每个 **q 个子查询 [3-5 个]** 执行以下操作：  <br>- 使用文本块嵌入及块-社区关系进行相似度排名（最佳优先）。  <br>- 按前 k 个文本块的排名对社区排序。  <br>- 使用 LLM 进行句子级别的相关性评估，对排名靠前的未测试文本块进行广度优先匹配。  <br>- 在连续 z 个社区无相关文本块时递归进入相关子社区（迭代加深）。  <br>- 当无相关社区或达到相关性测试预算时终止。 |
| **答案映射** | - 使用 LLM 并行处理随机批次的社区摘要，生成答案。                                                | 对每个 **q 个子查询 [3-5 个]** 执行以下操作：  <br>- 从相关文本块构建概念子图。  <br>- 基于社区分配将相关文本块分组。  <br>- 使用 LLM 从相关文本块组中提取与子查询相关的陈述（仅聚焦于相关内容）。  <br>- 对提取的陈述进行排序和过滤，以适应预设上下文窗口大小。                                                        |
| **答案合并** | - 使用 LLM 对映射的答案进行整合，生成最终答案。                                                 | - 使用 LLM 基于映射阶段提取的陈述回答扩展查询。                                                                                                                                                                                       |

## LazyGraphRAG 评估

[LazyGraphRAG 答案质量](https://www.microsoft.com/en-us/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/#LazyGraphRAG%20answer%20quality%20is%20state-of-the-art)

## LazyGraphRAG 的优势与局限性分析

LazyGraphRAG 展示了在本地-全局查询范围内，一个灵活且通用的查询机制可以显著优于一系列专用的查询机制，同时无需承担 LLM 数据摘要带来的前期成本。其**极快且几乎零成本的索引过程**使 LazyGraphRAG 成为**一次性查询、探索性分析和流数据处理**的理想选择。同时，它能够在增加相关性测试预算时平滑地提升答案质量，这使其成为**评估 RAG 方法的有效基准工具**（例如：“在任务 Z 中，RAG 方法 X 在预算 Y 下优于 LazyGraphRAG”）。

### 是否意味着所有基于图的 RAG 都应采用 LazyGraphRAG？

我们认为答案是否定的，原因有三：

1. **GraphRAG 数据索引的附加价值：**
	- GraphRAG 中基于实体、关系和社区摘要构建的数据索引不仅能用于问答任务，还具备**报告生成与共享的价值**（例如，作为可读性强的报告供用户查看和分享）。

2. **GraphRAG + LazyGraphRAG 联合使用可能效果更佳：**
	- 将 GraphRAG 数据索引（实体、关系和社区摘要）与 LazyGraphRAG 类似的查询机制结合，可能比单独使用 LazyGraphRAG 能够获得**更好的查询效果**。

3. **新型 GraphRAG 数据索引可进一步增强 LazyGraphRAG 的性能：**
	- 设计一种专为支持 LazyGraphRAG 查询机制而优化的 GraphRAG 数据索引（例如，通过**预提取声明与主题**），有望实现**最佳性能**。

LazyGraphRAG 在查询效率和成本上表现优异，但 GraphRAG 的**丰富索引价值和增强版索引机制**仍然有助于实现更全面和高效的 RAG 系统。最佳实践可能是在两者之间取得平衡，以实现成本与性能的双赢。