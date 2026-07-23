

**第一课 什么是Agent，ReAct从哪来**


先分清楚三个东西
1 Chatbot
2 Workflow
3 Agent

Agent和Workflow的本质区别在于控制流由谁来决定。 Workflow的路径是预先编排好的， Agent是模型在运行时根据观察结果动态决定下一步。 所以Agent更灵活但是更不可控， 成本和延迟也不可预测



**第二课 为什么会有ReAct**

先看两条失败的路线

1  路线A：只会想， 不会做(Chain -of -Thought)

2 路线B 只会做 不会想

3 ReAct的答案 是让“想” 和 “做” 交替进行


**关键洞察(这是论文的核心,必须背下来):**

> Reasoning 帮助模型规划和调整行动,Acting 让模型从外部获取真实信息来校正推理。二者交错进行,互相纠正。




**ReAct的循环长什么样？**

messages = [系统提示, 用户问题]

循环:
    回复 = 调用大模型(messages, 可用工具列表)
    
    如果 回复里没有工具调用:
        → 这就是最终答案,结束
    
    把 回复 加进 messages          ← 必须!
    
    对于 回复里的每个工具调用:
        结果 = 执行工具(...)
        把 结果 加进 messages       ← tool_call_id 要对上
    
    回到循环开头


有几个必须处理的细节，

**细节 1:Thought 到底存在哪?**
有两种实现流派

老派： Prompt式

在 system prompt 里要求模型按固定格式输出纯文本:

```
Thought: 我需要查北京人口
Action: search
Action Input: 北京 常住人口
```



现代： 原生tool_calls: 用 API 的 tool_calls 字段拿结构化的 Action,Thought 放在 `content` 字段里(模型自然会在调工具前说一段话)。

python

```python
resp.content         # → "我需要先查北京的人口数据"   ← 这是 Thought
resp.tool_calls[0]   # → {name: "search", args: {...}} ← 这是 Action
```

#### 细节 2:工具报错,不要 raise

这是 ReAct 最容易被浪费的优势。

**❌ 错误写法:**

python

```python
result = tools[name](**args)   # 报错直接崩,整个 Agent 挂掉
```

**✅ 正确写法:**

python

```python
try:
    result = tools[name](**args)
except Exception as e:
    result = {"error": str(e), "hint": "参数格式可能有误"}
# 不管成功失败,都作为 Observation 塞回去
```


#### 细节 3:必须有 max_iterations

没有的话,模型可能反复调同一个工具,一直烧钱。

python

```python
MAX_ITER = 5
for step in range(MAX_ITER):
    ...
else:
    return "达到最大步数,任务未完成"
```

**面试延伸题:除了 max_iter 还能怎么防死循环?**

- **重复动作检测**:记录 `(工具名, 参数)` 的哈希,连续出现相同的就中断或提示模型换个方法
- **无进展检测**:连续 N 步 Observation 都是空/错误,中断
- **成本上限**:累计 token 超过阈值就停
- **时间上限**:超过 30 秒中断

答出"重复动作检测"就够了,这是最实用的一个。


#### 细节 4:tool_call_id 必须一一对应

模型一轮可能返回**多个** tool_call:

python

```python
resp.tool_calls = [
    {id: "call_1", name: "search", args: {"q": "北京人口"}},
    {id: "call_2", name: "search", args: {"q": "上海人口"}},
]
```

你必须**每个都回一条** tool 消息,少一条 API 直接 400 报错。

python

```python
for tc in resp.tool_calls:
    result = execute(tc)
    messages.append({
        "role": "tool",
        "tool_call_id": tc.id,     # ← 必须对上
        "content": str(result)
    })
```

**串行还是并行?**

- 只读工具(查询) → 可以并行,快
- 有副作用的工具(写库、发消息、付款) → **必须串行**,并行可能产生竞态

#### 细节 5:messages 会无限膨胀

跑到第 5 轮,messages 里已经堆了一堆 Observation。这时候第一层的上下文管理就要接上了:token 预算、裁剪、摘要。不管这件事的话,长任务跑着跑着就爆上下文。



### 五、状态设计(你的后端优势在这里)

别把 Agent 当成一个函数,当成一个**状态机**。

python

```python
from dataclasses import dataclass, field

@dataclass
class AgentState:
    messages: list = field(default_factory=list)   # 给模型看的
    scratchpad: list = field(default_factory=list) # 给人看的
    step: int = 0
    status: str = "running"    # running / finished / max_steps / error
    total_tokens: int = 0      # 成本统计
    final_answer: str | None = None
```

**为什么 messages 和 scratchpad 要分开存?**

- `messages`:喂给模型的,格式受 API 约束,会被裁剪、会被压缩
- `scratchpad`:结构化的 `(thought, action, observation)` 三元组,**给人和监控系统看的**

python

```python
state.scratchpad.append({
    "step": 1,
    "thought": "我需要先查北京人口",
    "action": {"tool": "search", "args": {"q": "北京人口"}},
    "observation": "2183.2万",
    "duration_ms": 340,
})
```

有了这个,你才能:调试时看清模型为什么走错、接 Langfuse 做 trace、给用户展示"思考过程"、统计每步耗时和成本。

**这个设计还有一个隐藏价值:** 后面学 LangGraph 时,它的 `StateGraph` 就是这个东西的框架化版本。你自己写过一遍,学 LangGraph 就是"把我的 dataclass 换成它的 State",一小时就通。