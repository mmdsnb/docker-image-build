# Docker 镜像构建工具

使用 GitHub Actions 自动构建多平台 Docker 镜像并推送到阿里云私人仓库。

## 功能特性

- ✅ 支持多镜像管理（每个镜像一个文件夹）
- ✅ 支持多平台构建（x86/amd64 + ARM64）
- ✅ **动态镜像发现**：无需修改 workflow，只需添加目录即可
- ✅ **自动验证**：构建前自动检查 Dockerfile 是否存在
- ✅ 手动触发构建，输入镜像名称即可
- ✅ 自动推送到阿里云镜像仓库
- ✅ 利用 GitHub Actions 的网络和构建资源

## 项目结构

```
.
├── helloworld/
│   └── Dockerfile          # Hello World 镜像的 Dockerfile
└── .github/
    └── workflows/
        └── build-docker.yml # GitHub Actions 工作流
```

## 配置步骤

### 1. 配置 GitHub Secrets

在你的 GitHub 仓库中设置以下 Secrets：

进入：`Settings` → `Secrets and variables` → `Actions` → `New repository secret`

需要添加的 Secrets：

| Secret 名称 | 说明 | 示例 |
|------------|------|------|
| `ALIYUN_USERNAME` | 阿里云镜像仓库用户名 | `your_username` |
| `ALIYUN_PASSWORD` | 阿里云镜像仓库密码 | `your_password` |
| `ALIYUN_NAMESPACE` | 阿里云命名空间 | `your_namespace` |

### 2. 添加新镜像

要添加新镜像，只需创建新文件夹并添加 Dockerfile，**无需修改 workflow 文件**：

```bash
mkdir myimage
cat > myimage/Dockerfile << 'EOF'
FROM alpine:latest
CMD ["echo", "Hello from My Image!"]
EOF
```

提交代码后，在 GitHub Actions 中输入镜像名称 `myimage` 即可构建！

## 使用方法

### 手动触发构建

1. 进入你的 GitHub 仓库
2. 点击 `Actions` 标签页
3. 选择 `Build and Push Docker Image` 工作流
4. 点击 `Run workflow` 按钮
5. 在输入框中输入要构建的镜像名称（如 `helloworld`）
6. 点击 `Run workflow` 确认

**自动验证**：
- Workflow 会自动检查输入的目录是否存在 Dockerfile
- 如果目录不存在或没有 Dockerfile，会列出所有可用的镜像目录

### 拉取镜像

构建完成后，可以从阿里云拉取镜像：

```bash
# 拉取最新版本
docker pull registry.cn-hangzhou.aliyuncs.com/your_namespace/helloworld:latest

# 拉取特定提交版本
docker pull registry.cn-hangzhou.aliyuncs.com/your_namespace/helloworld:commit_sha
```

## 镜像标签说明

每次构建会生成两个标签：

- `latest` - 最新版本
- `<commit_sha>` - 特定 Git 提交的版本

## 注意事项

- 构建多平台镜像需要时间，请耐心等待
- GitHub Actions 提供的网络速度通常比本地快
- 如果阿里云区域不是杭州，需要修改 `registry.cn-hangzhou.aliyuncs.com` 为对应区域
- 确保阿里云镜像仓库已创建对应的命名空间

## 支持的平台

- `linux/amd64` (x86_64)
- `linux/arm64` (ARM64/aarch64)

如需添加更多平台，可修改 workflow 中的 `platforms` 配置。
