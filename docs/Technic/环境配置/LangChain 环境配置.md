---
title: LangChain 环境配置
date created: 2024/7/31 11:15
date modified: 2024/8/12 14:40
---
## 创建新的虚拟环境

```python
conda activate -n LangChain python=3.10.14
```

## 安装所需软件包

```python
pip install --upgrade langchain langchain-community langchain-openai langgraph faiss-cpu

pip install PyJWT httpx-sse
```