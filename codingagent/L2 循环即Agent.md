---
tags:
  - agent开发
  - 课程/L2
created: 2026-07-10
aliases:
  - L2
---

# L2 · 循环即 Agent

> [!abstract] 核心
> **把"问 → 执行 → append → 再问"包进一个 while 循环，那一刻脚本就变成了 agent。** 循环体只有 8 行。

## Agent 的全部

```python
for turn in range(max_turns):
    resp = client.chat.completions.create(model=MODEL, messages=messages, tools=TOOLS)
    msg = resp.choices[0].message
    messages.append(msg_to_dict(msg))          # 记下模型说的话
    if not msg.tool_calls:                      # 它不要工具了 → 任务完成
        return msg.content
    for tc in msg.tool_calls:                    # 它要工具 → 执行，喂回结果
        result = call_tool(tc.function.name, tc.function.arguments)
        messages.append({"role": "tool", "tool_call_id": tc.id, "content": result})
```

## 两个必须记住的点

- **控制流交给了模型。** 它想继续就吐 `tool_calls`，觉得完成就吐 `content`。你不写任何"判断任务是否完成"的逻辑——那在模型脑子里。这是 agent 和传统 workflow 最本质的区别。
- **判断有没有 tool_calls，别判断 content 是否为空。** 官方返回 `content=None`，DeepSeek 返回 `''`——"OpenAI 兼容"各家实现有差异，`if not msg.tool_calls` 才稳。

## 亲历的坑

- **并行 tool call**：模型一圈发起 6 个调用，所以必须**遍历** `msg.tool_calls`，不能偷懒取 `[0]`（否则另外 5 个结果永远回不去，模型死等）。
- **token 单调暴涨**：374 → 491 → 4727，读一个文件全文就永久驻留，见 [[L4 上下文管理]]。
- **自我纠错**：读不存在的文件，拿到错误后自己改用别的路径——[[错误即信息]]。而且路径的行为无法提前推演：agent 的路径是模型即兴决定的。

## 代码

`lessons/l2_the_loop.py`

---
上一课：[[L1 工具调用的真相]] · 下一课：[[L3 手脚与沙盒]] · 总览：[[Agent开发 MOC]]
