---
date created: 2024/8/29 10:55
date modified: 2024/8/29 15:34
---
## 文本分类任务 Label Studio 使用指南

### 1. 安装

**以下标注示例用到的环境配置：**

- Python 3.8+
- Label-studio == 1.6.0

在终端 (terminal) 使用 pip 安装 label-studio：

```shell
pip install label-studio==1.6.0
```

安装完成后，运行以下命令行：

```shell
label-studio start
```

在浏览器打开 [http://localhost:8080/](http://127.0.0.1:8080/)，输入用户名和密码登录，开始使用 label-studio 进行标注。

### 2. 文本分类任务标注

#### 2.1 项目创建

点击创建（Create）开始创建一个新的项目，填写项目名称、描述，然后在 ``Labeling Setup`` 中选择 ``Text Classification``。

- 填写项目名称、描述

![](https://user-images.githubusercontent.com/25607475/210772704-7d8ebe91-eeb7-4760-82ac-f3c6478b754b.png)

- 数据上传，从本地上传 txt 格式文件，选择 ``List of tasks``，然后选择导入本项目

![](https://user-images.githubusercontent.com/25607475/210775940-59809038-fa55-44cf-8c9d-1b19dcbdc8a6.png)

- 设置任务，添加标签

![](https://user-images.githubusercontent.com/25607475/210775986-6402db99-4ab5-4ef7-af8d-9a8c91e12d3e.png)

![](https://user-images.githubusercontent.com/25607475/210776027-c4beb431-a450-43b9-ba06-1ee5455a95c5.png)

#### 2.2 数据上传

项目创建后，可在 Project/文本分类任务中点击 ``Import`` 继续导入数据，同样从本地上传 txt 格式文件，选择 ``List of tasks``，详见[项目创建](#data) 。

#### 2.3 标签构建

项目创建后，可在 Setting/Labeling Interface 中继续配置标签，详见[项目创建](#label)

默认模式为单标签多分类数据标注。对于多标签多分类数据标注，需要将 `choice` 的值由 `single` 改为 `multiple`。

![](https://user-images.githubusercontent.com/25607475/222630045-8d6eebf7-572f-43d2-b7a1-24bf21a47fad.png)

#### 2.4 任务标注

![](https://user-images.githubusercontent.com/25607475/210778977-842785fc-8dff-4065-81af-8216d3646f01.png)

#### 2.5 数据导出

勾选已标注文本 ID，选择导出的文件类型为 ``JSON``，导出数据：

![](https://user-images.githubusercontent.com/25607475/210779879-7560116b-22ab-433c-8123-43402659bf1a.png)

#### 2.6 数据转换

将导出的文件重命名为 ``label_studio.json`` 后，放入 ``./data`` 目录下。通过 [label_studio.py](./label_studio.py) 脚本可转为 UTC 的数据格式。

在数据转换阶段，还需要提供标签候选信息，放在 `./data/label.txt` 文件中，每个标签占一行。例如在医疗意图分类中，标签候选为 ``["病情诊断", "治疗方案", "病因分析", "指标解读", "就医建议", "疾病表述", "后果表述", "注意事项", "功效作用", "医疗费用", "其他"]``，也可通过 ``options`` 参数直接进行配置。

```shell
python label_studio.py \
    --label_studio_file ./data/label_studio.json \
    --save_dir ./data \
    --splits 0.8 0.1 0.1 \
    --options ./data/label.txt
```

#### 2.7 更多配置

- ``label_studio_file``: 从 label studio 导出的数据标注文件。
- ``save_dir``: 训练数据的保存目录，默认存储在 ``data`` 目录下。
- ``splits``: 划分数据集时训练集、验证集所占的比例。默认为[0.8, 0.1, 0.1]表示按照 ``8:1:1`` 的比例将数据划分为训练集、验证集和测试集。
- ``options``: 指定分类任务的类别标签。若输入类型为文件，则文件中每行一个标签。
- ``is_shuffle``: 是否对数据集进行随机打散，默认为 True。
- ``seed``: 随机种子，默认为 1000.

备注：

- 默认情况下 [label_studio.py](./label_studio.py) 脚本会按照比例将数据划分为 train/dev/test 数据集
- 每次执行 [label_studio.py](./label_studio.py) 脚本，将会覆盖已有的同名数据文件
- 对于从 label_studio 导出的文件，默认文件中的每条数据都是经过人工正确标注的。
