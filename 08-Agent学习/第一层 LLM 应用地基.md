
· Function Calling / Tool use的完整协议流程
· 结构化输出： JSON Schema 约束/解析失败的重试策略
· 上下文管理： token预算/历史裁剪/ 摘要压缩
· 流式输出： SSE/ WebSocket，前后端怎么串


Funtion Calling的协议本质， 模型不调用任何函数， 它只是输出一段结构化的“我想调用这个函数/ 参数是 这些” 的JSON格式， 执行时是我的代码干的

完整4步：
1 请求带上tools的定义(JSON schema).  
2 模型返回 assistant 消息，含 '''tool_calls[{id, name, arguments}]'''
3 本地来执行 -> 构造role = “tool” 消息， tool_call_id 必须对上
4 把 assistant消息 + tool 的结果一起塞回message， 再请求一次


