# 前置/后置脚本内置变量参考

## HTTP 前置脚本

脚本中可直接使用以下全局变量：

### `af` — Apiflow 请求对象

| 属性 | 类型 | 说明 |
|------|------|------|
| `af.nodeId` | `string` | 当前请求节点 ID |
| `af.projectId` | `string` | 当前项目 ID |
| `af.request.method` | `string` | HTTP 方法（GET / POST / PUT / DELETE 等） |
| `af.request.url` | `string` | 完整 URL |
| `af.request.path` | `string` | URL 路径 |
| `af.request.replaceUrl(url)` | `Function` | 替换整个 URL |
| `af.request.headers` | `Proxy<Record>` | 请求头，修改自动生效 |
| `af.request.queryParams` | `Proxy<Record>` | 查询参数，修改自动生效 |
| `af.request.pathParams` | `Proxy<Record>` | 路径参数，修改自动生效 |
| `af.request.bodyType` | `string` | 请求体类型（json / urlencoded / formdata / raw / binary） |
| `af.request.body.json` | `Proxy<object>` | JSON 请求体 |
| `af.request.body.urlencoded` | `Proxy<Record>` | URL-encoded 请求体 |
| `af.request.body.formdata` | `Proxy<Record>` | FormData 请求体 |
| `af.request.body.raw` | `string` | 原始文本请求体 |
| `af.request.body.binary` | `string` | 二进制请求体 |
| `af.envs` | `Record` | 全局变量 + 环境变量合并（只读） |
| `af.currentEnv` | `Record` | 当前环境变量（只读） |
| `af.variables` | `Proxy<Record>` | 临时变量，支持直接赋值 |
| `af.cookies` | `Proxy<Record>` | Cookies，值必须为字符串 |
| `af.sessionStorage` | `SessionStorage` | 会话级存储，含 `.set(key, val)` / `.get(key)` / `.remove(key)` / `.clear()` |
| `af.localStorage` | `LocalStorage` | 本地持久存储，含 `.set(key, val)` / `.get(key)` / `.remove(key)` / `.clear()` |
| `af.http.request(url, options)` | `Function` | 在脚本内发起 HTTP 请求 |
| `af.http.get(url, options)` | `Function` | GET 请求快捷方法 |
| `af.http.post(url, options)` | `Function` | POST 请求快捷方法 |
| `af.http.put(url, options)` | `Function` | PUT 请求快捷方法 |
| `af.http.delete(url, options)` | `Function` | DELETE 请求快捷方法 |

### `options` — 辅助对象

| 属性 | 类型 | 说明 |
|------|------|------|
| `options.getFile(path)` | `Function` | 获取文件描述对象，用于 formdata / binary 上传 |
| `options.axios` | `AxiosInstance` | axios 实例 |

### `axios` — 独立变量

等同于 `options.axios`，可直接用于发起 HTTP 请求。

### `getFile` — 独立变量

等同于 `options.getFile`。

---

## HTTP 后置脚本

脚本中可直接使用一个全局变量：

### `af` — Apiflow 响应对象

| 属性 | 类型 | 说明 |
|------|------|------|
| `af.nodeId` | `string` | 当前请求节点 ID |
| `af.projectId` | `string` | 当前项目 ID |
| `af.request.method` | `string` | 原始请求 HTTP 方法（**只读**） |
| `af.request.url` | `string` | 原始请求完整 URL（**只读**） |
| `af.request.headers` | `Record` | 原始请求头（**只读**） |
| `af.response.statusCode` | `number` | HTTP 状态码 |
| `af.response.headers` | `Record` | 响应头 |
| `af.response.cookies` | `Record` | 响应中的 Cookies |
| `af.response.rt` | `number` | 响应时间（毫秒） |
| `af.response.size` | `number` | 响应大小（字节） |
| `af.response.ip` | `string` | 服务器 IP 地址 |
| `af.response.text` | `string` | 文本格式的响应体 |
| `af.response.json` | `object` | JSON 格式的响应体 |
| `af.response.body` | `any` | 原始响应体 |
| `af.variables` | `Proxy<Record>` | 临时变量，支持 `.set(key, val)` / `.get(key)` / `.remove(key)` 和直接赋值 |
| `af.sessionStorage` | `SessionStorage` | 会话级存储，同前置脚本 |
| `af.localStorage` | `LocalStorage` | 本地持久存储，同前置脚本 |
| `af.cookies` | `Record` | 响应 Cookies |
| `af.http.request(url, options)` | `Function` | 在脚本内发起 HTTP 请求 |
| `af.http.get(url, options)` | `Function` | GET 请求快捷方法 |
| `af.http.post(url, options)` | `Function` | POST 请求快捷方法 |
| `af.http.put(url, options)` | `Function` | PUT 请求快捷方法 |
| `af.http.delete(url, options)` | `Function` | DELETE 请求快捷方法 |

> **注意：** 后置脚本**没有** `options`、`axios`、`getFile` 这些辅助变量，仅 `af` 一个全局变量。

---

## WebSocket 前置脚本

### `ws` — WebSocket 请求对象

| 属性 | 类型 | 说明 |
|------|------|------|
| `ws.projectId` | `string` | 当前项目 ID |
| `ws.nodeId` | `string` | 当前节点 ID |
| `ws.request.protocol` | `string` | 协议：`ws` 或 `wss` |
| `ws.request.url` | `object` | URL 对象，含 `.prefix` / `.path` / `.url` |
| `ws.request.headers` | `Proxy<Record>` | 请求头 |
| `ws.request.queryParams` | `Proxy<Record>` | 查询参数 |
| `ws.request.replaceUrl(url)` | `Function` | 替换 URL |
| `ws.variables` | `Proxy<Record>` | 临时变量 |
| `ws.sessionStorage` | `SessionStorage` | 会话级存储 |
| `ws.localStorage` | `LocalStorage` | 本地持久存储 |
| `ws.cookies` | `Proxy<Record>` | Cookies |

### `axios` — 独立变量

可用于在脚本内发起 HTTP 请求。

---

## WebSocket 后置脚本

### `ws` — WebSocket 响应对象

| 属性 | 类型 | 说明 |
|------|------|------|
| `ws.projectId` | `string` | 当前项目 ID |
| `ws.nodeId` | `string` | 当前节点 ID |
| `ws.response.data` | `string \| ArrayBuffer` | 接收到的消息数据 |
| `ws.response.type` | `string` | 数据类型：`text` 或 `binary` |
| `ws.response.timestamp` | `number` | 接收时间戳 |
| `ws.response.size` | `number` | 数据大小（字节） |
| `ws.response.mimeType` | `string` | MIME 类型 |
| `ws.variables` | `Proxy<Record>` | 临时变量 |
| `ws.sessionStorage` | `SessionStorage` | 会话级存储 |
| `ws.localStorage` | `LocalStorage` | 本地持久存储 |
| `ws.cookies` | `Proxy<Record>` | Cookies |

### `axios` — 独立变量

可用于在脚本内发起 HTTP 请求。

---

## 关键规则

1. **所有脚本均支持 `await`**：脚本被包裹在 `async function` 中执行，可以自由使用 `await`。
2. **Proxy 对象自动同步**：标记为 `Proxy` 的属性（如 `headers`、`variables`、`cookies` 等），其修改会自动同步回主线程，无需手动提交。
3. **存储限制**：`sessionStorage` 和 `localStorage` 单值最大限制为 **100KB**。
4. **Cookies 值类型**：HTTP 前置脚本中，`af.cookies` 的值必须为**字符串类型**。
