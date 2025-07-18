---
date created: 2025/3/14 10:49
date modified: 2025/3/14 11:11
---

[P2！自有文档提取+拆分！部署中文模型批量化提取+段落拆分！html+pdf+jpg等全格式支持！40/45](https://mp.weixin.qq.com/s?__biz=MzU4NDk0MDAwMw==&mid=2247485870&idx=1&sn=810151ea6f79a7b7d9e712c68bde1fe9&scene=21#wechat_redirect)

2025开年，[DeepSeek-R1](https://zhida.zhihu.com/search?content_id=254724810&content_type=Article&match_order=1&q=DeepSeek-R1&zhida_source=entity)震惊中外，至今还风高火猛

671B满血版至少百万硬件，方可推理！

无硬件条件的，只能用低参数的蒸馏版！

那，他是如何蒸馏到Qwen、Llama上？

一意在【[2025年度，会员价值交付计划](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s%3F__biz%3DMzU4NDk0MDAwMw%3D%3D%26mid%3D2247487134%26idx%3D1%26sn%3Deb999366fea0deccf533df810d658892%26scene%3D21%23wechat_redirect)】提到！

即将开展-->**大模型蒸馏专题0-1**

**基于几种热门方法：[SFT](https://zhida.zhihu.com/search?content_id=254724810&content_type=Article&match_order=1&q=SFT&zhida_source=entity)、[Logit](https://zhida.zhihu.com/search?content_id=254724810&content_type=Article&match_order=1&q=Logit&zhida_source=entity)、[Hidden States-based](https://zhida.zhihu.com/search?content_id=254724810&content_type=Article&match_order=1&q=Hidden+States-based&zhida_source=entity)等**

真正根据自己的文档、任务要求

做一个，拥有自己公司能力、指定领域知识的模型！

**Word数据源（会员数据示例）：**

![](https://pic3.zhimg.com/v2-e503c11ec6edd6f6ca16f96257789a1a_1440w.jpg)

**做成R1的微调数据集：**

![](https://pic1.zhimg.com/v2-f1fe76aa3880ae7d6fb4275f165f3de0_1440w.jpg)

**开始微调-32B-R1：**

![](https://pic3.zhimg.com/v2-cec6a5ba589c5d5425972396d9d4dd86_1440w.jpg)

蒸馏方案很多，今天内容

是整个系列动手前**【先导篇】**

如何根据最常见的word、pdf等非结构化文档！

生成自己领域数据集，然后拿着这份数据集，去DeepSeek中！

做一个专注于你的行业、拥有你公司、个人知识的R1！

如果你，想在企业使用DeepSeek-R1，但不知如何开始！

或者不知道，如何让DeepSeek拥有自己公司的知识？

在末尾，马上找一意工程师-小胖

为你企业做定制开发，辅助或主导你公司的数据结构化转换，让你的数据，为你所用！

今天，一次性解答4个问题！

**① 微调DeepSeek-R1需要怎样数据？**

**② 回答中的<think>思考能力是如何产生的？**

**③ 把word等非结构化数据转为数据集的整套方法论**

**④ 动起手来！把自己的word做成价值非常的数据集！**

本系列全程代码在会员盘，直接下载！强烈建议一边对照代码，一边跑！

另外：数据集制作，一意已做大量实践！许多会员、工程师，都跟着一步步方法，做成了自己的项目

除数据集外，一意还围绕高阶RAG、知识图谱增效RAG、大模型微调、多智能体系统等，做了大量实践，没看的，往前翻公众号！

现在，我们正在为会员交付【[五大项目](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s%3F__biz%3DMzU4NDk0MDAwMw%3D%3D%26mid%3D2247487134%26idx%3D1%26sn%3Deb999366fea0deccf533df810d658892%26scene%3D21%23wechat_redirect)】，你一定要识别下方二维码，加入学习，或联系一意，为你赋能

**第一部分：微调DeepSeek-R1需要怎样数据？**

要看你的任务

接下来，我们会先看看市面上的常见数据集，都有哪些，然后去看看适合deepseek-r1的，有哪些，对应的微调方法又是什么

**1.1 常见的数据集有哪些**

目前用得最多的两种数据是[alpaca](https://zhida.zhihu.com/search?content_id=254724810&content_type=Article&match_order=1&q=alpaca&zhida_source=entity) 和 [sharegpt](https://zhida.zhihu.com/search?content_id=254724810&content_type=Article&match_order=1&q=sharegpt&zhida_source=entity) 格式的

先看数据示例，再分析任务

**alpaca格式**

是一个json，通常包含了instruction、input、output字段，如果你需要让大模型在历史对话中，学习，还制作系统提示system、history这些数据

今天实跑演示的，也是这种格式，如下：

```text
[
  {
    "instruction": "human instruction (required)",
    "input": "human input (optional)",
    "output": "model response (required)",
    "system": "system prompt (optional)",
    "history": [
      ["human instruction in the first round (optional)", "model response in the first round (optional)"],
      ["human instruction in the second round (optional)", "model response in the second round (optional)"]
    ]
  }
]
```

**sharegpt格式**

human、gpt、observation 和 function等多角色，显示在对象列表中

human 和 observation 应该出现在奇数位置，而 gpt 和 function 应该出现在偶数位置

```text
[
  {
    "conversations": [
      {
        "from": "human",
        "value": "human instruction"
      },
      {
        "from": "function_call",
        "value": "tool arguments"
      },
      {
        "from": "observation",
        "value": "tool result"
      },
      {
        "from": "gpt",
        "value": "model response"
      }
    ],
    "system": "system prompt (optional)",
    "tools": "tool description (optional)"
  }
]
```

其他的，还可以根据你的任务，设计数据集类型和格式的，先不展开，同时，一意在微调函数调用模型的专题中，也有0-1实操！

**1.2 需要怎样的数据？**

回到任务，我们要用三种方法，去蒸馏大模型

今天先演示SFT(监督微调)的方法，生产数据集

Logit、Hidden States-based两种，放在后面专题开始

不知道什么是SFT？自行去查阅哈！

SFT！**alpaca格式**格式！

_"instruction": "能给我讲一个寓意深刻的故事吗？",_

_"input": "留空",_

_"output": "<think>思考过程…</think>\n\n好的，我将按照您的要求创作一个富有寓意的故事…（完整故事内容）"_

我们实跑出来的数据集，是这样的！

```text
{'instruction': '四柱八字的起源和发展历程是怎样的？\n\n', 'input': '\n\n', 'output': '<think>首先，我们需要理解四柱八字的基本概念，它起源于远古时代的天文学和星象学，深赋哲理，关注人的命运。四柱命理学起源于西汉，完善于唐朝，特别是唐朝的李虚中被认为是四柱八字学的开山鼻祖。然而，李虚中只创立了三柱论命，直到北宋时期的徐子平加入了时辰，发明了四柱论命法。四柱八字以日干为核心，通过五行生克制化为方法，对后世命学影响深远。南宋时期，徐大升将徐子平的四柱预测方法编著成《渊海子平》，初步确立了子平命学的基本框架和理论体系。随后，四柱预测学在元朝发展缓慢，直到明朝初期刘伯温的《滴天髓》和明嘉靖年间万民英的《三命通会》等著作的出现，四柱八字学得到了进一步的发展。到了清初，沈孝瞻的《子平真诠》和清光绪年间余春台的《穷通宝鉴》等著作，使得四柱八字理论空前提高。民国时期的徐乐吾、袁树珊、韦千里三位大师也有相关著作传世。现代八字研究则是由邵伟华先生在90年代初打破了几十年的易学沉寂期，使得易学如雨后春笋般蓬勃兴起。</think> 四柱八字，又称子平八字，起源于远古时代的天文学和星象学，深赋哲理，关注人的命运。四柱命理学起源于西汉，完善于唐朝，李虚中被认为是四柱八字学的开山鼻祖。北宋时期的徐子平在李虚中三柱论命的基础上加入了时辰，发明了四柱论命法，即以日干为核心，通过五行生克制化为方法的论命体系。南宋时期，徐大升将徐子平的四柱预测方法编著成《渊海子平》，初步确立了子平命学的基本框架和理论体系。随后，四柱预测学在元朝发展缓慢，直到明朝初期刘伯温的《滴天髓》和明嘉靖年间万民英的《三命通会》等著作的出现，四柱八字学得到了进一步的发展。到了清初，沈孝瞻的《子平真诠》和清光绪年间余春台的《穷通宝鉴》等著作，使得四柱八字理论空前提高。民国时期的徐乐吾、袁树珊、韦千里三位大师也有相关著作传世。现代八字研究则是由邵伟华先生在90年代初打破了几十年的易学沉寂期，使得易学如雨后春笋般蓬勃兴起。\n\n', 'repo_name': '\n\n', 'prompt_tokens_len': '\n\n', 'reasoning_content_tokens_len': '\n\n', 'content_tokens_len': '\n\n', 'score': '\n\n'}
```

他里面的思考过程是如何生产的？如何对应自己的数据的？

别急，接下来，继续！

第二部分：**<think>思考能力是如何产生的？**

这是R1强化训练模型，无法关闭的功能（许多第一次使用的朋友提起）

本身训练的数据，就有思维链（COT功能），一种通过逐步推理来解决问题的方法，CoT的核心思想是让模型在生成答案之前，先展示推理过程，而不是直接给出结果

问题 ：

**一意AI知识星球原价688元，老会员续费580元，相当于打几折？**

思维链 cot：

**a.理解折扣的含义**

打折意味着现价是原价的某个百分比。例如，打8折就是原价的80%

**b.计算折扣率公式**

折扣率 = （现价 ÷ 原价）× 100%

**c.代入已知数据**

现价是580元，原价是688元

折扣率 = （580 ÷ 688）× 100%

**d.进行除法计算**

580 ÷ 688 ≈ 0.843

**e.转换为百分比**

0.843 × 100% = 84.3%

**f.得出结论**

老会员续费价格相当于原价的 84.3折

以上！

就是思维链！

比较通用的思维规律，是这样的：

_<think>_

_--先阅读理解用户的问题_

_--然后提出初步的答案回复_

_--再验证以上答案是否合理_

_--最后给出最终答案_

_<think>_

稍后我们也会用这个通用逻辑来让模型做数据集！

但是！

他有问题！

雄哥在会员群里也说了，这是让大模型拉高平均分的方法！

小学考作文，通常写**“蓝色天空下的我们”**比较容易拿高分

写“我在天空下”，通常拿不到分数！

那所有学生都这样写，至少能拿到一个及格分！

但天空还有灰色的、黑色的、甚至是红色的（夕阳），这些高阶的用法，需要LLM根据任务，自动化适配的

究竟思维链是否有价值，完全取决于，思维链的智能化及足够的多样性！

比如

简单理解是：思维链自适配算法

如果你想做一个适合自己任务的R1，不知道如何去生产数据，那不用多说，马上找工程师-小胖，帮你理清楚思路吧！

搞清楚我们所需要的目标数据集，接下来，就是动手做！

**第三部分：把word转为数据集的整套方法论**

为了做好这个系列，我们直接用Word文档

最最常见办公数据！

把他一步步做成对自己有价值的数据集！

整个转换背后的方法论，非常简单

![](https://pic1.zhimg.com/v2-b825859a3178f3a712e512127e8b7dbc_1440w.jpg)

**3.1 把word/pdf/票据等转为结构化json**

一意之前已经做过大量的转化实践

最好是一个完整的一段段文本！

还未学过？

[自有文档提取+拆分！部署中文模型批量化提取+段落拆分！html+pdf+jpg全格式支持！](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s%3F__biz%3DMzU4NDk0MDAwMw%3D%3D%26mid%3D2247485870%26idx%3D1%26sn%3D810151ea6f79a7b7d9e712c68bde1fe9%26scene%3D21%23wechat_redirect)

处理后的结果，是这样的！

![](https://pica.zhimg.com/v2-741de7373d258eeea22c05ba6c949ca8_1440w.jpg)

没时间跑吗？

不急！

我们重新写了一个专门做word文本处理的代码，稍后实跑的时候，会介绍！

现在，我们要把文本块，分好一块块，才能发请求，去让R1总结文本，并蒸馏到他对这个知识的理解+回答！

为什么分块？

大模型都有上下文长度，比如qwen的是32K，超出上下文长度

会截断的！

**3.2 把json发给deepseek-R1-671B做成数据集**

做好了一段段的json

接下来，我们要让大模型根据我们数据中的问题与答案

按照以上思维链的通用方案

反推！

从而得出问题+答案的思维链条！

思维链是根据你任务来的！

如果你的任务，觉得不适合通用方案！

一意过去做了大量的智能体思维链的算法实践

完全根据你场景和数据

开发对应匹配的COT

找小胖

现在，我们发给大模型，让他做成json

同时，也在蒸馏大模型对该条数据的理解和思维

但是！许多无意义的文本块，我们不需要发给大模型的，避免浪费token！占用算力资源！

![](https://pic2.zhimg.com/v2-31bb70f9e1ab9460fed87e79ce196cf1_1440w.jpg)

**3.2 数据集质量评估及筛选**

要记住，你的word数据本身可能存在大量干扰

比如注解中的某个无意义网址、图示的备注

这些标签非常零散，会影响数据集的质量

我们，不要！

这时，我们要对生成的数据集质量

再次进行精细化管理

我们引入一个完整的数据集评分机制

比如：

_相关性 ：是否符合任务要求（1-10 分）_

_逻辑性 ：情节是否连贯（1-10 分）_

_创意性 ：是否有新意（1-10 分）_

_语言 ：是否生动优美（1-10 分）_

_综合得分 = (相关性 + 逻辑性 + 创意性 + 语言) / 4_

这个评分完全是跟你的数据和质量要求来做的！

每一条数据都应该有一个分数，然后把分数过低的，干掉！

**3.3 直接进行监督微调SFT**

到了这步，直接开始微调！

在会员圈，我们做了大量实践！

还未看？

点击下方：

[以慢病MAS为例，微调model函数调用能力！](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s%3F__biz%3DMzU4NDk0MDAwMw%3D%3D%26mid%3D2247487078%26idx%3D1%26sn%3Ded75e59c785a025b33d41cbbdf9cefea%26scene%3D21%23wechat_redirect)

到这里，制作数据集的关键几步，全部讲完！

接下来，跟着实操！

**第四部分：跑起来，动手用word做一个数据集！**

本次全程实践，环境如下：

**操作系统：**win11、Linux(无要求)

**Python版本：**3.10

**环境管理工具：**conda、jupyter

**大模型：**deepseek-R1-671B(MTU蒋哥供应)，也可本地推理

会员，R1满血版直接免费调用！不限速！

如果你需要，也直接找工程师-小胖！

你现在去打开会员盘，下载代码及示例：

![](https://pic2.zhimg.com/v2-f60ddba7130d875d3785c834f9a0c7f3_1440w.jpg)

整个代码非常简单，全部代码不超过一千行！

在开始前，你需要搭建好一个AI环境，还未搭建？

在这里搭建：

[0基础+手把手安装AI必备环境！](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s%3F__biz%3DMzU4NDk0MDAwMw%3D%3D%26mid%3D2247484778%26idx%3D1%26sn%3De0a4211cbcb52145da168d9aaeb42b8d%26scene%3D21%23wechat_redirect)

不管你把上方的代码放在什么目录，都不要有中文目录！

然后，跟着实操！

**创建conda环境！**

我们统一使用conda管理环境，名称r1，Python版本3.10

```text
conda create --name r1 python=3.10
```

**激活环境！**

养成习惯，一个项目=一个环境！避免依赖版本冲突

```text
conda activate r1
```

**安装依赖！**

非常简单！我们已经打包好了依赖了，你cd进入工作目录，直接用以下指令安装！

```text
pip install -r requirements.txt
```

**word分块成json！**

这里，直接从word文件，把他转为json格式，并且按段落分好块！

在开始运行前，你要打开代码，把里面的路径换成你的word文档的！

```text
input_dir = r"D:\人工智能\数据集制作\文件预处理"  # Word 文件所在目录output_dir = r"D:\人工智能\数据集制作\文件预处理\output_json"  # 输出 JSON 文件的目录
```

你要把他重命名，不要中文，这里写上中文，是为了方便大家辨别！

直接运行即可！

```text
python 文件预处理.py
```

处理成功，会显示处理完成！

这里，提醒一下：

分块策略可以跟着你数据走，这里是按段落分块的！

![](https://pic2.zhimg.com/v2-c856a70b7445b51d4155396a1bed5b35_1440w.jpg)

代码就不详细展开了，注解非常清楚！

这里分块不能太大，否则超出大模型上下文，效果极差！

甚至缺失数据！

如果你拿不准方法的话，拿着你的数据找工程师-小胖

**开始生产数据集！**

一样的，开始之前，要先把代码中的路径，改为你本地的路径！

![](https://pic3.zhimg.com/v2-42e59541dbf2e47125e7ed6866a5c756_1440w.jpg)

改完之后，运行以下指令开始！

```text
python 数据集生成（deepseek）.py
```

这里默认使用R1模型，如果你没有本地推理条件，推荐使用API，但硅基流动API，在实践的时候，被限速率了！

会员可以申请免费的R1（蒋哥支持）

为了方便没有R1的朋友， 或者想试试其他模型的，可以使用openai版本

道理是一样的

接下来，我们展开部分重要代码

首先是提示词啦，我们要生成多组这样的数据

![](https://pica.zhimg.com/v2-e9e2f0d8ba6b998bc790e06ec1a9d7b6_1440w.jpg)

然后他会将你的json文件，一条条地发给模型，让他能完整地返回标准化数据集

同时还能有 不错能力！

其他代码一样，不展开了！

**转换为jsonl及数据集评估！**

其实，已收到数据集的半成品了

许多低质量+不安全的因素，导致重复发起请求，或者数据集评分较低的，你要把这些数据，干掉

更加简单了！

```text
python 文件终处理.py
```

眼泪没有必要出啦！

备注好，直接出

```text
import json
import os
def convert_nested_json_to_jsonl(input_folder, output_file):
    with open(output_file, 'w', encoding='utf-8') as outfile:
        # 遍历文件夹中的所有文件
        for filename in os.listdir(input_folder):
            if filename.endswith('.json'):
                file_path = os.path.join(input_folder, filename)
                print(f"Processing file: {file_path}")
                # 读取 JSON 文件内容
                with open(file_path, 'r', encoding='utf-8') as infile:
                    try:
                        data = json.load(infile)
                        print(f"Loaded JSON data: {data}")
                        # 检查是否存在 qa_pairs 字段
                        if "qa_pairs" in data:
                            print(f"qa_pairs found in {filename}, type: {type(data['qa_pairs'])}")
                            if isinstance(data["qa_pairs"], list):
                                for qa_pair in data["qa_pairs"]:
                                    print(f"Processing qa_pair: {qa_pair}")
                                    # 检查是否存在 generated 字段
                                    if "generated" in qa_pair:
                                        print(f"generated found in qa_pair, type: {type(qa_pair['generated'])}")
                                        if isinstance(qa_pair["generated"], list):
                                            for generated_item in qa_pair["generated"]:
                                                print(f"Writing generated item: {generated_item}")
                                                try:
                                                    json.dump(generated_item, outfile, ensure_ascii=False)
                                                    outfile.write('\n')
                                                except TypeError as e:
                                                    print(f"Error writing generated item to JSONL file: {e}")
                    except json.JSONDecodeError as e:
                        print(f"Error decoding JSON in {filename}: {e}")
                    except Exception as e:
                        print(f"Unexpected error processing {filename}: {e}")
# 使用示例
input_folder = r'D:\人工智能\数据集制作\文件终处理'  # 替换为你的实际文件夹路径
output_file = r'D:\人工智能\数据集制作\文件终处理\output.jsonl'  # 输出文件路径
convert_nested_json_to_jsonl(input_folder, output_file)
```

**到这里，已经搞掂了！**

先看看完整样本吧！

```text
{"instruction": "阴阳的概念是什么？\n\n", "input": "\n\n", "output": "<think>首先，我们需要理解阴阳的基本定义。阴阳是中国古代哲学中的一个基本概念，用来描述宇宙间事物的对立统一关系。阴阳不仅存在于自然界中，也体现在社会生活和人体健康等方面。阴阳的概念强调事物的对立性和统一性，即任何事物都可以分为阴阳两个方面，这两个方面既相互对立又相互依存。通过深入分析，我们可以发现阴阳的概念贯穿于中国传统文化和哲学的各个方面，是理解中国传统文化的重要钥匙。</think> 阴阳的概念是中国古代哲学中用来描述宇宙间事物对立统一关系的基本概念。它认为宇宙间的事物尽管种类繁多，但如果按照相反的属性划分，则可以分为对立的两类，即阴阳。阴阳不仅存在于自然界中，如白天为阳，黑夜为阴，也体现在社会生活和人体健康等方面。阴阳的概念强调事物的对立性和统一性，即任何事物都可以分为阴阳两个方面，这两个方面既相互对立又相互依存。\n\n", "repo_name": "\n\n", "prompt_tokens_len": "\n\n", "reasoning_content_tokens_len": "\n\n", "content_tokens_len": "\n\n", "score": "\n\n"},
```

就这样！搞掂啦！

最近太忙了！大家一定要动起手来啊！

马上就会做一个蒸馏的0-1系列！

希望都能学有所得！