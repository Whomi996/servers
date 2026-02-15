# Everything Server - 项目结构

**[架构](architecture.md)
| 项目结构
| [启动流程](startup.md)
| [服务特性](features.md)
| [扩展点](extension.md)
| [工作机制](how-it-works.md)**

```text
src/everything
     ├── index.ts
     ├── AGENTS.md
     ├── package.json
     ├── docs
     │   ├── architecture.md
     │   ├── extension.md
     │   ├── features.md
     │   ├── how-it-works.md
     │   ├── instructions.md
     │   ├── startup.md
     │   └── structure.md
     ├── prompts
     │   ├── index.ts
     │   ├── args.ts
     │   ├── completions.ts
     │   ├── simple.ts
     │   └── resource.ts
     ├── resources
     │   ├── index.ts
     │   ├── files.ts
     │   ├── session.ts
     │   ├── subscriptions.ts
     │   └── templates.ts
     ├── server
     │   ├── index.ts
     │   ├── logging.ts
     │   └── roots.ts
     ├── tools
     │   ├── index.ts
     │   ├── echo.ts
     │   ├── get-annotated-message.ts
     │   ├── get-env.ts
     │   ├── get-resource-links.ts
     │   ├── get-resource-reference.ts
     │   ├── get-roots-list.ts
     │   ├── get-structured-content.ts
     │   ├── get-sum.ts
     │   ├── get-tiny-image.ts
     │   ├── gzip-file-as-resource.ts
     │   ├── toggle-simulated-logging.ts
     │   ├── toggle-subscriber-updates.ts
     │   ├── trigger-elicitation-request.ts
     │   ├── trigger-long-running-operation.ts
     │   └── trigger-sampling-request.ts
     └── transports
         ├── sse.ts
         ├── stdio.ts
         └── streamableHttp.ts
```

# 项目内容说明

## `src/everything`

### `index.ts`

- CLI 入口。根据第一个命令行参数选择并运行传输模块：`stdio`、`sse` 或 `streamableHttp`。

### `AGENTS.md`

- 面向 Agent/LLM 的开发说明，定义编码规范和正确扩展方式。

### `package.json`

- 包元数据与脚本：
  - `build`：编译 TypeScript 到 `dist/`，复制 `docs/`，并将编译后的入口脚本标记为可执行
  - `start:stdio`、`start:sse`、`start:streamableHttp`：运行 `dist/` 中构建产物
- 依赖包括 `@modelcontextprotocol/sdk`、`express`、`cors`、`zod` 等

### `docs/`

- `architecture.md`
  - 当前文档（架构总览）
- `server-instructions.md`
  - 面向客户端/LLM 的可读说明，指导如何使用服务；服务启动时加载并在 `initialize` 流程中返回

### `prompts/`

- `index.ts`
  - `registerPrompts(server)` 汇总入口，调用各 prompt 文件的注册函数
- `simple.ts`
  - 注册 `simple-prompt`：无参数，返回单条用户消息
- `args.ts`
  - 注册 `args-prompt`：参数为 `city`（必填）和 `state`（可选），用于拼装消息
- `completions.ts`
  - 注册 `completable-prompt`：使用 SDK `completable(...)` 提供服务端补全能力（如先补全 `department`，再给上下文相关 `name` 建议）
- `resource.ts`
  - 导出 `registerEmbeddedResourcePrompt(server)`，注册 `resource-prompt`：
    - 接收 `resourceType`（`Text` / `Blob`）与 `resourceId`（整数）
    - 在返回消息中嵌入对应动态资源
    - 内部复用 `resources/templates.ts` 的辅助函数

### `resources/`

- `index.ts`
  - `registerResources(server)` 汇总入口
- `templates.ts`
  - 使用 `ResourceTemplate` 注册两类动态模板资源：
- 文字：`demo://resource/dynamic/text/{index}`（MIME：`text/plain`）
- Blob：`demo://resource/dynamic/blob/{index}`（MIME：`application/octet-stream`，Base64）
  - `{index}` 必须是有限正整数，内容按需生成并带时间戳
  - 导出 `textResource(...)`、`textResourceUri(...)`、`blobResource(...)`、`blobResourceUri(...)` 供其他模块直接构造资源（如 prompts）
- `files.ts`
  - 注册 `docs/` 下静态文件资源
  - URI 模式：`demo://resource/static/document/<filename>`
  - `.md` 返回 `text/markdown`，`.txt` 返回 `text/plain`，`.json` 返回 `application/json`，其他默认 `text/plain`

### `server/`

- `index.ts`
  - 服务工厂：创建 `McpServer`、声明能力、加载 server instructions，并注册 tools/prompts/resources
  - 通过 `setSubscriptionHandlers(server)` 安装资源订阅处理
  - 对传输层暴露 `{ server, cleanup }`；连接断开时 `cleanup` 会停止正在运行的定时器
- `logging.ts`
  - 模拟日志实现：周期性发送不同级别随机日志到会话，可通过专用工具按需启停

### `tools/`

- `index.ts`
  - `registerTools(server)` 汇总入口
- `echo.ts`
  - 注册 `echo`，输入消息后返回 `Echo: {message}`
- `get-annotated-message.ts`
  - 注册 `annotated-message`，演示带 `annotations` 的内容项；按 `messageType`（`error` / `success` / `debug`）返回主 `text`，可选附带注解 `image`
- `get-env.ts`
  - 注册 `get-env`，返回当前进程环境变量（格式化 JSON）
- `get-resource-links.ts`
  - 注册 `get-resource-links`，返回说明文本及多个 `resource_link`
- `get-resource-reference.ts`
  - 注册 `get-resource-reference`，返回指定动态资源引用
- `get-roots-list.ts`
  - 注册 `get-roots-list`，返回客户端最近发送的 roots 列表
- `gzip-file-as-resource.ts`
  - 注册 `gzip-file-as-resource`：
    - 从 URL/data URI 抓取并压缩内容
    - 默认返回会话级资源 `resource_link`，也可返回内联 `resource`
    - 资源在会话期间可通过 `resources/list` 发现
    - 通过 `resources/session.ts` 注册到 `demo://resource/session/<name>`，`mimeType: application/gzip`
  - 环境变量：
    - `GZIP_MAX_FETCH_SIZE`（字节，默认 10 MiB）
    - `GZIP_MAX_FETCH_TIME_MILLIS`（毫秒，默认 30000）
    - `GZIP_ALLOWED_DOMAINS`（逗号分隔白名单；为空表示不限制域名）
- `trigger-elicitation-request.ts`
  - 注册 `trigger-elicitation-request`，向客户端/LLM 发送 `elicitation/create`
- `trigger-sampling-request.ts`
  - 注册 `trigger-sampling-request`，向客户端/LLM 发送 `sampling/createMessage`
- `get-structured-content.ts`
  - 注册 `get-structured-content`，演示 `structuredContent` 响应
- `get-sum.ts`
  - 注册 `get-sum`（Zod 输入 schema），对 `a` 与 `b` 求和
- `get-tiny-image.ts`
  - 注册 `get-tiny-image`，返回小型 PNG MCP logo（`image`）及前后说明 `text`
- `trigger-long-running-operation.ts`
  - 注册 `long-running-operation`，按给定 `duration` 和 `steps` 模拟长任务；客户端提供 `progressToken` 时发送 `notifications/progress`
- `toggle-simulated-logging.ts`
  - 注册 `toggle-simulated-logging`，按会话启停模拟日志
- `toggle-subscriber-updates.ts`
  - 注册 `toggle-subscriber-updates`，按会话启停模拟资源订阅更新检查

### `transports/`

- `stdio.ts`
  - 启动 `StdioServerTransport`，调用 `createServer()` 创建服务并连接
  - 处理 `SIGINT`，优雅关闭并调用 `cleanup()`
- `sse.ts`
  - 基于 Express 提供：
    - `GET /sse`：建立每个会话的 SSE 连接
    - `POST /message`：接收客户端消息
  - 通过 transport map 管理多客户端
  - 为每个新连接创建 `SSEServerTransport` 与服务实例并连接
  - 断开时调用 `cleanup()`
- `streamableHttp.ts`
  - 基于 Express 提供 `/mcp`：`POST`（JSON-RPC）、`GET`（SSE）、`DELETE`（会话结束）
  - 使用 `InMemoryEventStore` 支持会话恢复，按 `sessionId` 跟踪 transport
  - 初始化 `POST` 时连接新服务实例，后续请求复用对应 transport
