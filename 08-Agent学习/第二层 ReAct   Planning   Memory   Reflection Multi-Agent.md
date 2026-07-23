

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


有几个必须处理的细节， Thoughts到底存在哪？
有两种实现流派

老派： Prompt式

现代： 原生tool_calls

