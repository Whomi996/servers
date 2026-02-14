# Everything Server - 特性

**[架构](architecture.md)
| [项目结构](structure.md)
| [启动流程](startup.md)
| 服务特性
| [扩展点](extension.md)
| [工作机制](how-it-works.md)**

## Tools

- `echo`（`tools/echo.ts`）：回显 `message: string`，输入使用 Zod 校验。
- `get-annotated-message`（`tools/get-annotated-message.ts`）：根据 `messageType`（`error` / `success` / `debug`）返回带 `priority` 与 `audience` 注解的 `text` 消息；可选附带带注解的 `image`。
- `get-env`（`tools/get-env.ts`）：返回运行进程中的全部环境变量（格式化 JSON 文本）。
- `get-resource-links`（`tools/get-resource-links.ts`）：返回一段介绍文本和多个 `resource_link`。根据 `count`（1–10）在动态 Text 与 Blob 资源之间交替生成 URI（来自 `resources/templates.ts`）。
- `get-resource-reference`（`tools/get-resource-reference.ts`）：输入 `resourceType`（`text` 或 `blob`）与 `resourceId`（正整数），返回具体 `resource` 内容块（含 `uri`、`mimeType` 和数据）与说明文字。
- `get-roots-list`（`tools/get-roots-list.ts`）：返回客户端最近一次发送的 roots 列表。
- `gzip-file-as-resource`（`tools/gzip-file-as-resource.ts`）：输入 `name` 与 `data`（URL 或 data URI），在大小/时间/域名限制下抓取数据并压缩，注册为会话资源 `demo://resource/session/<name>`（`mimeType: application/gzip`），并根据 `outputType` 返回 `resource_link`（默认）或内联 `resource`。
- `get-structured-content`（`tools/get-structured-content.ts`）：演示结构化响应。输入 `location`，返回兼容旧格式的 `content`（含 JSON 的 `text`）以及通过 `outputSchema` 校验的 `structuredContent`（温度、天气状况、湿度）。
- `get-sum`（`tools/get-sum.ts`）：输入数字 `a`、`b`，返回求和结果。使用 Zod 做输入校验。
- `get-tiny-image`（`tools/get-tiny-image.ts`）：返回一个很小的 MCP logo PNG（`image` 内容项），前后附带简短说明文本。
- `trigger-long-running-operation`（`tools/trigger-trigger-long-running-operation.ts`）：模拟多步骤长任务（给定 `duration` 和 `steps`）；客户端提供 `progressToken` 时通过 `notifications/progress` 推送进度。
- `toggle-simulated-logging`（`tools/toggle-simulated-logging.ts`）：按会话启动/停止随机日志级别的模拟日志，遵循客户端设置的最小日志级别。
- `toggle-subscriber-updates`（`tools/toggle-subscriber-updates.ts`）：按会话启动/停止已订阅 URI 的资源更新通知模拟。
- `trigger-sampling-request`（`tools/trigger-sampling-request.ts`）：向客户端/LLM 发起 `sampling/createMessage` 请求（含 `prompt` 与可选生成参数），返回 LLM 响应。
- `simulate-research-query`（`tools/simulate-research-query.ts`）：演示 MCP Tasks（SEP-1686）的多阶段研究任务。输入 `topic` 与 `ambiguous`。任务会分阶段推进并更新状态；若 `ambiguous=true` 且客户端支持 elicitation，会先发起澄清请求再完成任务。
- `trigger-sampling-request-async`（`tools/trigger-sampling-request-async.ts`）：演示双向任务：服务端发 sampling 请求，客户端以后台任务执行；服务端轮询状态并在完成后获取结果。要求客户端支持 `tasks.requests.sampling.createMessage`。
- `trigger-elicitation-request-async`（`tools/trigger-elicitation-request-async.ts`）：演示双向任务：服务端发 elicitation 请求，客户端后台执行；服务端轮询等待用户输入。要求客户端支持 `tasks.requests.elicitation.create`。

## Prompts

- `simple-prompt`（`prompts/simple.ts`）：无参数提示词，返回固定用户消息。
- `args-prompt`（`prompts/args.ts`）：两个参数：`city`（必填）与 `state`（可选），组合成提问。
- `completable-prompt`（`prompts/completions.ts`）：演示参数自动补全，使用 SDK 的 `completable`；`department` 补全可驱动上下文相关的 `name` 建议。
- `resource-prompt`（`prompts/resource.ts`）：输入 `resourceType`（`Text` 或 `Blob`）和 `resourceId`（可转整数字符串），返回消息中嵌入对应动态资源（由 `resources/templates.ts` 生成）。

## Resources

- 动态 Text：`demo://resource/dynamic/text/{index}`（按需生成）
- 动态 Blob：`demo://resource/dynamic/blob/{index}`（按需生成 base64 负载）
- 静态文档：`demo://resource/static/document/<filename>`（来自 `src/everything/docs/`）
- 会话资源：`demo://resource/session/<name>`（运行时动态注册，仅会话生命周期内可用）

## 资源订阅与通知

- 资源更新通知默认关闭，需要显式开启
- 客户端可通过 MCP `resources/subscribe` 与 `resources/unsubscribe` 订阅/取消订阅 URI
- 通过 `toggle-subscriber-updates` 启停会话级定时器，仅对该会话已订阅 URI 发送 `notifications/resources/updated { uri }`
- 支持多客户端并发；每个客户端订阅按会话独立追踪与投递

## 模拟日志

- 模拟日志默认关闭
- 通过 `toggle-simulated-logging` 启停会话级周期日志（级别包括 debug/info/notice/warning/error/critical/alert/emergency）
- 客户端可通过 MCP `logging/setLevel` 设置最小接收日志级别

## Tasks（SEP-1686）

服务端声明支持 MCP Tasks，可用于长任务状态跟踪：

- **声明能力**：`tasks.list`、`tasks.cancel`、`tasks.requests.tools.call`
- **任务存储**：使用 SDK experimental 的 `InMemoryTaskStore`
- **消息队列**：使用 `InMemoryTaskMessageQueue`

### 任务生命周期

1. 客户端调用 `tools/call` 并传 `task: true`
2. 服务端返回 `CreateTaskResult`（含 `taskId`），而非立即返回最终结果
3. 客户端轮询 `tasks/get` 获取状态与 `statusMessage`
4. 当状态为 `completed` 时，客户端调用 `tasks/result` 获取最终结果

### 任务状态

- `working`：执行中
- `input_required`：需要额外输入（服务端会直接发 elicitation 请求）
- `completed`：成功完成
- `failed`：执行失败
- `cancelled`：被客户端取消

### 演示工具

**服务端任务（Client -> Server）**  
使用 `simulate-research-query` 体验完整任务流程。设置 `ambiguous: true` 可触发 elicitation：服务端会直接发送 `elicitation/create` 并等待响应后完成。

**客户端任务（Server -> Client）**  
使用 `trigger-sampling-request-async` 或 `trigger-elicitation-request-async` 演示双向任务：服务端向客户端发请求，由客户端后台执行。客户端需要分别声明 `tasks.requests.sampling.createMessage` 或 `tasks.requests.elicitation.create` 能力。

### 双向任务流

MCP Tasks 是双向的，服务端和客户端都可以成为任务执行方：

| 方向 | 请求类型 | 任务执行方 | 演示工具 |
| --- | --- | --- | --- |
| Client -> Server | `tools/call` | Server | `simulate-research-query` |
| Server -> Client | `sampling/createMessage` | Client | `trigger-sampling-request-async` |
| Server -> Client | `elicitation/create` | Client | `trigger-elicitation-request-async` |

对于客户端侧任务：

1. 服务端发送带任务元数据的请求（如 `params.task.ttl`）
2. 客户端创建任务并返回 `CreateTaskResult`（含 `taskId`）
3. 服务端轮询 `tasks/get` 追踪状态
4. 任务完成后，服务端调用 `tasks/result` 拉取最终结果
