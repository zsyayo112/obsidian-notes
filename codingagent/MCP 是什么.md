---
tags:
  - agent开发
  - 盲区补课
  - 协议
created: 2026-07-13
---

# 🔌 MCP 是什么

> [!important] 一句话
> **MCP（Model Context Protocol）是 Anthropic 提出的开放协议，标准化"LLM 应用怎么连外部工具/数据/上下文"。可以理解成"AI 工具的 USB-C"：工具写成一个 MCP server，任何 MCP 客户端都能用，不用每个客户端单独对接。**

## 核心概念
- **客户端 Client**：LLM 应用（如 Claude Desktop、某些 IDE）。
- **服务器 Server**：暴露能力的进程，提供三类东西：
  - **Tools**：模型可调用的函数
  - **Resources**：可读取的数据
  - **Prompts**：预设提示词模板
- **传输**：JSON-RPC，走 stdio 或 HTTP/SSE。

## 好处
**写一次工具，到处能用**：工具做成 MCP server，Claude Desktop、IDE、任何 MCP 兼容客户端都能接上，省掉 N×M 的定制对接。

## 和 RepoPilot 的关系（呼应你最初的"mcp"提问）
因为 [[薄壳原则 接口与核心解耦|薄壳原则]]，RepoPilot 的 `solve()` 和各个工具跟"命令行"毫无耦合。所以完全可以**把它们暴露成一个 MCP server**——那样 Claude Desktop 或 IDE 就能直接调 RepoPilot 的能力，一行核心都不用改。这就是当初把 CLI 做薄的红利。

## 面试话术
> "MCP 是 Anthropic 的开放协议，标准化 LLM 应用怎么暴露工具/数据/提示词给客户端，像'AI 工具的 USB-C'——工具写成 server，任何 MCP 客户端都能用。我的项目是薄壳设计，核心逻辑在 solve() 里，所以能不动核心就包成一个 MCP server。"

关联：[[薄壳原则 接口与核心解耦]] · [[Agent 面试八股与答法]]
