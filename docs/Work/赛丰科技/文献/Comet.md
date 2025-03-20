---
date created: 2025/3/12 14:5
date modified: 2025/3/13 15:21
---
# 万卡集群真实部署，已节省数百万 GPU 小时！MoE 通信优化技术 COMET 开源

>当前，[MoE 架构](https://zhida.zhihu.com/search?content_id=254875249&content_type=Article&match_order=1&q=MoE+%E6%9E%B6%E6%9E%84&zhida_source=entity)是业界拓展模型规模的重要方向，然而，其在分布式训练中存在的大量通信开销，仍严重制约了训练效率和成本。为攻克这一瓶颈，豆包大模型团队提出了一个全新的**通信优化系统 COMET**，通过更精准、细粒度的计算-通信重叠技术，在大规模 MoE 模型上可达到单层 1.96 倍加速，端到端平均 1.71 倍效率提升，且在不同并行策略、输入规模及硬件环境下均表现稳定。
>
>目前，COMET **已实际应用于万卡级生产集群**，助力 MoE 模型高效训练，并已累计节省了数百万 GPU 小时资源。此外，COMET 还可与豆包大模型团队此前发布的新一代稀疏模型架构 [UltraMem](https://zhida.zhihu.com/search?content_id=254875249&content_type=Article&match_order=1&q=UltraMem&zhida_source=entity) 结合，实现协同优化
>
> **Comet: Fine-grained Computation-communication Overlapping for Mixture-of-Experts
> **
> **论文链接**：[https://arxiv.org/pdf/2502.19811](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/2502.19811)
>
> **开源地址**：[https://github.com/bytedance/flux](https://link.zhihu.com/?target=https%3A//github.com/bytedance/flux)

混合专家模型（MoE）通过稀疏激活机制突破了传统稠密模型（Dense Model）的计算瓶颈，然而，MoE 的分布式训练仍面临一项严峻挑战：**跨设备通信开销巨大**。例如，Mixtral-8x7B 模型在 Megatron-LM 框架中的通信时间占比可高达 40%，严重制约了训练效率和成本。

核心问题在于，MoE 的专家网络分布在多个 GPU 上，每次计算需频繁执行 Token 分发与结果聚合，导致 GPU 计算资源大量闲置。因此，如何将通信隐藏到计算的过程中，提升模型训练效率、节省计算资源，成为了 MoE 系统优化的关键。

## **1. 难点:「复杂的数据依赖」与「流水线气泡」**

为了掩盖巨大的通信开销，现有方案主要集中在**如何对「计算-通信」进行高效重叠**。

一种方案是将流水线调度与通信算子结合起来，即通过定制训练中流水线并行的调度方式，将不同 microbatch 的计算和通信进行重叠，如 [DeepSeek](https://zhida.zhihu.com/search?content_id=254875249&content_type=Article&match_order=1&q=DeepSeek&zhida_source=entity) 的 [DualPipe](https://zhida.zhihu.com/search?content_id=254875249&content_type=Article&match_order=1&q=DualPipe&zhida_source=entity)。但是，这一方式会导致较大的显存开销，并需要对现有训练框架进行复杂的**侵入性改动**。

其它 MoE 系统方案则是在 microbatch 内部采用了粗粒度的计算-通信流水线，将输入数据分割成「数据块」进行通信与计算的重叠。然而，这种粗粒度的重叠方式难以高效利用计算资源，且无法实现无缝的通信延迟隐藏，尤其在动态路由、异构硬件环境下，**性能损失显著**。

因此，团队认为现有的系统级 MoE 解决方案仍面临两大困境：

### **1）难以解决复杂的数据依赖**

MoE 架构的稀疏特性导致计算和通信间的依赖动态且复杂。MoE 会动态地将 Token 分配给不同专家，而传统的粗粒度矩阵分块方式，会导致 GPU 频繁等待远程数据，从而造成计算资源闲置。‍

如图 1 所示，当专家 0 需要在紫色「数据块」中进行 Tile-level 的计算时，必须先通过 Token-level 的通信接收远程数据（Token B），这种由于复杂数据依赖导致的计算-通信粒度上的错配，使得效率严重下滑。

![](https://pic2.zhimg.com/v2-76302f599e7110a187c8738124d11935_1440w.jpg)

图 1：单层 MoE 模型示意图（专家分布在 GPU0 和 GPU1 两张卡上）

### **2）难以消除计算-通信流水线气泡**

另一个问题是，现有方法无法精确控制计算任务和通信任务对硬件资源的使用，因而，也无法根据不同的模型结构和动态输入，来自适应地调整资源分配。这导致计算和通信无法实现无缝重叠，进而产生大量流水线气泡，增加了系统的延迟。

因此，团队认为：解决 MoE 模型中计算与通信的**粒度不匹配问题**是实现两者高效重叠的关键，同时，还需要根据负载情况自适应调整通信和计算的资源分配，以进一步实现无缝重叠。

## **2. COMET 核心方案**

COMET 是一个针对 MoE 模型的通信优化系统，通过细粒度计算-通信重叠技术，助力大模型训练优化。

团队分析发现，MoE 架构包含两条不同的生产-消费流水线:「计算-通信流水线」和「通信-计算流水线」。如图 2 所示，数据在流水线中流动时，各流水线内的操作会通过一个共享缓冲区链接，该缓冲区被称作「共享张量」。

![](https://picx.zhimg.com/v2-ff5055da98a5e83d69f68ec8edcd1b73_1440w.jpg)

图 2：COMET 的设计结构

基于此，COMET 引入**两项关键机制**，以最小化整体延迟并提升流水线性能。

### **1）共享张量依赖解析**

通过分解和重调度共享张量，解决通信与计算之间的粒度错配问题，实现细至单 Token 级的重叠。

**张量分解：** 将 MoE 层间传递的共享张量沿 Token 维度（M）或隐层维度（N）进行切割，使通信与计算的最小单元对齐。例如，在 MoE 第一层（Layer 0，图 3 左）沿 M 维度分解，使通信和计算在 M 维度进行对齐；在 MoE 第二层（Layer 1，图 3 右）沿 N 维度分解，细粒度传输 Token 结果，保证计算和通信的高效重叠。

![](https://pic3.zhimg.com/v2-c44b223f92653e212f83bbc060c17e46_1440w.jpg)

图 3：COMET 对共享张量进行依赖解析和分解

**计算重调度**：为了更好地隐藏计算与通信的延迟，COMET 会动态调整数据块的计算顺序。例如，优先计算本地数据块，同时异步拉取远程 Token。当某个专家需处理 Token A（本地）和 Token B（远程）时，系统会优先启动 Token A 的计算线程，并与 Token B 的通信线程并行执行，从而消除等待延迟。

![](https://pic3.zhimg.com/v2-314a19a1efbff01f3e79caba1b87a18c_1440w.jpg)

图 4：COMET 在 MoE layer0 中分解并重新调度共享张量

### **2）自适应负载分配**

动态分配 GPU 线程块资源，精准平衡通信与计算负载，消除流水线气泡。

**线程块隔离**：将通信与计算任务分别封装在独立线程块中，避免远程 I/O 阻塞计算核心。在 [Nvidia Hopper](https://zhida.zhihu.com/search?content_id=254875249&content_type=Article&match_order=1&q=Nvidia+Hopper&zhida_source=entity) 架构中，计算线程块专用于执行异步 TMA 指令的 GEMM 运算，通信线程块通过 NVSHMEM 实现单 Token 级数据传输，这种设计赋予了系统在算子级别进行资源管理的能力。

![](https://pic1.zhimg.com/v2-e825e08a72673f958b74303de15069cc_1440w.jpg)

图 5：COMET 的计算/通信线程块隔离设计

**动态负载平衡**：根据输入规模（如 Token 长度 M）、并行策略（EP/TP 比例）实时调整线程块分配。如图 6 所示，当 TP=8、EP=1 时，通信线程块占所有线程块的比例为 19.7%，而当 TP=4、EP=2，该比例需提升至 34.8%，系统通过预编译多个版本的计算-通信融合算子实现在运行时的「零开销」算子动态切换，并始终提供低延迟的算子。

![](https://pic1.zhimg.com/v2-f371a871d32b50113e5ddb2fdaa10368_1440w.jpg)

图 6：单个 MoE 层使用不同数量的通信线程块的时延结果

## **3. 大规模落地验证**

团队在多个大规模 MoE 模型中评估了 COMET 的端到端性能。结果表明，COMET 在 8 卡 H800 的实验集群中，**端到端 MoE 模型**（Mixtral-8x7B、Qwen2-MoE 等）的前向时延较其他基线系统可降低 **31.8%-44.4%**，且在不同并行策略、输入规模及硬件环境下均表现稳定。

![](https://pic2.zhimg.com/v2-cda15ff76d87eb974ecd5e160fe5d397_1440w.jpg)

图 7：COMET 在多个 MoE 模型中的测评结果

在单个 MoE 层上，当输入 Token 数量不同的情况下，COMET 的执行时间均显著短于基线方案，平均实现了 **1.28** 倍到 **2.37** 倍的速度提升。

![](https://pic4.zhimg.com/v2-d2c065c331611f0a2e2fea780cd3546f_1440w.jpg)

图 8：COMET 在单个 MoE 层不同输入 Token 长度下的延迟情况

目前，COMET 已实际应用于万卡级生产集群，助力 MoE 模型高效训练，**并已累计节省数百万 GPU 小时。**该工作在 MLSys 2025 会议获得 5/5/5/4 的评审高分，并被认为在大规模生产环境中极具应用潜力。

具体表现为——

**强鲁棒性**：COMET 采用的细粒度计算-通信重叠方案，即使在专家负载不均衡的场景下，也能保持低于其它基线系统的延迟，具有较好的鲁棒性；

**强泛化能力**：COMET 在 NVLink 与 PCIe 等不同网络环境下均能提供稳定的加速比；在使用不同并行策略时均能生成低时延算子，以供大规模训练框架使用。

## **4. 核心代码开源**

COMET 包含约 1.2 万行 C++ 和 CUDA 代码，以及 2 千行 Python 代码，并向开发者提供了一套友好的 Python API。

![](https://pic4.zhimg.com/v2-76b72aecc9bc07f1110674342e6f5205_1440w.jpg)

图9：COMET 开源页面

此外，COMET 建立了面向 MoE 的细粒度流水线编程范式，通过深度融合 NVSHMEM 通信库与 [CUTLASS](https://zhida.zhihu.com/search?content_id=254875249&content_type=Article&match_order=1&q=CUTLASS&zhida_source=entity) 高效计算算子，实现了通信操作与 GEMM 计算的算子内融合。例如，MoE Layer 1 的 GEMM 计算与 Token 聚合通信可在单一 GPU 算子内完成。这与此前提到的 Deepseek DualPipe 方案并不冲突，两者结合或将带来更好的优化空间。

此外，COMET 可直接接入已有的 MoE 训练框架，支持 TP/EP/EP+TP 多种并行模式，并提供了灵活的插拔式部署方案。

从终端输出的日志来看，有几个关键点需要解释和注意：

### 1. **NCCL 警告**

```plaintext
[W313 07:09:22.390173594 ProcessGroupNCCL.cpp:4561] [PG ID 0 PG GUID 0 Rank 0]  using GPU 0 to perform barrier as devices used by this process are currently unknown. This can potentially cause a hang if this rank to GPU mapping is incorrect. Specify device_ids in barrier() to force use of a particular device, or call init_process_group() with a device_id.
```

这个警告表明，在执行分布式训练时，NCCL（NVIDIA Collective Communications Library）无法自动识别当前进程使用的设备（GPU）。这可能会导致设备到进程的映射不一致，进而导致挂起问题。为了解决这个问题，您可以明确指定设备 ID。例如，在使用 `init_process_group()` 时，指定正确的 `device_id` 或在 `barrier()` 操作中指定设备 ID。

如果您的代码中有多 GPU 进程，需要确保每个进程使用正确的 GPU，并在 `barrier()` 操作中通过 `device_ids` 参数指定 GPU。

### 2. **Flux 警告**

```plaintext
WARNING:root:device NVIDIA H100 NVL not listed in flux. calculate tflops by estimation
```

这条警告表明，您的机器上使用的是 NVIDIA H100 NVL，但是 Flux 没有将其列为已知设备。Flux 通过估算计算了 TFLOPS（每秒万亿次浮点运算）。这并不一定会导致错误，但可能会影响性能计算的准确性。为了避免这个警告，可以尝试更新 Flux 或手动指定设备信息。

### 3. **性能比较**

```plaintext
torch #0: total 11.216 ms, gemm 11.128 ms, comm 0.088 ms
flux  #0: total 13.670 ms, gemm 11.372 ms, comm 2.298 ms, gemm_only 9.024 ms
flux(no-overlap) #0: total 13.708 ms, gemm 13.615 ms, comm 0.093 ms
```

- 这部分输出展示了 **Torch** 和 **Flux** 在执行矩阵乘法（GEMM）时的性能比较：
	- **Torch**：总时间为 11.216 毫秒，其中 GEMM 操作占 11.128 毫秒，通信操作占 0.088 毫秒。
	- **Flux**：总时间为 13.670 毫秒，GEMM 操作占 11.372 毫秒，通信占 2.298 毫秒。
	- **Flux(no-overlap)**：这是 Flux 在没有重叠操作时的结果，总时间为 13.708 毫秒，GEMM 占 13.615 毫秒，通信占 0.093 毫秒。

这些结果显示，**Flux** 的性能略低于 **Torch**，特别是在通信操作上有一定的开销。

### 4. **Bitwise 匹配**

```plaintext
is bitwise match:  True
✅  torch vs flux bitwise match
all close!
✅ flux check passed
```

这部分输出表明，尽管 Flux 和 Torch 的性能有所不同，但它们在比特级别上的输出是一致的，表明两者的计算结果完全相同。这表明 Flux 和 Torch 在这个测试中是功能匹配的，尽管性能差异可能源于实现方式或通信开销。

### 5. **资源泄漏警告**

```plaintext
[W313 07:09:25.751176941 ProcessGroupNCCL.cpp:1496] Warning: WARNING: destroy_process_group() was not called before program exit, which can leak resources.
```

这个警告说明程序退出时没有显式调用 `destroy_process_group()`，这可能导致资源泄漏。在分布式训练中，确保在结束时清理分布式进程组非常重要。可以在代码中添加如下清理代码：

```python
import torch.distributed as dist

def cleanup():
    dist.destroy_process_group()

# 在训练结束时调用
cleanup()
```

### 总结：

1. **NCCL 警告**：确保为每个进程正确指定设备，避免 GPU 到进程的映射不一致。
2. **Flux 设备警告**：Flux 未识别 H100 NVL 设备，可能需要手动指定或更新 Flux。
3. **性能比较**：Flux 和 Torch 在性能上有所差异，Flux 的通信开销较大，可能影响总性能。
4. **Bitwise 匹配**：计算结果匹配，表明两者在功能上没有问题。
5. **资源清理**：确保调用 `destroy_process_group()` 来避免资源泄漏。

以上信息可以帮助您更好地调试和优化分布式训练中的问题。