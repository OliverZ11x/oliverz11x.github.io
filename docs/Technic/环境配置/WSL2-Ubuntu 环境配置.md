---
title: WSL2-Ubuntu 安装
date created: 2024/7/31 11:15
date modified: 2024/8/13 17:27
aliases:
  - Win10 安装 WSL2-Ubuntu 并配置 anaconda
---

[Win10 安装WSL2-Ubuntu并配置anaconda_win10安装wsl并下载anconda-CSDN博客](https://blog.csdn.net/cosmic_potato/article/details/129135605)

## Win 10 安装 WSL 2-Ubuntu 并配置 anaconda

### 检查 windows 版本

必须运行 Windows 10 版本 2004 及更高版本（内部版本 19041 及更高版本），检查步骤如下：
1. Win+R 打开 winver

	![[IMG-2024-08-08-14-28-37.png]]
2. 查看版本号，内部版本需大于 19041
	![[IMG-2024-08-08-14-28-37-1.png]]
3. 如果是早先的版本建议更新下

### WSL 2 的安装

1. 打开控制面板->程序->启用或关闭 Windows 功能>适用于 Linux 的 Windows 子系统和虚拟机平台
	![[IMG-2024-08-08-14-28-37-2.png]]
2. 重启系统
3. 在管理员模式下打开 PowerShell，输入如下命令

```shell
wsl --install
```

4. 重新管理员身份打开 powershell，用下面的命令将 wsl 2 设置为默认。

```shell
wsl --set-default-version 2
```

### Ubuntu 的安装

打开 WSL 设置初始用户名和密码，自动安装 Ubuntu_22.04，此时 Ubuntu 安装在 C 盘默认文件夹，如需安装在指定路径，可参考

[Ubuntu_WSL 迁移-CSDN博客](https://blog.csdn.net/whitepu/article/details/136177105)

#### 更换成国内的镜像源（可选）

> [!NOTE]
> 注意换源之后 cuda 的安装可能受到[影响](https://blog.csdn.net/pazzn/article/details/125988205)，并且 VPN 也可能收到影响，请自行斟酌是否需要换源。

1. 将系统源文件复制一份备用

```shell
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
```

2. 用 vi 命令打开并编辑源文件

```shell
sudo vi /etc/apt/sources.list
```

3. 输入 `49dd`，清除所有内容
4. 输入 `i` 进行编辑，复制如下源

```txt
#添加阿里源
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
#添加清华源
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse multiversee
```

5. 先按 `esc`，输入 `：`，最后输入 `wq！` 保存退出
6. 更新

```shell
sudo apt-get -y update && sudo apt-get -y upgrade
```

#### 配置 VPN（可选）

不同代理软件配置 VPN 方法可能有一些细微差别，请根据自己的代理软件做调整。

[WSL2 中访问宿主机 Windows 的代理 - ZingLix Blog](https://zinglix.xyz/2020/04/18/wsl2-proxy/)

[WSL2配置V2ray等http代理服务器 - KarsonJo妙妙屋](https://www.karsonjo.com/wsl2%E9%85%8D%E7%BD%AEv2ray%E7%AD%89http%E4%BB%A3%E7%90%86%E6%9C%8D%E5%8A%A1%E5%99%A8/)

### CUDA 安装

[WSL 上的 CUDA (nvidia.com)](https://docs.nvidia.com/cuda/wsl-user-guide/index.html#cuda-support-for-wsl-2)

在系统上安装 Windows NVIDIA GPU 驱动程序后，CUDA 将在 WSL 2 中可用。安装在 Windows 主机上的 CUDA 驱动程序将作为存根存储在 WSL 2 中，因此**用户不得在 WSL 2 中安装任何 NVIDIA GPU Linux 驱动程序**。这里必须==非常小心==，因为默认的 CUDA 工具包附带一个驱动程序，并且很容易使用默认安装覆盖 WSL 2 NVIDIA 驱动程序。我们建议开发人员使用 [CUDA 工具包下载](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=WSL-Ubuntu&target_version=2.0)页面中提供的单独的 CUDA 工具包用于 WSL 2 （Ubuntu），以避免这种覆盖。此 WSL-Ubuntu CUDA 工具包安装程序不会覆盖已映射到 WSL 2 环境中的 NVIDIA 驱动程序。

1. 输入 `nvidia-smi`，显示当前驱动版本。
	![[IMG-2024-08-08-14-28-37-3.png]]
2. 根据[驱动版本、python 版本和 tensorflow 版本](https://tensorflow.google.cn/install/source?hl=en#gpu)安装 [cuda 11.2](https://developer.nvidia.com/cuda-11.2.0-download-archive?target_os=Linux&target_arch=x86_64&target_distro=WSLUbuntu&target_version=20&target_type=debnetwork)，cuda 版本号需要低于驱动版本号，如果驱动版本过低，请[[CUDA 环境配置#升级 CUDA 驱动|升级 CUDA 驱动]]。安装错误，请[[CUDA 环境配置#Wsl Ubuntu 22.04 cuda 12.5 卸载|卸载驱动]]重新安装。
3. 检查是否安装好 `nvcc -V` 输出结果：
	![[IMG-2024-08-08-14-28-37-4.png]]
	安装完毕之后，`nvcc -V` 有可能还是报没安装 CUDA-Toolkit，先去 `/usr/local/cuda/bin` 看一眼有没有 nvcc 的可执行文件，如果有的话打开 `vi ~/.bashrc`，把 cuda 的 bin 目录加到 PATH，也就是把下面这行加到. Bashrc 中, 然后使用 `source ~/.bashrc` 重新读取配置文件

```shell
export PATH=$PATH:/usr/local/cuda/bin
```

### Cudnn 安装

在 anaconda 环境中安装 cudnn，使用以下命令可以自动匹配版本，当然也可以自行设置版本，但需要注意版本兼容性问题。

```python
conda install cudnn
```

![[IMG-2024-08-08-14-28-38.png]]

### Anaconda 安装

1. 进入 [网址](https://repo.anaconda.com/archive/)手动下载 anaconda 安装包到 `D:\Ubuntu\anaconda` 中
	![[IMG-2024-08-08-14-28-38-1.png]]
2. 激活虚拟环境，复制安装包到虚拟环境中

```shell
mv /mnt/d/Ubuntu/anaconda/Anaconda3-2024.05-Linux-x86_64.sh /home/oliver/download/
```

3. 进入 `home/cxy/download` 中运行如下代码进行安装

```shell
$ bash Anaconda3-2024.05-Linux-x86_64.sh
```

4. 提示输入 yes 同意条款，输入 yes 同意安装路径

Do you wish to update your shell profile to automatically initialize conda?
This will activate conda on startup and change the command prompt when activated.
If you'd prefer that conda's base environment not be activated on startup,
   run the following command when conda is activated:

```shell
conda config --set auto_activate_base false
```

## Tips

### Win 10 与 WSL 2-Ubuntu 文件传输

```shell
mv /mnt/d/Ubuntu/anaconda/Anaconda3-2022.10-Linux-x86_64.sh /home/cxy/download/
```