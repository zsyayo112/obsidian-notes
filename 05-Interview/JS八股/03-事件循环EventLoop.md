---
tier: Tier-1
tech: JavaScript
topic: JS 异步基础
knowledge-point: 事件循环 · 微任务 vs 宏任务
difficulty: ⭐⭐⭐
status: learning
created: 2026-07-02
review-1: 2026-07-05
review-2: 2026-07-09
review-3: 2026-07-23
tags:
  - tech/javascript
  - tier-1
  - async
  - event-loop
  - 面试必考
---
# 📌 事件循环 Event Loop（⭐ 前端面试第一题）
> Topic: [[JS 异步基础]] · Tier: 1 · 难度: ⭐⭐⭐
> 💡 一句话：**同步先全跑完 → 微任务整队清空 → 才执行一个宏任务。**

## 📖 0. 术语扫盲
| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Synchronous | 同步 | 立刻执行、排队跑完（普通 `console.log`） |
| Asynchronous | 异步 | 先「登记」等会儿再跑（`setTimeout`/`.then` 的回调） |
| Microtask | 微任务 | 高优先级队列：`Promise.then`、`await` 之后、`queueMicrotask` |
| Macrotask | 宏任务 | 低优先级队列：`setTimeout`、`setInterval`、I/O、事件 |

## 📝 1. 实例代码
```javascript
// 代码读法说明：追踪每行进哪个队列、按什么顺序执行
console.log("1");                                  // 同步 → 立刻
setTimeout(() => console.log("2"), 0);             // 宏任务 → 垫底
Promise.resolve().then(() => console.log("3"));    // 微任务 → 优先
console.log("4");                                  // 同步 → 立刻
// 输出：1 4 3 2

// 时间线：
// ① 同步阶段：打印 1、4；cb2 进宏任务队列，cb3 进微任务队列
// ② 清空微任务队列：执行 cb3 → 打印 3
// ③ 取一个宏任务：执行 cb2 → 打印 2

// 进阶：微任务里再生微任务，也一起清完才轮到宏任务
console.log("A");
setTimeout(() => console.log("B"), 0);
Promise.resolve().then(() => {
  console.log("C");
  Promise.resolve().then(() => console.log("D"));  // C 里又生一个微任务 D
});
console.log("E");
// 输出：A E C D B
```
⚠️ 易错点：
1. **`setTimeout(fn, 0)` 的 0 ≠ 立刻**：它是「排到下一轮宏任务」，永远排在同轮微任务之后。
2. **同步代码插队**：`console.log("4")` 是同步的，比任何异步（含 setTimeout 0ms）都先跑。
3. **微任务清空是「整队 + 新生的也清」**：清微任务过程中新冒出的微任务，会一并清完，才轮到宏任务。

## 🎯 2. 使用场景
1. **SaaS 场景**：理解为什么 `setState` 后立刻读 state 读到旧值（更新被排进异步）。
2. **Fintech 场景**：批量任务调度——理解 Promise 微任务优先，避免误判执行顺序导致数据竞态。
3. **E-commerce 场景**：调试「为什么这段异步日志顺序和我想的不一样」，用事件循环模型解释。

## 🔄 3. 可替代技术
| 概念 | 归类 | 优先级 | 何时想到它 |
|---|---|---|---|
| `Promise.then` / `await` 后续 | 微任务 | 高 | 需要「尽快」在本轮收尾 |
| `queueMicrotask` | 微任务 | 高 | 手动排微任务 |
| `setTimeout` / `setInterval` | 宏任务 | 低 | 需要「下一轮再说」 |
| `requestAnimationFrame` | 渲染前回调 | 特殊 | 动画帧 |

**选型决策树**：
- 如果要「当前这轮同步结束后立刻执行」→ 微任务（Promise/queueMicrotask）
- 如果要「让出主线程、下一轮再执行」→ 宏任务（setTimeout）

## 💼 4. 面试题
### 题 1（🔴 高级）
**Q**: 解释这段代码的输出顺序，并说明原因：`console.log(1); setTimeout(()=>log(2),0); Promise.resolve().then(()=>log(3)); console.log(4);`
**核心 points**: `1 4 3 2`。同步先全跑(1,4)；微任务(.then→3)整队清空；再执行宏任务(setTimeout→2)。
**追问方向**:
1. 如果微任务里再注册微任务会怎样？（也在本轮清完，才轮到宏任务）
2. `async/await` 混进来怎么排？（await 之前同步，await 之后是微任务）
**加分回答**: `setTimeout(fn,0)` 常被误当「立刻执行」，其实只是「下一轮宏任务」——理解这点是判断异步竞态的关键。

## 🧪 5. 小测验
> 点击展开答案前先思考
```javascript
console.log("1");
async function f() {
  console.log("2");
  await Promise.resolve();
  console.log("3");
}
setTimeout(() => console.log("4"), 0);
f();
Promise.resolve().then(() => console.log("5"));
console.log("6");
```
<details>
<summary>展开答案</summary>

输出：`1 2 6 3 5 4`
- 同步：1 → f() 里 await 之前的 2 → 6
- 微任务队列：3（f 的 await 后续）、5（.then）→ 按登记顺序 3、5
- 宏任务：4
</details>

## 🌟 Senior 思维彩蛋
`setState` 后立刻 `console.log(state)` 读到旧值，本质就是事件循环 + 闭包：更新被排进异步，且本次渲染的 state 被闭包定死。Senior 不会 debug 半天，而是立刻反应「这是异步更新，用函数式 setState 或 useEffect 拿新值」。

## 🔗 关联算法题
- 无直接算法题（语言运行机制）
- 面试中常与「Promise 手写实现」一起考

---
## 📅 复习记录
- [ ] D+3 复习（2026-07-05）✅ Day1 已能讲通 1 4 3 2
- [ ] D+7 复习（2026-07-09）
- [ ] D+21 复习（2026-07-23）
## 🔗 相关笔记
- [[02-Promise 与 async-await]]
- [[04-闭包与 useState]]
