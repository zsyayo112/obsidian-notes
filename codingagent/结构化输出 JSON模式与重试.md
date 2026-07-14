---
tags:
  - agent开发
  - 机制
  - repopilot
created: 2026-07-13
---

# 🧩 结构化输出：JSON 模式与重试

> [!important] 一句话
> **让模型交回"数据"而非"散文"。三件事，零框架：①逼它进 JSON 模式 ②解析 ③解析失败把错误喂回去让它自己改。planner 和 reviewer 都靠这一个 `json_call`。**

## 两种拿结构化输出的方式（都在裸 SDK 范围内）
1. **JSON 模式**：`response_format={"type":"json_object"}`，逼模型只吐 JSON。RepoPilot 用这个。
   - DeepSeek 坑：JSON 模式要求**提示词里出现 "JSON" 字样**，所以 system prompt 都写"只输出 JSON"。
2. **function calling**：定义一个 `submit_plan` 工具，逼模型用它交计划。和派普通工具一个套路。

## json_call 的骨架（llm.py）
```python
def json_call(system, user, retries=2):
    client = get_client()                       # 第一次敲模型的门（懒加载）
    for _ in range(retries + 1):
        try:
            resp = client.chat.completions.create(
                model=MODEL, messages=messages,
                response_format={"type":"json_object"})
        except Exception:                       # 个别网关不支持就退化成普通调用
            resp = client.chat.completions.create(model=MODEL, messages=messages)
        try:
            return _parse(text)                 # 成功：返回 dict
        except ValueError as e:                 # 失败：把错误喂回，重试
            messages += [{"role":"assistant","content":text},
                         {"role":"user","content":f"你的输出不是合法 JSON（{e}）。只输出修正后的 JSON。"}]
    raise RuntimeError(...)
```
`_parse` 还做兜底：剥掉模型爱加的 ```` ```json ```` 围栏，再 `json.loads`。

## 暗线
"解析失败喂回重试"和 [[错误即信息]] 是同一枚指纹，只是这次喂回的是**格式错误**。见 [[三层循环 阶段-步骤-重试|重试循环]]。

## json.dumps vs json.loads
- `json.loads`：字符串 → 字典（llm._parse 用）
- `json.dumps(d, ensure_ascii=False, indent=2)`：字典 → 好看字符串（render_plan 用，中文原样+缩进）
- 同一个 json 模块，两个反方向。

## 面试
"怎么让 LLM 输出结构化数据 / 保证格式可靠" —— JSON 模式（或 function calling）+ 解析 + 解析失败喂回重试。能讲到"重试"就比只答 response_format 深一层。

关联：[[RepoPilot MOC]] · [[错误即信息]] · [[廉价假设加后续验证]]
