---
date created: 2025/4/25 10:20
date modified: 2025/4/25 10:27
---
# Xm-phi AI 助手 API 文档

## 基本信息

- 基础 URL: `http://101.126.131.184:8000`
- 认证方式: API Key
- API Key: 在请求头中通过 `X-API-Key` 传递

## 接口列表

### 1. 普通聊天接口

**接口说明**：用于发送消息并获取 AI 助手的回复

**请求方式**：POST

**接口地址**：`/api/chat`

**请求头**：

```python
Content-Type: application/json
X-API-Key: testkey
```

**请求参数**：

```json
{
    "message": "string",    // 用户发送的消息
    "token_id": "string"    // 用户标识
}
```

**响应示例**：

```json
{
    "response": "string"    // AI助手的回复
}
```

**状态码说明**：
- 200: 请求成功
- 401: API 密钥无效
- 500: 服务器内部错误
- 503: 服务未就绪

### 3. 健康检查接口

**接口说明**：用于检查服务是否正常运行

**请求方式**：GET

**接口地址**：`/health`

**请求头**：

```python
X-API-Key: testkey
```

**响应示例**：

```json
{
    "status": "healthy"    // 或 "unhealthy"
}
```

## 错误处理

所有接口在发生错误时会返回相应的 HTTP 状态码和错误信息：

```json
{
    "detail": "错误信息"
}
```

常见错误信息：

- "无效的 API 密钥"
- "服务未就绪"
- "服务器内部错误"

## 使用示例

### Python 示例

```python
import requests

def chat_with_ai(message, token_id, api_key):
    url = "http://localhost:8000/api/chat"
    headers = {
        "Content-Type": "application/json",
        "X-API-Key": api_key
    }
    data = {
        "message": message,
        "token_id": token_id
    }
    
    response = requests.post(url, headers=headers, json=data)
    return response.json()

# 使用示例
result = chat_with_ai(
    message="你好，请介绍一下自己",
    token_id="user_123456",
    api_key="testkey"
)
print(result)
```

### CURL 示例

```bash
# 普通聊天
curl -X POST "http://localhost:8000/api/chat" \
     -H "Content-Type: application/json" \
     -H "X-API-Key: testkey" \
     -d '{
         "message": "你好，请介绍一下自己",
         "token_id": "user_123456"
     }'
```

## 注意事项

1. 所有接口都需要提供有效的 API 密钥
2. 建议使用 HTTPS 进行生产环境部署
3. Token_id 用于标识用户，建议使用唯一标识符
4. 请妥善保管 API 密钥，避免泄露
