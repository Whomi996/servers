# Everything Server - 工作机制

**[架构](architecture.md)
| [项目结构](structure.md)
| [启动流程](startup.md)
| [服务特性](features.md)
| [扩展点](extension.md)
| 工作机制**

# 条件化工具注册

### 模块：`server/index.ts`

- 某些工具依赖客户端能力，只有客户端支持时才应注册：
  - `get-roots-list`
  - `trigger-elicitation-request`
  - `trigger-sampling-request`
- 客户端能力只有在初始化握手完成后才可确定。
- 大多数工具会在 Server Factory 执行阶段（客户端连接前）立即注册。
- 这些条件工具会延迟到 `oninitialized` 回调中，通过 `registerConditionalTools(server)` 注册。

## 资源订阅

### 模块：`resources/subscriptions.ts`

- 按 URI 追踪订阅者：`Map<uri, Set<sessionId>>`
- 通过 `setSubscriptionHandlers(server)` 安装订阅/取消订阅处理器并维护映射
- `toggle-subscriber-updates` 工具按需启停更新：
  - 启动：`beginSimulatedResourceUpdates(server, sessionId)`
  - 停止：`stopSimulatedResourceUpdates(sessionId)`
- `cleanup(sessionId?)` 会调用 `stopSimulatedResourceUpdates(sessionId)`，清理定时器与会话态数据

## 会话级资源（Session-scoped Resources）

### 模块：`resources/session.ts`

- `getSessionResourceURI(name: string)`：构建 URI：`demo://resource/session/<name>`
- `registerSessionResource(server, resource, type, payload)`：
  - 以给定 `uri`、`name`、`mimeType` 注册资源
  - 返回 `resource_link`
  - 内容仅驻留内存，在当前会话生命周期内可用
  - 支持 `type: "text" | "blob"`，并在对应字段返回数据
- 典型用法：工具在不落盘的情况下创建会话内产物
  - 例如 `tools/gzip-file-as-resource.ts` 会压缩抓取内容，注册为 `mimeType: application/gzip` 的会话资源，并按 `outputType` 返回 `resource_link` 或内联 `resource`

## 模拟日志

### 模块：`server/logging.ts`

- 周期性发送不同级别的随机日志；可包含 session ID，便于演示定位
- 通过 `toggle-simulated-logging` 工具按需启停：
  - 启动：`beginSimulatedLogging(server, sessionId?)`
  - 停止：`stopSimulatedLogging(sessionId?)`
  - 连接断开时 `cleanup()` 也会停止相关定时任务
- 使用 `server.sendLoggingMessage({ level, data }, sessionId?)` 发送日志，从而遵循客户端通过 SDK 设置的最小日志级别
