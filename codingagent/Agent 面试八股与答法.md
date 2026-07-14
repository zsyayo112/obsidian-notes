---
tags:
  - agent开发
  - 面试
  - 八股
created: 2026-07-13
---

# 📝 Agent 面试八股与答法

> [!important] 用法
> 每道题都能从 RepoPilot / L0–L9 里找到"我做过"的证据。**练到每个短语背后的"为什么"都能张口就来**——面试的追问永远是"为什么？"。

## 高频题 × 从项目哪里答

| 八股题 | 我从哪答（证据）|
|---|---|
| Agent 的本质是什么 | LLM + while 循环 + 工具；`executor.py` 就是实证（[[L2 循环即Agent]]）|
| 工具调用底层怎么发生 | 模型只吐结构化意图，我的代码在 `toolkit.execute` 真执行（[[L1 工具调用的真相]]）|
| ReAct vs Plan-and-Execute | 两个都做了：planner 先出计划，executor 再 ReAct 循环 |
| 怎么防幻觉 / 怎么确认真修好 | [[验证是灵魂]]：测试基线对比，代码当法官不是模型 |
| 怎么防改出回归 | `verifier.compare` 先查 new_failures，回归优先于一切 |
| 上下文爆炸怎么办 | compact 压缩 + 子 agent 隔离 + read 行号分段（[[L4 上下文管理]]）|
| 多 agent / 何时委派 | L8 并行 2.9×、L9 隔离 6.6× 上下文差（[[L9 何时委派]]）|
| 怎么保证 agent 安全 | 分层：jail + 黑名单 + 权限 + git 回滚；诚实说天花板是 Docker（[[硬约束vs软约束]]）|
| 结构化输出怎么做 | JSON 模式 / function calling + 解析失败喂回重试（`llm.py`）|
| 工具出错怎么处理 | [[错误即信息]]：变字符串喂回让模型自愈，不崩 |
| 怎么评测 agent | 对 `run.jsonl` 聚合：成功率看 report.ok，工具数看 tool 事件；SWE-bench Lite |
| 讲一个调过的 bug | 金牌故事：jail 剥前缀在 `tinydb/tinydb/` 炸掉（见 [[RepoPilot 面试语料]]）|
| 框架无关怎么做到 | `adapters` detector 注册表；核心只 log kind 从不 branch，可现场 grep 自证 |
| 怎么用 LLM 当裁判 | [[隔离即去偏见]]：只喂客观证据、屏蔽被审对象自述 |
| 怎么控制 agent 成本 | [[廉价假设加后续验证]]：Plan 只吃固定小常数上下文 |
| 怎么防 agent 失控 | MAX_TURNS/MAX_MODIFIED_FILES 等硬约束（代码兜底非提示词）|
| agent 主循环怎么写 | 问→有工具执行喂回→无工具收工 + MAX_TURNS + 流式碎片重组 |
| 为什么不用框架 | 见 [[RepoPilot 面试语料]] 的"框架三句话" |
| MCP 是什么 | 给客户端挂工具的协议；薄壳架构下 `solve()` 可包成 MCP server |

## 待补的盲区（面试前补齐）
1. **能可信复述 LangGraph**（节点/边/状态/checkpoint 四个词）
2. **一个真实评测数字**（SWE-bench Lite 跑几条）
3. **RAG/向量词汇**（我用 ripgrep 词法检索，能说清何时才上 embedding）

关联：[[RepoPilot 面试语料]] · [[RepoPilot MOC]]
