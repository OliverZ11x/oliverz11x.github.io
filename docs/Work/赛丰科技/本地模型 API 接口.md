---
date created: 2025/4/14 9:55
date modified: 2025/6/10 16:21
---
## 本地模型接口文档（支持涉台、通用）

### 请求头

```python
Content-Type: application/json
```

---

### 接口地址

- **通用接口地址**：`POST http://172.32.1.163:8000/api/chat`
- **涉台接口地址**：`POST http://172.32.1.162:8000/api/chat`

---

### 请求参数（JSON）

| 字段名           | 类型    | 是否必填 | 说明                                                                          |
| ------------- | ----- | ---- | --------------------------------------------------------------------------- |
| `message`     | list  | 是    | 多轮对话消息数组，至少包含一条 `user` 消息。                                                  |
| `max_tokens`  | int   | 否    | 生成回复的最大 token 数。通用模型默认值为 32768，支持范围为 0-32768；涉藏与涉台模型默认值为 8192，支持范围为 0-8192。 |
| `temperature` | float | 否    | 控制生成回复的随机性，默认值为 0.8，支持范围为 0.1~1.0。                                          |
| `top_p`       | float | 否    | 设置 nucleus sampling 截断比例，默认值为 0.9，支持范围为 0~1。                                |

示例：

```json
{
  "message": [
    {"role": "system", "content": "你是一个有帮助的助手"},
    {"role": "user", "content": "你好"}
  ],
  "max_tokens": 512,
  "temperature": 0.7,
  "top_p": 0.90
}
```

---

### 响应参数（JSON）

| 字段名                  | 类型     | 说明              |
| -------------------- | ------ | --------------- |
| `response`           | string | 模型生成的完整回复文本     |
| `input_token_count`  | Int    | 模型输入 token 数    |
| `output_token_count` | Int    | 模型输出 token 数    |
| `totol_token_count`  | Int    | 模型输入输出 token 总数 |

示例：

```json
{
    "response": "你好！很高兴为你提供帮助。有什么问题或需要我帮忙的吗？",
    "input_token_count": 18,
    "output_token_count": 66,
    "totol_token_count": 84
}
```

---

### 健康检查接口

- **接口地址**：`GET /health`
- **返回示例**：

```json
{ "status": "healthy" }
```
