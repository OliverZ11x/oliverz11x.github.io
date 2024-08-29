---
date created: 2024/8/29 14:23
date modified: 2024/8/29 17:48
---

> [!note] 代码地址
> [OliverZ11x/Public-Opinion-Analysis (github.com)](https://github.com/OliverZ11x/Public-Opinion-Analysis)

> [!note] 参考
> [小样本文本分类应用：基于UTC的医疗意图多分类，训练调优部署一条龙 - 飞桨AI Studio星河社区](https://aistudio.baidu.com/projectdetail/5951557)
> [零样本文本分类](https://github.com/PaddlePaddle/PaddleNLP/tree/release/2.8/applications/zero_shot_text_classification)
> [小样本场景下的二/多分类任务指南](https://github.com/PaddlePaddle/PaddleNLP/tree/develop/legacy/applications/text_classification/multi_class/few-shot)

## 概述

在舆情分析和情感分析中，尤其是在基于 VOC 标签体系的场景中，文本分类任务尤为重要。根据任务的复杂性和标签的特性，文本分类任务可以分为以下三种类型：

1. **多分类任务（Multi-Class Classification）**：每个文本只能归属于一个类别，类别之间互斥。
2. **多标签任务（Multi-Label Classification）**：每个文本可以同时属于多个类别，类别之间不互斥。
3. **层次分类任务（Hierarchical Classification）**：分类标签具有层次结构，需要在不同层级间进行分类。

![](https://user-images.githubusercontent.com/63761690/186378697-630d3590-4e67-49a0-8d5f-7cabd9daa894.png)

虽然 VOC 标签体系中的标签具有层次结构，但最终目标是准确确定四级指标，因此文本分类任务——对四级指标进行多分类。

### 分类任务面临的挑战

在实际应用中，针对 583 个四级分类标签，目前确定的几种分类方法存在以下挑战：

1. **基于文本检索的分类方式**：
   - **问题**：由于分类标签数量繁多，导致模型检索召回率较低，分类效果不佳。

2. **基于微调的语言模型分类方式**：
   - **问题**：此方法需要大量的标注样本，标注成本高，不适用于场景中标签繁杂的情况。

### 解决方案

为了应对上述问题，拟采用**基于 UTC 的零样本和小样本语言模型微调的分类**方式，具体步骤如下：

1. **少量样本标注**：
   - 对小部分样本数据进行标注，以便用于后续模型的微调。

2. **三级和四级指标分级分类**：
   - **步骤**：
	 - 首先进行三级指标的分类，以减少输入文本的长度和复杂性（目前该语言模型支持最大输入 token 长度为 2048）。
	 - 然后在四级指标上进行更精细的分类。

3. **采用 Hugging Face 的零样本分类模型**：
   - 调用 Hugging Face 的零样本分类模型，在四级指标上进一步提升分类效果。

通过这一策略，能够在分类精度和效率之间取得较好的平衡，使得复杂场景下的文本分类任务变得更加可行和有效。

## 通用文本分类技术 UTC 介绍

通用文本分类 UTC（Universal Text Classification） 模型微调的文本分类端到端应用方案，UTC 通过统一语义匹配方式 USM（Unified Semantic Matching）来将标签和文本的语义匹配能力进行统一建模。这种方法可以帮助更好地理解文本数据，并从中提取出有用的特征信息。

通过使用 PaddleNLP 的零样本文本分类应用 UTC，可以简单高效实现多任务文本分类数据标注、训练、调优、上线，降低文本分类落地技术门槛。

![UTC](https://user-images.githubusercontent.com/25607475/211755652-dac155ca-649e-470c-ac8b-06156b444b58.png)

## 基于 UTC 的舆情样本分类实现流程

### 文本清洗和文本分割

在舆情分析和情感分析的任务中，文本数据通常包含大量的噪声信息，例如 HTML 标签、特殊字符、多余的空格、无关标记等。因此，对文本进行清洗是确保分类模型性能的重要步骤。以下是基于你的代码总结的文本清洗流程。

#### 文本清洗步骤

##### 移除 HTML 标签

   - 使用正则表达式 `<.*?>` 匹配并移除文本中的 HTML 标签，保持文本的纯净。

   ```python
   def remove_html_tags(text):
       clean = re.compile('<.*?>')
       return re.sub(clean, '', text)
   ```

##### 移除特殊字符

   - 保留字母、数字和中文字符，去除所有其他特殊字符。这一步能有效清理文本中不必要的符号和标点。

   ```python
   def remove_special_characters(text):
       text = re.sub(r'[^a-zA-Z0-9\u4e00-\u9fa5\s]', '', text)
       return text
   ```

##### 去除多余空格

   - 使用正则表达式匹配多个连续的空格，并将其替换为单个空格，同时移除文本首尾的空格。

   ```python
   def remove_extra_spaces(text):
       return re.sub(r'\s+', ' ', text).strip()
   ```

##### 清除标签和标记

   - 去除文本中的特定标记（如 `#标签`、`@用户` 等），避免这些噪声干扰模型的分类。

   ```python
   def clean_tag(text):
       clean_text = re.sub(r'#.*? ', '', text)
       clean_text = re.sub(r'@.*? ', '', clean_text)
       clean_text = re.sub(r'#\w+', '', clean_text)
       clean_text = re.sub(r'@\w+', '', clean_text)
       return clean_text
   ```

##### 移除特定字符和短语

   - 进一步清除文本中特定的短语或标记，如“车友圈广场”，以确保内容的纯净性。

   ```python
   def remove_specific_characters_regex(text):
       text = re.sub('车友圈广场 ', '', text)
       text = re.sub('车友圈广场', '', text)
       return text
   ```

##### 综合清洗函数

   - 以上步骤可以综合为一个清洗函数，依次执行各个清洗步骤，确保文本数据的高质量。

   ```python
   def clean_text(text):
       text = clean_tag(text)
       text = remove_specific_characters_regex(text)
       text = remove_extra_spaces(text)
       return text
   ```

#### 文本清洗后的处理步骤

##### 处理缺失值

   - 将缺失值填充为空字符串，并将文本数据类型转换为字符串类型，确保后续处理的兼容性。

   ```python
   saic_data['内文'] = saic_data['内文'].fillna('').astype(str)
   ```

##### 文本清洗应用

   - 使用 `clean_text` 函数对数据进行清洗，移除噪声信息。

   ```python
   saic_data['清洗后内文'] = saic_data['内文'].apply(clean_text)
   ```

##### 文本分割

   - 将清洗后的文本按照标点符号进行分句，生成单独的句子列表，以便后续分析。

   ```python
   essays = saic_data['清洗后内文'].tolist()
   single_sentences_list = []
   for essay in essays:
       single_sentences_list.extend(re.split(r'(?<=[。?？!！])', essay))
   ```

##### 句子过滤

   - 过滤掉空文本和长度不合适的句子（长度小于 3 或大于 200），确保数据的有效性。
   - 进一步过滤掉非中文文本和包含广告的句子。

   ```python
   filtered_sentences_len = [sentence for sentence in single_sentences_list if 3 <= len(sentence) < 200]
   filtered_sentences_NoneAD = [sentence for sentence in filtered_sentences_len if '广告' not in sentence]
   filtered_sentences_CN = [sentence for sentence in filtered_sentences_NoneAD if chinese_pattern.search(sentence)]
   ```

##### 结果保存与可视化

   - 将清洗和分割后的句子保存为文本文件，并对句子长度分布进行可视化，以便进一步分析和处理。

   ```python
   with open('filtered_sentences_CN.txt', 'w', encoding='utf-8') as file:
       for sentence in filtered_sentences_CN:
           file.write(sentence + '\n')

   sentence_lengths = [len(sentence) for sentence in filtered_sentences_CN]
   plt.hist(sentence_lengths, bins=500)
   plt.xlabel('Sentence Length')
   plt.ylabel('Frequency')
   plt.title('Length Distribution of Sentences')
   plt.show()
   ```

![[IMG-2024-08-29-15-44-54.png]]

### 标签制作 (基于 Label Studio)

#### 安装

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

#### 文本分类任务标注

##### 项目创建

点击创建（Create）开始创建一个新的项目，填写项目名称、描述，然后在 ``Labeling Setup`` 中选择 ``Text Classification``。

- 填写项目名称、描述

![](https://user-images.githubusercontent.com/25607475/210772704-7d8ebe91-eeb7-4760-82ac-f3c6478b754b.png)

- 数据上传，从本地上传 filtered_sentences_CN. Txt 格式文件，选择 ``List of tasks``，然后选择导入本项目

![](https://user-images.githubusercontent.com/25607475/210775940-59809038-fa55-44cf-8c9d-1b19dcbdc8a6.png)

- 设置任务，添加标签

![](https://user-images.githubusercontent.com/25607475/210775986-6402db99-4ab5-4ef7-af8d-9a8c91e12d3e.png)

![](https://user-images.githubusercontent.com/25607475/210776027-c4beb431-a450-43b9-ba06-1ee5455a95c5.png)

##### 数据上传

项目创建后，可在 Project/文本分类任务中点击 ``Import`` 继续导入数据，同样从本地上传 txt 格式文件，选择 ``List of tasks``，详见[项目创建](#data) 。

##### 标签构建

项目创建后，可在 Setting/Labeling Interface 中继续配置标签，详见[项目创建](#label)

默认模式为单标签多分类数据标注。对于多标签多分类数据标注，需要将 `choice` 的值由 `single` 改为 `multiple`。

![](https://user-images.githubusercontent.com/25607475/222630045-8d6eebf7-572f-43d2-b7a1-24bf21a47fad.png)

##### 任务标注

![](https://user-images.githubusercontent.com/25607475/210778977-842785fc-8dff-4065-81af-8216d3646f01.png)

##### 数据导出

勾选已标注文本 ID，选择导出的文件类型为 ``JSON``，导出数据：

![](https://user-images.githubusercontent.com/25607475/210779879-7560116b-22ab-433c-8123-43402659bf1a.png)

##### 数据转换

将导出的文件重命名为 `label_studio.json` 后，放入 `./data` 目录下。通过 [label_studio.py](./label_studio.py) 脚本可转为 UTC 的数据格式。

在数据转换阶段，还需要提供标签候选信息，放在 `./data/label.txt` 文件中，每个标签占一行。例如在医疗意图分类中，标签候选为 ``["病情诊断", "治疗方案", "病因分析", "指标解读", "就医建议", "疾病表述", "后果表述", "注意事项", "功效作用", "医疗费用", "其他"]``，也可通过 ``options`` 参数直接进行配置。

```shell
python label_studio.py \
    --label_studio_file ./data/label_studio.json \
    --save_dir ./data \
    --splits 0.8 0.1 0.1 \
    --options ./data/label.txt
```

### 模型微调

当前使用的是 cpu 进行模型微调，也可以不进行模型微调，直接进行零样本预测。

```shell
# 单卡启动：
!python run_train.py  \
    --device cpu \
    --logging_steps 10 \
    --save_steps 10 \
    --eval_steps 10 \
    --seed 1000 \
    --model_name_or_path utc-base \
    --output_dir ./checkpoint/model_best \
    --dataset_path ./data/ \
    --max_seq_length 2048  \
    --per_device_train_batch_size 2 \
    --per_device_eval_batch_size 2 \
    --gradient_accumulation_steps 8 \
    --num_train_epochs 20 \
    --learning_rate 1e-5 \
    --do_train \
    --do_eval \
    --do_export \
    --export_model_dir ./checkpoint/model_best \
    --overwrite_output_dir \
    --disable_tqdm True \
    --metric_for_best_model macro_f1 \
    --load_best_model_at_end  True \
    --save_total_limit 1 \
    --save_plm
```

### 模型预测

在本次实验中，使用了 PaddleNLP 提供的 `Taskflow` 工具进行零样本文本分类任务。且由于设备原因，仅对 1196 条数据在 cpu 上进行预测，具体步骤如下：

#### 初始化分类任务

   - 首先，读取存储在 `label.txt` 中的分类标签，将其加载为一个列表 `schema`，然后使用 `Taskflow` 初始化零样本文本分类模型 `my_cls`，模型使用了 `utc-base`，并且由于设备性能限制，选择在 CPU上进行预测。

   ```python
   from paddlenlp import Taskflow
   import json
   from tqdm import tqdm

   # 读取 schema 文件
   with open('./data/3label.txt', 'r') as file:
       schema = [line.strip() for line in file]

   # 初始化分类任务
   my_cls = Taskflow("zero_shot_text_classification", model="utc-base", schema=schema, device_id=-1)
   ```

#### 逐行读取测试文件

   - 使用生成器函数逐行读取 `filtered_sentences_CN.txt` 文件中的句子，并对空行进行过滤。

   ```python
   # 定义生成器逐行读取文件
   def read_lines(file_path):
       with open(file_path, 'r', encoding='utf-8') as file:
           for line in file:
               stripped_line = line.strip()
               if stripped_line:
                   yield stripped_line
   ```

#### 分批处理数据

   - 为了防止内存占用过高，定义每批次处理 128 行文本，逐批对数据进行预测。
   - 使用 `pbar` 显示进度条，跟踪处理进度，并确保所有文本都能被处理。

   ```python
   # 分批处理数据
   batch_size = 128  # 每批处理的行数
   results = []

   # 计算文件总行数
   total_lines = sum(1 for _ in read_lines('./data/test.txt'))

   # 使用 tqdm 显示进度条
   with tqdm(total=total_lines, desc="Processing") as pbar:
       batch = []
       for line in read_lines('./data/test.txt'):
           batch.append(line)
           if len(batch) == batch_size:
               batch_result = my_cls(batch)
               results.extend(batch_result)
               batch = []
           pbar.update(1)

       # 处理剩余的行
       if batch:
           batch_result = my_cls(batch)
           results.extend(batch_result)
           pbar.update(len(batch))
   ```

#### 保存预测结果

   - 将分类结果保存为 `JSON` 格式的文件，以便后续分析和使用。

   ```python
   # 将结果保存为 JSON 文件
   with open('./result/result.json', 'w', encoding='utf-8') as f:
       json.dump(results, f, ensure_ascii=False, indent=4)
   ```

#### 性能优化建议

由于目前在内存和设备性能有限的条件下使用 CPU 进行预测，速度相对较慢。建议在内存和设备性能较好的平台上，可以通过以下方式优化模型预测性能：

1. **使用 GPU 加速**：在支持 GPU 的环境中，指定 `device_id` 为 GPU 设备 ID，可以显著提升模型预测速度。
2. **并行处理**：在性能较好的平台上可以采用并行处理方式，进一步加快模型预测过程。

## 结果分析