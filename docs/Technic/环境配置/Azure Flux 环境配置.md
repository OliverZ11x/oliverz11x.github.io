---
date created: 2025/3/13 10:33
date modified: 2025/5/26 11:11
---
### Ubuntu 安装 cuda 驱动

Ubuntu 打包 NVIDIA 专有驱动程序。 这些驱动程序直接来自 NVIDIA，只是由 Ubuntu 打包，以便系统可以自动管理这些驱动程序。 从另一个源下载和安装驱动程序可能会导致系统损坏。 此外，安装第三方驱动程序需要在启用了 TrustedLaunch 和安全启动的 VM 上执行额外步骤。 他们要求用户为系统添加新的计算机所有者密钥才能启动。 来自 Ubuntu 的驱动程序由 Canonical 签名，适用于安全启动。

[适用于 Linux 的 Azure N 系列 GPU 驱动程序安装 - Azure Virtual Machines | Microsoft Learn](https://learn.microsoft.com/zh-cn/azure/virtual-machines/linux/n-series-driver-setup#ubuntu)

[从 NVIDIA 下载并安装 CUDA 工具包，获取特定于每个版本的完整路径，自行选择和驱动对应的cuda-toolkit-x-x](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=24.04&target_type=deb_network)

### Azure 虚拟机 `H100` 区别

Azure H100 虚拟机支持 **两种不同版本的 H100 GPU**，分别是 **H100 NVL** 和 **H100 PCIe**，具体取决于所选的虚拟机系列：

1. **ND H100 v5 系列**——采用 **H100 PCIe**（标准 PCIe 连接的 H100）
2. **NCads H100 v5 系列**——采用 **H100 NVL**（专门用于推理和 AI 训练的 NVLink 版本）

### **主要区别**

| **H100 NVL**        | **H100 PCIe**               |                     |
| ------------------- | --------------------------- | ------------------- |
| **使用的 Azure VM 系列** | NCads H100 v5               | ND H100 v5          |
| **连接方式**            | NVLink（用于高带宽 GPU 互连）        | PCIe（适用于常规扩展）       |
| **适用场景**            | AI 训练、批量推理、机器学习、数据库加速       | 深度学习训练、大规模 HPC 计算   |
| **内存**              | 94GB HBM3（两个 NVL 互连 GPU 组合） | 80GB HBM3           |
| **计算能力**            | 2x NVL = 2x H100（更高推理性能）    | 独立 H100 GPU，适用于集群训练 |

如果你的任务是 **大规模深度学习训练**，ND H100 v5 的 **H100 PCIe** 可能更合适。

如果你的任务是 **推理和 AI 应用**，NCads H100 v5 的 **H100 NVL** 可能更优。

### Anaconda 安装

> [在Linux服务器上安装Anaconda超详细教程_Linux_脚本之家](https://www.jb51.net/server/336729ihf.htm)

美国，台独

两种方式

Sft 微调 RL 强化学习

方式

moe

Phi-3.5(moe)/4

千问 2.5 (musk 是 moe 但是不开源)

DeepSeek 蒸馏模型（moe）

---

数据清洗（14 B 三千条，32 B 六千条，数据清洗完后，）：

[OpenCSG 打造中国本土化 Huggingface plus 开源社区 开放传神 OpenCSG 传神社区 官网](http://192.168.0.9/codes/xiaohei/xunlianall/blob/main/yuliaotiqu/fkchuli.py)

RL增强学习（用自带数据集）：

[OpenCSG 打造中国本土化 Huggingface plus 开源社区 开放传神 OpenCSG 传神社区 官网](http://192.168.0.9/codes/xiaohei/xunlianall/blob/main/rl/gtirl.py)

Sft：

[OpenCSG 打造中国本土化 Huggingface plus 开源社区 开放传神 OpenCSG 传神社区 官网](http://192.168.0.9/codes/xiaohei/xunlianall/files/main/sft)

---

### 从教师模型蒸馏学生模型

### 对学生模型进行微调

[deepseek-ai/DeepSeek-R1](https://github.com/deepseek-ai/DeepSeek-R1?tab=readme-ov-file)

### 1. **提高特定任务的性能**

- **蒸馏模型**通常通过从一个较大、较复杂的教师模型（Teacher Model）中提取知识来训练，以获得较小且更高效的学生模型（Student Model）。这些模型通常在通用任务上有很好的性能，但它们可能没有针对特定任务或数据集的最优表现。
- **微调**是通过在特定任务的标注数据上进一步训练学生模型，以使其在该任务上达到最佳性能。通过这种方式，蒸馏模型可以充分利用学生模型的轻量化优势，同时通过微调适应特定任务的需求。

1、模型训练不挑方式，不挑实现逻辑。

> Phi-3.5(moe)/4, 千问 2.5 (musk 是 moe 但是不开源), DeepSeek 蒸馏模型（moe）

2、一切以项目为主，模型不能太大，最大不超过72b最小不要小于14B

3、限定设计新疆、西藏、美国、台湾四个方向，用我们特定的数据

创建进程

```shell
screen -S rl
```

Ctrl+A+D 退出该进程

在命令面板输入 `exit` 关闭该进程

进入该进程

```shell
screen -r
```

1. 训练医学推理大语言模型。
2. 制作 Deepseek-r 1-distill 的数据集，包含 reasoning 用于蒸馏模型的微调。

### 5. 强制删除 screen 会话

要强制关闭 screen 会话，您可以使用以下命令：

```bash
screen -X -S [会话ID] quit
```

或者更简单的方法是：

1. 先按 `Ctrl + A` 然后按 `K`，会提示 "Really kill this window [y/n]"
2. 输入 `y` 确认

如果您不确定会话 ID，可以先列出所有会话：

```bash
screen -ls
```

然后使用对应的会话 ID 强制关闭：

```bash
screen -X -S [会话ID] quit
```

这样就可以强制关闭 screen 会话了。
