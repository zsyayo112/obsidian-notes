---
tags:
  - agent开发
  - 盲区补课
  - 框架
created: 2026-07-13
---

# 🕸️ LangGraph 是什么

> [!important] 一句话
> **LangGraph 是 LangChain 团队做的框架，把 agent 的流程建模成一张"状态图"。核心四词：节点、边、共享状态、checkpoint。它卖的就是我 orchestrator 手写的那张状态机——外加持久化。**

## 核心四词（面试能复述就够）
- **节点 Node**：一个步骤/函数（比如"调模型""跑工具"）。
- **边 Edge**：节点间的转移；**条件边**可以根据状态决定下一步走哪（等于 if 分支）。支持**环**（循环）。
- **状态 State**：一个在节点间传递的共享对象（通常 TypedDict），带 reducer 合并更新。
- **Checkpoint**：把状态存盘 → **可中断、可恢复、可回放（time-travel）**、可 human-in-the-loop。

## 和 RepoPilot 的对照
| | 我的 orchestrator（手写）| LangGraph |
|--|--|--|
| 状态 + 转移 | `while` + `state` 变量 + if 分支 | 节点 + 边 + State 对象 |
| 持久化/续跑 | **没有**（跑一半崩了要重来）| checkpoint 内置 |
| 可调试性 | 报错直接定位到我的行 | 要穿过框架封装 |
| 代码量 | 几十行 | import 一个框架 |

## 面试话术
> "LangGraph = 节点+边+共享状态+checkpoint，把 agent 流程建成状态图。我的 orchestrator 就是同一个思想的手写版——少了持久化那层。我这规模手写更透、可调试；真需要'长任务中断续跑'，加个存盘也就 50 行。团队生产环境我会专门评估它的 checkpoint 层，但工具和 prompt 保持显式。"

姿态：不贬低，"理解权衡、做了选择"。见 [[手写状态机]] · [[RepoPilot 面试语料]]。

关联：[[手写状态机]] · [[Agent 面试八股与答法]]
