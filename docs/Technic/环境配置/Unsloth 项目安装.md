---
date created: 2025/3/14 14:31
date modified: 2025/4/2 9:37
---

[Unsloth 项目安装和配置指南-CSDN博客](https://blog.csdn.net/gitblog_07703/article/details/142222375)

```shell
conda create --name unsloth_env \
    python=3.11 \
    pytorch-cuda=12.1 \
    pytorch cudatoolkit xformers -c pytorch -c nvidia -c xformers \
    -y
conda activate unsloth_env
pip install unsloth
```

创建进程

```shell
Screen -S rl
```

Ctrl+A+D 退出该进程

在命令面板输入 `exit` 关闭该进程

进入该进程

```shell
Screen -r
```

1. 训练医学推理大语言模型。
2. 制作 Deepseek-r 1-distill 的数据集，包含 reasoning 用于蒸馏模型的微调。
