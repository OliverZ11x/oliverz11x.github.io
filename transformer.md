![](./docs/%E6%B1%82%E8%81%8C%E7%9F%A5%E8%AF%86%E5%82%A8%E5%A4%87/image/2024-06-02-19-37-11.png)
为什么要对qkv进行投影？
不投影attention层没有可训练参数，会让相同的qkv点积，导致每个词的注意力都在自己身上。
![](./docs/%E6%B1%82%E8%81%8C%E7%9F%A5%E8%AF%86%E5%82%A8%E5%A4%87/image/2024-06-02-20-05-02.png)

---

### attention计算

![](./docs/%E6%B1%82%E8%81%8C%E7%9F%A5%E8%AF%86%E5%82%A8%E5%A4%87/image/2024-06-02-22-22-44.png)
为什么要分为不同的注意力头（num_attention_heads）?
CNN中多通道思想，希望不同的注意力头学到不同的特征。
![](./docs/%E6%B1%82%E8%81%8C%E7%9F%A5%E8%AF%86%E5%82%A8%E5%A4%87/image/2024-06-02-20-13-24.png)

attention有加性和乘性，为什么不用加性attention？
计算机对于乘性效果更好
![](./docs/%E6%B1%82%E8%81%8C%E7%9F%A5%E8%AF%86%E5%82%A8%E5%A4%87/image/2024-06-02-20-19-52.png)

除以标准差根号d的作用是：变回均值为0方差为1的正态分布
![](./docs/%E6%B1%82%E8%81%8C%E7%9F%A5%E8%AF%86%E5%82%A8%E5%A4%87/image/2024-06-02-20-08-47.png)

mask为什么为-10000？
这是为了在softmax函数中使得被mask掉的位置的概率趋近于0。注意softmax的公式。
![](./docs/%E6%B1%82%E8%81%8C%E7%9F%A5%E8%AF%86%E5%82%A8%E5%A4%87/image/2024-06-02-20-47-30.png)

---

### 残差连接和normalization

transformer中为什么要用layernorm，和batchnorm有什么区别？
在Transformer模型中使用Layer Normalization（LN）是为了在每个特征维度上独立地进行标准化，以减少内部协变量转移（Internal Covariate Shift）并加速训练过程。不同于Batch Normalization（BN），LN不是在batch维度上进行标准化，而是在特征维度上进行标准化。这样做有助于保持每个特征维度的分布稳定性，避免了BN在处理变长序列时可能引入的问题。

另外，LN相较于BN更适合用于RNN和Transformer等序列模型中，因为LN不需要在整个batch上进行统计，而是在每个样本上进行统计，因此更适应于**变长序列**的处理。此外，LN对于小批量的数据更稳定，而BN对小批量的数据可能会引入较大的噪声。

总的来说，LN和BN都是用来加快训练速度，减少内部协变量转移的方法，但在应用场景和计算方式上有所不同。

transformer中为什么要用prenorm和postnorm有什么区别?
![](./docs/%E6%B1%82%E8%81%8C%E7%9F%A5%E8%AF%86%E5%82%A8%E5%A4%87/image/2024-06-02-22-20-01.png)

---

### FFN

![](./docs/%E6%B1%82%E8%81%8C%E7%9F%A5%E8%AF%86%E5%82%A8%E5%A4%87/image/2024-06-02-20-55-20.png)