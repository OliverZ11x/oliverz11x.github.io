---
date created: 2025/3/27 14:53
date modified: 2025/4/7 14:58
---

```python
Access Key
j45mMkM1otecJk98UsXQ
Secret Key
qZCLgKl6QwOLmZLb8CVINWGkqI0sWh5S47DXnJFV
```

`admin`

`By@123456`

运行 MinIO 客户端后，你可以使用 `mc` 命令与 MinIO 服务进行交互，比如创建存储桶、上传/下载文件、设置访问权限等。以下是详细的使用方法：

---

### ✅ **一、运行 MinIO 客户端**

在命令行中运行：

```powershell
E:\zsy\mc.exe
```

💡 如果你已经将 MinIO 客户端加入环境变量，可以直接运行：

```powershell
mc
```

运行后你可以通过以下命令查看版本和帮助信息：

```powershell
mc --version   # 查看版本
mc --help      # 查看帮助
```

---

### ✅ **二、配置 MinIO 服务别名**

在使用 MinIO 客户端与服务器交互前，你需要配置服务别名：

```powershell
mc alias set <别名> <MinIO地址> <用户名> <密码>
```

- 示例：

```powershell
mc alias set myminio http://localhost:9000 admin password
```

参数解释：

- `<别名>`：自定义名称，用于标记 MinIO 服务（如 `myminio`）
- `<MinIO地址>`：MinIO 服务地址，默认是 `http://localhost:9000`
- `<用户名>`：MinIO 管理员账户名
- `<密码>`：MinIO 密码

✅ 查看所有配置的服务：

```powershell
mc alias list
```

---

### ✅ **三、存储桶操作**

MinIO 中的存储桶类似于 S3 中的存储桶（Bucket），用于存放文件和数据。

1. **创建存储桶**

```powershell
mc mb <别名>/<存储桶名>
```

- 示例：

```powershell
mc mb myminio/my-bucket
```

2. **列出存储桶**

```powershell
mc ls myminio
```

3. **删除存储桶**

```powershell
mc rb myminio/my-bucket
```

- ⚠️ 注意：存储桶必须为空才能删除。

---

### ✅ **四、文件操作**

1. **上传文件**

```powershell
mc cp <本地文件路径> <别名>/<存储桶名>
```

- 示例：

```powershell
mc cp E:\data\file.txt myminio/my-bucket
```

上传整个目录：

```powershell
mc cp --recursive E:\data\ myminio/my-bucket
```

2. **下载文件**

```powershell
mc cp <别名>/<存储桶名>/<文件名> <本地路径>
```

- 示例：

```powershell
mc cp myminio/my-bucket/file.txt E:\backup\
```

3. **列出存储桶中的文件**

```powershell
mc ls myminio/my-bucket
```

4. **删除文件**

```powershell
mc rm myminio/my-bucket/file.txt
```

- 删除整个目录：

```powershell
mc rm --recursive --force myminio/my-bucket/data
```

---

### ✅ **五、共享与访问权限**

1. **生成临时访问链接**

- 生成文件的预签名下载链接：

```powershell
mc share download myminio/my-bucket/file.txt --expire 1h
```

- 生成文件上传链接：

```powershell
mc share upload myminio/my-bucket/file.txt --expire 1h
```

- 默认有效期为 `1小时`，你可以修改时间，如 `30m`、`2h` 等。

2. **设置存储桶的访问权限**

- 设置存储桶为公开可访问：

```powershell
mc anonymous set download myminio/my-bucket
```

- 禁止匿名访问：

```powershell
mc anonymous set none myminio/my-bucket
```

---

### ✅ **六、查询 MinIO 服务状态**

1. **查看 MinIO 服务信息**

```powershell
mc admin info myminio
```

2. **查看存储桶占用空间**

```powershell
mc du myminio/my-bucket
```

3. **监控 MinIO 日志**

```powershell
mc admin console myminio
```

---

### ✅ **七、Python 操作 MinIO**

你也可以通过 Python 使用 `boto3` 与 MinIO 交互：

📌 **安装依赖**

```bash
pip install boto3
```

📌 **Python 上传示例**

```python
import boto3

# MinIO 连接信息
s3 = boto3.client(
    's3',
    endpoint_url='http://localhost:9000',  # MinIO 地址
    aws_access_key_id='admin',             # 用户名
    aws_secret_access_key='password'       # 密码
)

# 上传文件
s3.upload_file('E:\\data\\file.txt', 'my-bucket', 'file.txt')

print("文件上传成功！")
```

---

### ✅ **八、常见问题**

1. **连接失败**

	- 检查 MinIO 服务是否启动：

	```powershell
    docker ps
    ```

	- 启动 MinIO：

	```powershell
    docker start minio
    ```

2. **权限不足**

	- 如果访问存储桶失败，检查权限：

	```powershell
    mc anonymous set download myminio/my-bucket
    ```

---

✅ 如果你有其他 MinIO 相关问题或需求，欢迎随时交流！ 😊