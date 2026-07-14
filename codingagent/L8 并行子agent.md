---
tags:
  - agent开发
  - 课程/L8
  - 多agent
created: 2026-07-10
aliases:
  - L8
  - 并行
---

# L8 · 并行子 agent

> [!abstract] 核心
> 主 agent 常在【一圈里】发出多个 spawn_agent，说明这些子任务彼此独立、可以同时做。但 `for tc in tool_calls:` 把它们排成了队。这一课把队拆掉——**用线程池同时开跑**。

## 实测加速

3 个独立子 agent：**串行 8.8s → 并行 3.0s → 2.9×**。
- 串行墙钟 ≈ 各子 agent 耗时之【和】
- 并行墙钟 ≈ 最慢那个的耗时（其余被它罩住）
- 子 agent 越多、越慢，省得越多

并行时各子 agent 的日志会**交错打印**——那就是它们真在同时跑的肉眼证据。

## 为什么线程就够（很多人搞混）

子 agent 的时间几乎全花在**等 API 返回**（I/O 等待），不是算 CPU。
Python 的 GIL 只锁 CPU 计算、不锁 I/O 等待 → 一个线程等 API 时别的照跑。
**这类"I/O 密集"任务用线程池就能接近 N 倍加速，用不着 asyncio、用不着多进程。**

## 一个死规矩：并行可以，顺序不能丢

用 `ThreadPoolExecutor.map`，它**保证结果顺序 = 输入顺序**。
因为每个 tool_call 的结果必须配回正确的 `tool_call_id`（见 [[消息配对约束]]），
并行执行完若乱序收集，配对错乱 → API 直接 400。

```python
with ThreadPoolExecutor(max_workers=len(TASKS)) as pool:
    return list(pool.map(run_subagent, TASKS))   # 保序
```

## 代价

- 日志交错，出错更难定位。
- **多个子 agent 同时写文件会互相踩** → 所以并行的通常是【只读】子 agent。
  要并行【写】，得像 Claude Code 那样给每个子 agent 独立的工作副本（git worktree），改完再合并。

## 代码

`lessons/l8_parallel.py`

---
上一课：[[L7 子agent与上下文隔离]] · 下一课：[[L9 何时委派]] · 总览：[[Agent开发 MOC]]
