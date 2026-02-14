# Everything MCP Server
**[架构](docs/architecture.md)
| [项目结构](docs/structure.md)
| [启动流程](docs/startup.md)
| [服务特性](docs/features.md)
| [扩展点](docs/extension.md)
| [工作机制](docs/how-it-works.md)**

该 MCP 服务器旨在覆盖 MCP 协议的几乎全部特性。它并非面向实际业务价值的服务器，而是为 MCP 客户端开发者提供的测试服务器。它实现了 prompts、tools、resources、sampling 等能力，用于展示 MCP 的能力边界。

## Tools、Resources、Prompts 与其他特性

已注册的 MCP 原语与其他协议特性的完整列表见 [服务特性](docs/features.md)。

## 在 Claude Desktop 中使用（使用 [stdio 传输](https://modelcontextprotocol.io/specification/2025-03-26/basic/transports#stdio)）

将以下内容添加到 `claude_desktop_config.json`：

```json
{
  "mcpServers": {
    "everything": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-everything"
      ]
    }
  }
}
```

## 在 VS Code 中使用

可使用以下一键安装按钮：

[![Install with NPX in VS Code](https://img.shields.io/badge/VS_Code-NPM-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=everything&config=%7B%22command%22%3A%22npx%22%2C%22args%22%3A%5B%22-y%22%2C%22%40modelcontextprotocol%2Fserver-everything%22%5D%7D) [![Install with NPX in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-NPM-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=everything&config=%7B%22command%22%3A%22npx%22%2C%22args%22%3A%5B%22-y%22%2C%22%40modelcontextprotocol%2Fserver-everything%22%5D%7D&quality=insiders)

[![Install with Docker in VS Code](https://img.shields.io/badge/VS_Code-Docker-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=everything&config=%7B%22command%22%3A%22docker%22%2C%22args%22%3A%5B%22run%22%2C%22-i%22%2C%22--rm%22%2C%22mcp%2Feverything%22%5D%7D) [![Install with Docker in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Docker-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=everything&config=%7B%22command%22%3A%22docker%22%2C%22args%22%3A%5B%22run%22%2C%22-i%22%2C%22--rm%22%2C%22mcp%2Feverything%22%5D%7D&quality=insiders)

手动安装可选两种方式：

**方式 1：用户级配置（推荐）**  
在命令面板（`Ctrl + Shift + P`）执行 `MCP: Open User Configuration`，编辑用户 `mcp.json`。

**方式 2：工作区配置**  
在工作区添加 `.vscode/mcp.json`，方便共享。

> 更多信息见 [VS Code MCP 官方文档](https://code.visualstudio.com/docs/copilot/customization/mcp-servers)。

#### NPX

```json
{
  "servers": {
    "everything": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-everything"]
    }
  }
}
```

## 从源码运行（[HTTP+SSE 传输](https://modelcontextprotocol.io/specification/2024-11-05/basic/transports#http-with-sse)，已在 [2025-03-26](https://modelcontextprotocol.io/specification/2025-03-26/basic/transports) 标记废弃）

```shell
cd src/everything
npm install
npm run start:sse
```

## 从源码运行（[Streamable HTTP 传输](https://modelcontextprotocol.io/specification/2025-03-26/basic/transports#streamable-http)）

```shell
cd src/everything
npm install
npm run start:streamableHttp
```

## 作为已安装包运行

### 安装

```shell
npm install -g @modelcontextprotocol/server-everything@latest
```

### 运行默认（stdio）服务

```shell
npx @modelcontextprotocol/server-everything
```

### 或显式指定 stdio

```shell
npx @modelcontextprotocol/server-everything stdio
```

### 运行 SSE 服务

```shell
npx @modelcontextprotocol/server-everything sse
```

### 运行 streamable HTTP 服务

```shell
npx @modelcontextprotocol/server-everything streamableHttp
```
