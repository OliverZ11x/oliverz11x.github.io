---
date created: 2024/7/31 11:15
date modified: 2024/8/13 9:48
---

- [[#Conda 环境移除|Conda 环境移除]]
- [[#Conda 环境回滚|Conda 环境回滚]]
- [[#Conda 环境复制|Conda 环境复制]]
- [[#Conda 环境迁移|Conda 环境迁移]]
- [[#额外信息|额外信息]]

## Conda 环境移除

要移除一个 Conda 环境，您可以使用 `conda env remove` 命令，后面跟上您要删除的环境名称。以下是详细步骤：
1. **打开您的终端或命令提示符。**
2. **列出所有现有环境**以确保您知道要删除的环境名称：

   ```bash
   conda env list
   ```

3. **移除环境**，指定环境名称。将 `env_name` 替换为您要删除的环境的实际名称：

   ```bash
   conda env remove --name env_name
   ```

   例如，如果环境名称是 `myenv`，您需要运行：

   ```bash
   conda env remove --name myenv
   ```

4. **验证环境已被移除**，再次列出所有环境：

   ```bash
   conda env list
   ```

## Conda 环境回滚

Conda 环境回滚是指将环境恢复到之前的某个状态。这通常在您对环境进行了更改并希望撤销这些更改时非常有用。以下是回滚 conda 环境的步骤：

1. 使用 conda 命令查看当前环境的版本信息。运行以下命令：

```shell
conda list --revision
```

2. 这将显示环境中安装的所有包及其版本。
3. 确定要回滚到的版本号。记下您要回滚到的特定版本号。
4. 卸载当前环境中的包并重新安装旧版本。运行以下命令：

```shell
conda install —-revision [版本号]
```

5. 这将卸载当前环境中的所有包，并重新安装指定版本的包。

## Conda 环境复制

Conda 环境复制是将一个环境完全复制到另一个环境中。这通常在您需要在新服务器或计算机上创建与现有环境完全相同的环境时非常有用。以下是复制 conda 环境的步骤：
1. 创建一个包含环境中所有包和其版本信息的文本文件。运行以下命令：

```shell
conda list —export > [文件名].txt
```

这将创建一个包含环境中所有包和其版本信息的文本文件。
2. 在目标服务器或计算机上创建一个新的 conda 环境，并使用步骤 1 中创建的文本文件来安装包。运行以下命令：

```shell
conda create -n [新环境名称] —file=[文件名].txt
```

这将创建一个新的 conda 环境，并使用提供的文本文件安装所有包和其版本信息。

## Conda 环境迁移

Conda 环境迁移是指将一个环境从一个服务器或计算机迁移到另一个服务器或计算机。这通常在您需要将环境从一个位置移动到另一个位置时非常有用。以下是迁移 conda 环境的步骤：
1. 导出当前环境的依赖关系。运行以下命令：

```shell
conda env export > environment.yml
```

这将创建一个包含环境中所有依赖关系的 YAML 文件。
2. 将 YAML 文件复制到目标服务器或计算机上。您可以使用 SCP、FTP 或其他文件传输工具来完成此操作。
3. 在目标服务器或计算机上创建一个新的 conda 环境，并使用步骤 1 中创建的 YAML 文件来安装依赖关系。运行以下命令：

```shell
conda env create -f environment.yml
```

这将创建一个新的 conda 环境，并使用提供的 YAML 文件安装所有依赖关系。

## Conda 换源

执行以下命令来配置阿里云镜像源：

```shell
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --set show_channel_urls yes
```

如果你想要确认是否已经切换成功，可以查看 `.condarc` 文件或者执行以下命令：

```shell
conda config --show
```

重置为默认 conda 镜像源：

```shell
conda config --remove-key channels
conda config --add channels defaults
```

### 国内常用镜像源

- **清华大学：** https://pypi.tuna.tsinghua.edu.cn/simple/
- **阿里云：** http://mirrors.aliyun.com/pypi/simple/
- **中国科技大学：** https://pypi.mirrors.ustc.edu.cn/simple/
- **豆瓣 (douban)：** http://pypi.douban.com/simple/
- **中国科学技术大学：** http://pypi.mirrors.ustc.edu.cn/simple/

## 额外信息

这些步骤将帮助您有效地管理 Conda 环境，删除不再需要的环境。有关更多详细信息，您可以参考官方的 [Conda管理环境文档](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#removing-an-environment)。

```shell
conda update -n base -c defaults conda
```

Python 版本包搜索
[Search results · PyPI](https://pypi.org/search/)