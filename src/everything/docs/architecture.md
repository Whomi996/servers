# Everything Server - 架构

**架构
| [项目结构](structure.md)
| [启动流程](startup.md)
| [服务特性](features.md)
| [扩展点](extension.md)
| [工作机制](how-it-works.md)**

本文档概述了 `src/everything` 包当前的布局与运行时架构。
它说明了服务如何启动、传输层如何接线、tools/prompts/resources 在哪里注册，以及如何扩展系统。

## 高层概览

### 目标

一个最小、模块化的 MCP 服务器，用于展示 Model Context Protocol 的核心能力。它暴露基础工具、提示词和资源，并支持多种传输（STDIO、SSE、Streamable HTTP）。

### 设计

使用小型“server factory”来构建 MCP 服务并注册能力。
传输层作为独立入口，负责创建/连接服务以及网络通信。
tools、prompts、resources 分别按子模块组织。

### 多客户端

服务器支持多个并发客户端。通过资源订阅与模拟日志，演示按 session 维度跟踪数据。

## 构建与分发

- TypeScript 源码通过 `npm run build` 编译到 `dist/`
- `build` 脚本会把 `docs/` 复制到 `dist/`，让说明文档随编译产物一起发布
- CLI 可执行入口在 `package.json` 中配置为 `mcp-server-everything` → `dist/index.js`

## [项目结构](structure.md)

## [启动流程](startup.md)

## [服务特性](features.md)

## [扩展点](extension.md)

## [工作机制](how-it-works.md)
