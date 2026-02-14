# SQLBot 本地运行与调试指南（开发环境）

本文档面向本地开发调试，推荐使用：
- 前后端源码本地运行
- PostgreSQL 使用 Docker 单独启动

## 1. 环境要求

- Git
- Node.js 18+
- Python 3.11
- `uv`（Python 依赖管理）
- Docker（仅用于启动 PostgreSQL）

## 2. 启动 PostgreSQL

在任意终端执行：

```powershell
docker run -d --name sqlbot-pg `
  -e POSTGRES_DB=sqlbot `
  -e POSTGRES_USER=root `
  -e POSTGRES_PASSWORD=Password123@pg `
  -p 5432:5432 postgres:16
```

如容器已存在，可直接启动：

```powershell
docker start sqlbot-pg
```

## 3. 启动后端（FastAPI）

在 `backend` 目录执行：

```powershell
cd backend
uv sync --extra cpu
uv run uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

说明：
- 后端启动时会自动执行 Alembic 迁移（`main.py` 的 `lifespan` 中触发）。
- API 文档地址：`http://localhost:8000/docs`

## 4. 启动前端（Vue 3 + Vite）

新开终端，在 `frontend` 目录执行：

```powershell
cd frontend
npm install
npm run dev
```

默认前端地址：
- `http://localhost:5173`

前端开发环境 API 基地址（已在仓库中配置）：
- `frontend/.env.development` -> `VITE_API_BASE_URL=http://localhost:8000/api/v1`

## 5. 可选服务

### 5.1 MCP 服务（端口 8001）

新开终端：

```powershell
cd backend
uv run uvicorn main:mcp_app --reload --host 0.0.0.0 --port 8001
```

### 5.2 图表 SSR 服务（端口 3000）

新开终端：

```powershell
cd g2-ssr
npm install
node app.js
```

## 6. 默认登录账号

- 用户名：`admin`
- 密码：`SQLBot@123456`

## 7. 推荐调试方式

### 7.1 后端断点（VS Code / PyCharm）

- 入口模块：`backend/main.py`
- 调试目标：`main:app`
- 常用断点位置：
  - 路由入口：`backend/apps/*/api/*.py`
  - 业务逻辑：`backend/apps/*/crud/*.py`
  - 数据访问：`backend/apps/db/*.py`

### 7.2 前端断点

- 直接在浏览器 DevTools 调试（Sources）
- 或在 VS Code 使用 JavaScript Debugger 附加 Vite 页面

## 8. 常见问题排查

1. 前端 401 / 无法登录
- 检查后端是否正常启动：`http://localhost:8000/docs`
- 检查前端 API 地址是否正确：`frontend/.env.development`

2. 数据库连接失败
- 检查 PostgreSQL 容器状态：`docker ps`
- 检查端口 `5432` 是否被占用
- 检查数据库账号密码是否与后端配置一致（默认 `root / Password123@pg`）

3. 启动后端报 Alembic 相关错误
- 确保命令在 `backend` 目录执行
- 先手动执行一次：
  ```powershell
  cd backend
  uv run alembic upgrade head
  ```

4. Windows 本机安装某些数据库驱动失败
- 建议后端改在 WSL2 Ubuntu 运行，通常更稳定

## 9. 一键本地最小启动顺序（建议）

1. 启动 PostgreSQL 容器
2. 启动后端 `main:app`
3. 启动前端 `vite`
4. 需要 MCP 时再启动 `main:mcp_app`
5. 需要图表服务时再启动 `g2-ssr`
