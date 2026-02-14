# Project Context

## Purpose
SQLBot 是一个基于大语言模型（LLM）与 RAG 的智能问数系统（ChatBI）。目标是让用户通过自然语言完成数据查询、图表展示与分析，降低 SQL 使用门槛，并通过术语库、数据训练和历史反馈持续提升问答准确率。

## Tech Stack
- Backend: Python 3.11, FastAPI, Uvicorn, SQLModel/SQLAlchemy, Alembic
- AI/RAG: LangChain、LangGraph、LlamaIndex、Sentence Transformers、pgvector
- Frontend: Vue 3 + TypeScript, Vite, Pinia, Vue Router, Vue I18n, Element Plus
- Chart/Visualization: AntV G2/S2/X6，独立 `g2-ssr` Node 服务（PM2 启动）
- Data/Infra: PostgreSQL（默认主库）、Redis（可选缓存）、Docker/Docker Compose
- Protocol/Integration: FastAPI MCP（内置 MCP Server，默认 8001 端口）

## Project Conventions

### Code Style
- Python:
  - 使用 Ruff 做 lint + format，Mypy 开启 strict（`backend/pyproject.toml`）
  - 模块命名采用 `snake_case`，接口按 `apps/<domain>/{api,crud,models,schemas}` 分层
- Frontend:
  - ESLint + Prettier（单引号、无分号、2 空格缩进、`printWidth=100`）
  - Vue SFC 组件名以 PascalCase 为主，TS 逻辑尽量强类型
- 全仓库：
  - 启用 pre-commit（YAML/TOML 校验、行尾空格修复、ruff）
  - 变更优先小步提交，避免一次性混入重构和功能改动

### Architecture Patterns
- Monorepo 结构：`backend`（API 与业务）、`frontend`（管理与问答 UI）、`g2-ssr`（图表服务端渲染）
- Backend 采用领域模块化路由聚合（`backend/apps/api.py`），统一挂载到 `/api/v1`
- 通过中间件处理 Token 鉴权、统一响应包装、请求上下文与异常处理
- MCP 与主应用同进程构建：`main:app` 提供业务 API，`main:mcp_app` 提供 MCP 能力与图片静态资源
- 数据源访问使用统一抽象（`apps/db` + `DB` 枚举），按不同数据库走 SQLAlchemy 或原生驱动
- RAG 能力包含术语、数据训练、表结构 embedding，并在启动时做必要初始化与补全

### Testing Strategy
- 后端标准脚本：
  - `backend/scripts/lint.sh`: `mypy` + `ruff check` + `ruff format --check`
  - `backend/scripts/test.sh`: `pytest` + `coverage`
- 当前仓库自动化测试用例较少（几乎无独立 test 文件）；新增功能应至少补充关键路径单元测试/接口测试
- 前端目前以 `eslint` + `vue-tsc` + `vite build` 作为主要质量门禁

### Git Workflow
- 提交信息遵循 Conventional Commits 风格（如 `feat:`, `fix:`, `perf:`, `style:`，可带 scope）
- 建议流程：
  - 从功能分支开发并发起 PR
  - 合并前至少通过 lint/build，涉及后端逻辑的改动应补充测试
  - 避免在同一提交中混入无关格式化噪音

## Domain Context
- 核心业务是 Text-to-SQL + 可视化问答，支持“问数 -> SQL -> 执行 -> 图表/结论”链路
- 关键实体：工作空间、数据源、表/字段、术语库、数据训练、助手配置、权限规则
- 权限模型包含工作空间隔离、行级/列级权限过滤
- 支持嵌入式助手、MCP 调用、API 文档多语言展示（zh/en 等）
- 支持多数据源：MySQL、PostgreSQL、SQL Server、Oracle、ClickHouse、Doris、StarRocks、Redshift、Kingbase、Elasticsearch、Excel/CSV

## Important Constraints
- SQL 执行层默认只允许读操作（通过 SQL 解析拦截写操作）
- Python 版本固定 `3.11`，后端依赖与镜像构建流程对版本敏感
- 默认部署形态为 Linux + Docker（容器内含 PostgreSQL、后端 API、MCP、SSR）
- 安全相关配置（`SECRET_KEY`、数据库密码、模型 Key）必须通过环境变量管理，禁止硬编码到提交
- 项目采用 FIT2CLOUD Open Source License，二开需遵守许可证中对 Logo/版权及 GPLv3 衍生义务的要求
- OpenSpec 变更流程：新增功能、破坏性变更、架构变更需先提 proposal 并通过评审后再实现

## External Dependencies
- LLM/OpenAI-compatible API（OpenAI、Gemini、DeepSeek、通义、Kimi、腾讯/火山等供应商）
- PostgreSQL（主库）与可选 Redis 缓存
- 外部数据源连接（企业数据库与数据仓库）
- Docker Hub / 阿里云镜像仓库（镜像发布）
- 可选第三方集成场景：n8n、Dify、MaxKB、DataEase、MCP Client
