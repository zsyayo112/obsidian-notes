---
tags:
  - agent开发
  - 盲区补课
  - 评测
created: 2026-07-13
---

# 📊 SWE-bench 与 agent 评测

> [!important] 一句话
> **SWE-bench 是评测 coding agent 的行业标准基准：真实 GitHub issue，agent 产出补丁，用仓库的隐藏测试判分。RepoPilot 干的正是这件事，我的 verifier 就是它判分逻辑的迷你版。**

## SWE-bench 是什么
- 每个任务 = 一个真实仓库在某 commit 的状态 + 一个 issue 文本；agent 要产出 patch。
- **判分靠跑测试**：`FAIL_TO_PASS`（原本失败、修完要通过）+ `PASS_TO_PASS`（原本通过、不能弄坏 = 防回归）。
- 规模：全量 ~2294 条；**SWE-bench Lite ~300 条**（精简，适合起步）；**SWE-bench Verified 500 条**（人工验证过可解）。
- 指标：**% resolved**（补丁让隐藏测试通过的比例）。

## 和 RepoPilot 的关系
- SWE-bench 的判分（FAIL_TO_PASS + PASS_TO_PASS）**就是我 [[验证是灵魂|verifier]] 的基线对比 + 回归优先**，一模一样的思想。
- 所以我**别自建数据集**，直接拿 SWE-bench Lite 当样本，跑出一个 % resolved 就是我最缺的"评测数字"。

## agent 评测方法论（更宽的答法）
- **成功率**（pass@1 / % resolved）
- **成本**（token / 美元）
- **效率**（工具调用次数、wall-clock）
- **怎么算**：对 [[三层循环 阶段-步骤-重试|每次运行]]的 `run.jsonl`（[[RepoPilot MOC|trace]]）做聚合——成功率看 report.ok，工具数数 tool 事件。

## 面试话术
> "SWE-bench 是标准：真实 issue，agent 出补丁，隐藏测试判分，Lite 300 条、Verified 500 条。我的 verifier 的基线对比就是它判分逻辑的迷你版。我用 SWE-bench Lite 跑出 % resolved，再从 trace 聚合出 token 成本和工具调用数。"

关联：[[验证是灵魂]] · [[三层循环 阶段-步骤-重试]] · [[Agent 面试八股与答法]]
