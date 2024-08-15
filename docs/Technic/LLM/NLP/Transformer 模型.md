---
date created: 2024/7/31 11:15
date modified: 2024/8/15 10:21
---
[Transformer 模型 · Transformers快速入门](https://transformers.run/c1/transformer/)

![transformer 模型架构图](https://huggingface.co/datasets/huggingface-course/documentation-images/resolve/main/en/chapter1/transformers.svg)

# Tansformer Decoder 的输入是什么？

在 Transformer 模型中，Decoder 的输入由两个主要部分组成：

1. **目标序列的已生成部分 (Target Sequence So Far) **：
   - 在训练阶段，Decoder 的输入是已生成的目标序列，即原始目标序列（target sequence）的左移版本。具体来说，对于目标序列 `[y1, y2, …, yn]`， Decoder 的输入是 `[<s>, y1, y2, …, yn-1]`，其中 `<s>` 是起始标记。
   - 在推理阶段（Inference），Decoder 在每一步中生成一个新的令牌（token），并将其添加到已生成的序列中。然后，Decoder 在下一步中使用更新后的序列作为输入。

2. **编码器的输出（Encoder Outputs）**：
   - Encoder 的输出是整个源序列 (source sequence) 经过 Encoder 处理后的上下文表示 (contextual representation)。这些表示提供了源序列的全局信息，帮助 Decoder 在生成目标序列时参考源序列的内容。

总结起来，Transformer Decoder 的输入可以表示为：
- 在训练时：`[目标序列的已生成部分,  Encoder 的输出]`
- 在推理时：`[当前生成的目标序列部分,  Encoder 的输出]`

这样， Decoder 能够根据源序列的信息和目标序列的上下文来生成新的令牌。

## NLP 的例子

当然，下面是一个基于 NLP 的示例，展示了 Transformer Decoder 在机器翻译任务中的输入。

假设我们有一个简单的机器翻译任务，将英语句子翻译成法语。源序列是一个英语句子，目标序列是相应的法语句子。

### 示例句子

- **源序列（英语句子）**: "I love machine learning."
- **目标序列（法语句子）**: "J'aime l'apprentissage automatique."

### Encoder 的输入和输出

1. ** Encoder 的输入**:
	- 源序列（英语句子）: "I love machine learning."

1. ** Encoder 的输出**:
	- Encoder 将每个单词或子词映射到一个向量表示，并输出整个句子的上下文表示。这些表示将传递给 Decoder 。

### Decoder 的输入

#### 在训练阶段

在训练过程中， Decoder 的输入包括目标序列的已生成部分（即左移版本）和 Encoder 的输出。

假设目标序列是"J'aime l'apprentissage automatique."

- **第一个时间步**:
	- Decoder 的输入：`[<s>]`
	- Decoder 的输出：`J'`
- **第二个时间步**:
	- Decoder 的输入：`[<s>, J']`
	- Decoder 的输出：`aime`
- **第三个时间步**:
	- Decoder 的输入：`[<s>, J', aime]`
	- Decoder 的输出：`l'`
- 依此类推，直到整个目标序列被生成。

#### 在推理阶段

在推理过程中， Decoder 逐步生成目标序列，每一步的输入是之前生成的部分和 Encoder 的输出。

- **第一个时间步**:
	- Decoder 的输入：`[<s>]`（起始标记）
	- Decoder 的输出：`J'`
- **第二个时间步**:
	- Decoder 的输入：`[<s>, J']`
	- Decoder 的输出：`aime`
- **第三个时间步**:
	- Decoder 的输入：`[<s>, J', aime]`
	- Decoder 的输出：`l'`
- 依此类推，直到生成完整的句子"J'aime l'apprentissage automatique."

# 解码策略

>[一网打尽文本生成策略（beam search/top-k/top-p/温度系数） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/676398366)

Transformer 解码（Decoding）就是在词表中抓阄。具体而言：模型根据下一个词的概率分布，从词表中选择下一个词。这个选择的策略就是解码策略，也称解码算法、解码方法等。

## Beam search

Beam search 通过扩大每次选择的范围增加了生成的随机性。其存在一个超参数 num_beams，用于设置每次选择的可能结果的个数。具体来说，beam search 不严格要求选择当前最大概率的 token，而是考虑**累积概率最大的 k 个序列**。为了方便叙述，我们假设输出的词表是’A, B, C’这三个字母。当输入为“I am a”, 设置 num_beams=2，beam search 的过程可表达为：

**第一步：** 假设 A, B, C 对应的概率分别为 0.5, 0.2, 0.3。那么此时选择每个 token 并组成序列的概率分别为：

```python
"I am a A": 0.5

"I am a B": 0.3

"I am a C": 0.2
```

![](https://pic1.zhimg.com/80/v2-ea30e0e10cc4975e8858e047dc4acafc_720w.webp)

beam search 会选择概率最大的 num_beams=2 个序列，参与后续的生成。因此 `I am a A` 和 `I am a B` 会参与下一个 token 的预测。

**第二步：**将两个当前序列送入下一个 token 的预测中：

![](https://pic3.zhimg.com/80/v2-09a91b17dcc05a0ca9128b681b8d4b0a_720w.webp)

![](https://pic3.zhimg.com/80/v2-b6ee45f8e5a8de104fe7759758c803fe_720w.webp)

由上图可知 `I am a A A` 的概率为 `0.5\*0.1 = 0.05`，`I am a A B` 的概率为 `0.5\*0.4=0.2` 。以此类推，可以算出六个可能序列对应的概率。此时再保留 num_beams=2 个最大概率的序列，分别是 `I am a A C=0.25`，`I am a B C=0.27`，再把这两个序列送到下一个 token 的预测中。

上述步骤会一直重复，直到遇到结束符或指定的长度。最终，会有 num_beams 个序列被预测出来。此时可以对这几个概率最大的序列做进一步的处理，例如选一个概率最大的作为最终的输出，或者根据概率做个采样作为输出。

Beam search 有什么好处呢？相比于每次都选最大的贪心，beam search 显然**增大了搜索空间**。而且更重要的是，beam search 会**防止潜在概率很大的序列被过早的抛弃**。例如”I am a B C”，在预测到 B 的时候序列的概率还是不大的，但是到 C 就变得更大了。

我们可以使用 huggingface 的 GPT 2 模型感受一下 beam search:

```python
from transformers import GPT2LMHeadModel, GPT2Tokenizer

tokenizer = GPT2Tokenizer.from_pretrained('gpt2')
model = GPT2LMHeadModel.from_pretrained('gpt2')

text = "I am a"
input_ids = tokenizer.encode(text, return_tensors='pt')

output = model.generate(input_ids, 
            max_length=10,  
            num_beams=5, 
            num_return_sequences=5, 
            early_stopping=True)

print(output.shape)
# torch.Size([5, 10])

for out in output:
  print(tokenizer.decode(out))
# I am a big believer in the power of the
# I am a big fan of this book. I
# I am a big fan of this book and I
# I am a big fan of this book. It
# I am a member of the United States Senate.
```

上面我们设置 num_beams=5，用于生成 5 个最大概率的序列。同时 num_return_sequences 用于设置返回多少个结果。如果不设置就默认只会返回 5 个序列中概率最大的那个作为最终输出。Early_stopping 用于控制遇到终止符就停止生成。当 num_beams=1 时，beam search 等价于贪心。

## Top-k sampling

Top-k 采样其实很好理解。就是每一步只考虑概率最大的 k 个 token，并且**把这 k 个 token 的概率做重归一化，** 并随机采样得到预测的 token。假设在一步中，ABC 对应的概率 ligits 是[5,3,2]，k 设置为 2。那么会选出字母 A, B，并把其对应的概率 logits[5,3]进行重新归一化。这个归一化可以是 softmax:

$$𝑠𝑜𝑓𝑡𝑚𝑎𝑥([5,3])=[0.8808,0.1192]$$

随后基于归一化后的概率随机选 A 或 B，拼接到“I am a”后面，并进行下一个 token 的预测，如此反复。

```python
...
output = model.generate(input_ids, 
            max_length=10,  
            do_sample=True,  
            top_k=50)

print(tokenizer.decode(output[0]))
# I am a man and do not belong in a
```

除了概率归一化，top-k 和 beam search 有什么区别呢？看上去他们都是只考虑概率最大的那几个。他们的**核心区别**在于 top-k 自始至终只有一个序列进行预测，k 只用于规定采样的范围，每步只采样一个 token 作为结果。而 beam search 会保留 num_beams 个序列进行预测。

## Top-p sampling

Top-p sampling 也叫 Nucleus sampling。这种策略会把 token 的概率按照递减的次序累加，直到累加的概率值超过了阈值 p，在这些 token 中做采样得到预测。

假设 p=0.7，ABC 在第一步预测的概率分布为[0.5,0.3,0.2]。那么 A 和 B 的概率值加起来超过了 0.7，第一步就会在 A, B 中采样得到预测。假设第二步概率分布为[0.3,0.3,0.4]，那么 ABC 三个加起来才会超过 0.7，此时第二步就会在这三个里面采样，如此反复。

可以看出 top-p 的每一步实际上会在一个动态长度的范围里做采样。这样的优点是可以排除一些概率不高的单词，例如分布为[0.9,0.05,0.05]时，只考虑第一个就足够了，而 top-k 还是会考虑前 k 个。并且在分布相对均衡时，top-p 会增加输出的多样性。

```python
output = model.generate(input_ids, 
            max_length=10,  
            do_sample=True,  
            top_p=0.9)

print(tokenizer.decode(output[0]))
# I am a former New York City police commissioner,
```

## 温度系数

在上面提到的归一化中，我们还可以引入温度系数调整概率分布：

$$Softmax(x_i) = \frac{e^{x_i/T}}{\sum_j e^{x_j/T}}$$

相信大家在之前都听说过，大模型的生成需要调一些参数，其中温度系数就是经常出现的一个超参数。这里面温度系数就起到了对预测 token 的概率分布做调整的作用。还是以 ABC 对应的概率 logits 为[5,3,2]为例，我们试一下不同的 T 下归一化的结果：

```python
import torch.nn.functional as F
import torch

a = torch.tensor([5, 3, 2])

print(F.softmax(a/5., dim=0)) # t=5, [0.4506, 0.3021, 0.2473]
print(F.softmax(a/1., dim=0)) # t=1, [0.8438, 0.1142, 0.0420]
print(F.softmax(a/0.5, dim=0)) # t=0.5 [0.9796, 0.0179, 0.0024]
```

对比结果可以看出，T 越大归一化后的概率分布越均匀，而 T 越小概率分布越陡峭。因此大的 T 会增加模型输出的随机性，而小的 T 则会让模型的输出更加固定。这也是“温度系数影响模型创造性”说法的原因。

# Llama

>[以LLAMA为例，快速入门LLM的推理过程 (qq.com)](https://mp.weixin.qq.com/s/5lbrqbqiHPZIARsVW6l6tA)

>[浅谈LLAMA2核心函数generate源码 (qq.com)](https://mp.weixin.qq.com/s/vnke00f7kzlA16Pw_FJFjQ)

## 架构

对于 llama 来说，只用了**decoder 部分**

llama-7 B 有 32 个 `LlamaDecoderLayer`，每个 Decoder 包含一个 LlamaAttention 和 LlamaMLP，然后是 LlamaRMSNorm 和 head 部分，核心的结构是 LlamaDecoderLayer。

先看核心的 `LlamaDecoderLayer`，llama-7 B 有 32 个，而 30 B 的话有 60 个，30 B 和 7 B 的差别也就是 decoder 的个数和 decoder 的不同配置。

```python
model = LlamaForCausalLM.from_pretrained(model, torch_dtype='auto')

LlamaForCausalLM(  
  (model): LlamaModel(  
    (embed_tokens): Embedding(32000, 4096, padding_idx=31999)  
    (layers): ModuleList(  
      (0-31): 32 x LlamaDecoderLayer(
        (self_attn): LlamaAttention(
          (q_proj): Linear(in_features=4096, out_features=4096, bias=False)  
          (k_proj): Linear(in_features=4096, out_features=4096, bias=False)  
          (v_proj): Linear(in_features=4096, out_features=4096, bias=False)  
          (o_proj): Linear(in_features=4096, out_features=4096, bias=False)  
          (rotary_emb): LlamaRotaryEmbedding()  
        )  
        (mlp): LlamaMLP(  
          (gate_proj): Linear(in_features=4096, out_features=11008, bias=False)  
          (down_proj): Linear(in_features=11008, out_features=4096, bias=False)  
          (up_proj): Linear(in_features=4096, out_features=11008, bias=False)  
          (act_fn): SiLUActivation()  
        )  
        (input_layernorm): LlamaRMSNorm()  
        (post_attention_layernorm): LlamaRMSNorm()  
      )  
    )  
    (norm): LlamaRMSNorm()  
  )  
  (lm_head): Linear(in_features=4096, out_features=32000, bias=False)  
)
```

看下 7 B 模型的 config，可以看到**模型类型为 float 16，use_cache 设置为 true**：

```json
{  
    "architectures": [  
        "LLaMAForCausalLM"  
    ],  
    "bos_token_id": 0,  
    "eos_token_id": 1,  
    "hidden_act": "silu",  
    "hidden_size": 4096,  
    "intermediate_size": 11008,  
    "initializer_range": 0.02,  
    "max_sequence_length": 2048,  
    "model_type": "llama",  
    "num_attention_heads": 32,  
    "num_hidden_layers": 32,  
    "pad_token_id": 0,  
    "rms_norm_eps": 1e-06,  
    "torch_dtype": "float16",  
    "transformers_version": "4.27.0.dev0",  
    "use_cache": true,  
    "vocab_size": 32000  
}
```

## Tokenizer

LlamaTokenizer

这个类是 LLAMA 模型的分词器（tokenizer）的实现，基于字节级的字节对编码（Byte-Pair Encoding）。这个分词器的主要功能是将文本字符串转换为模型可以理解的数字序列，反之亦然。

# GPT 2

>[Training and Fine-Tuning GPT-2 and GPT-3 Models Using Hugging Face Transformers](https://www.it-jim.com/blog/training-and-fine-tuning-gpt-2-and-gpt-3-models-using-hugging-face-transformers-and-openai-api/#info-block-1)

>[GPT2 源码解析 - 知乎](https://zhuanlan.zhihu.com/p/630970209)

GPT-2 是一个 Transformer Decoder 模型。Encoder 和 Decoder 之间的唯一区别是后者是因果性的，即它不能及时返回。这里的“时间”是指序列中标记（单词）的位置 t=1.. T。只有 Decoder 才能用于文本生成。 GPT 模型几乎就是普通的 Transformer Decoder，不同的 GPT 版本仅在大小、次要细节和数据集+训练机制方面有很大差异。如果您了解 GPT-2 甚至 GPT-1 的工作原理，那么在很大程度上也可以了解 GPT-4。

具有明确张量维度的简化 GPT-2 图

![Transformer Decoder模型](https://www.it-jim.com/wp-content/uploads/2023/06/Architecture-of-the-GPT-2-Transformer-model-768x633.webp)

![](https://www.it-jim.com/wp-content/uploads/2023/06/GPT2-768x452.webp)

---
