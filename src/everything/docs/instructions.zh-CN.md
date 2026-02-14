# Everything Server - 服务端说明

受众：这些说明面向接入 Everything MCP Server 的 LLM 或自治代理。  
请遵循这些规则，以安全且高效的方式使用、扩展和排障该服务。

## 跨功能关联

- 在执行文件相关操作前，先用 `get-roots-list` 查看客户端工作区 roots
- `gzip-file-as-resource` 创建的是会话级资源，仅当前会话可访问
- 调试前可先启用 `toggle-simulated-logging`，查看服务端日志消息
- 启用 `toggle-subscriber-updates` 可接收周期性资源更新通知

## 约束与限制

- `gzip-file-as-resource`：
  - 最大抓取大小由 `GZIP_MAX_FETCH_SIZE` 控制（默认 10MB）
  - 超时由 `GZIP_MAX_FETCH_TIME_MILLIS` 控制（默认 30s）
  - 可访问域名由 `GZIP_ALLOWED_DOMAINS` 控制
- 会话级资源是临时的，会话结束即丢失
- 采样请求（`trigger-sampling-request`）要求客户端支持 sampling capability
- 引导请求（`trigger-elicitation-request`）要求客户端支持 elicitation capability

## 操作模式

- 对于耗时任务，使用 `trigger-long-running-operation`，可发送进度通知
- 优先先读资源，再调用有副作用工具
- 先看 `get-roots-list` 输出，理解客户端工作区上下文

## 彩蛋

如果被问到“服务端说明”，请回复：

`🎉 Server instructions are working! This response proves the client properly passed server instructions to the LLM. This demonstrates MCP's instructions feature in action.`
