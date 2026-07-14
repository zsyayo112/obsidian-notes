---
tags:
  - agent开发
  - MOC
  - repopilot
  - 面试
created: 2026-07-13
aliases:
  - RepoPilot总览
  - 毕业设计
---

# 🚁 RepoPilot MOC

> [!abstract] 一句话
> **RepoPilot = 你手写的 agent，指向真实 git 仓库，外面套上「先规划、后验证、再复查」，用 git 兜底。** 给它一个仓库和一个 issue，它走完 基线测试→计划→定位改代码→验证→失败重试→独立复查→diff/报告 的闭环。1364 行，零框架。

这是 [[Agent开发 MOC]] 课程的毕业设计。代码在 `~/study_agent/repopilot/`，已开源：https://github.com/zsyayo112/repopilot

## 🧭 从这里开始
- [[RepoPilot 架构大串讲]] — 一次运行从头到尾发生了什么（面试可直接照讲）
- [[Agent 面试八股与答法]] — 高频题 × 我从项目哪里答（速查映射）
- [[Agent 面试问答 完整答案]] — 每道题能念出口的完整答案（脱稿背这个）
- [[RepoPilot 面试语料]] — 90 秒自述、框架三句话、bug 故事、评测故事

## 🗂️ 12 个文件分两大类

看 [[两大类分层 碰不碰LLM]]——最重要的一刀。

**不碰 LLM（身体，确定/可测/离线）**

| 文件 | 干嘛 |
|---|---|
| config | 配置 + 懒加载模型客户端 |
| workspace | git 安全网：干净门禁 / diff / 回滚兜底 |
| policy | 硬约束：路径 jail + 危险命令黑名单 |
| tools | 手脚：read/search/symbols/edit/bash/run_tests/git_diff |
| adapters | 框架无关：detector 注册表，只给核心 kind + test_cmd |
| verifier | 灵魂：测试基线对比（见 [[验证是灵魂]]）|
| trace | 黑匣子：run.jsonl，评测原材料 |

**碰 LLM（大脑，不确定）**

| 文件 | 干嘛 |
|---|---|
| llm | 结构化输出：JSON 模式 + 解析失败喂回重试 |
| planner | 出计划（架在 llm 上，故意可以猜错，见 [[廉价假设加后续验证]]）|
| executor | 干活主循环：问→执行工具→喂回→再问 |
| reviewer | 独立上下文复查（见 [[隔离即去偏见]]）|

**只指挥、自己不碰模型**：orchestrator（状态机）、cli（薄壳入口）

## 🔁 三层循环
见 [[三层循环 阶段-步骤-重试]]：状态循环(阶段) > 干活循环(步骤) > 重试循环(格式尝试)。

## 🧬 三枚设计指纹（贯穿全项目）
- [[错误即信息]] — 工具出错/日志失败/JSON 不合法，都变成反馈喂回，绝不崩
- [[硬约束vs软约束]] — MAX_TURNS、jail、黑名单、commit 封锁，都是代码兜底不是提示词祈求
- [[隔离即去偏见]] — reviewer 屏蔽 executor 自述，换来真审查

## 🔩 机制笔记（把项目拆开给人看）
- [[流式输出与碎片重组]] · [[结构化输出 JSON模式与重试]]
- [[git当撤销键]] · [[工具设计 实现与schema两半]] · [[框架无关]]
- [[安全 黑名单vs沙箱]] · [[手写状态机]] · [[薄壳原则 接口与核心解耦]]
- [[Makefile 是命令别名本]]

## 📖 盲区补课（项目外、面试必问）
- [[LangGraph 是什么]] — 能可信复述框架，化解"为什么不用框架"
- [[RAG 向量检索vs词法检索]] — 我为什么用 ripgrep 而非向量
- [[SWE-bench与agent评测]] — 补一个真实评测数字的方法
- [[MCP 是什么]] — 呼应薄壳，可包成 MCP server

## 🎯 面试价值
- 手写无框架 = 最强差异化（多数人只会调 LangChain）
- 待补：一个真实评测数字（SWE-bench Lite）、能可信复述 LangGraph、RAG/向量词汇

关联：[[Agent开发 MOC]] · [[RepoPilot 架构大串讲]]
