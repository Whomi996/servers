# 时间 MCP 服务器

<!-- mcp-name: io.github.modelcontextprotocol/server-time -->

一个提供时间与时区转换能力的 Model Context Protocol 服务器。该服务器让 LLM 可以获取当前时间，并使用 IANA 时区名称进行时区转换，同时支持自动检测系统时区。

### 可用工具

- `get_current_time` - 获取指定时区（或系统时区）的当前时间。
  - 必填参数：
    - `timezone`（string）：IANA 时区名（例如 `America/New_York`、`Europe/London`）

- `convert_time` - 在两个时区之间进行时间转换。
  - 必填参数：
    - `source_timezone`（string）：源 IANA 时区名
    - `time`（string）：24 小时制时间（`HH:MM`）
    - `target_timezone`（string）：目标 IANA 时区名

## 安装

### 使用 uv（推荐）

使用 [`uv`](https://docs.astral.sh/uv/) 时通常无需额外安装。我们直接用 [`uvx`](https://docs.astral.sh/uv/guides/tools/) 运行 *mcp-server-time*。

```bash
uvx mcp-server-time
```

### 使用 PIP

也可以通过 pip 安装 `mcp-server-time`：

```bash
pip install mcp-server-time
```

安装后可通过以下方式启动：

```bash
python -m mcp_server_time
```

## 配置

### 配置 Claude.app

将以下内容添加到 Claude 配置中：

<details>
<summary>使用 uvx</summary>

```json
{
  "mcpServers": {
    "time": {
      "command": "uvx",
      "args": ["mcp-server-time"]
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
    "time": {
      "command": "docker",
      "args": ["run", "-i", "--rm", "-e", "LOCAL_TIMEZONE", "mcp/time"]
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
    "time": {
      "command": "python",
      "args": ["-m", "mcp_server_time"]
    }
  }
}
```
</details>

### 配置 Zed

将以下内容添加到 Zed 的 `settings.json`：

<details>
<summary>使用 uvx</summary>

```json
"context_servers": [
  "mcp-server-time": {
    "command": "uvx",
    "args": ["mcp-server-time"]
  }
],
```
</details>

<details>
<summary>使用 pip 安装版本</summary>

```json
"context_servers": {
  "mcp-server-time": {
    "command": "python",
    "args": ["-m", "mcp_server_time"]
  }
},
```
</details>

### 配置 VS Code

快速安装可使用以下一键安装按钮：

[![Install with UV in VS Code](https://img.shields.io/badge/VS_Code-UV-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=time&config=%7B%22command%22%3A%22uvx%22%2C%22args%22%3A%5B%22mcp-server-time%22%5D%7D) [![Install with UV in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-UV-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=time&config=%7B%22command%22%3A%22uvx%22%2C%22args%22%3A%5B%22mcp-server-time%22%5D%7D&quality=insiders)

[![Install with Docker in VS Code](https://img.shields.io/badge/VS_Code-Docker-0098FF?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=time&config=%7B%22command%22%3A%22docker%22%2C%22args%22%3A%5B%22run%22%2C%22-i%22%2C%22--rm%22%2C%22mcp%2Ftime%22%5D%7D) [![Install with Docker in VS Code Insiders](https://img.shields.io/badge/VS_Code_Insiders-Docker-24bfa5?style=flat-square&logo=visualstudiocode&logoColor=white)](https://insiders.vscode.dev/redirect/mcp/install?name=time&config=%7B%22command%22%3A%22docker%22%2C%22args%22%3A%5B%22run%22%2C%22-i%22%2C%22--rm%22%2C%22mcp%2Ftime%22%5D%7D&quality=insiders)

手动安装时，将以下 JSON 添加到 VS Code 的用户设置 JSON（`Ctrl + Shift + P` 后执行 `Preferences: Open User Settings (JSON)`）。

也可以放到工作区的 `.vscode/mcp.json` 中，以便共享给团队。

> 使用 `mcp.json` 文件时需要包含 `mcp` 顶层键。

<details>
<summary>使用 uvx</summary>

```json
{
  "mcp": {
    "servers": {
      "time": {
        "command": "uvx",
        "args": ["mcp-server-time"]
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
      "time": {
        "command": "docker",
        "args": ["run", "-i", "--rm", "mcp/time"]
      }
    }
  }
}
```
</details>

### 配置 Zencoder

1. 打开 Zencoder 菜单（...）
2. 下拉中选择 `Agent Tools`
3. 点击 `Add Custom MCP`
4. 按下方配置填入名称与服务配置，并点击 `Install`

<details>
<summary>使用 uvx</summary>

```json
{
  "command": "uvx",
  "args": ["mcp-server-time"]
}
```
</details>

### 自定义 - 系统时区

默认情况下服务器会自动检测系统时区。你可以在配置 `args` 中添加 `--local-timezone` 覆盖该行为。

示例：

```json
{
  "command": "python",
  "args": ["-m", "mcp_server_time", "--local-timezone=America/New_York"]
}
```

## 交互示例

1. 获取当前时间：

```json
{
  "name": "get_current_time",
  "arguments": {
    "timezone": "Europe/Warsaw"
  }
}
```

返回：

```json
{
  "timezone": "Europe/Warsaw",
  "datetime": "2024-01-01T13:00:00+01:00",
  "is_dst": false
}
```

2. 时区转换：

```json
{
  "name": "convert_time",
  "arguments": {
    "source_timezone": "America/New_York",
    "time": "16:30",
    "target_timezone": "Asia/Tokyo"
  }
}
```

返回：

```json
{
  "source": {
    "timezone": "America/New_York",
    "datetime": "2024-01-01T12:30:00-05:00",
    "is_dst": false
  },
  "target": {
    "timezone": "Asia/Tokyo",
    "datetime": "2024-01-01T12:30:00+09:00",
    "is_dst": false
  },
  "time_difference": "+13.0h"
}
```

## 调试

可以使用 MCP inspector 调试服务。uvx 安装时：

```bash
npx @modelcontextprotocol/inspector uvx mcp-server-time
```

如果你在本地开发或在指定目录安装了包：

```bash
cd path/to/servers/src/time
npx @modelcontextprotocol/inspector uv run mcp-server-time
```

## 可向 Claude 提问的示例

1. “现在几点了？”（使用系统时区）
2. “东京现在几点？”
3. “纽约下午 4 点时，伦敦是几点？”
4. “把东京上午 9:30 换算成纽约时间”

## 构建

Docker 构建：

```bash
cd src/time
docker build -t mcp/time .
```

## 贡献

欢迎贡献以扩展和改进 mcp-server-time。无论是新增时间相关工具、增强现有功能还是改进文档，都非常有价值。

可参考其他 MCP 服务器实现模式：
https://github.com/modelcontextprotocol/servers

欢迎提交 PR。

## 许可证

mcp-server-time 使用 MIT 许可证。你可以在遵守 MIT 条款的前提下自由使用、修改和分发本软件。详情见仓库中的 LICENSE 文件。
