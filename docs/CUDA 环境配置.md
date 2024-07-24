
[【保姆级教程】Windows安装CUDA及cuDNN_windows安装cudnn-CSDN博客](https://blog.csdn.net/qq_40968179/article/details/128996692)

[CUDA12.2 conda安装torch_12.2torch-CSDN博客](https://blog.csdn.net/XXXTOWER/article/details/135139310)

[使用虚拟环境conda安装不同版本的cuda，cudnn，pytorch_conda安装cuda-CSDN博客](https://blog.csdn.net/qq_42537872/article/details/132322398)

系统自带 Nvidia 驱动，请勿卸载：
![[./docs/01attachment/Pasted image 20240718181144.png|Pasted image 20240718181144.png]]

## 升级 CUDA 驱动

在 cmd 中用 `nvidia-smi` 命令查看 CUDA 版本为 12.6，因此，CUDA 12.0 以下的版本应该都可以安装。
![[./docs/01attachment/1721612811074.png|1721612811074.png]]

如果需要升级驱动，打开 Nvidia Geforce experience，点击驱动程序更新驱动，即可安装更高版本的 CUDA。
![[./docs/01attachment/1721613232449.png|1721613232449.png]]

## Conda installation CUDA

[CUDA Installation Guide for Linux (nvidia.com)](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#conda-overview)

## Wsl Ubuntu 22.04 cuda 12.5 卸载

[Ubuntu20.04卸载cuda12.0-CSDN博客](https://blog.csdn.net/weixin_44405843/article/details/129816716)
