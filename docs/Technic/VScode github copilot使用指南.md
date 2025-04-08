---
date created: 2025/4/8 10:23
date modified: 2025/4/8 10:27
---
## 名词解释

- **聊天类型**

copilot中的使用模式，对应不同应用场景。

- **聊天参与者**

聊天参与者就像专家一样，拥有他们可以帮助解决各种专业领域的问题，可以通过使用`@`符号提及他们在聊天中与他们交谈。

- **意图**

copilot回答问题，最为关键的一项就是确定使用者的意图，了解你想做什么。

可以使用自然语言来描述你的意图，但它可能很模糊。为了帮助解决这个问题，你可以使用“/”命令。

聊天参与者可以贡献斜杠命令，它们是该聊天参与者提供的特定常用功能的快捷方式。

- **上下文变量**

copilot 会根据使用者的自然语言聊天提示尝试确定意图和问题的范围。

为了帮助 Copilot 提供最佳和最相关的答案，我们应当在聊天提示中添加上下文。例如，附加特定文件，甚至添加工作区中的所有内容、当前编辑器选择内容等。

使用"#"来指定上下问

## 聊天类型

- **官方推荐的使用场景**

下图是官方给出的使用场景，我会在下面给出我自己实际使用时的使用场景

![](https://pic2.zhimg.com/v2-d2d64a0af61ec4b67a8ca8599498a4db_1440w.jpg)

- **内联聊天**

特点：单个文件编辑器内快速唤起，交互视图简介

适用范围：当前文件内的小型修改

快捷键：⌘I

![](https://pica.zhimg.com/v2-8ad6cd091bdf9bce70a56f7f897d790e_1440w.jpg)

- **聊天视图**

特点：集大成者，几乎什么功能都集合在聊天视图中，可以启用聊天参与者来进行专业对话

适用范围：项目整体结构上的问题（个人不常用）

快捷键：⌃⌘I

![](https://pic3.zhimg.com/v2-5f96d919d1e13d317c7c3a175d6be538_1440w.jpg)

- **编辑视图**

特点：属于copilot最新版本新加入的功能。它将 Copilot 聊天的对话流程和内联聊天的快速反馈结合在一个体验中，非常适合迭代跨多个文件的大型更改

适用场景：多文件修改

快捷键：⇧⌘I

![](https://pic1.zhimg.com/v2-a5a8449ca050c13cb925136a079b32a4_1440w.jpg)

- **快速聊天**

特点：顶部唤起，交互界面短小

使用场景：询问技术或者是工程相关的问题，适用于提问场景

快捷键：⇧⌥⌘L

![](https://pic3.zhimg.com/v2-e0dc3f1786e880565a5e46c046e1af54_1440w.jpg)

## 聊天参与者（@）

可在「**快速聊天」**以及「**聊天视图」**场景下唤起

- @workspace

具有有关工作区中代码的上下文，可以帮助您浏览它，找到相关文件或类

- @vscode

了解 VS Code 编辑器本身的命令和功能，可以帮助你使用它们

- @terminal

对集成终端 shell 及其内容有上下文信息

- @github

了解你的 GitHub 仓库、问题、拉取请求和主题，还可以使用 [Bing API](https://zhida.zhihu.com/search?content_id=250584326&content_type=Article&match_order=1&q=Bing+API&zhida_source=entity) 进行网络搜索

- 扩展程序还可以新增聊天者

![](https://pic4.zhimg.com/v2-18ac386c21105086f19cb789f7c79a7f_1440w.jpg)

## 意图（/）

意图可以理解为快捷键或者是宏，用于快速下达命令。配合**聊天参与者**使用（一般是隐形的，常见的**意图**不需要特殊指定），不同的聊天参与者可以提供不同的意图。

- **/clear**

开始一个新的聊天会话

- **/help**

获取有关使用 GitHub Copilot 的帮助

- **/doc**

生成代码文档（行内聊天）

- **/explain（@workspace /explain）**

解释所选代码的工作原理

- **/fix（@workspace /fix）**

为所选代码中的问题提出解决方案

- **@workspace /fixTestFailure（预览）**

为失败的测试提出解决方案

- **@workspace /setupTests**

为你的工作区配置测试框架

- **/tests（@workspace /tests）**

为所选代码生成单元测试

- **/new（@workspace /new）**

为新工作区或新文件搭建代码

- **/newNotebook（@workspace /newNotebook）**

创建一个新的 [Jupyter 笔记本](https://zhida.zhihu.com/search?content_id=250584326&content_type=Article&match_order=1&q=Jupyter+%E7%AC%94%E8%AE%B0%E6%9C%AC&zhida_source=entity)

- **@vscode /runCommand**

搜索或运行 VS Code 命令

- **/search（@vscode /search）**

为搜索视图生成查询参数

- **@vscode /startDebugging（实验性）**

生成 launch.json 文件以设置调试配置并启动调试

- **@terminal /explain**

解释终端功能或 shell 命令

## 上下文变量（#）

- **#codebase**

将整个工作区内容（整个项目）作为上下文添加到您的提示中

- **#editor**

将活动编辑器的可见内容作为上下文添加到您的提示中

- **#selection**

将当前编辑器内光标选中区域作为上下文添加到您的提示中

- **#terminalSelection**

将当前终端选择作为上下文添加到您的聊天提示中。

- **#terminalLastCommand**

将上次运行的终端命令作为上下文添加到您的聊天提示中。

- **#VSCodeAPI**

将 VS Code API 作为上下文添加到您的提示中，以提出与 VS Code 扩展开发相关的问题。

## 提升聊天性能

可以根据平时对话时的赞成或反对来反向激励，从而未来提供更好的答案

## 官方功能速查表

[VS Code 中的 GitHub Copilot 速查表](https://link.zhihu.com/?target=https%3A//vscode.js.cn/docs/copilot/copilot-vscode-features)

[github copilot使用指南 - 知乎](https://zhuanlan.zhihu.com/p/7808027849)
