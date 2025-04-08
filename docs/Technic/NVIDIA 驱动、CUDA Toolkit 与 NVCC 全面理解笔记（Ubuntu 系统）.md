---
date created: 2025/4/8 9:48
date modified: 2025/4/8 10:7
---
## 🔍 一图胜千言：三者的关系

```python
[NVIDIA Driver] ←—— 控制和驱动 GPU
      ↑
      |
[CUDA Toolkit] ←—— 提供 CUDA 编译器（nvcc）、库、工具等
      ↑
      |
  安装方式：sudo apt install cuda-toolkit-12-8
      ↑
      |
    [nvcc] ← CUDA Toolkit 的一部分，用于编译 CUDA 程序
```

当然可以！以下是我们讨论的关于 **NVIDIA Driver、CUDA Toolkit、NVCC、nvidia-smi 等**相关内容的完整笔记，适合作为 CUDA 环境配置与理解的速查手册：

---

# 🚀 NVIDIA 驱动、CUDA Toolkit 与 NVCC 全面理解笔记（Ubuntu 系统）

---

## 📌 一、关键组件总览

| 组件                | 说明                                                            | 是否必装                 | 常用命令             |
| ----------------- | ------------------------------------------------------------- | -------------------- | ---------------- |
| **NVIDIA Driver** | 驱动程序，控制 GPU 硬件，支持 CUDA 程序运行                                   | ✅ 必须                 | `nvidia-smi`     |
| **CUDA Toolkit**  | 包含 CUDA 编译器（`nvcc`）与 `cuBLAS`, `cuDNN`, `Thrust`, `NCCL` 等加速库 | ✅ 用于开发               | `nvcc --version` |
| **NVCC**          | CUDA 编译器，编译 `.cu` 文件                                          | ⛔ 不能单独安装，Toolkit 中包含 | `nvcc`           |
| **nvidia-smi**    | 查看 GPU 状态、驱动版本、支持的 CUDA 运行时版本                                 | ✅ 安装 Driver 即可用      | `nvidia-smi`     |

---

## 🧠 二、nvidia-smi 中 CUDA Version 的含义

```bash
nvidia-smi
```

输出中的：

```text
Driver Version: 550.120.00
CUDA Version: 12.4
```

- **Driver Version**：系统当前 NVIDIA 驱动版本（比如 550.120）
- **CUDA Version**：驱动所支持的**最高 CUDA Runtime API**版本
	⚠️ 并不是你安装的 CUDA Toolkit 版本！

> 你可以不安装 CUDA Toolkit，也能看到这个版本。它只是告诉你，这个驱动**能运行**由 CUDA 12.4 版本编译的程序。

---

## 🧱 三、NVCC 与 CUDA Toolkit 的关系

- `nvcc` 是 CUDA Toolkit 中提供的编译器
- 你无法单独安装 `nvcc`，必须通过安装 CUDA Toolkit 获得
- 使用 `nvcc` 编译 `.cu` 文件后可以生成 CPU+GPU 混合程序

示例编译命令：

```bash
nvcc hello.cu -o hello
```

---

## 🔗 四、NVIDIA Driver 和 CUDA Toolkit 的版本依赖关系

|项目|说明|
|---|---|
|驱动支持向下兼容|Driver 550 支持 CUDA 12.4/12.3/11.8 等旧版 CUDA 程序运行|
|Toolkit 需 Driver 支持|CUDA Toolkit 12.8 至少需要 Driver 535+|
|不匹配情况|若 CUDA 版本 > Driver 支持版本，将无法运行|

👉 参考官方表格：[CUDA Compatibility Guide](https://docs.nvidia.com/deploy/cuda-compatibility/)

---

## 🧹 五、Ubuntu 上高效卸载和重装 Driver + Toolkit

- **建议用 `apt` 安装驱动和 Toolkit，容易统一管理和卸载**
- 安装前可用 `ubuntu-drivers devices` 查看推荐的 Driver
- 用 Docker 跑多版本 CUDA 环境是更优雅的做法

### 1. 卸载 NVIDIA 驱动：

```bash
sudo apt-get --purge remove "*nvidia*"
sudo reboot
```

或 `.run` 文件安装的卸载方式：

```bash
sudo /usr/bin/nvidia-uninstall
```

---

### 2. 卸载 CUDA Toolkit：

```bash
sudo apt-get --purge remove "cuda*"
sudo apt-get --purge remove "libcudnn*"
sudo apt-get --purge remove "libnccl*"
sudo rm -rf /usr/local/cuda*
```

---

### 3. 安装推荐方式（APT）

![[Azure Flux 环境配置#Ubuntu 安装 cuda 驱动]]

## 🧪 六、常用检查命令

```bash
nvidia-smi                 # 查看驱动和 GPU 状态
nvcc --version             # 查看 CUDA 编译器版本
which nvcc                 # 检查是否正确安装 Toolkit
ls /usr/local/             # 查看安装的 CUDA 目录
```

---

## 🧭 七、常见误区对比表

|错误理解|正确解释|
|---|---|
|CUDA Version 是我装的 Toolkit|❌ 是 Driver 支持的最大版本|
|有 CUDA Version 就能开发|❌ 还需安装 Toolkit|
|装了 Toolkit 就能运行|❌ 还需要 NVIDIA 驱动支持|
|NVCC 可以单独装|❌ 它属于 CUDA Toolkit，不能单独安装|

---

## ✅ 八、总结建议

- 🛠 **开发者**：需要安装 NVIDIA Driver + CUDA Toolkit（推荐 `apt` 方式）
- 📦 **只运行别人程序**：只需安装 Driver
- 🎯 使用 Docker：可以避免多版本冲突，非常推荐！
- 🧪 经常使用 `nvidia-smi` + `nvcc` 检查环境是否一致

