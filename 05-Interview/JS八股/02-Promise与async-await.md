---
tier: Tier-1
tech: JavaScript
topic: JS 异步基础
knowledge-point: Promise · async/await
difficulty: ⭐⭐
status: learning
created: 2026-07-02
review-1: 2026-07-05
review-2: 2026-07-09
review-3: 2026-07-23
tags:
  - tech/javascript
  - tier-1
  - async
  - promise
---
# 📌 Promise & async / await
> Topic: [[JS 异步基础]] · Tier: 1 · 难度: ⭐⭐
> 💡 核心比喻：**Promise = 取餐号**（现在没好、未来会有的值）。

## 📖 0. 术语扫盲
| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Promise | 承诺 / 取餐号 | 代表「现在没好、未来会有结果」的值 |
| pending / fulfilled / rejected | 进行中/成功/失败 | Promise 的三种状态，只能变一次 |
| resolve | 兑现 / 叫号按钮 | 按下它，取餐号变「成功」 |
| reject | 拒绝 | 按下它，取餐号变「失败」 |
| async / await | 异步 / 等待 | 让异步代码写得「像同步」 |

## 📝 1. 实例代码
```javascript
// 代码读法说明：
// - delay 自己造一张「ms 毫秒后成功」的取餐号
// - 三种取值方式：.then / await
// - async 函数永远返回 Promise

// 自己造取餐号（因为 setTimeout 太老，不给现成 Promise）
const delay = (ms) => new Promise(resolve => setTimeout(resolve, ms));
// 拆解：
//   setTimeout(resolve, ms) → ms 毫秒后按下叫号按钮 resolve（不加 () = 交给 setTimeout 调）
//   new Promise(resolve => …) → 造取餐号，拿到叫号按钮
//   (ms) => …               → 包成函数，ms 可传

async function 拿用户(id) {         // async → 这个函数永远返回 Promise
  const 响应 = await fetch(`/api/user/${id}`);  // await → 暂停，等取餐号成功取出值
  if (!响应.ok) throw new Error(`失败 ${响应.status}`);  // 抛错 → Promise reject
  return 响应.json();               // return 普通值 → 外面自动包成 Promise
}

// 取值方式
拿用户(1).then(用户 => console.log(用户));   // .then：登记回调，成功后取餐号调用它
const 用户 = await 拿用户(1);                // await：直接把值给你（须在 async 内）
```
⚠️ 易错点：
1. **async 函数永远返回 Promise**：`return 42` 拿到的是 `Promise{42}`，不是 42。调用方必须 `await` / `.then` 才能取到值；忘了 await → 拿到取餐号 → 报错。
2. **await 必须在 async 里**：await 要「暂停函数」，只有 async 开了「暂停开关」才允许（例外：ES 模块顶层 top-level await）。
3. **.then 不需要 async**：`.then` 哪都能放；只有 `await` 受「必须在 async 内」的约束。

## 🎯 2. 使用场景
1. **SaaS 场景**：调用后端 API 拿数据 `const data = await fetch(...).then(r=>r.json())`。
2. **Fintech 场景**：串行依赖的交易流程——先建订单 → 再扣款 → 再发通知，用 `await` 一步步来。
3. **E-commerce 场景**：首页并发加载互不依赖的数据，用 `Promise.all([拿用户(), 拿商品(), 拿公告()])` 省一半时间。

## 🔄 3. 可替代技术
| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| 回调 callback | 最底层、无依赖 | 回调地狱（嵌套金字塔） | 极简单的一次性异步 |
| Promise `.then` 链 | 拍平嵌套、可组合 | 一堆 .then 包着，读着绕 | 需要链式转换时 |
| async/await | 像同步一样读，配 try/catch | 底层仍是 Promise | **默认首选** |

**选型决策树**：
- 如果后一步依赖前一步结果 → `await` 串行
- 如果多个请求互不依赖 → `Promise.all` 并发
- 如果只是简单一步 → 直接 `await`

## 💼 4. 面试题
### 题 1（🟡 进阶）
**Q**: `await` 串行和 `Promise.all` 并发有什么区别？什么时候用哪个？
**核心 points**: 串行 = 一个等完再下一个（有依赖时用）；并发 = 一起发车全部好了再继续（互不依赖时用，省时间）。
**追问方向**:
1. `Promise.all` 里有一个失败会怎样？（整体 reject，用 `Promise.allSettled` 可拿到每个结果）
2. `await` 之后的代码属于同步还是微任务？（微任务，见 [[03-事件循环 Event Loop]]）
**加分回答**: 无依赖却写成串行，是常见的性能浪费——Senior review 时会一眼揪出「这两个 await 可以并发」。

## 🧪 5. 小测验
> 点击展开答案前先思考
```javascript
async function f() { return 42; }
const x = f();
console.log(x);              // 打印什么？
```
<details>
<summary>展开答案</summary>

打印 `Promise { 42 }`，不是 42。
因为 async 函数永远返回 Promise。要拿 42：`f().then(v=>console.log(v))` 或 `await f()`。
</details>

## 🌟 Senior 思维彩蛋
async「传染性」：一个底层函数用了 await 变成 async，调它的人也得 await，一路往上传染到调用链顶端。设计时要想清楚异步边界画在哪，别让 async 无节制蔓延到本可同步的地方。

## 🔗 关联算法题
- RAG 文本切块 → 滑动窗口 LeetCode #3(M), #76(H)
- （异步本身非算法，但 AI 项目里处理流式响应会大量用到）

---
## 📅 复习记录
- [ ] D+3 复习（2026-07-05）
- [ ] D+7 复习（2026-07-09）
- [ ] D+21 复习（2026-07-23）
## 🔗 相关笔记
- [[01-函数作为值与回调函数]]
- [[03-事件循环 Event Loop]]
