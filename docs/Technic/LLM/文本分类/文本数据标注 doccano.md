---
date created: 2024/8/15 16:5
date modified: 2024/8/15 17:10
---
# Paddlenlp 之 UIE 分类模型【以情感倾向分析、新闻分类为例】

## 0.1 如何对数据进行标注---doccano

官方文档： https://github.com/doccano/doccano

**记的进虚拟环境！！！！！**

**Step 1. 本地安装 doccano（请勿在 AI Studio 内部运行，本地测试环境 python=3.8）**

```shell
$ pip install doccano
```

**Step 2. 初始化数据库和账户（用户名和密码可替换为自定义值）**

```shell
# 初始化，设置用户名= admin,密码=pass
$ doccano init
$ doccano createuser --username OliverZ11 --password pwdpwdpwd
```

**Step 3. 启动 doccano**

在一个窗口启动 doccano 的 WebServer，保持窗口

```shell
$ doccano webserver --port 8000
```

在另一个窗口启动 doccano 的任务队列

```shell
$ doccano task
```

浏览器访问 http://127.0.0.1:8000/

## 0.2 智能标注

当你数据样本很大的时候，一条条标注会很费时，效率很低这里推荐去**hugging face**加载一些预训练模型进行一次标注再进行人工复核。

[基于 hugging face 预训练模型的实体识别智能标注方案：生成doccano要求json格式](https://blog.csdn.net/sinat_39620217/article/details/125616709?spm=1001.2014.3001.5502)
