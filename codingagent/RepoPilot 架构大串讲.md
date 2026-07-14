---
tags:
  - agent开发
  - repopilot
  - 面试
created: 2026-07-13
---

# 🚁 RepoPilot 架构大串讲

> [!abstract] 用途
> 面试时"讲讲你这个项目怎么运作"，照着这条线一口气讲完。一次运行从 issue 到报告，12 个文件依次登场。

## 一次运行的完整故事

用户敲 `repo-pilot solve --repo ../tinydb --issue-file bug.md`：

1. **入口 `cli.py`**：解析参数 → 把 issue 读成文字 → 交给 `orchestrator.solve()`。薄壳，逻辑不跟界面绑死（换 Web/MCP 不动核心）。
2. **门禁 `workspace`/`adapters`/`config`**：确认是 git 仓库 + 工作区干净（否则拒绝，git 是唯一撤销键）；`adapters` 探测"python + pytest"；`config` 攥着模型钥匙但门还关着（懒加载）。
3. **总指挥 `orchestrator`**：进入 `while` 状态机，自己不碰模型，只按顺序调度：
   - **BASELINE** → `verifier.run_tests` 存基线 `214 passed, 1 failed`（"修没修好"的参照系）
   - **PLAN** → `planner`（架在 `llm.json_call`）出 JSON 计划；只看目录树、故意可以猜错；第一次敲模型
   - **EXECUTE** → `executor` 干活循环：问模型→要工具就执行喂回→再问，直到不再要工具。这圈里 `permissions`+`tools`+`trace` 全汇齐；模型 search 定位、read 看清、`edit_file` 改 `<`→`<=`、run_tests 验证
   - **VERIFY** → `verifier.compare` 比基线：先查回归、再判 `fixed`；数改动文件数，超限判跑偏中止
   - **REVIEW** → `reviewer` 独立上下文复查，只看 issue/计划/diff/测试四样证据，屏蔽 executor 自述；approve + 提醒"缺防复发测试"
   - **REPORT** → 打印对比/裁决/diff + "满意 commit / 不满意回滚"，最后一步归人（`policy` 拉黑 commit）；置 DONE 退出
4. 全程每步落进 `run.jsonl`（`trace`），可回放、可统计成功率。

## 一句话收尾
1364 行、零框架、理解每一行。三层循环各转各的（见 [[三层循环 阶段-步骤-重试]]），三枚指纹贯穿（[[错误即信息]] / [[硬约束vs软约束]] / [[隔离即去偏见]]）。

## 从 playground 到真实仓库的三处认知跳变
1. **git 成了撤销键**——敢让它放手干的底气
2. **仓库读不完**——先 search 再 read
3. **验证才是主角**——[[验证是灵魂]]，模型说"好了"不算数

关联：[[RepoPilot MOC]] · [[Agent开发 MOC]]
