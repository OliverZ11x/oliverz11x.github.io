---
date created: 2024年7月31日,星期三,上午,11:15:58
date modified: 2024年8月8日,星期四,下午,4:36:18
---
[Hello! · Transformers快速入门](https://transformers.run/)

[课程简介 - Hugging Face NLP Course](https://huggingface.co/learn/nlp-course/zh-CN/chapter0/1?fw=pt)

Hugging Face 专门为使用 Transformer 模型编写了一个 Transformers 库，建立在 Pytorch 框架之上（Tensorflow 的版本功能并不完善），所有 Transformer 模型都可以在 Hugging Face Hub 中找到并且加载使用，包括训练、推理、量化等。

# Pipelines

Transformers 库将目前的 NLP 任务归纳为几下几类：

- **文本分类：** 例如情感分析、句子对关系判断等；
- **对文本中的词语进行分类：** 例如词性标注 (POS)、命名实体识别 (NER) 等；
- **文本生成：** 例如填充预设的模板 (prompt)、预测文本中被遮掩掉 (masked) 的词语；
- **从文本中抽取答案：** 例如根据给定的问题从一段文本中抽取出对应的答案；
- **根据输入文本生成新的句子：** 例如文本翻译、自动摘要等。

Transformers 库最基础的对象就是 `pipeline()` 函数，它封装了预训练模型和对应的前处理和后处理环节。我们只需输入文本，就能得到预期的答案。目前常用的 [pipelines](https://huggingface.co/docs/transformers/main_classes/pipelines) 有：

- `feature-extraction` （获得文本的向量化表示）
- `fill-mask` （填充被遮盖的词、片段）
- `ner`（命名实体识别）
- `question-answering` （自动问答）
- `sentiment-analysis` （情感分析）
- `summarization` （自动摘要）
- `text-generation` （文本生成）
- `translation` （机器翻译）
- `zero-shot-classification` （零训练样本分类）

## 这些 pipeline 背后做了什么？（管道内部）

这些简单易用的 pipeline 模型实际上封装了许多操作，下面我们就来了解一下它们背后究竟做了啥。以第一个情感分析 pipeline 为例，我们运行下面的代码

```python 
from transformers import pipeline

classifier = pipeline("sentiment-analysis")
result = classifier("I've been waiting for a HuggingFace course my whole life.")
print(result)
```

就会得到结果：

```json
[{'label': 'POSITIVE', 'score': 0.9598048329353333}]
```

实际上它的背后经过了三个步骤：
1. 预处理 (preprocessing)，将原始文本转换为模型可以接受的输入格式；
2. 将处理好的输入送入模型；
3. 对模型的输出进行后处理 (postprocessing)，将其转换为人类方便阅读的格式。

![](https://huggingface.co/datasets/huggingface-course/documentation-images/resolve/main/en/chapter2/full_nlp_pipeline.svg)

# 模型与分词器

Transformers 库中的两个重要组件：**模型**和**分词器**。

## 模型

除了像之前使用 `AutoModel` 根据 checkpoint 自动加载模型以外，我们也可以直接使用模型对应的 `Model` 类，例如 BERT 对应的就是 `BertModel`：

```python
from transformers import BertModel

model = BertModel.from_pretrained("bert-base-cased")
```

注意，**在大部分情况下，我们都应该使用 `AutoModel` 来加载模型。**这样如果我们想要使用另一个模型（比如把 BERT 换成 RoBERTa），只需修改 checkpoint，其他代码可以保持不变。

### 加载模型

所有存储在 HuggingFace [Model Hub](https://huggingface.co/models) 上的模型都可以通过 `Model.from_pretrained()` 来加载权重，参数可以像上面一样是 checkpoint 的名称，也可以是本地路径（预先下载的模型目录），例如：

```python
from transformers import BertModel

model = BertModel.from_pretrained("./models/bert/")
```

`Model.from_pretrained()` 会自动缓存下载的模型权重，默认保存到 _~/.cache/huggingface/transformers_，我们也可以通过 HF_HOME 环境变量自定义缓存目录。

> [!note] notion
> 由于 checkpoint 名称加载方式需要连接网络，因此在大部分情况下我们都会采用本地路径的方式加载模型。
>
> 部分模型的 Hub 页面中会包含很多文件，我们通常只需要下载模型对应的 _config.json_ 和 _pytorch_model.bin_，以及分词器对应的 _tokenizer.json_、_tokenizer_config.json_ 和 _vocab.txt_。

### 保存模型

保存模型通过调用 `Model.save_pretrained()` 函数实现，例如保存加载的 BERT 模型：

```python
from transformers import AutoModel

model = AutoModel.from_pretrained("bert-base-cased")
model.save_pretrained("./models/bert-base-cased/")
```

这会在保存路径下创建两个文件：

- _config.json_：模型配置文件，存储模型结构参数，例如 Transformer 层数、特征空间维度等；
- _pytorch_model.bin_：又称为 state dictionary，存储模型的权重。

简单来说，配置文件记录模型的**结构**，模型权重记录模型的**参数**，这两个文件缺一不可。我们自己保存的模型同样通过 `Model.from_pretrained()` 加载，只需要传递保存目录的路径。

## 分词器

由于神经网络模型不能直接处理文本，因此我们需要先将文本转换为数字，这个过程被称为**编码 (Encoding)**，其包含两个步骤：

1. 使用分词器 (tokenizer) 将文本按词、子词、字符切分为 tokens；
2. 将所有的 token 映射到对应的 token ID。

### 分词策略

根据切分粒度的不同，分词策略可以分为以下几种：

- **按词切分 (Word-based)**

	![word_based_tokenization](https://transformers.run/assets/img/transformers-note-2/word_based_tokenization.png)

	例如直接利用 Python 的 `split()` 函数按空格进行分词：

	```python
    tokenized_text = "Jim Henson was a puppeteer".split()
    print(tokenized_text)
    ```

	```json
    ['Jim', 'Henson', 'was', 'a', 'puppeteer']
    ```

	这种策略的问题是会将文本中所有出现过的独立片段都作为不同的 token，从而产生巨大的词表。而实际上很多词是相关的，例如 “dog” 和 “dogs”、“run” 和 “running”，如果给它们赋予不同的编号就无法表示出这种关联性。

	> 词表就是一个映射字典，负责将 token 映射到对应的 ID（从 0 开始）。神经网络模型就是通过这些 token ID 来区分每一个 token。

	当遇到不在词表中的词时，分词器会使用一个专门的 [UNK] token 来表示它是 unknown 的。显然，如果分词结果中包含很多 [UNK] 就意味着丢失了很多文本信息，因此一个好的分词策略，应该尽可能不出现 unknown token。

- **按字符切分 (Character-based)**

	![character_based_tokenization](https://transformers.run/assets/img/transformers-note-2/character_based_tokenization.png)

	这种策略把文本切分为字符而不是词语，这样就只会产生一个非常小的词表，并且很少会出现词表外的 tokens。

	但是从直觉上来看，字符本身并没有太大的意义，因此将文本切分为字符之后就会变得不容易理解。这也与语言有关，例如中文字符会比拉丁字符包含更多的信息，相对影响较小。此外，这种方式切分出的 tokens 会很多，例如一个由 10 个字符组成的单词就会输出 10 个 tokens，而实际上它们只是一个词。

	因此现在广泛采用的是一种同时结合了按词切分和按字符切分的方式——按子词切分 (Subword tokenization)。

- **按子词切分 (Subword) **

	高频词直接保留，低频词被切分为更有意义的子词。例如 “annoyingly” 是一个低频词，可以切分为 “annoying” 和 “ly”，这两个子词不仅出现频率更高，而且词义也得以保留。下图展示了对 “Let’s do tokenization!“ 按子词切分的结果：

	![bpe_subword](https://transformers.run/assets/img/transformers-note-2/bpe_subword.png)

	可以看到，“tokenization” 被切分为了 “token” 和 “ization”，不仅保留了语义，而且只用两个 token 就表示了一个长词。这种策略只用一个较小的词表就可以覆盖绝大部分文本，基本不会产生 unknown token。尤其对于土耳其语等黏着语，几乎所有的复杂长词都可以通过串联多个子词构成。

### 直接使用分词器

在实际使用时，我们应该直接使用分词器来完成包括分词、转换 token IDs、Padding、构建 Attention Mask、截断等操作。

**Padding 操作**通过 `padding` 参数来控制：

- `padding="longest"`： 将序列填充到当前 batch 中最长序列的长度；
- `padding="max_length"`：将所有序列填充到模型能够接受的最大长度，例如 BERT 模型就是 512。

**截断操作**通过 `truncation` 参数来控制：

- `truncation=True`，那么大于模型最大接受长度的序列都会被截断，例如对于 BERT 模型就会截断长度超过 512 的序列。
- `max_length` 参数来控制截断长度：

分词器还可以通过 `return_tensors` 参数指定返回的张量格式

- `pt` 则返回 PyTorch 张量；
- `tf` 则返回 TensorFlow 张量，
- `np` 则返回 NumPy 数组。

实际使用分词器时，我们通常会同时进行 padding 操作和截断操作，并设置返回格式为 Pytorch 张量，这样就可以直接将分词结果送入模型：

```python
from transformers import AutoTokenizer, AutoModelForSequenceClassification

checkpoint = "distilbert-base-uncased-finetuned-sst-2-english"
tokenizer = AutoTokenizer.from_pretrained(checkpoint)
model = AutoModelForSequenceClassification.from_pretrained(checkpoint)

sequences = [
    "I've been waiting for a HuggingFace course my whole life.", 
    "So have I!"
]

tokens = tokenizer(sequences, padding=True, truncation=True, return_tensors="pt")
print(tokens)
output = model(**tokens)
print(output.logits)
```

```python
{'input_ids': tensor([
    [  101,  1045,  1005,  2310,  2042,  3403,  2005,  1037, 17662, 12172,
      2607,  2026,  2878,  2166,  1012,   102],
    [  101,  2061,  2031,  1045,   999,   102,     0,     0,     0,     0,
         0,     0,     0,     0,     0,     0]]), 
 'attention_mask': tensor([
    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
    [1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]])}

tensor([[-1.5607,  1.6123],
        [-3.6183,  3.9137]], grad_fn=<AddmmBackward0>)
```

在 `padding=True, truncation=True` 设置下，同一个 batch 中的序列都会 padding 到相同的长度，并且大于模型最大接受长度的序列会被自动截断。

