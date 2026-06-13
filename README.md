# koi-commune-bootstrap

Koi Commune 的课程验收一键启动仓库。

本仓库用于把 `backend` 和 `frontend` 两个子模块编排成一套可直接验收的 Docker 环境，目标是降低老师检查成本，而不是提供通用本地开发模板。

- 默认启动 Web、后端、PostgreSQL、Redis
- PostgreSQL 和 Redis 只在 Docker 内部网络开放，不映射宿主机端口
- 如需单独调试后端，请使用后端仓库自带的 `docker compose`

## 验收运行方式

1. 初始化子模块：

```bash
git submodule update --init --recursive
```

2. 将提交者单独提供的 `.env` 放到仓库根目录。

如果暂时没有验收用 `.env`，可先复制模板再自行补全：

```bash
cp .env.example .env
```

3. 启动整套服务：

```bash
docker compose up -d --build
```

如果在全局代理环境下执行 `docker compose up -d --build` 时出现镜像拉取很慢、长时间卡住或部分基础镜像拉不下来的情况，通常可以先在终端单独预拉取所需镜像，再重新执行上面的启动命令：

```bash
docker pull postgres:16-alpine
docker pull redis:7-alpine
docker pull eclipse-temurin:17-jdk-jammy
docker pull eclipse-temurin:17-jre-jammy
docker pull node:20-bookworm-slim
docker pull nginx:1.30.1-alpine
```

4. 查看状态：

```bash
docker compose ps
```

5. 查看后端日志：

```bash
docker compose logs -f koi-backend
```

6. 停止服务：

```bash
docker compose down
```

## 访问地址

- Web: `http://127.0.0.1:3000`
- Backend API: `http://127.0.0.1:8080`
- Weapp 导出目录: `./koi-commune-weapp-dist`

## 小程序后端地址说明

Web 构建产物默认通过 `KOI_API_BASE_URL=/backend` 访问本地 Docker 后端。

微信小程序导出产物默认访问正式后端：

`https://api.koimune.com`

这是刻意设计，用于降低验收成本。老师导入小程序后无需再配置宿主机局域网地址，也不需要保证手机或微信开发者工具能够访问本机 Docker 网络。

如果微信开发者工具中出现网络请求超时、域名校验失败或 HTTPS 证书相关报错，可在右上角“详情”或“本地设置”中勾选“不校验合法域名、web-view（业务域名）、TLS 版本以及 HTTPS 证书”后重新编译。

如需让小程序改连本地后端，可在构建前覆盖：

```bash
KOI_WEAPP_API_BASE_URL=http://<宿主机局域网 IP>:8080
```

## 环境变量说明

`.env.example` 仅用于说明变量含义，不代表验收时必须由老师逐项填写。

验收时会随项目单独提供 `.env`，其中包含课程检查所需的临时数据库密码、管理员密码、COS 配置和可选 AI 模型配置。这些密钥仅用于验收，不用于生产环境，也不应提交到公开仓库。

本仓库不提供宿主机直连 PostgreSQL / Redis 的开发模式。这样设计是为了避免占用老师本机的 `5432` / `6379` 端口，并减少本机已有数据库服务对验收结果的干扰。

## 管理员账号

根 compose 默认以 `dev` 配置启动后端，首次启动时会自动创建管理员账号。

- 用户名: `admin`
- 邮箱: `admin@koimune.local`
- 密码: `.env` 中的 `ADMIN_PASSWORD`

如果数据库中已经存在该账号，再修改 `.env` 不会覆盖旧密码。

## 重置数据

如需清空 PostgreSQL、Redis 和本地导出数据，可执行：

```bash
docker compose down -v --remove-orphans
docker compose up -d --build
```
