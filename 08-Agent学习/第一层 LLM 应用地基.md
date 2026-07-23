
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

 三个层级(从弱到强)

|方式|可靠性|说明|
|---|---|---|
|Prompt 里说"只返回 JSON"|差|会带 markdown 围栏、会加解释|
|JSON mode|中|保证是合法 JSON,但不保证符合你的 schema|
|Structured Output / 约束解码|强|语法层面强制符合 schema|


**上下文管理**

Token预算怎么算
 `context_window = system + tools定义 + 历史 + 当前输入 + max_tokens(输出预留)` 

输出预留很容易忘记 

裁剪策略

滑动窗口： 保留最近N轮。 最简单， 但是会丢失早期的关键约束
首尾保留： system + 最早几轮 + 最近N论。 因为Lost in the Middle现象 -- 模型对中间位置的信息注意力最弱。
摘要压缩： 超阈值时用 LLM 把早期历史压成摘要。代价是多一次调用 + 信息损失。
向量检索历史： 把历史存进向量库,按当前 query 检索相关片段。这就通向 Memory 了(第二层)。



**流式输出**

为什么这么做 
首Token延迟， 决定体感。 非流式的Agent用户会以为卡死了

技术选型：
SSE 是默认答案： 单向