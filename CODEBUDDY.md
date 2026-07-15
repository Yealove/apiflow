# CODEBUDDY.md This file provides guidance to CodeBuddy when working with code in this repository.

## 常用命令

### 开发
```bash
# 同时启动前端和后端
npm run dev

# 仅启动前端（Electron + Vite HMR，端口 5173）
npm run web:dev

# 仅启动后端（端口 7001，需要 MongoDB）
npm run server:dev

# 纯 Web 模式启动前端（不含 Electron）
npm run web:web
```

### 构建
```bash
# 构建纯 Web 前端（产物：packages/web/dist/web/）
npm run web:build:web

# 本地构建 Electron 应用，不发布（产物：packages/web/release/）
npm run web:build:local:mac      # macOS .dmg
npm run web:build:local:win      # Windows .exe
npm run web:build:local:linux    # Linux .AppImage

# CI 构建并自动发布到 GitHub Release（仅 CI 环境使用）
npm run web:build:ci:mac
npm run web:build:ci:win
npm run web:build:ci:linux

# 构建后端
npm run server:build
```

### 测试
```bash
# 后端单元测试（含覆盖率）
npm run server:cov

# E2E 测试（Playwright，需要先构建）
npm --prefix packages/web run test:e2e
npm --prefix packages/web run test:e2e:ui       # 交互式 UI 模式
npm --prefix packages/web run test:e2e:headed   # 有头浏览器模式
```

### Docker（开发/测试环境，推荐）

首次启动步骤：

```bash
# 1. 创建环境变量文件（若不存在）
cp .env.example .env
# 编辑 .env，确保 MONGO_ROOT_PASSWORD 已设置

# 2. 构建前端产物（web 容器以 volume 挂载 dist/web）
npm run web:build:web

# 3. 启动全部服务（MongoDB + Server + Web + Website）
docker compose up -d

# 4. 验证服务状态
docker compose ps
# 四个服务全部显示 "healthy" 或 "running" 即为正常
```

服务端口：

| 服务 | 端口 | 说明 |
|------|------|------|
| web | 80 | Nginx 提供 Vue SPA，挂载本地 `packages/web/dist/web` |
| server | 7001 | Midway.js 后端 API，健康检查 `/api/health` |
| mongo | 27017 | MongoDB 6，含认证 |
| website | 3000 | 官网/产品落地页 |

日常开发（修改前端代码后）：

```bash
# 重新构建前端产物即可，无需重启容器（volume 实时挂载）
npm run web:build:web
```

重置环境（清除所有数据重新开始）：

```bash
docker compose down -v   # 删除容器和 volume（包括数据库数据）
docker compose up -d     # 重新启动
```

### Git
```bash
# 推送代码到 GitHub（remote 名称为 'github'）
git add -A
git commit -m "描述信息"
git push github main
```

注意：项目有两个远程仓库：
- `origin` — https://cnb.cool/iwonder/apiflow.git
- `github` — https://github.com/Yealove/apiflow.git

添加 GitHub 远程仓库：
```bash
git remote add github https://github.com/Yealove/apiflow.git
```

推送代码：
```bash
git push origin main   # 推送到 cnb.cool
git push github main   # 推送到 GitHub
```

## 架构概览

Apiflow 是一个基于 Electron 的 API 开发平台（Postman 替代品），后端使用 Midway.js。支持离线使用（IndexedDB 缓存）和在线协作（MongoDB 服务端）。

### 三个子包

| 包 | 技术栈 | 用途 |
|---|-------|------|
| `packages/web` | Vue 3 + Vite + Electron + Pinia + Element Plus | 前端：Electron 桌面应用 + Web SPA |
| `packages/server` | Midway.js（基于 Koa）+ Typegoose/Mongoose | 后端 REST API + WebSocket 服务 |
| `packages/website` | Express + 静态页面 | 官网/产品落地页 |

根目录 `package.json` 仅作为编排器，通过 `--prefix` 代理到子包。**不是** npm workspaces 单体仓库。

### 前端（`packages/web`）

**双模式架构**：同一份代码既可运行 Electron 桌面应用，也可作为 Web SPA，由环境变量 `BUILD_TARGET` 控制（`web` vs Electron）。

**Electron 主进程**（`src/main/`）：
- `main.ts`：创建 `BrowserWindow` + `WebContentsView`（顶栏 + 内容视图），注册 `app://` 自定义协议，系统托盘，单实例锁，全局快捷键，自动更新
- `preload.ts`：通过 context bridge 暴露 `window.electronAPI`（WebSocket、IPC、文件系统等）
- `ipcMessage/`：渲染进程与主进程之间的所有 IPC 处理器（31KB 模块）
- `mcp/`：通过 stdio 运行的 MCP JSON-RPC 服务，用于 AI 工具/资源集成
- `mock/`：在主进程中运行的本地 HTTP 和 WebSocket Mock 服务器
- `sendRequest.ts`：在主进程中实际执行 HTTP 请求（got 库），由渲染进程代理而来
- `lifecycle/contentViewLifecycle.ts`：在线/离线内容视图，含回退、重试、超时逻辑
- `update/`：通过 electron-updater 实现自动更新，含下载管理器

**Electron 渲染进程**（`src/renderer/`）：
- **框架**：Vue 3 Composition API，Pinia 状态管理，Vue Router（hash 模式），Element Plus UI
- **入口**：`main.ts` 创建 Vue 实例，安装 Pinia、路由、i18n、Element Plus、自定义指令
- **6 条路由**：`/home`（首页）、`/workbench`（核心 API 工具）、`/header`（Electron 顶栏）、`/login`（登录）、`/share`（分享）、`/settings`（设置）
- **离线优先**：IndexedDB 缓存所有项目数据（`src/renderer/cache/`）。路由进入工作区前预初始化缓存。在线/离线模式由 `runtimeStore` 追踪。

**状态管理**（`src/renderer/store/`）：
按功能领域组织。核心 store：
- `runtime/runtimeStore.ts` — 用户信息、网络模式（在线/离线）、应用运行时状态
- `httpNode/httpNodeStore.ts` — HTTP 请求节点的增删改查、URL、请求头、请求体、参数
- `httpNode/httpNodeRequestStore.ts` — 根据节点 + 环境 + 变量计算 `fullUrl`，并监听变化
- `projectWorkbench/environmentStore.ts` — 环境管理（baseUrl、各环境变量）
- `projectWorkbench/variablesStore.ts` — 工作区级模板变量
- `ai/agentStore.ts` — AI Agent 状态、LLM 客户端管理
- `ai/skillStore.ts` — 133KB 的 AI 技能定义和工具注册

**请求引擎**（`src/renderer/server/request/`）：
两个独立的层：
1. **后端 API 客户端**（`api/api.ts`）：基于 Axios，请求签名（SHA-256 + 时间戳 + 随机数），用于所有服务端通信
2. **HTTP 代理引擎**（`server/request/request.ts`）：实际的 API 测试引擎。解析模板变量、拼接环境 baseUrl、在 Web Worker 中执行前置/后置脚本、处理 Cookie、重定向、multipart、SSE 流

请求链路：渲染进程 → `request.ts` → `request.web.ts`（IPC 代理）→ 主进程 `sendRequest.ts`（got）→ 响应 → Web Worker（后置脚本）→ 最终结果。

**模板变量解析**（`request.ts` 中的 `getUrl`）：
- 合并：工作区变量 + 环境变量（来自当前激活环境）+ 全局请求头
- `activeEnvironment.baseUrl` 会拼接到相对 URL 前面
- `httpNodeRequestStore` 监听 `environmentStore.activeEnvironment?.baseUrl`，变更时实时更新 URL

**IndexedDB 缓存**（`src/renderer/cache/`）：
每个缓存模块封装一个 IndexedDB 存储（Dexie.js）。用于离线模式——项目、API 节点、变量、环境、响应历史、Cookie 全部本地持久化，上线后与服务端同步。

**共享代码执行**（`src/shared/`）：
- `execCode.renderer.ts` — Electron 渲染进程中的代码执行（Node.js 沙箱）
- `execCode.web.ts` — 纯 Web 模式下的代码执行（Function 构造器沙箱）
- 前置/后置脚本在 Web Worker 中运行（`src/renderer/worker/`），保证隔离

### 后端（`packages/server`）

**框架**：Midway.js v3（基于 Koa 的 IoC 框架，装饰器依赖注入），监听 7001 端口。

**架构模式**：基于装饰器的 MVC，使用 Midway IoC 容器：
- `@Controller('/api/xxx')` 装饰路由控制器
- `@Inject()` / `@InjectEntityModel()` 依赖注入
- `@Provide()` 注册服务
- 自定义装饰器：`@ReqLimit`（限流）、`@ReqSign`（请求签名校验）

**核心模块**：
- `controller/` — 路由处理器：`api`、`common`、`docs`、`project`、`proxy`、`security`
- `service/` — 业务逻辑：文档、项目、安全、附件、代理
- `entity/` — Typegoose 实体（Mongoose 模型）：User、Role、Project、Doc、Environment、Variable 等
- `middleware/` — `ResponseWrapperMiddleware`（统一响应格式）、`PermissionMiddleware`（JWT 鉴权）
- `filter/` — 校验错误和服务端错误过滤器
- `socket/` — WebSocket 控制器，用于 Mock 数据生成（JSON、XML、CSV、二进制）

**数据库**：MongoDB，通过 Mongoose v7 + Typegoose 提供类型化实体模型。连接字符串来自 `MONGODB_URI` 环境变量。

**API 端点**：
- `/api/security/*` — 认证（登录、验证码、注册、用户/角色/路由/菜单 CRUD）
- `/api/project/*` — 项目 CRUD、环境、变量、分享、公共请求头
- `/api/docs/*` — 文档管理、导入/导出
- `/api/proxy/http` — HTTP 代理（转发客户端 API 测试请求）
- `/api/attachment/*` — 文件上传
- WebSocket 端点 — Mock 数据生成引擎

**认证**：基于 JWT。`PermissionMiddleware` 根据白名单/黑名单校验 token。请求签名使用 SHA-256 + 时间戳 + 随机数。

### Docker 部署

`docker-compose.yml` 包含四个服务：
1. `mongo` — MongoDB 6，含健康检查
2. `server` — Midway.js 后端（内部 7001 端口，健康检查 `/api/health`）
3. `website` — Express 官网页面（3000 端口）
4. `web` — Nginx 提供 Vue SPA（80 端口）。本地开发时 `packages/web/dist/web` 以 volume 挂载覆盖 Docker 镜像的静态文件

`.env` 环境变量：`MONGO_ROOT_USERNAME`、`MONGO_ROOT_PASSWORD`、`MONGO_DATABASE`、`DEPLOYMENT_TYPE`，以及可选的阿里云邮件/短信凭证。

### CI/CD

**GitHub Actions**（`.github/workflows/build-electron.yml`）：
由标签推送（`v*.*.*`）、手动触发或 Release 事件触发。并行构建 Windows、Linux、macOS 的 Electron 应用，通过 `electron-builder --publish always` 自动发布到 GitHub Release。同时通过 rsync 将更新产物部署到自定义更新服务器。发布目标（owner/repo）在 `packages/web/package.json` 的 `build.publish` 中配置。

### 关键架构模式

- **离线优先 + 同步**：所有项目数据缓存在 IndexedDB。离线模式下写入本地缓存；恢复在线后同步到服务端。
- **代码执行沙箱**：前置/后置脚本在 Web Worker（渲染进程）或 Node 沙箱（主进程）中运行，不阻塞 UI 线程。
- **模板变量解析**：多来源变量（工作区、环境、全局请求头）在请求执行前合并，通过 Mustache 风格模板解析。
- **请求代理**：Electron 中 HTTP 请求通过 IPC 发送到主进程（绕过 CORS）；Web 模式下通过后端 `/api/proxy/http` 端点转发。
