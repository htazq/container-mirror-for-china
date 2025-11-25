# 容器镜像加速服务

一个简单的 Docker 镜像代理服务，帮助国内开发者加速容器镜像拉取。

## 支持的镜像源

| 镜像源 | 域名 |
|--------|------|
| Docker Hub | registry-1.docker.io |
| Google Container Registry | gcr.io |
| Kubernetes (旧) | k8s.gcr.io |
| Kubernetes | registry.k8s.io |
| Red Hat Quay | quay.io |
| GitHub Container Registry | ghcr.io |
| Microsoft Container Registry | mcr.microsoft.com |
| NVIDIA GPU Cloud | nvcr.io |
| Elastic | docker.elastic.co |
| GitLab | registry.gitlab.com |

## 快速部署

### 前置要求
- Docker 和 Docker Compose
- 一台海外 VPS（推荐美国）
- 一个域名（可选）

### 部署步骤

1. **克隆项目到服务器**

使用 Git Clone（推荐）：
```bash
cd /opt
git clone https://github.com/htazq/container-mirror-for-china.git docker-mirror
cd docker-mirror
```

或者下载 ZIP 包：
```bash
cd /opt
wget https://github.com/htazq/container-mirror-for-china/archive/refs/heads/main.zip
unzip main.zip
mv container-mirror-for-china-main docker-mirror
cd docker-mirror
```

2. **修改域名（如果有自己的域名）**
```bash
# 将 docker.at9.net 替换为你的域名
sed -i 's/docker.at9.net/your-domain.com/g' config/nginx.conf config/index.html
```

3. **启动服务**
```bash
docker-compose up -d
```

4. **查看服务状态**
```bash
docker-compose ps
docker-compose logs -f
```

### 配置 SSL（推荐）

```bash
# 安装 certbot
apt update && apt install -y certbot

# 停止 nginx
docker-compose stop nginx

# 获取证书（替换为你的域名）
certbot certonly --standalone -d your-domain.com

# 复制证书
mkdir -p ssl
cp /etc/letsencrypt/live/your-domain.com/fullchain.pem ssl/
cp /etc/letsencrypt/live/your-domain.com/privkey.pem ssl/

# 修改 nginx.conf，取消 HTTPS server 块的注释

# 重启服务
docker-compose up -d
```

## 使用方法

### 方式一：直接拉取（推荐）

显式指定源（推荐，明确知道从哪里拉取）：
```bash
# Docker Hub
docker pull your-domain.com/docker.io/library/nginx:latest

# Google GCR
docker pull your-domain.com/gcr.io/distroless/static-debian11

# Kubernetes
docker pull your-domain.com/registry.k8s.io/pause:3.9

# Quay
docker pull your-domain.com/quay.io/prometheus/prometheus:latest

# GitHub
docker pull your-domain.com/ghcr.io/project-zot/zot-linux-amd64:latest

# Microsoft
docker pull your-domain.com/mcr.microsoft.com/dotnet/runtime:8.0

# NVIDIA
docker pull your-domain.com/nvcr.io/nvidia/cuda:12.0.0-base

# Elastic
docker pull your-domain.com/docker.elastic.co/elasticsearch/elasticsearch:8.11.0

# GitLab
docker pull your-domain.com/registry.gitlab.com/gitlab-org/gitlab-runner:latest
```

隐式拉取（自动匹配 Docker Hub）：
```bash
docker pull your-domain.com/nginx:latest
```

### 方式二：配置为镜像源

**Docker 配置** (`/etc/docker/daemon.json`):
```json
{
  "registry-mirrors": ["https://your-domain.com"]
}
```

重启 Docker:
```bash
sudo systemctl restart docker
```

**Kubernetes (containerd) 配置** (`/etc/containerd/config.toml`):
```toml
[plugins."io.containerd.grpc.v1.cri".registry.mirrors]
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
    endpoint = ["https://your-domain.com"]
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors."gcr.io"]
    endpoint = ["https://your-domain.com"]
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors."registry.k8s.io"]
    endpoint = ["https://your-domain.com"]
```

重启 containerd:
```bash
sudo systemctl restart containerd
```

## 常用管理命令

```bash
# 启动
docker-compose up -d

# 停止
docker-compose down

# 重启
docker-compose restart

# 查看日志
docker-compose logs -f

# 查看状态
docker-compose ps

# 更新服务
docker-compose pull
docker-compose up -d
```

## 工作原理

- **Zot**：开源的容器镜像代理，按需拉取并缓存镜像
- **Nginx**：前端代理，处理路由和提供首页
- **按需缓存**：首次拉取镜像时从上游源获取，之后使用缓存
- **自动清理**：定期 GC 清理过期镜像，节省磁盘空间

## 故障排查

```bash
# 检查服务是否运行
docker-compose ps

# 查看 Zot 日志
docker-compose logs zot

# 查看 Nginx 日志
docker-compose logs nginx

# 测试服务
curl http://localhost/health
curl http://localhost/v2/
```

## 项目结构

```
.
├── docker-compose.yml          # Docker Compose 配置
├── config/
│   ├── zot-config.json        # Zot 配置
│   ├── nginx.conf             # Nginx 配置
│   └── index.html             # 首页
├── ssl/                       # SSL 证书（可选）
└── README.md
```

---

**为云原生进步一起加油！**
