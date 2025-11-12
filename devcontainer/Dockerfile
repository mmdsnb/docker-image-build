# 开发容器 Dockerfile
# 基于 Ubuntu 22.04 LTS 的 Docker-in-Docker 全栈开发环境

FROM ubuntu:22.04

# 避免交互式安装提示
ENV DEBIAN_FRONTEND=noninteractive \
    TZ=Asia/Shanghai \
    # 工具版本定义
    DOCKER_COMPOSE_VERSION=v2.24.0 \
    NVM_VERSION=v0.39.7 \
    # 目录定义
    NVM_DIR=/root/.nvm \
    SDKMAN_DIR=/root/.sdkman

# ============================================
# 阶段 1: 系统基础配置
# ============================================

# 配置阿里云镜像源（多架构自动适配）
RUN cp /etc/apt/sources.list /etc/apt/sources.list.backup && \
    ARCH=$(dpkg --print-architecture) && \
    if [ "$ARCH" = "amd64" ] || [ "$ARCH" = "i386" ]; then \
        UBUNTU_REPO="ubuntu"; \
    else \
        UBUNTU_REPO="ubuntu-ports"; \
    fi && \
    echo "检测到架构: $ARCH, 使用源: $UBUNTU_REPO" && \
    cat > /etc/apt/sources.list <<EOF
deb http://mirrors.aliyun.com/$UBUNTU_REPO/ jammy main restricted universe multiverse
deb http://mirrors.aliyun.com/$UBUNTU_REPO/ jammy-security main restricted universe multiverse
deb http://mirrors.aliyun.com/$UBUNTU_REPO/ jammy-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/$UBUNTU_REPO/ jammy-backports main restricted universe multiverse
EOF

# ============================================
# 阶段 2: 安装系统包和基础工具
# ============================================

RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y \
        # 基础工具
        curl \
        wget \
        zip \
        unzip \
        tar \
        vim \
        git \
        build-essential \
        make \
        software-properties-common \
        ca-certificates \
        gnupg \
        lsb-release \
        # 网络工具
        net-tools \
        iputils-ping \
        telnet \
        netcat-openbsd \
        # 系统监控
        htop \
        procps \
        # JSON 处理
        jq \
        # Python 开发
        python3-pip \
        python3-dev \
        # SSH 服务
        openssh-server \
        # Shell
        zsh && \
    # 清理缓存
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# ============================================
# 阶段 3: 安装 Docker 和 Docker Compose
# ============================================

# 安装 Docker（使用官方脚本）
RUN curl -fsSL https://get.docker.com | sh

# 安装 Docker Compose（固定版本）
RUN ARCH=$(uname -m) && \
    if [ "$ARCH" = "x86_64" ]; then \
        COMPOSE_ARCH="x86_64"; \
    elif [ "$ARCH" = "aarch64" ]; then \
        COMPOSE_ARCH="aarch64"; \
    else \
        COMPOSE_ARCH=$ARCH; \
    fi && \
    curl -L "https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-linux-${COMPOSE_ARCH}" \
        -o /usr/local/bin/docker-compose && \
    chmod +x /usr/local/bin/docker-compose && \
    docker-compose --version

# ============================================
# 阶段 4: 配置 Python 环境
# ============================================

# 创建 python 软链接
RUN ln -sf /usr/bin/python3 /usr/bin/python && \
    python --version

# 配置 pip 镜像源
RUN mkdir -p /root/.pip && \
    cat > /root/.pip/pip.conf <<'EOF'
[global]
index-url = http://mirrors.aliyun.com/pypi/simple/
trusted-host = mirrors.aliyun.com
EOF

# 安装 uv（现代 Python 包管理器）
RUN pip3 install --no-cache-dir uv && \
    uv --version

# 配置 uv 镜像源
RUN mkdir -p /root/.config/uv && \
    cat > /root/.config/uv/uv.toml <<'EOF'
[index]
url = "http://mirrors.aliyun.com/pypi/simple/"

[[index.trusted-host]]
host = "mirrors.aliyun.com"
EOF

# ============================================
# 阶段 5: 安装 Node.js 和 Java 版本管理器
# ============================================

# 安装 nvm（Node.js 版本管理器）
RUN curl -o- "https://raw.githubusercontent.com/nvm-sh/nvm/${NVM_VERSION}/install.sh" | bash && \
    # 验证安装
    . "$NVM_DIR/nvm.sh" && nvm --version

# 安装 SDKMAN（Java/JVM 工具版本管理器）
RUN curl -s "https://get.sdkman.io" | bash && \
    # 验证安装
    bash -c "source $SDKMAN_DIR/bin/sdkman-init.sh && sdk version"

# ============================================
# 阶段 6: 配置 Zsh 和 Oh My Zsh
# ============================================

# 设置 zsh 为默认 shell
RUN chsh -s /usr/bin/zsh root

# 安装 Oh My Zsh
RUN sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended

# 安装 zsh 插件
RUN git clone --depth=1 https://github.com/zsh-users/zsh-autosuggestions \
        ${ZSH_CUSTOM:-/root/.oh-my-zsh/custom}/plugins/zsh-autosuggestions && \
    git clone --depth=1 https://github.com/zsh-users/zsh-syntax-highlighting \
        ${ZSH_CUSTOM:-/root/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# 配置 Oh My Zsh 插件
RUN sed -i 's/plugins=(git)/plugins=(git docker docker-compose python pip extract z colored-man-pages command-not-found zsh-autosuggestions zsh-syntax-highlighting)/g' /root/.zshrc

# 配置 zsh 环境变量
RUN cat >> /root/.zshrc <<'EOF'

# ============================================
# 开发环境配置
# ============================================

# NVM（Node.js 版本管理）
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"

# SDKMAN（JVM 工具版本管理）
export SDKMAN_DIR="/root/.sdkman"
[[ -s "/root/.sdkman/bin/sdkman-init.sh" ]] && source "/root/.sdkman/bin/sdkman-init.sh"

# Python uv 工具
export PATH="$HOME/.local/bin:$PATH"

# Docker 别名
alias d='docker'
alias dc='docker-compose'
alias dps='docker ps'
alias di='docker images'

# 常用别名
alias ll='ls -lah'
alias ..='cd ..'
alias ...='cd ../..'

# 欢迎信息
echo "🚀 开发环境已就绪！"
echo "📦 Docker: $(docker --version 2>/dev/null || echo 'Not running')"
echo "🐍 Python: $(python --version 2>/dev/null || echo 'N/A')"
echo "📊 可用命令: d (docker), dc (docker-compose), uv (python)"
EOF

# ============================================
# 阶段 7: 配置 SSH 服务
# ============================================

RUN mkdir -p /var/run/sshd /root/.ssh && \
    chmod 700 /root/.ssh && \
    # 生成 SSH 密钥
    ssh-keygen -t ed25519 -f /root/.ssh/id_ed25519 -N "" -C "devcontainer" && \
    ssh-keygen -t rsa -b 4096 -f /root/.ssh/id_rsa -N "" -C "devcontainer-rsa" && \
    # 创建 authorized_keys
    touch /root/.ssh/authorized_keys && \
    chmod 600 /root/.ssh/authorized_keys && \
    # 配置 SSH 服务（安全配置）
    sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin prohibit-password/' /etc/ssh/sshd_config && \
    sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config && \
    sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/' /etc/ssh/sshd_config && \
    sed -i 's/#AuthorizedKeysFile/AuthorizedKeysFile/' /etc/ssh/sshd_config && \
    # 禁用密码登录，只允许密钥认证
    echo "PasswordAuthentication no" >> /etc/ssh/sshd_config && \
    echo "ChallengeResponseAuthentication no" >> /etc/ssh/sshd_config

# 配置 SSH 客户端（Git 使用）
RUN cat > /root/.ssh/config <<'EOF'
Host *
    StrictHostKeyChecking accept-new
    ServerAliveInterval 60
    ServerAliveCountMax 3
EOF

RUN chmod 600 /root/.ssh/config

# ============================================
# 阶段 8: 安装 VSCode CLI (支持 Tunnel)
# ============================================

# 安装 VSCode CLI（支持远程隧道连接）
RUN ARCH=$(uname -m) && \
    if [ "$ARCH" = "x86_64" ]; then \
        VSCODE_ARCH="x64"; \
    elif [ "$ARCH" = "aarch64" ]; then \
        VSCODE_ARCH="arm64"; \
    else \
        echo "不支持的架构: $ARCH" && exit 1; \
    fi && \
    echo "下载 VSCode CLI for ${VSCODE_ARCH}..." && \
    curl -fSL "https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-${VSCODE_ARCH}" \
        --output /tmp/vscode-cli.tar.gz && \
    tar -xzf /tmp/vscode-cli.tar.gz -C /usr/local/bin && \
    rm /tmp/vscode-cli.tar.gz && \
    code --version && \
    echo "✅ VSCode CLI 安装成功"

# ============================================
# 阶段 9: 最终配置
# ============================================

# 创建工作目录
RUN mkdir -p /workspace

# 配置 Git
RUN git config --global init.defaultBranch main && \
    git config --global pull.rebase false && \
    git config --global core.editor vim

# 设置工作目录
WORKDIR /workspace

# 暴露端口
# 22: SSH
# 3000: React/Next.js 等前端开发
# 8000: Python/Django 等后端开发
# 8080: 通用 HTTP
# 9000: 其他服务
EXPOSE 22 3000 8000 8080 9000

# 默认启动 zsh shell
CMD ["/usr/bin/zsh"]
