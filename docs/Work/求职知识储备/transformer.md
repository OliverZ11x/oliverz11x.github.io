---
title: transformer
date created: 2024年7月31日,星期三,上午,11:15:59
date modified: 2024年8月6日,星期二,下午,2:29:00
---
![](docs/01attachment/docs/Work/求职知识储备/transformer/IMG-2024-07-31-15-50.png)
为什么要对qkv进行投影？
不投影attention层没有可训练参数，会让相同的qkv点积，导致每个词的注意力都在自己身上。
![](docs/01attachment/docs/Work/求职知识储备/transformer/IMG-2024-07-31-15-50-1.png)

---

### attention计算

![](docs/01attachment/docs/Work/求职知识储备/transformer/IMG-2024-07-31-15-50-2.png)
为什么要分为不同的注意力头（num_attention_heads）?
CNN中多通道思想，希望不同的注意力头学到不同的特征。
![](docs/01attachment/docs/Work/求职知识储备/transformer/IMG-2024-07-31-15-50-3.png)

attention有加性和乘性，为什么不用加性attention？
计算机对于乘性效果更好
![](docs/01attachment/docs/Work/求职知识储备/transformer/IMG-2024-07-31-15-50-4.png)

除以标准差根号d的作用是：变回均值为0方差为1的正态分布
![](docs/01attachment/docs/Work/求职知识储备/transformer/IMG-2024-07-31-15-50-5.png)

mask为什么为-10000？
这是为了在softmax函数中使得被mask掉的位置的概率趋近于0。注意softmax的公式。
![](docs/01attachment/docs/Work/求职知识储备/transformer/IMG-2024-07-31-15-50-6.png)

---

### 残差连接和normalization

transformer中为什么要用layernorm，和batchnorm有什么区别？
在Transformer模型中使用Layer Normalization（LN）是为了在每个特征维度上独立地进行标准化，以减少内部协变量转移（Internal Covariate Shift）并加速训练过程。不同于Batch Normalization（BN），LN不是在batch维度上进行标准化，而是在特征维度上进行标准化。这样做有助于保持每个特征维度的分布稳定性，避免了BN在处理变长序列时可能引入的问题。

另外，LN相较于BN更适合用于RNN和Transformer等序列模型中，因为LN不需要在整个batch上进行统计，而是在每个样本上进行统计，因此更适应于**变长序列**的处理。此外，LN对于小批量的数据更稳定，而BN对小批量的数据可能会引入较大的噪声。

总的来说，LN和BN都是用来加快训练速度，减少内部协变量转移的方法，但在应用场景和计算方式上有所不同。

transformer中为什么要用prenorm和postnorm有什么区别?
![](docs/01attachment/docs/Work/求职知识储备/transformer/IMG-2024-07-31-15-50-7.png)

---

### FFN

![](docs/01attachment/docs/Work/求职知识储备/transformer/IMG-2024-07-31-15-50-8.png)
