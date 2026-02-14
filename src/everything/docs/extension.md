# Everything Server - 扩展点

**[架构](architecture.md)
| [项目结构](structure.md)
| [启动流程](startup.md)
| [服务特性](features.md)
| 扩展点
| [工作机制](how-it-works.md)**

## 添加工具（Tools）

- 在 `tools/` 下新建文件，提供 `registerXTool(server)` 函数，内部通过 `server.registerTool(...)` 注册。
- 在 `tools/index.ts` 的 `registerTools(server)` 中导出并调用。

## 添加提示词（Prompts）

- 在 `prompts/` 下新建文件，提供 `registerXPrompt(server)` 函数，内部通过 `server.registerPrompt(...)` 注册。
- 在 `prompts/index.ts` 的 `registerPrompts(server)` 中导出并调用。

## 添加资源（Resources）

- 在 `resources/` 下新建文件，提供 `registerXResources(server)`，通过 `server.registerResource(...)`（可选结合 `ResourceTemplate`）注册。
- 在 `resources/index.ts` 的 `registerResources(server)` 中导出并调用。
