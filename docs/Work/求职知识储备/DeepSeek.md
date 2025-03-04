---
date created: 2025/3/4 12:33
date modified: 2025/3/4 14:21
---
### 一、论文背景：为啥要搞这个研究？

大型语言模型（比如ChatGPT这类）这几年发展很快，已经在朝“通用人工智能”（AGI）迈进。推理能力是AI变得更聪明的重要一环，比如解决数学题、写代码、做科学推理等等。之前OpenAI推出了o1系列模型，通过延长推理过程（Chain-of-Thought, CoT）在推理任务上表现得很强，但具体怎么做到的，大家只能猜。

这篇论文的作者想搞清楚：能不能不用传统的那种监督微调（Supervised Fine-Tuning, SFT），直接靠强化学习让模型自己学会推理？他们用了DeepSeek-V3-Base作为基础模型，试着用纯RL打造一个推理高手，结果还真搞出了名堂。

---

### 二、主角登场：DeepSeek-R1-Zero和DeepSeek-R1

论文里主要讲了两个模型，一个是“原始版”DeepSeek-R1-Zero，一个是“升级版”DeepSeek-R1。

#### 1. DeepSeek-R1-Zero：纯RL的试验田

- **怎么做的？**
- 直接拿基础模型（DeepSeek-V3-Base），不给任何监督数据，就用强化学习去训练。
- 用了一种叫GRPO（Group Relative Policy Optimization）的算法，简单说就是让模型自己试错，试出一堆答案，然后根据“对不对”和“格式好不好”给奖励。
- 奖励分两块：一是答案正确性（比如数学题对不对），二是格式（要求模型把思考过程写在`<think>`标签里，答案写在`<answer>`里）。
- **结果咋样？**
- 牛得很！比如在AIME 2024（美国数学邀请赛）上，正确率从15.6%飙到71%，用多数投票（cons@64）还能到86.7%，跟OpenAI的o1-0912差不多。
- 更厉害的是，模型自己学会了反思、验证，还能生成很长的推理过程，完全没人为干预。
- **有啥问题？**
- 可读性差，回答乱七八糟，有时还中英混杂，看着头晕。
- 这让作者觉得，得优化一下，不能光推理强还得让人看得懂。

#### 2. DeepSeek-R1：加点料的升级版

- **怎么改进的？**
- 先用少量高质量的“冷启动数据”（cold-start data）微调基础模型，让它有个好起点。
- 然后分四步走：

1. **冷启动**：收集几千条带长推理过程的数据，教模型怎么写得清楚又好看。
2. **推理强化学习**：跟R1-Zero一样用RL，但加了个“语言一致性”奖励，避免中英混杂。
3. **拒绝采样+SFT**：用RL训练到差不多时，生成一大堆数据（60万推理+20万非推理），再微调模型，让它不只会推理，还能写文章、回答常识问题。
4. **全面RL**：再来一轮RL，优化帮助性和安全性，兼顾各种场景。

- **结果咋样？**
- 性能直接对标OpenAI的o1-1217。比如AIME 2024上79.8%，MATH-500上97.3%，代码任务Codeforces上Elo达到2029（超过96%的人类选手）。
- 比R1-Zero可读性好多了，还能干更多事，比如写作、问答，长上下文理解也很强。

---

### 三、顺手开源：小模型也能很强

- **咋搞的？**
- 用DeepSeek-R1生成的数据（80万条），直接微调了一些开源小模型（Qwen和Llama系列，1.5B到70B不等），叫“蒸馏”（distillation）。
- 没用RL，就简单SFT，结果也很猛。
- **效果如何？**
- 7B的Qwen模型在AIME 2024上55.5%，14B的超了QwQ-32B-Preview，32B和70B甚至干翻o1-mini。
- 证明大模型的推理能力可以“传”给小模型，比直接在小模型上用RL效果好还省力。

---

### 四、实验结果：硬碰硬的数据对比

论文里给了详细的测试结果，跟一堆强模型（Claude-3.5、GPT-4o、o1系列）比了个遍。简单总结：

- **推理任务**：DeepSeek-R1跟o1-1217不相上下，秒杀其他模型。
- **知识任务**：MMLU 90.8%，GPQA Diamond 71.5%，比DeepSeek-V3强，但略逊o1-1217。
- **其他任务**：写作、问答啥的也很牛，AlpacaEval 2.0胜率87.6%，ArenaHard 92.3%。

蒸馏的小模型也很有竞争力，尤其是14B、32B、70B，性价比很高。

---

### 五、聊聊得失：成功的秘密和踩过的坑

#### 1. 为啥成功？

- **纯RL可行**：DeepSeek-R1-Zero证明不靠监督数据也能练出推理能力，RL自己就能让模型进化。
- **冷启动+多阶段**：DeepSeek-R1用少量数据打底，再RL+SFT循环，效果更好还更人性化。
- **蒸馏效率高**：大模型的智慧能便宜地传给小模型。

#### 2. 踩了啥坑？

- **过程奖励模型（PRM）**：想细化每步奖励，但定义难、标注烦，还容易被模型“钻空子”，最后放弃了。
- **蒙特卡洛树搜索（MCTS）**：想模仿AlphaGo用搜索提升推理，但语言模型的搜索空间太大，效果不理想。

---

### 六、未来咋办？

- **通用能力**：现在R1在函数调用、多轮对话上不如V3，得继续优化。
- **语言混杂**：目前只优化了中英文，其他语言容易乱，得修。
- **软件工程**：这块数据少，RL没发挥好，后面要加码。

---

### 七、大白话总结

这论文讲的就是DeepSeek团队怎么用强化学习把一个普通语言模型调教成推理高手。DeepSeek-R1-Zero是纯RL的实验品，证明这路子走得通；DeepSeek-R1是加了料的成品，性能顶尖还好用。他们还顺手把大模型的本事“蒸馏”到小模型上，开源给大家玩。整个过程既有惊喜（模型自己学会反思），也有教训（有些方法行不通），但总的来说，是AI推理领域的一次漂亮突破。

![[IMG-2025-03-04-14-18-29.png]]

 [4000字！DeepSeek-R1 核心强化学习算法 GRPO 详解，收藏这一篇就够了！！](https://blog.csdn.net/2401_85325557/article/details/145677553) 的笔记整理

# 4000字！DeepSeek-R1 核心强化学习算法 GRPO 详解，收藏这一篇就够了！！

## 前言

在大语言模型（LLMs）的训练中，强化学习算法一直是提升模型性能的关键。然而，传统算法如 PPO 面临着计算开销大、策略更新不稳定等问题。本文将深入解析 DeepSeek-R1 模型中采用的新型强化学习算法——GRPO（Group Relative Policy Optimization），包括其原理、实现细节，以及在数学推理和代码生成任务中的卓越表现。

## 1. GRPO算法概述

### 1.1 算法背景与动机

在大语言模型（LLM）的微调过程中，强化学习（RL）扮演着至关重要的角色。传统的近端策略优化（PPO）算法虽然被广泛应用于 LLM 的微调，但其在处理大规模模型时面临着巨大的计算和存储负担。PPO 需要维护一个与策略模型大小相当的价值网络来估计优势函数，这在大模型场景下会导致显著的内存占用和计算代价。

此外，PPO 在更新策略时可能会导致策略分布发生剧烈变化，影响训练的稳定性。为了解决这些问题，DeepSeek 提出了一种新的强化学习算法——组相对策略优化（GRPO），旨在减少对价值网络的依赖，同时保持策略更新的稳定性和高效性。

### 1.2 GRPO核心思想

GRPO 的核心思想是通过组内相对奖励来优化策略模型，而不是依赖传统的批评模型（critic model）。具体而言，GRPO 会在每个状态下采样一组动作，然后根据这些动作的相对表现来调整策略，而不是依赖一个单独的价值网络来估计每个动作的价值。

这种方法的优势在于：

- **减少计算负担**：通过避免维护一个与策略模型大小相当的价值网络，GRPO 显著降低了训练过程中的内存占用和计算代价。

- **提高训练稳定性**：GRPO 通过组内比较来估计优势函数，减少了策略更新的方差，从而确保了更稳定的学习过程。

- **增强策略更新的可控性**：GRPO 引入了 KL 散度约束，防止策略更新过于剧烈，从而保持了策略分布的稳定性。

从数学角度来看，GRPO 的目标是最大化预期累积奖励，同时保持策略更新的稳定性。其目标函数可以表示为：

![GRPO 目标函数公式](https://chatgpt.com/c/image_url)

其中：

- G 是采样动作的组大小。

- $a_i$ 是第 i 个动作的输出序列。

- $T_i$ 是输出序列的长度。

- $r(a_i)$ 是当前策略与旧策略的概率比。

- $\hat{A}(a_i)$ 是分组相对奖励，通过对每个动作的奖励进行归一化得到。

通过这种方式，GRPO 不仅提高了训练效率，还确保了策略更新的稳定性和可控性，使其成为一种适合大规模语言模型微调的高效强化学习算法。

## 2. GRPO算法原理

### 2.1 算法流程

GRPO（Group Relative Policy Optimization）算法的流程可以分为以下几个关键步骤，这些步骤共同协作，实现了对策略模型的高效优化，同时避免了传统强化学习算法中常见的计算瓶颈和稳定性问题。

1. **采样动作组**：对于每个输入状态 $s$，GRPO 从当前策略 $\Pi_\theta$ 中采样一组动作 $a_1, a_2, \ldots, a_G$。这些动作的采样基于策略模型的概率分布，确保了多样性。

2. **奖励评估**：每个采样动作 $a_i$ 都会通过奖励函数 $r(a_i)$ 进行评估，得到对应的奖励值 $r_i$。奖励函数可以根据具体任务设计，例如在数学推理任务中，奖励函数可以基于答案的正确性。

3. **计算相对优势**：将每个动作的奖励值进行归一化处理，得到相对优势 $\hat{A}(a_i)$。具体来说，相对优势可以通过以下公式计算：


	$\hat{A}(a_i) = \frac{r(a_i) - \mu_r}{\sigma_r}$



	其中，$\mu_r$ 和 $\sigma_r$ 分别是奖励值的均值和标准差。


4. **策略更新**：根据计算得到的相对优势 $\hat{A}(a_i)$，更新策略模型的参数 $\theta$。更新的目标是增加具有正相对优势的动作的概率，同时减少具有负相对优势的动作的概率。

5. **KL散度约束**：为了防止策略更新过于剧烈，GRPO 在更新过程中引入了 KL 散度约束。通过限制新旧策略之间的 KL 散度，确保策略分布的变化在可控范围内。

通过以上步骤，GRPO 能够在不依赖价值网络的情况下，实现对策略模型的有效优化，同时保持训练过程的稳定性和高效性。

## 3. GRPO与PPO对比

### 3.1 算法结构对比

GRPO 相比于 PPO 主要的改进点如下：

|对比项|PPO (Proximal Policy Optimization)|GRPO (Group Relative Policy Optimization)|
|---|---|---|
|价值估计|依赖单独的 Critic 网络估计优势|采用组内相对奖励计算优势|
|计算成本|需要训练额外的 Critic 网络，计算量较大|仅需组内奖励计算，减少计算负担|
|更新稳定性|可能出现策略更新剧烈变化，需裁剪机制|组内相对比较，更新更稳定|
|KL 约束|采用 KL 散度约束策略更新|同样采用 KL 约束，防止剧烈变化|
|适用性|适用于一般强化学习任务|适用于大语言模型（LLM）微调|

GRPO 通过避免单独的价值网络，减少计算需求，同时在策略更新过程中利用组内相对比较，使其比 PPO 更加稳定，并且更适用于大模型优化。

---

## 4. 代码示例

以下是 GRPO 的基本伪代码框架：

```python
import torch

class GRPO:
    def __init__(self, policy_model, lr=1e-5, kl_beta=0.1):
        self.policy_model = policy_model
        self.optimizer = torch.optim.Adam(self.policy_model.parameters(), lr=lr)
        self.kl_beta = kl_beta

    def compute_relative_advantage(self, rewards):
        mean_reward = rewards.mean()
        std_reward = rewards.std()
        return (rewards - mean_reward) / (std_reward + 1e-8)

    def update_policy(self, states, actions, rewards, old_log_probs):
        advantages = self.compute_relative_advantage(rewards)
        log_probs = self.policy_model.get_log_probs(states, actions)
        kl_div = (old_log_probs - log_probs).mean()
        
        loss = -torch.mean(log_probs * advantages) + self.kl_beta * kl_div
        self.optimizer.zero_grad()
        loss.backward()
        self.optimizer.step()

        return loss.item()
```

该代码包含了 GRPO 核心步骤：

1. **计算相对优势**：基于组内奖励计算相对优势函数。
2. **策略更新**：使用 KL 散度约束防止策略更新过快。
3. **优化策略模型**：利用 Adam 优化器进行参数更新。

---

## **3.1 算法结构对比**

GRPO 相比于 PPO 主要的改进点如下：

|对比项|PPO (Proximal Policy Optimization)|GRPO (Group Relative Policy Optimization)|
|---|---|---|
|价值估计|依赖单独的 Critic 网络估计优势|采用组内相对奖励计算优势|
|计算成本|需要训练额外的 Critic 网络，计算量较大|仅需组内奖励计算，减少计算负担|
|更新稳定性|可能出现策略更新剧烈变化，需裁剪机制|组内相对比较，更新更稳定|
|KL 约束|采用 KL 散度约束策略更新|同样采用 KL 约束，防止剧烈变化|
|适用性|适用于一般强化学习任务|适用于大语言模型（LLM）微调|

GRPO 通过避免单独的价值网络，减少计算需求，同时在策略更新过程中利用组内相对比较，使其比 PPO 更加稳定，并且更适用于大模型优化。

---

### **4. 代码示例**

以下是 GRPO 的基本伪代码框架：

```python
import torch

class GRPO:
    def __init__(self, policy_model, lr=1e-5, kl_beta=0.1):
        self.policy_model = policy_model
        self.optimizer = torch.optim.Adam(self.policy_model.parameters(), lr=lr)
        self.kl_beta = kl_beta

    def compute_relative_advantage(self, rewards):
        mean_reward = rewards.mean()
        std_reward = rewards.std()
        return (rewards - mean_reward) / (std_reward + 1e-8)

    def update_policy(self, states, actions, rewards, old_log_probs):
        advantages = self.compute_relative_advantage(rewards)
        log_probs = self.policy_model.get_log_probs(states, actions)
        kl_div = (old_log_probs - log_probs).mean()
        
        loss = -torch.mean(log_probs * advantages) + self.kl_beta * kl_div
        self.optimizer.zero_grad()
        loss.backward()
        self.optimizer.step()

        return loss.item()
```

该代码包含了 GRPO 核心步骤：

1. **计算相对优势**：基于组内奖励计算相对优势函数。
2. **策略更新**：使用 KL 散度约束防止策略更新过快。
3. **优化策略模型**：利用 Adam 优化器进行参数更新。

---

## **5. GRPO 在 LLM 微调中的应用**

GRPO 被 DeepSeek-R1 应用于强化学习微调（RLHF）任务，特别是在数学推理和代码生成领域展现出显著优势：

- **数学推理**：提升了模型解题能力，能更稳定地优化解题策略。
- **代码生成**：减少了错误代码输出，提高了代码可读性和正确率。

实验表明，使用 GRPO 进行强化学习优化后，DeepSeek-R1 的推理能力比传统 PPO 微调的 LLM 更稳定，生成质量也更高。

---

## **6. 总结**

GRPO 作为一种新型强化学习算法，相比传统的 PPO，带来了更高效、稳定的策略优化方法，特别适用于 LLM 的强化学习微调任务。其主要特点包括：

- **减少计算成本**：无需单独的 Critic 网络，降低了计算资源需求。
- **稳定的策略更新**：通过组内相对比较降低更新方差，提高训练稳定性。
- **适用于大模型优化**：在 LLM 微调任务中表现优异，提升数学推理和代码生成质量。

随着大模型的不断发展，GRPO 可能会成为未来强化学习优化 LLM 的关键方法之一。

---

**参考文献：**

- DeepSeek 官方论文
- OpenAI PPO 相关研究
- 强化学习最新进展

---

这样整理的 Markdown 版本不仅包含了原文的核心内容，还对 GRPO 进行了清晰的结构化拆解，方便后续阅读和参考。你需要进一步调整格式或补充内容吗？

GRPO 被 DeepSeek-R1 应用于强化学习微调（RLHF）任务，特别是在数学推理和代码生成领域展现出显著优势：

- **数学推理**：提升了模型解题能力，能更稳定地优化解题策略。
- **代码生成**：减少了错误代码输出，提高了代码可读性和正确率。

实验表明，使用 GRPO 进行强化学习优化后，DeepSeek-R1 的推理能力比传统 PPO 微调的 LLM 更稳定，生成质量也更高。

---

## **6. 总结**

GRPO 作为一种新型强化学习算法，相比传统的 PPO，带来了更高效、稳定的策略优化方法，特别适用于 LLM 的强化学习微调任务。其主要特点包括：

- **减少计算成本**：无需单独的 Critic 网络，降低了计算资源需求。
- **稳定的策略更新**：通过组内相对比较降低更新方差，提高训练稳定性。
- **适用于大模型优化**：在 LLM 微调任务中表现优异，提升数学推理和代码生成质量。

随着大模型的不断发展，GRPO 可能会成为未来强化学习优化 LLM 的关键方法之一。
