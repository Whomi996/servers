# 获取 MCP 服务器

<!-- mcp-name: io.github.modelcontextprotocol/server-fetch -->

一个提供网页内容抓取能力的 Model Context Protocol 服务器。它可以让 LLM 抓取网页并处理内容，将 HTML 转换为更易消费的 Markdown。

> [!警告]
> 此服务器可以访问本地/内网 IP 地址，可能带来安全风险。使用时请谨慎，避免泄露敏感数据。

`fetch` 工具会截断返回内容。你可以通过 `start_index` 指定从哪个字符位置开始提取，从而让模型按分块方式读取网页，直到定位到需要的信息。

### 可用工具

- `fetch` - 从互联网抓取 URL，并将内容提取为 Markdown。
  - `url`（string，必填）：目标 URL
  - `max_length`（integer，可选）：最大返回字符数（默认 `5000`）
  - `start_index`（integer，可选）：从该字符索引开始返回（默认 `0`）
  - `raw`（boolean，可选）：返回原始内容，不做 Markdown 转换（默认 `false`）

### 提示

- **拿来**
  - 抓取 URL 并提取为 Markdown
  - 参数：
    - `url`（string，必填）：目标 URL

## 安装

可选：安装 Node.js。安装后 fetch server 会使用更稳健的 HTML 简化器。

### 使用 uv（推荐）

使用 [`uv`](https://docs.astral.sh/uv/) 时通常无需额外安装，直接通过 [`uvx`](https://docs.astral.sh/uv/guides/tools/) 运行 *mcp-server-fetch*。

### 使用 PIP

也可以通过 pip 安装 `mcp-server-fetch`：

```bash
pip install mcp-server-fetch
```

安装后可运行：

```bash
python -m mcp_server_fetch
```

## 配置

### 配置 Claude.app

将以下内容添加到 Claude 配置：

<details>
<summary>使用 uvx</summary>

```json
{
  "mcpServers": {
    "fetch": {
      "command": "uvx",
      "args": ["mcp-server-fetch"]
    }
  }
}
```
</details>

<details>
<summary>使用 docker</summary>

```json
{
  "mcpServers": {
    "fetch": {
      "command": "docker",
      "args": ["run", "-i", "--rm", "mcp/fetch"]
    }
  }
}
```
</details>

<details>
<summary>使用 pip 安装版本</summary>

```json
{
  "mcpServers": {
    "fetch": {
      "command": "python",
      "args": ["-m", "mcp_server_fetch"]
    }
  }
}
```
</details>

### 配置 VS Code

快速安装可使用以下按钮：

[![Install with UV in VS Code](https://img.shields.io/badge/VS_Code-UV-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=fetch&config=%7B%22command%22%3A%22uvx%22%2C%22args%22%3A%5B%22mcp-server-fetch%22%5D%7D) [![Install with UV in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-UV-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=fetch&config=%7B%22command%22%3A%22uvx%22%2C%22args%22%3A%5B%22mcp-server-fetch%22%5D%7D&quality=insiders)

[![Install with Docker in VS Code](https://img.shields.io/badge/VS_Code-Docker-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=fetch&config=%7B%22command%22%3A%22docker%22%2C%22args%22%3A%5B%22run%22%2C%22-i%22%2C%22--rm%22%2C%22mcp%2Ffetch%22%5D%7D) [![Install with Docker in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Docker-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=fetch&config=%7B%22command%22%3A%22docker%22%2C%22args%22%3A%5B%22run%22%2C%22-i%22%2C%22--rm%22%2C%22mcp%2Ffetch%22%5D%7D&quality=insiders)

手动安装时，将对应 JSON 添加到 VS Code 用户设置 JSON（`Ctrl + Shift + P`，执行 `Preferences: Open User Settings (JSON)`）。

也可以写到工作区 `.vscode/mcp.json` 以便共享。

> 使用 `mcp.json` 时需要包含 `mcp` 顶层键。

<details>
<summary>使用 uvx</summary>

```json
{
  "mcp": {
    "servers": {
      "fetch": {
        "command": "uvx",
        "args": ["mcp-server-fetch"]
      }
    }
  }
}
```
</details>

<details>
<summary>使用 Docker</summary>

```json
{
  "mcp": {
    "servers": {
      "fetch": {
        "command": "docker",
        "args": ["run", "-i", "--rm", "mcp/fetch"]
      }
    }
  }
}
```
</details>

### 自定义 - robots.txt

默认情况下：若请求由模型通过工具发起，服务器会遵守目标站点的 `robots.txt`；若请求由用户通过 prompt 发起，则不受该限制。  
可在配置 `args` 中添加 `--ignore-robots-txt` 关闭此行为。

### 自定义 - User-Agent

根据请求来源不同（模型工具调用 vs 用户主动请求），默认使用以下 User-Agent 之一：

```text
ModelContextProtocol/1.0 (Autonomous; +https://github.com/modelcontextprotocol/servers)
```

或

```text
ModelContextProtocol/1.0 (User-Specified; +https://github.com/modelcontextprotocol/servers)
```

可通过在 `args` 中添加 `--user-agent=YourUserAgent` 覆盖。

### 自定义 - 代理

可使用 `--proxy-url` 参数配置代理。

## Windows 配置

如果你在 Windows 上遇到超时问题，可能需要设置 `PYTHONIOENCODING` 环境变量以确保字符编码正常：

<details>
<summary>Windows 配置（uvx）</summary>

```json
{
  "mcpServers": {
    "fetch": {
      "command": "uvx",
      "args": ["mcp-server-fetch"],
      "env": {
        "PYTHONIOENCODING": "utf-8"
      }
    }
  }
}
```
</details>

<details>
<summary>Windows 配置（pip）</summary>

```json
{
  "mcpServers": {
    "fetch": {
      "command": "python",
      "args": ["-m", "mcp_server_fetch"],
      "env": {
        "PYTHONIOENCODING": "utf-8"
      }
    }
  }
}
```
</details>

该设置可避免字符编码问题导致的超时。

## 调试

可以使用 MCP inspector 调试。uvx 场景：

```bash
npx @modelcontextprotocol/inspector uvx mcp-server-fetch
```

若在本地开发或指定目录安装：

```bash
cd path/to/servers/src/fetch
npx @modelcontextprotocol/inspector uv run mcp-server-fetch
```

## 贡献

欢迎贡献以扩展和改进 mcp-server-fetch。你可以新增工具、增强现有功能或改进文档。

可参考其他 MCP 服务器实现：
https://github.com/modelcontextprotocol/servers

欢迎提交 PR。

## 许可证

mcp-server-fetch 使用 MIT 许可证。你可以在遵守 MIT 条款的前提下自由使用、修改和分发软件。详见项目仓库中的 LICENSE。
