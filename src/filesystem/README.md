# 文件系统 MCP 服务器

一个基于 Node.js 的 Model Context Protocol（MCP）文件系统服务器实现。

## 特性

- 读写文件
- 创建/列出/删除目录
- 移动文件/目录
- 搜索文件
- 获取文件元数据
- 基于 [Roots](https://modelcontextprotocol.io/docs/learn/client-concepts#roots) 的动态目录访问控制

## 目录访问控制

服务端使用灵活的目录访问控制机制。允许访问的目录可以通过命令行参数指定，也可以通过 [Roots](https://modelcontextprotocol.io/docs/learn/client-concepts#roots) 动态提供。

### 方式 1：命令行参数

启动时传入允许目录：

```bash
mcp-server-filesystem /path/to/dir1 /path/to/dir2
```

### 方式 2：MCP Roots（推荐）

支持 Roots 协议的 MCP 客户端可以动态更新允许目录。

当客户端通过 Roots 通知目录后，会**完全替换**服务端现有允许目录。

**重要**：如果服务端启动时没有命令行目录，同时客户端也不支持 roots（或返回空 roots），服务端会在初始化时抛错。

这是推荐方式，因为你可以通过 `roots/list_changed` 在运行时更新目录，无需重启服务，更灵活。

### 工作流程

1. **服务端启动**
   - 如提供命令行参数，先使用参数中的目录
   - 如未提供参数，则允许目录初始为空

2. **客户端连接与初始化**
   - 客户端发送 `initialize`（含 capabilities）
   - 服务端检查客户端是否支持 roots（`capabilities.roots`）

3. **Roots 处理（客户端支持时）**
   - 初始化阶段：服务端调用 `roots/list` 请求 roots
   - 客户端返回其 roots
   - 服务端用客户端 roots 替换全部允许目录
   - 运行时：客户端可发送 `notifications/roots/list_changed`
   - 服务端再次请求 roots 并整体替换允许目录

4. **回退行为（客户端不支持 roots）**
   - 服务端继续仅使用命令行目录
   - 无法动态更新

5. **访问控制**
   - 所有文件系统操作都限制在允许目录内
   - 可通过 `list_allowed_directories` 查看当前允许目录
   - 服务端至少需要一个允许目录才能工作

**说明**：服务端只允许在 `args` 或 Roots 指定的目录内进行操作。

## API

### Tools

- **读取文本文件**
  - 读取文本文件完整内容
  - 输入：
- `path`（字符串）
    - `head`（number，可选）：前 N 行
    - `tail`（number，可选）：后 N 行
  - 无论扩展名，均按 UTF-8 文本处理
  - `head` 与 `tail` 不能同时指定

- **读取媒体文件**
  - 读取图片或音频文件
  - 输入：
- `path`（字符串）
  - 流式读取并返回 base64 数据和 MIME 类型

- **读取多个文件**
  - 同时读取多个文件
  - 输入：`paths`（string[]）
  - 单文件失败不会中断整体调用

- **写入文件**
  - 新建文件或覆盖已有文件（请谨慎）
  - 输入：
- `path`（字符串）
- `content`（字符串）

- **编辑文件**
  - 通过高级模式匹配执行选择性编辑
  - 特性：
    - 支持单行/多行匹配
    - 空白标准化并保留缩进
    - 多处编辑自动定位
    - 自动识别并保留缩进风格
    - 返回带上下文的 git 风格 diff
    - 支持 dry run 预览
  - 输入：
- `path`（字符串）
- `edits`（数组）
      - `oldText`（string）：待匹配文本（可为子串）
      - `newText`（string）：替换文本
    - `dryRun`（boolean）：仅预览不落盘（默认 `false`）
  - `dryRun` 时返回匹配与差异信息；否则直接应用
  - 最佳实践：先用 `dryRun` 预览再执行

- **创建目录**
  - 创建目录（若已存在则保持不变）
  - 输入：`path`（string）
  - 可自动创建父目录

- **列表目录**
  - 列出目录内容（带 `[FILE]` / `[DIR]` 前缀）
  - 输入：`path`（string）

- **list_directory_with_size**
  - 列出目录内容并显示大小
  - 输入：
- `path`（字符串）
    - `sortBy`（string，可选）：`name` 或 `size`（默认 `name`）
  - 返回大小明细与汇总统计（文件数、目录数、总大小）

- **移动文件**
  - 移动/重命名文件和目录
  - 输入：
- `source`（字符串）
- `destination`（字符串）
  - 若目标已存在则失败

- **搜索文件**
  - 递归搜索文件/目录（支持排除模式）
  - 输入：
    - `path`（string）：起始目录
    - `pattern`（string）：匹配模式
    - `excludePatterns`（string[]）：排除模式
  - 使用 glob 风格匹配
  - 返回匹配项完整路径

- **目录树**
  - 获取目录递归 JSON 树
  - 输入：
- `path`（字符串）
- `excludePatterns`（字符串[]）
  - 返回：
    - 数组条目包含：
- `name`（字符串）
- `type`（`file` / `directory`）
      - `children`（仅目录有）
        - 空目录为 `[]`
        - 文件省略该字段
  - 输出使用 2 空格缩进

- **获取文件信息**
  - 获取文件/目录详细元数据
  - 输入：`path`（string）
  - 返回：
    - 大小
    - 创建时间
    - 修改时间
    - 访问时间
    - 类型（文件/目录）
    - 权限

- **列出允许的目录**
  - 列出当前服务可访问目录
  - 无输入
  - 返回当前可读写目录列表

### Tool annotations（MCP 提示）

本服务为每个工具设置了 [MCP ToolAnnotations](https://modelcontextprotocol.io/specification/2025-03-26/server/tools#toolannotations)，让客户端可以：

- 区分只读工具和写操作工具
- 识别哪些写操作是幂等的（可重试）
- 标记可能具有破坏性的操作（覆盖或重度修改）

映射如下：

| Tool                        | readOnlyHint | idempotentHint | destructiveHint | 说明 |
|-----------------------------|--------------|----------------|-----------------|------|
| `read_text_file`            | `true`       | –              | –               | 纯读取 |
| `read_media_file`           | `true`       | –              | –               | 纯读取 |
| `read_multiple_files`       | `true`       | –              | –               | 纯读取 |
| `list_directory`            | `true`       | –              | –               | 纯读取 |
| `list_directory_with_sizes` | `true`       | –              | –               | 纯读取 |
| `directory_tree`            | `true`       | –              | –               | 纯读取 |
| `search_files`              | `true`       | –              | –               | 纯读取 |
| `get_file_info`             | `true`       | –              | –               | 纯读取 |
| `list_allowed_directories`  | `true`       | –              | –               | 纯读取 |
| `create_directory`          | `false`      | `true`         | `false`         | 重复创建同目录无副作用 |
| `write_file`                | `false`      | `true`         | `true`          | 会覆盖已有文件 |
| `edit_file`                 | `false`      | `false`        | `true`          | 重复应用可能失败或重复改动 |
| `move_file`                 | `false`      | `false`        | `false`         | 移动/重命名，重复执行通常报错 |

> 注：根据 MCP 规范，`idempotentHint` 与 `destructiveHint` 仅在 `readOnlyHint=false` 时有意义。

## 在 Claude Desktop 中使用

将以下内容添加到 `claude_desktop_config.json`。

说明：可将沙箱目录挂载到 `/projects`。加 `ro` 标记即只读挂载。

### 码头工人

注意：默认需要把所有目录挂载到 `/projects` 下。

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "--mount", "type=bind,src=/Users/username/Desktop,dst=/projects/Desktop",
        "--mount", "type=bind,src=/path/to/other/allowed/dir,dst=/projects/other/allowed/dir,ro",
        "--mount", "type=bind,src=/path/to/file.txt,dst=/projects/path/to/file.txt",
        "mcp/filesystem",
        "/projects"
      ]
    }
  }
}
```

### NPX

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/username/Desktop",
        "/path/to/other/allowed/dir"
      ]
    }
  }
}
```

## 在 VS Code 中使用

快速安装可使用以下按钮：

[![Install with NPX in VS Code](https://img.shields.io/badge/VS_Code-NPM-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=filesystem&config=%7B%22command%22%3A%22npx%22%2C%22args%22%3A%5B%22-y%22%2C%22%40modelcontextprotocol%2Fserver-filesystem%22%2C%22%24%7BworkspaceFolder%7D%22%5D%7D) [![Install with NPX in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-NPM-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=filesystem&config=%7B%22command%22%3A%22npx%22%2C%22args%22%3A%5B%22-y%22%2C%22%40modelcontextprotocol%2Fserver-filesystem%22%2C%22%24%7BworkspaceFolder%7D%22%5D%7D&quality=insiders)

[![Install with Docker in VS Code](https://img.shields.io/badge/VS_Code-Docker-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=filesystem&config=%7B%22command%22%3A%22docker%22%2C%22args%22%3A%5B%22run%22%2C%22-i%22%2C%22--rm%22%2C%22--mount%22%2C%22type%3Dbind%2Csrc%3D%24%7BworkspaceFolder%7D%2Cdst%3D%2Fprojects%2Fworkspace%22%2C%22mcp%2Ffilesystem%22%2C%22%2Fprojects%22%5D%7D) [![Install with Docker in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Docker-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=filesystem&config=%7B%22command%22%3A%22docker%22%2C%22args%22%3A%5B%22run%22%2C%22-i%22%2C%22--rm%22%2C%22--mount%22%2C%22type%3Dbind%2Csrc%3D%24%7BworkspaceFolder%7D%2Cdst%3D%2Fprojects%2Fworkspace%22%2C%22mcp%2Ffilesystem%22%2C%22%2Fprojects%22%5D%7D&quality=insiders)

手动安装可用两种方式：

**方法 1：用户级配置（推荐）**  
打开命令面板（`Ctrl + Shift + P`）执行 `MCP: Open User Configuration`，编辑用户级 `mcp.json`。

**方法 2：工作区配置**  
在工作区创建 `.vscode/mcp.json`，共享给团队。

> 详细说明见 [VS Code MCP 官方文档](https://code.visualstudio.com/docs/copilot/customization/mcp-servers)。

你也可以把沙箱目录挂载到 `/projects`，使用 `ro` 做只读挂载。

### Docker（手动）

```json
{
  "servers": {
    "filesystem": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "--mount", "type=bind,src=${workspaceFolder},dst=/projects/workspace",
        "mcp/filesystem",
        "/projects"
      ]
    }
  }
}
```

### NPX（手动）

```json
{
  "servers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "${workspaceFolder}"
      ]
    }
  }
}
```

## 构建

Docker 构建：

```bash
docker build -t mcp/filesystem -f src/filesystem/Dockerfile .
```

## 许可证

本 MCP 服务器使用 MIT 许可证。你可以在遵守 MIT 条款的前提下自由使用、修改和分发软件。详情见项目仓库中的 LICENSE 文件。
