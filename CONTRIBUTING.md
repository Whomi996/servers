# 为 MCP Servers 做贡献

感谢你对贡献的兴趣！以下是你可以帮助改进此仓库的方式。

我们通过 [GitHub 标准工作流（GitHub Flow）](https://docs.github.com/en/get-started/using-github/github-flow) 接收变更。

## 服务列表（Server Listings）

我们**不再接受**通过 PR 往 README 中新增服务器链接。请改为将你的服务器发布到 [MCP Server Registry](https://github.com/modelcontextprotocol/registry)。可参考 [快速开始指南](https://github.com/modelcontextprotocol/registry/blob/main/docs/modelcontextprotocol-io/quickstart.mdx)。

你也可以通过这个简单界面浏览已发布服务器：[https://registry.modelcontextprotocol.io/](https://registry.modelcontextprotocol.io/)。

## 服务实现（Server Implementations）

我们欢迎：
- **Bug 修复**：帮助我们消灭恼人的缺陷。
- **可用性改进**：让人类和智能体更容易使用这些服务器。
- **展示 MCP 协议能力的增强**：我们鼓励能更好展示 MCP 协议能力的贡献，不要只局限于 Tools，也包括 Resources、Prompts、Roots 等。例如，为 filesystem-server 增加 Roots 支持，就能展示这个重要但相对少见的特性。

我们会更谨慎评估：
- **其他新功能**：尤其是与服务器核心目标关联不强、或主观性较高的功能。现有服务器主要是参考实现，用于启发社区。如果你需要特定功能，我们鼓励你构建增强版本并发布到 [MCP Server Registry](https://github.com/modelcontextprotocol/registry)。我们认为多样化的服务器生态对所有人都有益。

我们不接受：
- **新增服务器实现**：请改为发布到 [MCP Server Registry](https://github.com/modelcontextprotocol/registry)。

## 测试

当你为 TypeScript 实现的服务器新增或配置测试时，请使用 **vitest** 作为测试框架。Vitest 对 ESM 支持更好、执行更快、测试体验也更现代。

## 文档

欢迎改进现有文档。不过一般来说，我们更偏好通过可用性改进来消除痛点，而不是仅仅记录痛点。

对于全新的文档内容，我们会更谨慎，特别是那些不够厂商中立的内容（例如“如何用某个特定客户端运行某个特定服务器”）。

## 社区

[了解 MCP 社区如何沟通](https://modelcontextprotocol.io/community/communication)。

感谢你帮助 MCP servers 变得更好！
