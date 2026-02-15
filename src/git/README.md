# mcp-server-git：一个 Git MCP 服务器

<!-- mcp-name: io.github.modelcontextprotocol/server-git -->

## 概览

这是一个用于 Git 仓库交互与自动化的 Model Context Protocol 服务器。该服务器提供工具，让大语言模型可以读取、搜索并操作 Git 仓库。

请注意：mcp-server-git 目前仍处于早期开发阶段。功能和可用工具会持续变化与扩展。

### 工具列表

1.`git_status`
   - 显示工作区状态
   - 输入：
     - `repo_path`（string）：Git 仓库路径
   - 返回：工作目录当前状态文本

2.`git_diff_unstaged`
   - 显示尚未暂存的工作区改动
   - 输入：
- `repo_path`（字符串）
     - `context_lines`（number，可选）：上下文行数（默认 `3`）
   - 返回：未暂存 diff

3.`git_diff_staged`
   - 显示已暂存待提交改动
   - 输入：
- `repo_path`（字符串）
     - `context_lines`（number，可选，默认 `3`）
   - 返回：已暂存 diff

4.`git_diff`
   - 比较分支或提交之间的差异
   - 输入：
- `repo_path`（字符串）
     - `target`（string）：比较目标（分支或提交）
     - `context_lines`（number，可选，默认 `3`）
   - 返回：当前状态与目标的 diff

5.`git_commit`
   - 提交仓库改动
   - 输入：
- `repo_path`（字符串）
     - `message`（string）：提交信息
   - 返回：提交成功确认及新 commit hash

6.`git_add`
   - 将文件加入暂存区
   - 输入：
- `repo_path`（字符串）
     - `files`（string[]）：要暂存的文件路径数组
   - 返回：暂存成功确认

7.`git_reset`
   - 取消暂存所有已暂存改动
   - 输入：
- `repo_path`（字符串）
   - 返回：重置操作确认

8.`git_log`
   - 查看提交日志（支持日期过滤）
   - 输入：
- `repo_path`（字符串）
     - `max_count`（number，可选）：最多返回提交数（默认 `10`）
     - `start_timestamp`（string，可选）：起始时间（支持 ISO8601、相对时间、绝对日期）
     - `end_timestamp`（string，可选）：结束时间（同上）
   - 返回：包含 hash、作者、日期、消息的提交数组

9.`git_create_branch`
   - 创建新分支
   - 输入：
- `repo_path`（字符串）
- `branch_name`（字符串）
     - `base_branch`（string，可选）：起始分支（默认当前分支）
   - 返回：分支创建确认

10.`git_checkout`
   - 切换分支
   - 输入：
- `repo_path`（字符串）
- `branch_name`（字符串）
   - 返回：切换确认

11.`git_show`
   - 显示指定修订版本内容
   - 输入：
- `repo_path`（字符串）
- `revision`（字符串）：提交哈希/分支/标签
   - 返回：指定修订内容

12.`git_branch`
   - 列出分支
   - 输入：
- `repo_path`（字符串）
- `branch_type`（字符串）：`local` / `remote` / `all`
     - `contains`（string，可选）：需包含的 commit sha
     - `not_contains`（string，可选）：不得包含的 commit sha
   - 返回：分支列表

## 安装

### 使用 uv（推荐）

使用 [`uv`](https://docs.astral.sh/uv/) 时无需额外安装，直接使用 [`uvx`](https://docs.astral.sh/uv/guides/tools/) 运行 *mcp-server-git*。

### 使用 PIP

```bash
pip install mcp-server-git
python -m mcp_server_git
```

## 配置

### 在 Claude Desktop 中使用

将以下内容添加到 `claude_desktop_config.json`：

<details>
<summary>使用 uvx</summary>

```json
"mcpServers": {
  "git": {
    "command": "uvx",
    "args": ["mcp-server-git", "--repository", "path/to/git/repo"]
  }
}
```
</details>

<details>
<summary>使用 docker</summary>

> 注意：将 `/Users/username` 替换为你希望开放访问的真实路径。

```json
"mcpServers": {
  "git": {
    "command": "docker",
    "args": ["run", "--rm", "-i", "--mount", "type=bind,src=/Users/username,dst=/Users/username", "mcp/git"]
  }
}
```
</details>

<details>
<summary>使用 pip 安装版本</summary>

```json
"mcpServers": {
  "git": {
    "command": "python",
    "args": ["-m", "mcp_server_git", "--repository", "path/to/git/repo"]
  }
}
```
</details>

### 在 VS Code 中使用

快速安装按钮：

[![Install with UV in VS Code](https://img.shields.io/badge/VS_Code-UV-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=git&config=%7B%22command%22%3A%22uvx%22%2C%22args%22%3A%5B%22mcp-server-git%22%5D%7D) [![Install with UV in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-UV-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=git&config=%7B%22command%22%3A%22uvx%22%2C%22args%22%3A%5B%22mcp-server-git%22%5D%7D&quality=insiders)

[![Install with Docker in VS Code](https://img.shields.io/badge/VS_Code-Docker-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=git&config=%7B%22command%22%3A%22docker%22%2C%22args%22%3A%5B%22run%22%2C%22--rm%22%2C%22-i%22%2C%22--mount%22%2C%22type%3Dbind%2Csrc%3D%24%7BworkspaceFolder%7D%2Cdst%3D%2Fworkspace%22%2C%22mcp%2Fgit%22%5D%7D) [![Install with Docker in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Docker-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=git&config=%7B%22command%22%3A%22docker%22%2C%22args%22%3A%5B%22run%22%2C%22--rm%22%2C%22-i%22%2C%22--mount%22%2C%22type%3Dbind%2Csrc%3D%24%7BworkspaceFolder%7D%2Cdst%3D%2Fworkspace%22%2C%22mcp%2Fgit%22%5D%7D&quality=insiders)

手动安装：

**方法 1：用户级配置（推荐）**  
命令面板执行 `MCP: Open User Configuration`，在用户 `mcp.json` 添加。

**方法 2：工作区配置**  
在工作区创建 `.vscode/mcp.json` 共享配置。

> 详情见 [VS Code MCP 官方文档](https://code.visualstudio.com/docs/copilot/customization/mcp-servers)。

```json
{
  "servers": {
    "git": {
      "command": "uvx",
      "args": ["mcp-server-git"]
    }
  }
}
```

Docker 版本：

```json
{
  "mcp": {
    "servers": {
      "git": {
        "command": "docker",
        "args": [
          "run",
          "--rm",
          "-i",
          "--mount", "type=bind,src=${workspaceFolder},dst=/workspace",
          "mcp/git"
        ]
      }
    }
  }
}
```

### 在 [Zed](https://github.com/zed-industries/zed) 中使用

<details>
<summary>使用 uvx</summary>

```json
"context_servers": [
  "mcp-server-git": {
    "command": {
      "path": "uvx",
      "args": ["mcp-server-git"]
    }
  }
],
```
</details>

<details>
<summary>使用 pip 安装版本</summary>

```json
"context_servers": {
  "mcp-server-git": {
    "command": {
      "path": "python",
      "args": ["-m", "mcp_server_git"]
    }
  }
},
```
</details>

### 在 [Zencoder](https://zencoder.ai) 中使用

1. 打开 Zencoder 菜单（...）
2. 选择 `Agent Tools`
3. 点击 `Add Custom MCP`
4. 填写名称（如 `git`）和配置后点击 `Install`

<details>
<summary>使用 uvx</summary>

```json
{
  "command": "uvx",
  "args": ["mcp-server-git", "--repository", "path/to/git/repo"]
}
```
</details>

## 调试

可使用 MCP inspector：

```bash
npx @modelcontextprotocol/inspector uvx mcp-server-git
```

本地开发时：

```bash
cd path/to/servers/src/git
npx @modelcontextprotocol/inspector uv run mcp-server-git
```

可通过以下命令查看 Claude 日志辅助排障：

```bash
tail -n 20 -f ~/Library/Logs/Claude/mcp*.log
```

## 开发

本地改动可通过两种方式验证：

1. 用 MCP inspector 测试（见上方调试章节）。
2. 在 Claude Desktop 中联调（修改 `claude_desktop_config.json`）。

### Docker（开发配置示例）

```json
{
  "mcpServers": {
    "git": {
      "command": "docker",
      "args": [
        "run",
        "--rm",
        "-i",
        "--mount", "type=bind,src=/Users/username/Desktop,dst=/projects/Desktop",
        "--mount", "type=bind,src=/path/to/other/allowed/dir,dst=/projects/other/allowed/dir,ro",
        "--mount", "type=bind,src=/path/to/file.txt,dst=/projects/path/to/file.txt",
        "mcp/git"
      ]
    }
  }
}
```

### UVX（开发配置示例）

```json
{
"mcpServers": {
  "git": {
    "command": "uv",
    "args": [
      "--directory",
      "/<path to mcp-servers>/mcp-servers/src/git",
      "run",
      "mcp-server-git"
    ]
    }
  }
}
```

## 构建

Docker 构建：

```bash
cd src/git
docker build -t mcp/git .
```

## 许可证

本 MCP 服务器使用 MIT 许可证。你可以在遵守 MIT 条款的前提下自由使用、修改和分发软件。详情见项目仓库中的 LICENSE 文件。
