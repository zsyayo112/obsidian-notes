---
tags:
  - agent开发
  - 机制
  - repopilot
created: 2026-07-13
---

# 🔧 工具设计：实现 + schema 两半

> [!important] 一句话
> **一个工具永远有两半：实现半（普通 Python 函数，你的代码，模型看不见）+ 说明书半（JSON schema，发给模型看，模型只见这半）。模型看着说明书吐意图，你的代码照实现半真执行。**

## 两半
```python
# 实现半：普通方法
def search_code(self, pattern, glob=""): ...
# 说明书半：发给模型的 schema
{"type":"function","function":{"name":"search_code",
  "description":"用正则搜索代码…","parameters":{...}}}
```
`description` 是**软约束**：引导模型"何时用、怎么用"，但它可以不听（[[硬约束vs软约束]]）。

## 真实仓库逼出来的三只新手
- `search_code`（ripgrep）：仓库读不完，先搜再读
- `list_symbols`（ast）：千行大文件先看类/函数骨架
- `edit_file`（精确片段替换）：不整文件覆盖

## edit_file 的"恰好一次"约束
```python
n = text.count(old_string)
if n == 0: return "错误：old_string 不存在。先 read_file 看清…"
if n > 1:  return f"错误：出现 {n} 次，无法确定改哪处，请包含更多上下文…"
```
逼模型**先 read 看清、再改**，杜绝凭记忆瞎改。（防的正是 tinydb 那次它绕去用 sed 全局替换、差点把 `__lt__` 也改坏。）

## 两条铁律（execute 分派器）
1. **永远返回字符串** 2. **永远不抛异常**（出错返回错误串，[[错误即信息]]）3. **超长自动截断**（保护上下文）。
还有职责分离：**工具只管"怎么做"，"该不该做"归 permissions/policy**，在 executor 里串。

## 只读/改动分界
`SAFE_TOOLS`（read/search/symbols/git_diff/run_tests）自动放行；改动类（edit/write/bash）过权限闸。这个分界是权限系统的地基。

## 面试
"agent 的工具怎么实现" —— 两半（实现+schema，模型只见 schema）；工具出错返回字符串喂回（错误即信息）；只读/改动分权限。

关联：[[RepoPilot MOC]] · [[错误即信息]] · [[L1 工具调用的真相]] · [[L3 手脚与沙盒]]
