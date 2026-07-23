
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



**为什么 assistant 的 tool_calls 消息必须放回 messages?不放会怎样?：**
	LLM 推理是无状态的,每轮要重建完整上下文。而且 tool 消息和 assistant 的 tool_calls 是配对结构,tool_call_id 对不上 API 会直接报错。语义上 tool_call 是提问、tool result 是回答,拆开就没有归属信息了。




**工具选错了,你有几种排查/优化手段?**
	我会首先看选错的工具有哪些， 然后从 工具的description， 或者是参数， 或者是问题的上传， 还有工具数量， 如果工具很多， 使用分组或者是rag，
	- **工具名本身也是信号**。`get_data` vs `get_user_order_history`,后者准确率高得多。
	- **description 里写"什么时候不要用"**。比如 `search_web`:"仅用于查询实时信息;若问题可由 get_user_profile 回答,不要使用本工具。" 负向约束比正向描述更能区分相似工具。
	- **Few-shot**:在 system prompt 里给 2-3 个「query → 应该调哪个工具」的例子。相似工具打架时最有效。
	- 兜底:**用小模型/规则做一层路由**,先把工具集缩小到 3-5 个再交给主模型







JSON mode 和 Structured Output 的区别?各自底层怎么实现的?
	`JSON mode{response_format: {“type”:"json_object"}}`
	
	从而保证输出是语法合适的JSON， 但是不能保证字段名 字段类型 必填项符合你的期望
	底层是约束解码， 但是只约束JSON语法(括号配对/引号闭合/逗号位置)
	实际风险是输出的格式可能不是你期望的合法JSON 类型，例如你要 `{"name": str, "age":int}`，它可能给你 `{"用户名": "张三", "年龄": "十八"}`

	Structured Output(传 JSON Schema)：
	- 保证:**100% 符合你给的 schema**,字段名、类型、enum、必填全部强制
	- 底层:**约束解码 / Grammar-guided decoding**

答案：

	模型每一步都在输出 token 的概率分布。约束解码在采样前,根据当前已生成的部分和 schema,算出**哪些 token 是合法的**,把非法 token 的 logits 直接设成 `-inf`,然后再采样。
	比如 schema 说下一个字段名必须是 `"age"`,那模型生成完 `{"` 之后,只有 `a` 这个 token 的概率被保留,其他全部屏蔽。schema 说 `age` 是 integer,那生成值的时候只有 `0-9` 这些 token 合法,`"` 和字母全被屏蔽。
	实现上通常把 JSON Schema 编译成一个**有限状态机(FSM)**,每生成一个 token 就在状态机上转移一次,状态机决定下一步的合法 token 集合。









**Lost in the Middle 是什么?怎么缓解?**

	Lost in the Middle 是长上下文下的位置偏置现象,模型对中间位置信息的召回明显弱于首尾,呈 U型。缓解上我会做首尾保留式裁剪、把 rerank 后最相关的文档放在最靠近提问的位置、以及用结构化标签帮模型定位。






**流式返回时,tool_call 的参数怎么拼?**

每个 chunk 里的 delta 长这样:

```
chunk1: tool_calls[{index:0, id:"call_abc", function:{name:"get_weather", arguments:""}}]
chunk2: tool_calls[{index:0, function:{arguments:"{\"ci"}}]
chunk3: tool_calls[{index:0, function:{arguments:"ty\":\"北"}}]
chunk4: tool_calls[{index:0, function:{arguments:"京\"}"}]
```

要点:

- **`id` 和 `name` 只在第一个 chunk 出现**,后续 chunk 只有 arguments 碎片
- 用 `index` 做 key 存进 dict,**并行多工具时 index 会有 0 和 1 交错出现**
- arguments 是**字符串拼接**,不是 JSON 合并,拼完整了才能 `json.loads`
- 判断结束靠 `finish_reason == "tool_calls"`


`acc = {}  # index -> {id, name, args_str}`
`for chunk in stream:`
    `for tc in chunk.choices[0].delta.tool_calls or []:`
        `cur = acc.setdefault(tc.index, {"id": "", "name": "", "args": ""})`
        `if tc.id: cur["id"] = tc.id`
        `if tc.function.name: cur["name"] = tc.function.name`
        `if tc.function.arguments: cur["args"] += tc.function.arguments`

答案：
	tool_calls 的 argument是一个字符一个字符的流出来的， 必须按index累计拼接以后才能parse





**用户中途取消请求,你的后端要做哪些清理?**

	首先 SSE 也能感知断连， 不需要WebSocket. TCP连接断了， 服务端写入的时候会抛出异常`BrokenPipeError` / `ClientDisconnected` ， 或者框架提供request.is_disconnected()

要清理掉的东西，按照重要性排：
	停掉上游LLM请求
	把已经生成的部分持久化
	释放工具侧资源
	写trace记录
	释放并发额度

回答：
	SSE 场景下服务端写入时能感知客户端断开。核心是把取消信号传播到上游 LLM 调用,避免继续计费;同时持久化已生成内容支持重连、释放工具侧资源和并发额度,并在 trace 里标记为 cancelled。




**一个工具返回了 50k token 的 JSON,你怎么处理?**

首先应该按照优先级处理
	**① 源头治理(最优,但常被忽略)**  
	在**工具实现里**就限制返回。加 `fields` 参数只返回需要的列,加 `limit` 限制条数,分页返回。数据库查询别 `SELECT *`。这是后端最自然的思路,面试第一个说这个显得很成熟。
	**② 应用层裁剪**  
	工具返回后、塞进 messages 前,代码层面做投影:嵌套结构拍平、去掉 null 字段、长文本截断加 `"...(truncated)"`。
	**③ 摘要 / 二次提取**  
	用便宜的小模型(Haiku 级别)把 50k 压成 500 token 的摘要再喂给主模型。代价是多一次调用 + 可能丢信息。
	**④ 落地成引用(生产级方案)**  
	把完整结果存到 Redis / 文件,给模型返回:
	json
	```json
	{
	  "summary": "查询到 1200 条订单,总额 45 万",
	  "ref_id": "res_a3f9",
	  "schema": ["order_id", "amount", "created_at"],
	  "preview": [前 3 条],
	  "hint": "使用 query_result(ref_id, filter) 进一步查询"
	}
	```
	再提供一个工具让模型按需取。**这本质上是让 Agent 用"游标"而不是"全量加载"**,是真正的工程解法。
	**⑤ 转成 RAG**  
	结果切块 + 向量化,让模型用检索的方式查这 50k。适合结果是大段非结构化文本时。
	**⑥ Code Interpreter 路线**  
	不让模型看数据,让模型**写代码处理数据**。50k 的 JSON 交给沙箱里的 Python,模型只看代码执行结果。数据量再大也不怕。这是目前处理大数据量最优雅的方案。


答案：先在工具侧就限制返回字段和条数,源头解决。已经拿到大结果的话,应用层做投影裁剪;还是太大就落地存储,只回给模型摘要+ref_id+schema,再提供查询工具让它按需取,相当于给 Agent 一个游标。如果是非结构化文本就转 RAG。数据量特别大时更倾向让模型写代码在沙箱里处理,而不是把数据塞进上下文。



Prompt Caching 的命中条件是什么， 你会怎么组织 prompt 来最大化命中率?

机制：服务器缓存的是前缀的KV。命中条件只有一条--  从第一个token开始，逐字节完全相同的最长前缀才能命中。中间改一个字， 后面全部失效。

**推论(这才是考点):prompt 必须按"稳定性"排序。**

```
[ system prompt      ]  ← 最稳定
[ tools 定义         ]  ← 稳定
[ few-shot 示例      ]  ← 稳定
──────── 缓存边界 ────────
[ 历史对话           ]  ← 追加式增长,前缀仍然稳定 ✅
[ 当前用户输入       ]  ← 每次变
```

**答法:**

> Prompt Caching 命中要求前缀逐字节一致,所以我会把 system prompt、tools 定义、few-shot 这些稳定内容放最前面,动态内容一律放尾部,绝不在头部放时间戳这类每次都变的东西。多轮对话是追加式增长,天然能命中。需要注意上下文裁剪会破坏前缀,和缓存策略要一起设计。

