---
date created: 2024/7/31 11:15
date modified: 2024/8/21 15:24
---

[【保姆级教程】Windows安装CUDA及cuDNN_windows安装cudnn-CSDN博客](https://blog.csdn.net/qq_40968179/article/details/128996692)

[CUDA12.2 conda安装torch_12.2torch-CSDN博客](https://blog.csdn.net/XXXTOWER/article/details/135139310)

[使用虚拟环境conda安装不同版本的cuda，cudnn，pytorch_conda安装cuda-CSDN博客](https://blog.csdn.net/qq_42537872/article/details/132322398)

系统自带 Nvidia 驱动，请勿卸载：

![[IMG-2024-07-31-15-50-12.png]]

## 升级 CUDA 驱动

在 cmd 中用 `nvidia-smi` 命令查看 CUDA 版本为 12.6，因此，CUDA 12.0 以下的版本应该都可以安装。

![[IMG-2024-07-31-15-50-14.png]]

如果需要升级驱动，打开 Nvidia Geforce experience，点击驱动程序更新驱动，即可安装更高版本的 CUDA。

![[IMG-2024-07-31-15-50-13.png]]

## Conda installation CUDA

[CUDA Installation Guide for Linux (nvidia.com)](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#conda-overview)

[如何在conda虚拟环境中安装不同版本的CUDA和cuDNN (baidu.com)](https://cloud.baidu.com/article/3173953)

[Win10系统下使用anaconda在虚拟环境下安装CUDA及CUDNN-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/1209157)

## Wsl Ubuntu 22.04 cuda 12.5 卸载

[Ubuntu20.04卸载cuda12.0-CSDN博客](https://blog.csdn.net/weixin_44405843/article/details/129816716)
