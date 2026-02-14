# Knowledge Graph Memory Server

一个基于本地知识图谱的持久化记忆基础实现。它可以让 Claude 在多轮对话之间记住用户信息。

## 核心概念

### Entities（实体）

实体是知识图谱中的核心节点。每个实体包含：
- 唯一名称（标识符）
- 实体类型（例如 `person`、`organization`、`event`）
- 一组观察信息（observations）

示例：

```json
{
  "name": "John_Smith",
  "entityType": "person",
  "observations": ["Speaks fluent Spanish"]
}
```

### Relations（关系）

关系用于定义实体之间的有向连接。关系始终使用主动语态，描述实体如何交互或关联。

示例：

```json
{
  "from": "John_Smith",
  "to": "Anthropic",
  "relationType": "works_at"
}
```

### Observations（观察）

观察是关于实体的离散信息片段，特点是：

- 以字符串存储
- 附着在具体实体上
- 可独立新增或删除
- 应尽量保持原子性（每条 observation 表达一个事实）

示例：

```json
{
  "entityName": "John_Smith",
  "observations": [
    "Speaks fluent Spanish",
    "Graduated in 2019",
    "Prefers morning meetings"
  ]
}
```

## API

### Tools

- **create_entities**
  - 在知识图谱中创建多个实体
  - 输入：`entities`（对象数组）
    - 每个对象包含：
      - `name`（string）：实体标识
      - `entityType`（string）：实体类型
      - `observations`（string[]）：关联观察
  - 同名实体会被忽略

- **create_relations**
  - 创建多个实体关系
  - 输入：`relations`（对象数组）
    - 每个对象包含：
      - `from`（string）：源实体名
      - `to`（string）：目标实体名
      - `relationType`（string）：主动语态关系类型
  - 重复关系会被跳过

- **add_observations**
  - 为已有实体追加观察信息
  - 输入：`observations`（对象数组）
    - 每个对象包含：
      - `entityName`（string）：目标实体
      - `contents`（string[]）：要添加的观察
  - 返回每个实体新增的观察内容
  - 若实体不存在则失败

- **delete_entities**
  - 删除实体及其关系
  - 输入：`entityNames`（string[]）
  - 会级联删除关联关系
  - 实体不存在时静默处理

- **delete_observations**
  - 删除实体上的指定观察
  - 输入：`deletions`（对象数组）
    - 每个对象包含：
      - `entityName`（string）：目标实体
      - `observations`（string[]）：要删除的观察
  - 观察不存在时静默处理

- **delete_relations**
  - 删除指定关系
  - 输入：`relations`（对象数组）
    - 每个对象包含：
      - `from`（string）：源实体名
      - `to`（string）：目标实体名
      - `relationType`（string）：关系类型
  - 关系不存在时静默处理

- **read_graph**
  - 读取完整知识图谱
  - 无需输入
  - 返回所有实体与关系

- **search_nodes**
  - 按查询词搜索节点
  - 输入：`query`（string）
  - 搜索范围：
    - 实体名
    - 实体类型
    - observation 内容
  - 返回匹配实体及其关系

- **open_nodes**
  - 按名称获取指定节点
  - 输入：`names`（string[]）
  - 返回：
    - 请求的实体
    - 这些实体之间的关系
  - 不存在的节点会被静默忽略

# 在 Claude Desktop 中使用

### 配置

将以下配置添加到 `claude_desktop_config.json`：

#### Docker

```json
{
  "mcpServers": {
    "memory": {
      "command": "docker",
      "args": ["run", "-i", "-v", "claude-memory:/app/dist", "--rm", "mcp/memory"]
    }
  }
}
```

#### NPX

```json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-memory"
      ]
    }
  }
}
```

#### NPX（自定义配置）

服务器支持以下环境变量：

```json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-memory"
      ],
      "env": {
        "MEMORY_FILE_PATH": "/path/to/custom/memory.jsonl"
      }
    }
  }
}
```

- `MEMORY_FILE_PATH`：记忆存储 JSONL 文件路径（默认是服务目录下的 `memory.jsonl`）

# VS Code 安装说明

可使用以下一键安装按钮：

[![Install with NPX in VS Code](https://img.shields.io/badge/VS_Code-NPM-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=memory&config=%7B%22command%22%3A%22npx%22%2C%22args%22%3A%5B%22-y%22%2C%22%40modelcontextprotocol%2Fserver-memory%22%5D%7D) [![Install with NPX in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-NPM-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=memory&config=%7B%22command%22%3A%22npx%22%2C%22args%22%3A%5B%22-y%22%2C%22%40modelcontextprotocol%2Fserver-memory%22%5D%7D&quality=insiders)

[![Install with Docker in VS Code](https://img.shields.io/badge/VS_Code-Docker-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=memory&config=%7B%22command%22%3A%22docker%22%2C%22args%22%3A%5B%22run%22%2C%22-i%22%2C%22-v%22%2C%22claude-memory%3A%2Fapp%2Fdist%22%2C%22--rm%22%2C%22mcp%2Fmemory%22%5D%7D) [![Install with Docker in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Docker-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=memory&config=%7B%22command%22%3A%22docker%22%2C%22args%22%3A%5B%22run%22%2C%22-i%22%2C%22-v%22%2C%22claude-memory%3A%2Fapp%2Fdist%22%2C%22--rm%22%2C%22mcp%2Fmemory%22%5D%7D&quality=insiders)

手动安装可选两种方式：

**方法 1：用户级配置（推荐）**  
通过命令面板（`Ctrl + Shift + P`）运行 `MCP: Open User Configuration`，在用户 `mcp.json` 中添加。

**方法 2：工作区配置**  
在工作区创建 `.vscode/mcp.json` 并添加配置。

> 更多细节请参考 [VS Code MCP 官方文档](https://code.visualstudio.com/docs/copilot/customization/mcp-servers)。

#### NPX

```json
{
  "servers": {
    "memory": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-memory"
      ]
    }
  }
}
```

#### Docker

```json
{
  "servers": {
    "memory": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "-v",
        "claude-memory:/app/dist",
        "--rm",
        "mcp/memory"
      ]
    }
  }
}
```

### System Prompt（示例）

如何使用 memory，很大程度取决于你的提示词。调整提示词可以帮助模型决定记忆的频率和类型。

下面是一个用于聊天个性化的示例提示词，可放到 [Claude.ai Project](https://www.anthropic.com/news/projects) 的 “Custom Instructions”：

```text
Follow these steps for each interaction:

1. User Identification:
   - You should assume that you are interacting with default_user
   - If you have not identified default_user, proactively try to do so.

2. Memory Retrieval:
   - Always begin your chat by saying only "Remembering..." and retrieve all relevant information from your knowledge graph
   - Always refer to your knowledge graph as your "memory"

3. Memory
   - While conversing with the user, be attentive to any new information that falls into these categories:
     a) Basic Identity (age, gender, location, job title, education level, etc.)
     b) Behaviors (interests, habits, etc.)
     c) Preferences (communication style, preferred language, etc.)
     d) Goals (goals, targets, aspirations, etc.)
     e) Relationships (personal and professional relationships up to 3 degrees of separation)

4. Memory Update:
   - If any new information was gathered during the interaction, update your memory as follows:
     a) Create entities for recurring organizations, people, and significant events
     b) Connect them to the current entities using relations
     c) Store facts about them as observations
```

## 构建

Docker：

```sh
docker build -t mcp/memory -f src/memory/Dockerfile .
```

注意：旧版 `mcp/memory` Docker volume 可能包含 `index.js`，新容器会覆盖该文件。如果你使用 volume 持久化，请在启动新容器前删除旧 volume 中的 `index.js`。

## 许可证

本 MCP 服务器使用 MIT 许可证。你可以在遵守 MIT 条款的前提下自由使用、修改和分发本软件。详情见项目仓库中的 LICENSE 文件。
