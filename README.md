# Zot 公益镜像仓库

🚀 **高速、稳定、免费的容器镜像加速服务** - `docker.at9.net`

> 🎉 **最新更新**: 支持 **7 个主流容器仓库**，包括 Docker Hub、GitHub、Microsoft 等

## 🌐 支持的镜像源

| 原始仓库 | 说明 |
|---------|------|
| **Docker Hub** | Docker 官方镜像库 |
| **gcr.io** | Google Container Registry |
| **registry.k8s.io** | Kubernetes 官方镜像 |
| **k8s.gcr.io** | Kubernetes 镜像（兼容） |
| **quay.io** | Red Hat Quay 镜像仓库 |
| **ghcr.io** | GitHub Container Registry 🆕 |
| **mcr.microsoft.com** | Microsoft Container Registry 🆕 |

## 🚀 使用方法

### 基本用法
```bash
# 原理：在任何镜像前加 docker.at9.net/
docker pull docker.at9.net/{原始镜像路径}
```

### 经过验证的可用镜像
```bash
# Docker Hub 镜像
docker pull docker.at9.net/nginx:latest
docker pull docker.at9.net/node:18-alpine
docker pull docker.at9.net/mysql:8.0
docker pull docker.at9.net/redis:7-alpine
docker pull docker.at9.net/tomcat:latest

# Google Container Registry
docker pull docker.at9.net/distroless/static-debian11:latest

# Microsoft Container Registry
docker pull docker.at9.net/dotnet/runtime:8.0
docker pull docker.at9.net/dotnet/sdk:8.0

# Quay.io
docker pull docker.at9.net/prometheus/prometheus:latest

# Kubernetes
docker pull docker.at9.net/pause:3.9
docker pull docker.at9.net/coredns/coredns:v1.10.1
```

## ⚙️ 配置镜像源（推荐）

### Docker 配置
编辑 `/etc/docker/daemon.json`:
```json
{
  "registry-mirrors": ["https://docker.at9.net"]
}
```

### Kubernetes containerd 配置
编辑 `/etc/containerd/config.toml`:
```toml
[plugins."io.containerd.grpc.v1.cri".registry]
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
      endpoint = ["https://docker.at9.net"]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."registry.k8s.io"]  
      endpoint = ["https://docker.at9.net"]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."gcr.io"]
      endpoint = ["https://docker.at9.net"]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."quay.io"]
      endpoint = ["https://docker.at9.net"]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."ghcr.io"]
      endpoint = ["https://docker.at9.net"]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."mcr.microsoft.com"]
      endpoint = ["https://docker.at9.net"]
```

### K3s 配置
创建 `/etc/rancher/k3s/registries.yaml`:
```yaml
mirrors:
  docker.io:
    endpoint: ["https://docker.at9.net"]
  registry.k8s.io:
    endpoint: ["https://docker.at9.net"]
  gcr.io:
    endpoint: ["https://docker.at9.net"]
  quay.io:
    endpoint: ["https://docker.at9.net"]
  ghcr.io:
    endpoint: ["https://docker.at9.net"]
  mcr.microsoft.com:
    endpoint: ["https://docker.at9.net"]
```

## 📊 服务状态

- **Web 界面**: [https://docker.at9.net](https://docker.at9.net)
- **API 检查**: `curl https://docker.at9.net/v2/`
- **已缓存镜像**: `curl https://docker.at9.net/v2/_catalog`

## 🔧 工作原理

**Zot 智能路由机制**:

1. **按需同步**: 首次请求时自动从 7 个上游仓库拉取
2. **智能匹配**: 自动尝试各个仓库，找到第一个可用源
3. **本地缓存**: 后续访问直接从本地高速返回
4. **内容去重**: 相同层只存储一份，节省空间
5. **自动清理**: 定期垃圾回收，保持系统健康

**配置示例**（简化）:
```json
{
  "extensions": {
    "sync": {
      "registries": [
        {"urls": ["https://registry-1.docker.io"]},
        {"urls": ["https://gcr.io"]},
        {"urls": ["https://registry.k8s.io"]},
        {"urls": ["https://quay.io"]},
        {"urls": ["https://ghcr.io"]},
        {"urls": ["https://mcr.microsoft.com"]}
      ]
    }
  }
}
```

**路由决策流程**:
```
用户请求 → 本地检查 → 缓存命中？
    ↓ 否
按顺序尝试上游 → 第一个成功 → 缓存 → 返回
```

## ❓ 常见问题

- **第一次慢?** 需要从上游同步，后续极速
- **某些镜像失败?** GitHub/私有镜像可能需要认证
- **如何验证?** 访问 [Web 界面](https://docker.at9.net) 确认可用性

## 📄 许可证

MIT License

---

**⭐ 如果有帮助，请给个 Star！**
