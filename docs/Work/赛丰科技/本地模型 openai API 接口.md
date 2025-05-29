---
date created: 2025/5/23 14:0
date modified: 2025/5/28 9:40
---
## 概述

本文档提供了本地部署的三个 OpenAI 兼容模型的 API 调用指南，包括通用模型、涉台专用模型和涉藏专用模型。这些模型通过不同端口提供服务，用户可以根据需求选择合适的模型进行调用。

## 基本信息

### 服务地址

- **通用模型**：`http://172.32.1.63:8000/v1`
- **涉台专用模型**：`http://172.32.1.62:8000/v1`
- **涉藏专用模型**：`http://172.32.1.61:8000/v1`

### 认证方式

所有 API 请求需要在请求头中包含 `Authorization` 字段，格式为：

```python
Authorization: Bearer token-abc123
```

| 模型类型   | 模型名                  | 适用场景                       |
| ------ | -------------------- | -------------------------- |
| 通用模型   | `Qwen/Qwen3-14B`     | 适用于一般场景的通用对话和任务            |
| 涉台专用模型 | `model/model_taiwan` | 专注于涉台相关内容的理解和生成，提供更精准的语义处理 |
| 涉藏专用模型 | `model/model_xizang` | 专注于涉藏相关内容的理解和生成，提供更精准的语义处理 |

### 响应格式

所有 API 响应将以 JSON 格式返回。

## API 端点

### 创建聊天完成 (Chat Completions)

**URL**: `/chat/completions`

**Method**: `POST`

**请求参数**:

| 参数名             | 类型      | 是否必需 | 描述                                                              |
| --------------- | ------- | ---- | --------------------------------------------------------------- |
| model           | string  | 是    | 模型名称，`Qwen/Qwen3-14B`，`model/model_taiwan`，`model/model_xizang` |
| messages        | array   | 是    | 聊天消息数组，每个消息包含 `role` (system/user/assistant) 和 `content`        |
| temperature     | float   | 否    | 控制随机性，范围 0-2，默认 0.7                                             |
| top_p           | float   | 否    | 核采样参数，范围 0-1，默认 1                                               |
| n               | integer | 否    | 每个提示生成的回复数量，默认 1                                                |
| max_tokens      | integer | 否    | 最大生成令牌数                                                         |
| stream          | boolean | 否    | 是否流式返回，默认 false                                                 |
| enable_thinking | boolean | 否    | 是否开启 think 模式，默认 true                                           |

**示例请求**:

```python
import os
from openai import OpenAI

# 选择模型对应的端口
client = OpenAI(
    base_url="http://172.32.1.63:8000/v1",  # 通用模型
    api_key="token-abc123",
)

response = client.chat.completions.create(
    model="Qwen/Qwen3-14B",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What's the weather today?"}
    ],
    temperature=0.7,
    extra_body={"chat_template_kwargs": {"enable_thinking": False}}
)

print(response.choices[0].message.content)
```

**响应参数**:

| 参数名               | 类型      | 描述                                |
| ----------------- | ------- | --------------------------------- |
| id                | string  | 响应唯一标识符                           |
| object            | string  | 响应对象类型，固定为 "chat. Completion"     |
| created           | integer | 创建时间戳                             |
| model             | string  | 使用的模型名称                           |
| choices           | array   | 生成的回复数组                           |
| index             | integer | 回复索引                              |
| message           | object  | 消息对象                              |
| role              | string  | 消息角色 (assistant)                  |
| content           | string  | 消息内容                              |
| finish_reason     | string  | 完成原因 (stop/length/content_filter) |
| usage             | object  | 令牌使用统计                            |
| prompt_tokens     | integer | 提示令牌数                             |
| completion_tokens | integer | 生成令牌数                             |
| total_tokens      | integer | 总令牌数                              |

**示例响应**:

```json
{
    "id": "chatcmpl-123456",
    "object": "chat.completion",
    "created": 1693764729,
    "model": "Qwen/Qwen3-14B",
    "choices": [
        {
            "index": 0,
            "message": {
                "role": "assistant",
                "content": "I don't have real-time data, but you can check a weather app for current conditions."
            },
            "finish_reason": "stop"
        }
    ],
    "usage": {
        "prompt_tokens": 15,
        "completion_tokens": 20,
        "total_tokens": 35
    }
}
```

## 错误处理

API 可能返回以下错误码：

| 错误码 | 描述 |
|--------|------|
| 400 | 错误请求，检查参数格式和必填项 |
| 401 | 未授权，检查 API 密钥 |
| 404 | 请求的资源不存在 |
| 500 | 服务器内部错误，请稍后重试 |

错误响应示例：

```json
{
    "error": {
        "message": "Invalid API key",
        "type": "authentication_error",
        "param": null,
        "code": "invalid_api_key"
    }
}
```

## 调用示例

### 通用模型调用

```python
client = OpenAI(
    base_url="http://172.32.1.63:8000/v1",
    api_key="token-abc123",
)
```

### 涉台专用模型调用

```python
client = OpenAI(
    base_url="http://172.32.1.62:8000/v1",
    api_key="token-abc123",
)
```

### 涉藏专用模型调用

```python
client = OpenAI(
    base_url="http://172.32.1.61:8000/v1",
    api_key="token-abc123",
)
```

## 注意事项

1. 请根据具体需求选择合适的模型端口，以获得最佳效果。
2. 模型可能对特定领域的术语和上下文有更好的理解和处理能力。
3. 所有模型均在本地运行，确保服务器资源充足以保证性能。
4. API 密钥需要妥善保管，避免泄露。