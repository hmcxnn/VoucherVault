# GitHub Container Registry 部署指南

本项目已配置 GitHub Actions 自动构建并推送 Docker 镜像到 GitHub Container Registry (GHCR)。

## 自动部署

### 触发条件

GitHub Action 会在以下情况下自动触发：

1. **推送到 main 分支** - 构建并推送 `latest` 标签
2. **创建版本标签** (如 `v1.0.0`) - 构建并推送版本标签
3. **Pull Request** - 仅构建，不推送

### 生成的镜像标签

- `ghcr.io/[username]/vouchervault:latest` - 最新版本
- `ghcr.io/[username]/vouchervault:main` - main 分支
- `ghcr.io/[username]/vouchervault:v1.0.0` - 具体版本
- `ghcr.io/[username]/vouchervault:1.0` - 主要.次要版本
- `ghcr.io/[username]/vouchervault:1` - 主要版本

## 使用镜像

### 拉取镜像

```bash
# 拉取最新版本
docker pull ghcr.io/[username]/vouchervault:latest

# 拉取特定版本
docker pull ghcr.io/[username]/vouchervault:v1.0.0
```

### 运行容器

```bash
# 使用 SQLite (简单部署)
docker run -d \
  --name vouchervault \
  -p 8000:8000 \
  -v $(pwd)/database:/opt/app/database \
  ghcr.io/[username]/vouchervault:latest

# 使用 PostgreSQL
docker run -d \
  --name vouchervault \
  -p 8000:8000 \
  -v $(pwd)/database:/opt/app/database \
  -e DB_ENGINE=postgres \
  -e POSTGRES_HOST=your-postgres-host \
  -e POSTGRES_DB=vouchervault \
  -e POSTGRES_USER=your-user \
  -e POSTGRES_PASSWORD=your-password \
  ghcr.io/[username]/vouchervault:latest
```

### 使用 Docker Compose

```yaml
version: '3.8'

services:
  app:
    image: ghcr.io/[username]/vouchervault:latest
    ports:
      - "8000:8000"
    volumes:
      - ./database:/opt/app/database
    environment:
      - DB_ENGINE=sqlite3
    restart: unless-stopped
```

## 权限设置

### 公开镜像

默认情况下，推送到 GHCR 的镜像是私有的。要使镜像公开可访问：

1. 访问 GitHub 仓库的 **Packages** 页面
2. 点击你的包名称
3. 在右侧点击 **Package settings**
4. 滚动到 **Danger Zone**
5. 点击 **Change visibility**
6. 选择 **Public**

### 访问私有镜像

如果镜像是私有的，需要先登录：

```bash
# 创建 Personal Access Token (需要 read:packages 权限)
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin

# 然后拉取镜像
docker pull ghcr.io/[username]/vouchervault:latest
```

## 本地构建

如果需要本地构建镜像：

```bash
# 克隆仓库
git clone https://github.com/[username]/VoucherVault.git
cd VoucherVault

# 构建镜像
docker build -f docker/Dockerfile -t vouchervault:local .

# 运行
docker run -d -p 8000:8000 -v $(pwd)/database:/opt/app/database vouchervault:local
```

## 多架构支持

构建的镜像支持以下架构：
- `linux/amd64` (x86_64)
- `linux/arm64` (ARM64/Apple Silicon)

Docker 会自动选择适合你系统的架构。

## 故障排除

### 构建失败

1. 检查 GitHub Actions 日志
2. 确保 Dockerfile 语法正确
3. 验证所有依赖文件存在

### 推送失败

1. 检查仓库的 **Actions** 权限设置
2. 确保 `GITHUB_TOKEN` 有 `packages:write` 权限
3. 验证仓库名称和用户名正确

### 镜像拉取失败

1. 检查镜像名称和标签是否正确
2. 如果是私有镜像，确保已正确登录
3. 验证网络连接

## 环境变量

容器支持以下环境变量：

- `DB_ENGINE`: 数据库引擎 (`sqlite3` 或 `postgres`)
- `POSTGRES_HOST`: PostgreSQL 主机地址
- `POSTGRES_DB`: 数据库名称
- `POSTGRES_USER`: 数据库用户名
- `POSTGRES_PASSWORD`: 数据库密码
- `POSTGRES_PORT`: 数据库端口 (默认 5432)

## 默认凭据

首次运行时，系统会创建默认管理员账户：
- **用户名**: `admin`
- **密码**: `admin`
- **邮箱**: `admin@example.com`

**重要**: 首次登录后请立即更改默认密码！