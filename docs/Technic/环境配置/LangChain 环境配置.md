---
title: LangChain 环境配置
date created: 2024年7月31日,星期三,上午,11:15:59
date modified: 2024年8月8日,星期四,下午,2:28:30
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