---
date created: 2025/4/14 9:55
date modified: 2025/4/14 10:37
---

涉台：

接口地址： POST `http://10.10.10.42:8000/api/chat`

**请求头**: `Content-Type: application/json`
**请求参数**（JSON）: 

```json  
{  
    "message": [
    {"role":"system",
    "content":""},
    {"role":"user",
    "content":""},
    ]
}  
```  

**响应参数**（JSON）:

```json  
{  
    "response": "你好，很高兴为你提供帮助。<|im_end|>"  
}  
```

涉藏：

接口地址： POST `http://10.10.10.41:8000/api/chat`

**请求头**: `Content-Type: application/json`  
**请求参数**（JSON）:   

```json    
{    
    "message": [  
    {"role":"system",  
    "content":""},  
    {"role":"user",  
    "content":""},  
    ]  
}    
```    
**响应参数**（JSON）:    
```json    
{    
    "response": "你好，很高兴为你提供帮助。"    
}    
```

通用：

接口地址： POST `http://10.10.10.43:8000/api/chat`

**请求头**: `Content-Type: application/json`  
**请求参数**（JSON）:   

```json    
{    
    "message": [  
    {"role":"system",  
    "content":""},  
    {"role":"user",  
    "content":""},  
    ]  
}    
```    
**响应参数**（JSON）:    
```json    
{    
    "response": "你好，很高兴为你提供帮助。"    
}    
```