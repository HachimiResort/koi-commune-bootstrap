# Koi Commune Bootstrap

`koi-commune-bootstrap` 是 Koi Commune 的集成部署仓库，通过 Git 子模块组合前后端，并使用 Docker Compose 启动完整的本地环境。

Koi Commune 面向高校课堂学习与知识互助场景，目前主要适配北京交通大学（BJTU）。项目支持课程同步、媒体上传与转写、AI 摘要与问答、笔记协作、社区讨论、积分及管理后台等功能。

## 仓库组成

- [koi-commune-backend](https://github.com/HachimiResort/koi-commune-backend)：Spring Boot 后端
- [koi-commune-frontend](https://github.com/HachimiResort/koi-commune-frontend)：Web 与微信小程序前端

两个子仓库均可独立开发和部署；本仓库仅负责固定兼容版本，并提供一键构建与联调环境。

```text
.
├── backend/              # 后端 Git 子模块
├── frontend/             # 前端 Git 子模块
├── docker-compose.yml    # PostgreSQL、Redis、后端与 Web 编排
├── Dockerfile.backend    # 后端镜像构建
├── Dockerfile.web        # Web 镜像及小程序产物构建
└── .env.example          # 根 compose 环境变量模板
```

## 快速启动

### 1. 克隆仓库

推荐直接初始化子模块：

```bash
git clone --recurse-submodules https://github.com/HachimiResort/koi-commune-bootstrap.git
cd koi-commune-bootstrap
```

如果已经克隆了本仓库：

```bash
git submodule update --init --recursive
```

### 2. 准备配置

```bash
cp .env.example .env
```

至少需要修改以下配置：

- `POSTGRES_PASSWORD`、`REDIS_PASSWORD`
- `JWT_SECRET`、`ADMIN_PASSWORD`
- `APP_AI_API_KEY_ENCRYPTION_KEY`
- `APP_MEDIA_MINIO_INTERNAL_ENDPOINT`
- `APP_MEDIA_MINIO_EXTERNAL_ENDPOINT`
- `APP_MEDIA_MINIO_ACCESS_KEY`、`APP_MEDIA_MINIO_SECRET_KEY`
- `APP_MEDIA_MINIO_BUCKET`

SMTP、阿里云听悟和 AI 模型服务按需配置；缺少这些配置时，对应的邮件、转写或 AI 功能不可用，但基础服务仍可启动。`.env` 包含敏感信息，已被 Git 忽略，请勿提交。

### 3. 构建并启动

```bash
docker compose up -d --build
```

首次启动需要下载镜像和依赖，耗时取决于网络环境。启动后可检查状态和日志：

```bash
docker compose ps
docker compose logs -f koi-backend
```

### 4. 访问服务

- Web：`http://127.0.0.1:3000`
- Backend API：`http://127.0.0.1:8080`
- Swagger UI：`http://127.0.0.1:8080/swagger-ui.html`
- 微信小程序构建产物：`./koi-commune-weapp-dist`

后端以 `dev` profile 启动时会初始化管理员账号：用户名 `admin`，邮箱 `admin@koimune.local`，密码由 `ADMIN_PASSWORD` 指定。若账号已存在，修改环境变量不会重置其密码。

### 5. 停止服务

```bash
docker compose down
```

需要同时删除 PostgreSQL 和 Redis 数据卷时使用：

```bash
docker compose down -v --remove-orphans
```

该命令会永久删除本仓库 Compose 创建的本地数据库与缓存数据。

## 对象存储

本仓库默认使用独立部署的 MinIO，不会额外启动 MinIO 容器：

- `APP_MEDIA_MINIO_INTERNAL_ENDPOINT` 供后端服务端上传使用，应能从后端容器访问。
- `APP_MEDIA_MINIO_EXTERNAL_ENDPOINT` 用于生成浏览器可访问的预签名 URL，生产环境应配置公网 HTTPS 地址。
- `APP_MEDIA_STORAGE_PUBLIC_BASE_URL` 可选；未设置时使用 external endpoint 生成公开媒体地址。
- Bucket 的媒体前缀需允许匿名读取，并配置浏览器直传所需的 CORS。

若启用阿里云听悟，公开媒体地址还必须能被听悟服务从公网访问；仅内网或 `127.0.0.1` 地址无法完成云端转写。

当前媒体处理默认在后端容器内完成并直接写入 MinIO，`APP_MEDIA_NAS_ENABLED=false`。腾讯云 COS 仍作为后端兼容选项保留，详细配置见 [后端 README](./backend/README.md)。

## 前端 API 地址

- Web 默认使用 `KOI_API_BASE_URL=/backend`，由 Web 容器反向代理到本地后端。
- 微信小程序在构建时读取 `KOI_WEAPP_API_BASE_URL`，默认值见 `.env.example`。

如需让小程序访问本机后端，应将 `KOI_WEAPP_API_BASE_URL` 设置为手机或微信开发者工具可访问的宿主机地址，然后重新构建：

```bash
docker compose build koi-web
docker compose up -d koi-web
```

## 更新子模块

拉取本仓库记录的前后端版本：

```bash
git pull
git submodule update --init --recursive
```

前后端的独立开发、环境配置、测试与生产部署方式，请分别阅读 [backend/README.md](./backend/README.md) 和 [frontend/README.md](./frontend/README.md)。

## 许可证

本项目采用 [MIT License](./LICENSE)。
