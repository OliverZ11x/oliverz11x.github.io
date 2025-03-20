---
date created: 2025/3/18 14:19
date modified: 2025/3/18 15:21
---
### **1. 确保你的 Hugging Face Token 具有正确权限**

运行以下命令查看当前使用的 Token：

```bash
huggingface-cli whoami --token
```

如果输出中 `access token` 不是 `write` 或 `admin`，你需要重新生成一个具有写入权限的 Token。

**如何生成新的 Token：**

1. 打开 [Hugging Face 访问令牌页面](https://huggingface.co/settings/tokens)。
2. 创建一个新的 Token，确保权限级别选择 **Write** 或 **Admin**。
3. 运行以下命令重新登录：

	```bash
    huggingface-cli login
    ```

4. 输入新的 Token。

### **4. 直接上传模型（跳过 Git）**

如果 Git 无法推送，你可以用 `huggingface_hub` 直接上传模型：

```python
from huggingface_hub import HfApi

api = HfApi()
api.upload_folder(
    folder_path="./your_model_directory",
    repo_id="重庆埃克斯曼得信息技术有限公司/your-model-name",
    repo_type="model",
)
```

这会把 `your_model_directory` 里的所有文件直接上传到 Hugging Face。
