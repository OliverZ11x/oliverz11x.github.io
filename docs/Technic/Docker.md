---
date created: 2025/4/2 9:40
date modified: 2025/4/8 9:29
---
## Docker 命令总结

在你的工作中，你可能会经常使用 Docker 进行模型微调、推理加速、环境管理等。以下是你可能需要的 Docker 命令总结：

- **`docker`** 是命令行工具，适合**单个容器**的构建与运行。
- **`docker-compose`** 是编排工具，适合**多个服务（多个容器）组合部署**。

|维度|`docker`|`docker-compose`|
|---|---|---|
|**使用方式**|命令行（逐条运行）|YAML 文件（集中管理）|
|**适用场景**|单个容器，简单服务|多容器组合（如前端 + 后端 + 数据库）|
|**配置方式**|每次运行需写全参数|一次性写好 `docker-compose.yml`|
|**启动命令**|`docker run …`|`docker-compose up`|
|**依赖管理**|无服务依赖管理|可定义依赖顺序（`depends_on`）|
|**可维护性**|配置零散，维护困难|所有配置集中在一个文件中，便于协作|

### Docker Compose（适用于多容器环境）

```bash
# 后台（-d）运行 docker-compose.yml 配置的所有服务
docker-compose up -d

# 停止 docker-compose 服务
docker-compose down

# 持续（-f）查看日志
docker-compose logs -f
```

### 基础命令

```bash
# 检查 Docker 版本
docker version

# 显示 Docker 运行状态
docker info

# 启动 Docker（Linux/macOS）
sudo systemctl start docker

# 关闭 Docker
sudo systemctl stop docker

# 重启 Docker
sudo systemctl restart docker
```

在 Docker 容器运行时，你可以使用以下命令来查看程序运行日志：

### 查看容器日志

```bash
docker logs <container_id>  # 查看容器标准输出日志
```

### 实时跟踪容器日志（类似 `tail -f`）

```bash
docker logs -f <container_id>
```

（`-f` 选项用于实时查看日志输出）

### 限制日志输出的行数

```bash
docker logs --tail 100 <container_id>  # 仅查看最近 100 行日志
```

### 过滤日志（按时间）

```bash
docker logs --since 30m <container_id>  # 查看最近 30 分钟的日志
docker logs --since "2024-04-01T12:00:00" <container_id>  # 查看指定时间后的日志
```

### 结合 `grep` 进行日志筛选

```bash
docker logs <container_id> | grep "error"  # 仅查看包含 "error" 的日志行
```

### 获取日志并写入文件

```bash
docker logs <container_id> > output.log  # 导出日志到本地文件
```

### 查看所有容器日志（包括已停止的）

```bash
docker ps -a --format '{{.ID}}' | xargs -I {} docker logs {}
```

（适用于批量检查多个容器的日志）

如果你的 Docker 容器运行的是一个服务（如 Flask 或 FastAPI），也可以进入容器后检查应用程序的日志：

```bash
docker exec -it <container_id> /bin/bash
cd /var/log  # 或者应用日志存放目录
cat application.log  # 查看应用日志
```

如果你在 `docker-compose` 环境下运行：

```bash
docker-compose logs -f  # 查看所有服务的日志
docker-compose logs -f <service_name>  # 查看指定服务的日志
```

### 镜像管理

```bash
# 搜索镜像
docker search <image_name>

# 拉取镜像
docker pull <image_name>:<tag>  # 例如：docker pull ubuntu:latest

# 列出本地镜像
docker images

# 删除镜像
docker rmi <image_id>  # 或 docker rmi <image_name>:<tag>

# 强制删除（如果有容器依赖）
docker rmi -f <image_id>
```

### 容器管理

```bash
# 运行容器（前台模式）
docker run -it --name <container_name> <image_name>:<tag> bash

# 运行容器（后台模式）
docker run -dit --name <container_name> <image_name>:<tag>

# 显示正在运行的容器
docker ps

# 显示所有容器（包括已停止）
docker ps -a

# 进入正在运行的容器
docker exec -it <container_id> /bin/bash

# 停止容器
docker stop <container_id>

# 启动已停止的容器
docker start <container_id>

# 删除容器
docker rm <container_id>

# 强制删除（如果容器正在运行）
docker rm -f <container_id>
```

### 挂载目录与端口映射

```bash
# 挂载本地目录到容器
docker run -it -v /local/path:/container/path <image_name> bash

# 端口映射（宿主机:容器）
docker run -it -p 8080:80 <image_name>
```

### 容器与镜像管理（进阶）

```bash
# 提交容器为新镜像
docker commit <container_id> <new_image_name>:<tag>

# 复制文件到容器
docker cp <local_file> <container_id>:<container_path>

# 复制文件从容器到宿主机
docker cp <container_id>:<container_path> <local_file>
```

### 网络管理

```bash
# 查看 Docker 网络
docker network ls

# 创建 Docker 网络
docker network create <network_name>

# 让容器加入指定网络
docker network connect <network_name> <container_name>

# 从网络中移除容器
docker network disconnect <network_name> <container_name>
```