---
date created: 2025/3/17 11:35
date modified: 2025/4/10 10:0
---

![[Azure Flux 环境配置#Ubuntu 安装 cuda 驱动]]

![[Linux Ubuntu#Ubuntu 创建 python 虚拟环境]]

![[Unsloth 项目安装]]

```shell
uvicorn src.main:app --host 0.0.0.0 --port 8000
```

在 Ubuntu 24 上使用 VS Code 部署 Docker，可以按照以下步骤进行：

---

## **1. 安装 Docker**

### **（1）更新系统并安装必要依赖**

```bash
sudo apt install -y ca-certificates curl gnupg
```

### **（2）添加 Docker 官方 GPG 密钥**

```bash
sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo tee /etc/apt/keyrings/docker.asc > /dev/null

sudo chmod a+r /etc/apt/keyrings/docker.asc
```

### **（3）添加 Docker 仓库**

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
```

### **（4）安装 Docker**

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### **1. 检查 Docker 运行状态**

```bash
sudo systemctl status docker
```

如果 Docker 没有运行，启动它：

```bash
sudo systemctl start docker
```

并设置开机自启：

```bash
sudo systemctl enable docker
```

### **3. 检查 `/var/run/docker.sock` 权限**

如果问题仍然存在，检查 `docker.sock` 文件权限：

```bash
ls -lah /var/run/docker.sock
```

如果所有者不是 `root:docker`，修正它：

```bash
sudo chown root:docker /var/run/docker.sock
sudo chmod 666 /var/run/docker.sock
```

然后再尝试运行：

```bash
docker run hello-world
```

### **（5）验证 Docker 安装**

```bash
docker --version
docker run hello-world
```

### **（6）配置非 root 用户使用 Docker**

```bash
sudo usermod -aG docker $USER
newgrp docker
docker ps  # 测试是否可以运行
```

---

## **5. 额外优化**

### **（1）开启 Docker 服务开机自启**

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

### **（2）升级 Docker Compose（如果需要）**

```bash
sudo apt install -y docker-compose-plugin
docker compose version
```

---

这样，你就成功在 **Ubuntu 24 + VS Code + Docker** 上完成了开发环境的部署！ 🚀

````markdown
# 解决 `docker run --gpus all -it` 报错问题

在使用以下命令创建容器时：

```bash
docker run --gpus all -it <image_name>
````

如果遇到 GPU 相关报错，可按照以下步骤解决：

## 解决步骤

### 1. 添加 NVIDIA 的 GPG 密钥

```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
```

### 2. 添加 NVIDIA 的仓库

```bash
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
```

### 3. 更新包列表

```bash
sudo apt-get update
```

### 4. 安装 NVIDIA Container Toolkit

```bash
sudo apt-get install -y nvidia-container-toolkit
```

### 5. 重启 Docker 服务

```bash
sudo systemctl restart docker
```

查看 docker 日志

```shell
docker-compose logs chat_service
```