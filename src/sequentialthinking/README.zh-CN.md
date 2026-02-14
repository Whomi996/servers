# Sequential Thinking MCP Server

一个 MCP 服务器实现，提供结构化思考工具，用于动态、可反思的问题求解。

## 功能特性

- 将复杂问题拆解为可管理步骤
- 随着理解加深不断修订和完善思路
- 支持分支推理路径
- 可动态调整总思考步数
- 生成并验证解题假设

## 工具

### sequential_thinking

提供详细的逐步思考流程，用于问题求解与分析。

**输入参数：**
- `thought`（string）：当前思考内容
- `nextThoughtNeeded`（boolean）：是否需要下一步思考
- `thoughtNumber`（integer）：当前思考序号
- `totalThoughts`（integer）：预估总思考步数
- `isRevision`（boolean，可选）：是否在修订之前的思考
- `revisesThought`（integer，可选）：正在修订哪一步
- `branchFromThought`（integer，可选）：从哪一步分支
- `branchId`（string，可选）：分支标识
- `needsMoreThoughts`（boolean，可选）：是否需要追加更多思考

## 使用场景

Sequential Thinking 适合：

- 将复杂问题拆解成步骤
- 需要可修订空间的规划与设计
- 可能中途调整方向的分析任务
- 初始时难以明确完整范围的问题
- 需要跨多步保持上下文的任务
- 需要过滤无关信息的场景

## 配置

### 在 Claude Desktop 中使用

将以下配置添加到 `claude_desktop_config.json`：

#### npx

```json
{
  "mcpServers": {
    "sequential-thinking": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-sequential-thinking"
      ]
    }
  }
}
```

#### docker

```json
{
  "mcpServers": {
    "sequentialthinking": {
      "command": "docker",
      "args": [
        "run",
        "--rm",
        "-i",
        "mcp/sequentialthinking"
      ]
    }
  }
}
```

如需关闭思考日志输出，设置环境变量 `DISABLE_THOUGHT_LOGGING=true`。

### 在 VS Code 中使用

快速安装可使用以下按钮：

[![Install with NPX in VS Code](https://img.shields.io/badge/VS_Code-NPM-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=sequentialthinking&config=%7B%22command%22%3A%22npx%22%2C%22args%22%3A%5B%22-y%22%2C%22%40modelcontextprotocol%2Fserver-sequential-thinking%22%5D%7D) [![Install with NPX in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-NPM-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=sequentialthinking&config=%7B%22command%22%3A%22npx%22%2C%22args%22%3A%5B%22-y%22%2C%22%40modelcontextprotocol%2Fserver-sequential-thinking%22%5D%7D&quality=insiders)

[![Install with Docker in VS Code](https://img.shields.io/badge/VS_Code-Docker-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=sequentialthinking&config=%7B%22command%22%3A%22docker%22%2C%22args%22%3A%5B%22run%22%2C%22--rm%22%2C%22-i%22%2C%22mcp%2Fsequentialthinking%22%5D%7D) [![Install with Docker in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Docker-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=sequentialthinking&config=%7B%22command%22%3A%22docker%22%2C%22args%22%3A%5B%22run%22%2C%22--rm%22%2C%22-i%22%2C%22mcp%2Fsequentialthinking%22%5D%7D&quality=insiders)

手动安装有两种方式：

**方法 1：用户级配置（推荐）**  
在命令面板（`Ctrl + Shift + P`）执行 `MCP: Open User Configuration`，在用户级 `mcp.json` 中添加配置。

**方法 2：工作区配置**  
在工作区新建 `.vscode/mcp.json` 并添加配置，便于团队共享。

> 详细说明见 [VS Code MCP 官方文档](https://code.visualstudio.com/docs/copilot/customization/mcp-servers)。

NPX 配置：

```json
{
  "servers": {
    "sequential-thinking": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-sequential-thinking"
      ]
    }
  }
}
```

Docker 配置：

```json
{
  "servers": {
    "sequential-thinking": {
      "command": "docker",
      "args": [
        "run",
        "--rm",
        "-i",
        "mcp/sequentialthinking"
      ]
    }
  }
}
```

### 在 Codex CLI 中使用

执行：

```bash
codex mcp add sequential-thinking npx -y @modelcontextprotocol/server-sequential-thinking
```

## 构建

Docker：

```bash
docker build -t mcp/sequentialthinking -f src/sequentialthinking/Dockerfile .
```

## 许可证

本 MCP 服务器使用 MIT 许可证。你可以在遵守许可证条款的前提下自由使用、修改和分发软件。详情见项目仓库中的 LICENSE 文件。
