
· Function Calling / Tool use的完整协议流程
· 结构化输出： JSON Schema 约束/解析失败的重试策略
· 上下文管理： token预算/历史裁剪/ 摘要压缩
· 流式输出： SSE/ WebSocket，前后端怎么串


**Function Calling / Tool use的完整协议流程**


Funtion Calling的协议本质， 模型不调用任何函数， 它只是输出一段结构化的“我想调用这个函数/ 参数是 这些” 的JSON格式， 执行时是我的代码干的

完整4步：
1 请求带上tools的定义(JSON schema).  
2 模型返回 assistant 消息，含 '''tool_calls[{id, name, arguments}]'''
3 本地来执行 -> 构造role = “tool” 消息， tool_call_id 必须对上
4 把 assistant消息 + tool 的结果一起塞回message， 再请求一次


工具定义的工程细节：
	description是给模型看的prompt，不是给人看的， 写清楚“什么时候用/什么时候不用”。 工具选错90%的情况是tool的description写的烂

	参数用enum就别用自由字符串,能约束就约束

	工具数量超过 ~20个， 准确率明显下降 -> 解决方案： 工具分组/ 检索式工具选择(RAG over
	tools)  这是个加分答案


并行tool_calls
一轮回答可能返回多个。 串行执行简单， 并行快 但是要注意：有副作用的工具， 写库， 发消息 不可以并行， 只读工具随便并行



**结构化输出**

#### 三个层级(从弱到强)

|方式|可靠性|说明|
|---|---|---|
|Prompt 里说"只返回 JSON"|差|会带 markdown 围栏、会加解释|
|JSON mode|中|保证是合法 JSON,但不保证符合你的 schema|
|Structured Output / 约束解码|强|语法层面强制符合 schema|