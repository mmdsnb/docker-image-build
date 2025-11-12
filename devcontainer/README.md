# DevContainer 开发环境镜像

基于 Ubuntu 22.04 的全栈开发容器镜像，支持 Docker-in-Docker、多语言开发环境。

**使用 GitHub Actions 构建，解决本地网络和多平台构建问题。**

## 🎯 特性

### 核心功能
- ✅ **Docker-in-Docker**: 容器内可以运行 Docker
- ✅ **多架构支持**: x86_64 和 ARM64（GitHub Actions 自动构建）
- ✅ **多语言环境**: Python、Node.js、Java
- ✅ **现代工具**: uv、nvm、SDKMAN
- ✅ **SSH 支持**: 密钥认证（安全）
- ✅ **Zsh + Oh My Zsh**: 增强的终端体验

### 预装工具

#### 系统工具
- Git、Vim、curl、wget
- htop、jq
- 网络工具：ping、telnet、netcat

#### 容器工具
- Docker (latest)
- Docker Compose v2.24.0

#### 开发环境
- **Python 3**: python3 + pip + uv
- **Node.js**: nvm (版本管理器)
- **Java/JVM**: SDKMAN (版本管理器)

#### Shell
- Zsh + Oh My Zsh
- 插件：autosuggestions、syntax-highlighting
- 预设别名：d, dc, dps, di, ll

## 🚀 构建方式

### 使用 GitHub Actions 构建（推荐）

1. 进入 GitHub 仓库的 `Actions` 页面
2. 选择 `Build and Push Docker Image`
3. 点击 `Run workflow`
4. 输入镜像名称：`devcontainer`
5. 等待构建完成（约 10-15 分钟）

构建完成后，镜像会自动推送到阿里云：
```
registry.cn-hangzhou.aliyuncs.com/your_namespace/devcontainer:latest
```

### 本地构建（可选）

如果需要本地构建：

```bash
cd devcontainer
docker build -t devcontainer:latest .
```

## 📦 使用方式

### 从阿里云拉取镜像

```bash
# 拉取最新版本
docker pull registry.cn-hangzhou.aliyuncs.com/your_namespace/devcontainer:latest

# 运行容器
docker run -it --privileged --network=host \
  --name my-dev \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v $(pwd):/workspace \
  registry.cn-hangzhou.aliyuncs.com/your_namespace/devcontainer:latest
```

### 使用 Docker Compose

创建 `docker-compose.yml`：

```yaml
version: '3.8'

services:
  devcontainer:
    image: registry.cn-hangzhou.aliyuncs.com/your_namespace/devcontainer:latest
    container_name: devcontainer
    privileged: true
    network_mode: host
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./:/workspace
    stdin_open: true
    tty: true
```

启动：
```bash
docker-compose up -d
docker-compose exec devcontainer zsh
```

## 🔧 使用说明

### 容器启动后

容器默认启动 Zsh shell，**需要手动启动服务**：

```bash
# 启动 Docker daemon（后台运行）
dockerd > /var/log/dockerd.log 2>&1 &

# 等待 Docker 启动
sleep 3
docker --version

# 启动 SSH 服务（可选）
service ssh start
```

**推荐方式**：长期运行容器，服务启动一次即可。

### 安装 Node.js 版本

```bash
# 安装 LTS 版本
nvm install --lts

# 安装特定版本
nvm install 20

# 设置默认版本
nvm alias default 20
```

### 安装 Java/Kotlin/Gradle

```bash
# 安装 Java
sdk install java 17.0.9-tem

# 安装 Gradle
sdk install gradle

# 安装 Kotlin
sdk install kotlin
```

### 使用 uv 管理 Python 项目

```bash
# 创建虚拟环境
uv venv

# 安装依赖
uv pip install fastapi uvicorn

# 运行脚本
uv run script.py
```

### Docker 命令别名

容器内预设了便捷别名：

```bash
d       # docker
dc      # docker-compose
dps     # docker ps
di      # docker images
ll      # ls -lah
```

## 🔐 SSH 配置

容器已生成 SSH 密钥，位于 `/root/.ssh/`：
- `id_ed25519` - Ed25519 密钥（推荐用于 GitHub）
- `id_rsa` - RSA 4096 密钥

### 查看公钥

```bash
# 进入容器
docker exec -it my-dev zsh

# 查看公钥
cat ~/.ssh/id_ed25519.pub
```

### 添加到 GitHub

```bash
# 复制公钥内容
cat ~/.ssh/id_ed25519.pub

# 添加到 GitHub Settings > SSH and GPG keys > New SSH key
```

## 📊 镜像源配置

已配置国内镜像源加速：

- **Ubuntu APT**: 阿里云
- **Python pip**: 阿里云
- **Python uv**: 阿里云

## 🔄 更新镜像

当 Dockerfile 更新后：

1. 推送代码到 GitHub
2. 在 Actions 中重新构建 `devcontainer` 镜像
3. 本地重新拉取：
   ```bash
   docker pull registry.cn-hangzhou.aliyuncs.com/your_namespace/devcontainer:latest
   ```

## 💡 为什么使用 GitHub Actions 构建？

### 优势
- ✅ **网络快**: GitHub 服务器网络好，下载速度快
- ✅ **多平台**: 自动构建 x86_64 和 ARM64 两种架构
- ✅ **省资源**: 不占用本地机器资源
- ✅ **可重现**: 构建过程完全一致

### 构建时间
- 首次构建：约 10-15 分钟
- 后续构建：利用缓存，约 5-8 分钟

## 🐛 常见问题

### Docker 无法启动

```bash
# 检查日志
cat /var/log/dockerd.log

# 手动启动
dockerd > /var/log/dockerd.log 2>&1 &

# 检查状态
docker ps
```

### SSH 服务无法启动

```bash
# 检查状态
service ssh status

# 启动 SSH
service ssh start

# 查看公钥
cat ~/.ssh/id_ed25519.pub
```

### 权限问题

```bash
# 确保使用 --privileged 参数运行容器
docker run -it --privileged ...

# 检查 Docker socket 是否挂载
ls -la /var/run/docker.sock
```

### 镜像拉取失败

```bash
# 检查镜像名称是否正确
# 确保已在 GitHub 中构建完成
# 检查阿里云镜像仓库配置
```

## 🎨 自定义配置

### 修改时区

编辑 `Dockerfile`:

```dockerfile
ENV TZ=America/New_York
```

### 添加更多工具

在 `Dockerfile` 对应阶段添加：

```dockerfile
# 阶段 2: 安装系统包
RUN apt-get install -y \
    your-package-here
```

然后推送到 GitHub，重新构建镜像。

## 📝 目录结构

```
devcontainer/
├── Dockerfile       # 镜像构建文件（无代理配置）
└── README.md        # 本文档
```

## 🚀 快速开始完整流程

1. **首次使用**：在 GitHub Actions 构建 `devcontainer` 镜像

2. **拉取镜像**：
   ```bash
   docker pull registry.cn-hangzhou.aliyuncs.com/your_namespace/devcontainer:latest
   ```

3. **运行容器**（长期运行）：
   ```bash
   docker run -it --privileged --network=host \
     -v /var/run/docker.sock:/var/run/docker.sock \
     -v $(pwd):/workspace \
     --name dev \
     registry.cn-hangzhou.aliyuncs.com/your_namespace/devcontainer:latest
   ```

4. **启动服务**：
   ```bash
   # 启动 Docker
   dockerd > /var/log/dockerd.log 2>&1 &

   # 启动 SSH（可选）
   service ssh start
   ```

5. **开始开发**！

### 后续使用

容器长期运行，重新连接时：

```bash
# 进入已存在的容器
docker exec -it dev zsh

# 或者重新启动已停止的容器
docker start -ai dev
```

## 📚 相关文档

- [Docker-in-Docker](https://github.com/jpetazzo/dind)
- [Oh My Zsh](https://ohmyz.sh/)
- [uv Documentation](https://github.com/astral-sh/uv)
- [nvm](https://github.com/nvm-sh/nvm)
- [SDKMAN](https://sdkman.io/)
