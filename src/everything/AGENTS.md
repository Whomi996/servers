# MCP “Everything” Server - 开发指南

## 构建、测试与运行命令

- Build：`npm run build` - 将 TypeScript 编译为 JavaScript
- Watch mode：`npm run watch` - 监听变更并自动重建
- Run STDIO server：`npm run start:stdio` - 使用 stdio 传输启动 MCP 服务
- Run SSE server：`npm run start:sse` - 使用 SSE 传输启动 MCP 服务
- Run StreamableHttp server：`npm run start:stremableHttp` - 使用 StreamableHttp 传输启动 MCP 服务
- Prepare release：`npm run prepare` - 为发布构建项目

## 代码风格指南

- 使用 ES modules，import 路径带 `.js` 扩展名
- 对函数与变量使用严格 TypeScript 类型
- 工具输入校验遵循 zod schema 模式
- 优先使用 async/await，避免回调和 Promise 链式写法
- 所有 import 放在文件顶部，按“外部依赖 -> 内部模块”分组
- 变量命名应语义清晰、表达明确
- 服务关闭时要正确清理定时器与资源
- 使用 try/catch 做错误处理，并提供清晰错误信息
- 保持一致缩进（2 空格）和多行对象尾随逗号
- 遵循所在目录既有风格（代码风格、import 顺序、模块布局）
- 变量/函数用 camelCase
- 类型/类用 PascalCase
- 常量用 UPPER_CASE
- 文件名及注册的 tools/prompts/resources 用 kebab-case
- 工具名称应使用动词，例如 `get-annotated-message`，不要用 `annotated-message`

## 扩展服务

Everything Server 设计为在明确扩展点进行扩展。  
参见 [Extension Points](docs/extension.md) 与 [Project Structure](docs/structure.md)。

服务工厂在 `src/everything/server/index.ts`，负责启动时注册全部特性，并处理连接后初始化逻辑。

### 高层结构

- Tools 在 `src/everything/tools/`，通过 `registerTools(server)` 注册
- Resources 在 `src/everything/resources/`，通过 `registerResources(server)` 注册
- Prompts 在 `src/everything/prompts/`，通过 `registerPrompts(server)` 注册
- 订阅与模拟更新在 `src/everything/resources/subscriptions.ts`
- 日志辅助在 `src/everything/server/logging.ts`
- 传输管理在 `src/everything/transports/`

### 新增功能时

- 遵循该目录现有文件/模块模式（命名、导出、注册函数）
- 导出 `registerX(server)` 函数，按现有风格向 MCP SDK 注册能力
- 把新模块接入对应 central index（如 `tools/index.ts`、`resources/index.ts`、`prompts/index.ts`）
- 工具 schema 应为准确的 JSON Schema，并提供有用描述与示例
- 注意 `server/index.ts` 以及 `logging.ts`、`subscriptions.ts` 中的相关使用
- 如新增/修改了重要特性，同步更新 `src/everything/docs/` 文档
