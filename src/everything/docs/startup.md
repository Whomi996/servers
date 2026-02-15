# Everything Server - 启动流程

**[架构](architecture.md)
| [项目结构](structure.md)
| 启动流程
| [服务特性](features.md)
| [扩展点](extension.md)
| [工作机制](how-it-works.md)**

## 1. Everything Server 启动器

- 用法：`node dist/index.js [stdio|sse|streamableHttp]`
- 根据指定参数运行对应的**传输管理器**
- 通过命令行指定传输类型（默认 `stdio`）
- `stdio` → `transports/stdio.js`
- `sse` → `transports/sse.js`
- `streamableHttp` → `transports/streamableHttp.js`

## 2. 传输管理器（Transport Manager）

- 通过 `server/index.ts` 的 `createServer()` 创建服务实例
  - 并连接到 MCP SDK 中对应传输实现
- 按 MCP 规范处理该传输下的通信

- **STDIO**
    - 进程绑定、单连接模型
    - 连接后调用 `clientConnect()`
    - `SIGINT` 时关闭并调用 `cleanup()`

- **上海证券交易所**
    - 支持多客户端连接
    - 客户端传输实例按 `sessionId` 进行映射
    - 连接后调用 `clientConnect(sessionId)`
    - 通过 `onclose` 清理并移除 session
    - 暴露接口：
      - `/sse`（`GET`，SSE 流）
      - `/message`（`POST`，JSON-RPC 消息）

- **流式 HTTP**
    - 支持多客户端连接
    - 客户端传输按 `sessionId` 映射
    - 连接后调用 `clientConnect(sessionId)`
    - 暴露 `/mcp`：
      - `POST`（JSON-RPC 消息）
      - `GET`（SSE 流）
      - `DELETE`（终止会话）
    - 使用 event store 实现可恢复能力，并按 `sessionId` 保存 transport
    - 在 `DELETE` 时调用 `cleanup(sessionId)`

## 3. 服务工厂（Server Factory）

- 调用 `server/index.ts` 中的 `createServer()`
- 创建新的 `McpServer` 实例，包含：
- **能力**
- `tools: {}`
- `logging: {}`
- `prompts: {}`
- `resources: { subscribe: true }`
- **服务器说明**
    - 从 docs 目录（`server-instructions.md`）加载
  - **注册项**
    - `registerTools(server)` 注册 tools
    - `registerResources(server)` 注册 resources
    - `registerPrompts(server)` 注册 prompts
  - **其他处理器**
    - `setSubscriptionHandlers(server)` 设置资源订阅处理
    - roots list change 处理器在连接后阶段添加
  - **返回值**
    - `McpServer` 实例
    - `clientConnect(sessionId)`：连接后初始化钩子
    - `cleanup(sessionId?)`：停止定时任务并清理会话态数据

## 启用多客户端

`transports` 目录中部分传输管理器支持多客户端。  
要实现多客户端，需把相关状态按 `sessionId` 进行映射管理。
