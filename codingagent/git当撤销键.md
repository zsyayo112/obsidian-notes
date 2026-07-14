---
tags:
  - agent开发
  - 机制
  - repopilot
  - 安全
created: 2026-07-13
---

# ↩️ git 当撤销键

> [!important] 一句话
> **从 playground 到真实仓库最重要的认知跳变：git 成了 agent 的撤销键。敢让它放手改真代码的唯一底气，是随时能一键回到原样。`workspace.py` 就是这张安全网。**

## 三件事，全围绕"安全网"
1. **门禁**：开工前确认 ①是 git 仓库 ②工作区干净，否则拒绝启动。
2. **看账**：`diff` 看改了什么、`modified_files` 数改了几个。
3. **兜底**：`reset_hard` 一键还原。

## 门禁为什么必须"工作区干净"
`reset_hard` = `git checkout -- .` + `git clean -fd`，是**无差别核平**所有未提交改动。若开工前工作区就脏（你自己有没提交的活儿）：
- **diff 污染**：最后 diff 把 agent 改动和你的改动搅在一起，分不清。
- **回滚误伤**：用户点"不满意回滚"，把你没提交的活儿也一起删了，不可逆。

> **门禁 ↔ 回滚是一对**：正因为回滚是核弹，才必须开工前保证干净，让核弹只炸得到 agent 的改动。

## 借力 git，不造轮子
给 planner 的文件清单用 `git ls-files`（而非 `os.walk`）——**天然跳过 .git、遵守 .gitignore**。git 已经知道"哪些文件算数"，直接借它的知识。

## 已知局限（诚实）
`git diff` 看不到**未跟踪的新文件**（write_file 新建的）。若 agent 新建文件，REPORT 的 diff 可能不完整——待改进（可 `git add -A` 后 `git diff --cached`）。

## 面试
"怎么保证 agent 改坏了能恢复、改动可审计" —— git 当撤销键 + 开工干净门禁 + reset_hard 兜底；门禁与回滚是一对。

关联：[[RepoPilot MOC]] · [[硬约束vs软约束]] · [[安全 黑名单vs沙箱]]
