
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
SSE 是默认答案： 单向/ 基于HTTP / 自动断连
WebSocket 只在需要双向(打断、实时协作)时用。


#### 流式下的难点(这才是面试想问的)

1. **流式 + 工具调用**:tool_calls 的 arguments 是一个字符 一个字符 流出来的,你必须按 `index` 累积拼接,拼完才能 parse。这是实现细节里最容易写错的地方。
2. **流式 + 结构化输出**:输出到一半不是合法 JSON,想边流边渲染要用增量解析(partial JSON parser)。
3. **中断/取消**:用户关页面了,后端得能感知并停掉上游请求,否则钱照烧。
4. **错误发生在流中间**:HTTP 200 已经发出去了,只能在 SSE 里发一个 error 事件,前端要单独处理。
5. **Nginx 缓冲**:要关 `proxy_buffering`,否则流被攒着一次性发,白做。


问题：

为什么 assistant 的 tool_calls 消息必须放回 messages?不放会怎样?：
	LLM 推理是无状态的,每轮要重建完整上下文。而且 tool 消息和 assistant 的 tool_calls 是配对结构,tool_call_id 对不上 API 会直接报错。语义上 tool_call 是提问、tool result 是回答,拆开就没有归属信息了。

工具选错了,你有几种排查/优化手段?
	我会首先看选错的工具有哪些， 然后从 工具的description， 或者是参数， 或者是问题的上传， 还有工具数量， 如果工具很多， 使用分组或者是rag，
	- **工具名本身也是信号**。`get_data` vs `get_user_order_history`,后者准确率高得多。
	- **description 里写"什么时候不要用"**。比如 `search_web`:"仅用于查询实时信息;若问题可由 get_user_profile 回答,不要使用本工具。" 负向约束比正向描述更能区分相似工具。
	- **Few-shot**:在 system prompt 里给 2-3 个「query → 应该调哪个工具」的例子。相似工具打架时最有效。
	- 兜底:**用小模型/规则做一层路由**,先把工具集缩小到 3-5 个再交给主模型


JSON mode 和 Structured Output 的区别?各自底层怎么实现的?
	JSON mode{respo}