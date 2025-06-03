---
date created: 2025/4/2 9:38
date modified: 2025/5/29 15:8
---
## 创建进程

```shell
screen -S rl
```

Ctrl+A+D 退出该进程

在命令面板输入 `exit` 关闭该进程

进入该进程

```shell
screen -r
```

## Ubuntu 清理空间

### **1. 检查大文件占用**

```shell
# 安装ncdu
apt-get install ncdu

# 使用ncdu扫描根目录
ncdu /
```

先查找哪些目录占用的空间最大：

```bash
du -h --max-depth=1 / | sort -hr | head -n 20
```

如果 `/home`、`/var` 或 `/usr` 这些目录占用较多，进入相应目录继续执行：

```bash
du -h --max-depth=1 /var | sort -hr | head -n 20
```

---

### **2. 清理不必要的文件**

#### **（1）清理 APT 缓存**

```bash
apt-get autoremove -y  # 删除不再需要的软件包
apt-get autoclean -y   # 清理旧的安装包
apt-get clean -y       # 删除所有缓存的安装包
```

#### **（3）清理 Docker 数据（如果使用了 Docker）**

检查 Docker 占用的空间：

```bash
docker system df
```

清理无用的 Docker 资源：

```bash
docker system prune -a -f --volumes
```

#### **（4）清理临时文件**

```bash
rm -rf /tmp/*
rm -rf /var/tmp/*
```

#### **（5）清理缓存**

```bash
rm -rf ~/.cache/*
```

---

### **3. 查找并删除大文件**

查找大于 1GB 的文件：

```bash
find / -type f -size +1G -exec ls -lh {} \;
```

如果发现无用的大文件，可以使用 `rm` 删除：

```bash
rm -rf /path/to/large/file
```

---

### **4. 卸载不必要的软件**

列出安装的软件包：

```bash
dpkg-query -W --showformat='${Installed-Size;10}\t${Package}\n' | sort -k1,1n | tail -n 20
```

卸载不常用的软件：

```bash
apt-get remove --purge <package_name> -y
```

---

### **5. 重新检查磁盘空间**

```bash
df -h
```

如果清理后仍然空间不足，可以重复上述步骤，或者考虑扩容磁盘。

## Ubuntu 创建 python 虚拟环境

| 方式        | 创建环境                                   | 进入环境                        | 退出环境               |
| --------- | -------------------------------------- | --------------------------- | ------------------ |
| **Conda** | `conda create -n myenv python=3.11 -y` | `conda activate myenv`      | `conda deactivate` |
| **venv**  | `python3 -m venv myenv`                | `source myenv/bin/activate` | `deactivate`       |

在 **Ubuntu 22.04** 上，你可以使用 `conda` 或 `venv` 创建新的 Python 虚拟环境。以下是两种方法：

---

### **方法 1：使用 Conda（推荐）**

如果你已经安装了 **Anaconda** 或 **Miniconda**，可以使用 `conda` 创建和管理 Python 环境。

#### **1. 确保 Conda 已安装**

如果你还没有安装 Miniconda，可以先安装：

```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
```

安装完成后，重启终端或运行：

```bash
source ~/.bashrc
```

#### **2. 创建 Python 环境**

创建一个新的 Python 3.11 虚拟环境：

```bash
conda create -n myenv python=3.11 -y
```

将 `myenv` 替换为你喜欢的环境名称。

#### **3. 进入环境**

激活 Python 环境：

```bash
conda activate myenv
```

#### **4. 退出环境**

如果想退出环境：

```bash
conda deactivate
```

---

### **方法 2：使用 Python 自带的 venv**

如果你不想安装 Conda，可以使用 Python 内置的 `venv` 创建虚拟环境。

#### **1. 安装 Python venv（如果未安装）**

```bash
sudo apt update
sudo apt install -y python3-venv
```

#### **2. 创建 Python 虚拟环境**

创建一个名为 `myenv` 的虚拟环境：

```bash
python3 -m venv myenv
```

（`myenv` 是环境名称，你可以换成其他名称）

#### **3. 进入虚拟环境**

```bash
source myenv/bin/activate
```

进入后，终端会显示 `(myenv)`，表示你已经进入了该环境。

#### **4. 退出环境**

```bash
deactivate
```
