---
tier: Tier-1
tech: React
topic: React Hooks
knowledge-point: setState 用法 · 函数式更新 · 不可变更新
difficulty: ⭐⭐
status: learning
created: 2026-07-02
review-1: 2026-07-05
review-2: 2026-07-09
review-3: 2026-07-23
tags:
  - tech/react
  - tier-1
  - hooks
  - state
---
# 📌 setState 用法（React 实战天天用）
> Topic: [[React Hooks]] · Tier: 1 · 难度: ⭐⭐
> 💡 三条铁律：本次 state 不变 · 依赖旧值用函数式 · 对象数组要造新的。

## 📖 0. 术语扫盲
| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Functional update | 函数式更新 | `setX(prev => ...)`，prev 是 React 塞的最新值 |
| Immutable update | 不可变更新 | 不改旧对象，用 `{...}`/`[...]` 造新的 |
| Batching | 批处理 | 多个 setState 合并成一次渲染 |

## 📝 1. 实例代码
```javascript
// 代码读法说明：直接设值 / 函数式 / 对象数组的不可变更新
const [count, setCount] = useState(0);
const [user, setUser] = useState({ name: "Hal", age: 25 });
const [list, setList] = useState([1, 2, 3]);

// ① 直接设值
setCount(5);
setCount(count + 1);

// ② 函数式更新（新值依赖旧值时必用）
//    prev 是 React 调用你的回调时塞进来的「当前最新值」（回调机制！）
setCount(prev => prev + 1);
setCount(prev => prev + 1);   // 连续更新也正确（不会都用旧值）

// ③ 对象：造新的（不能直接改原对象，否则引用没变，React 可能不重渲染）
setUser(prev => ({ ...prev, age: 26 }));   // 复制旧的，覆盖 age

// ④ 数组
setList(prev => [...prev, 4]);             // 加
setList(prev => prev.filter(x => x !== 2)); // 删
```
⚠️ 易错点：
1. **setState 后本次读不到新值**：`setCount(5); console.log(count)` 还是旧值——更新被排进异步，本次渲染 count 已定死。
2. **连续 setState 用函数式**：`setCount(count+1)` 连写三次只加 1（count 都是旧值）；`setCount(prev=>prev+1)` 才正确加 3。
3. **对象/数组必须造新的**：直接 `user.age=26; setUser(user)` 引用没变，React 可能不重渲染。

## 🎯 2. 使用场景
1. **SaaS 场景**：聊天框追加消息 `setMessages(prev => [...prev, 新消息])`——不可变更新保证渲染。
2. **Fintech 场景**：连续多次调整数量/金额，用函数式更新避免基于旧值算错。
3. **E-commerce 场景**：购物车增删商品，全用 `[...]`/`filter` 造新数组。

## 🔄 3. 可替代技术
| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| `useState` | 简单直接 | 复杂状态逻辑分散 | 单个/少量独立状态 |
| `useReducer` | 状态逻辑集中、可测 | 样板代码多 | 复杂状态转换、多动作 |
| 状态库(Zustand/Redux) | 跨组件共享 | 引入依赖 | 全局状态 |

**选型决策树**：
- 如果单个简单状态 → `useState`
- 如果一组相关状态 + 多种更新动作 → `useReducer`
- 如果要跨组件共享 → Zustand/Context

## 💼 4. 面试题
### 题 1（🟡 进阶）
**Q**: 为什么 `setCount(count+1)` 连续调用三次只加了 1，而不是 3？怎么修？
**核心 points**: 本次渲染 count 被闭包定死，三次都是 `setCount(0+1)`。改用函数式 `setCount(prev=>prev+1)`，prev 是 React 塞的最新值。
**追问方向**:
1. setState 是同步还是异步？（异步/批处理，本次读不到新值）
2. React 18 的自动批处理是什么？（多个 setState 合并成一次渲染，减少重渲染）
**加分回答**: 「本次渲染的 state 是不可变快照」——理解这点就理解了 React 的心智模型，是很多 setState bug 的根源。

## 🧪 5. 小测验
> 点击展开答案前先思考
```javascript
const [n, setN] = useState(0);
function click() {
  setN(n + 1);
  setN(prev => prev + 1);
  setN(n + 1);
}
// 点一次，n 最终是几？
```
<details>
<summary>展开答案</summary>

n 最终是 **1**。
- `setN(n+1)` → 基于旧 n=0 → 排队值 1
- `setN(prev=>prev+1)` → 基于上一步 1 → 2
- `setN(n+1)` → 又基于旧 n=0 → **覆盖成 1**
最后一个直接值覆盖了前面，结果是 1。教训：混用直接值和函数式很危险，要么全函数式。
</details>

## 🌟 Senior 思维彩蛋
`setX(prev => ...)` 里的 prev 是 React 塞给你的（和 `.then(食物=>)`、`arr.map(x=>)` 同一个回调机制）——不是你定义的变量，是接收方在调用回调时传进来的。看懂这层，回调机制在整个 JS/React 里就打通了。

## 🔗 关联算法题
- 无直接算法题（React API）

---
## 📅 复习记录
- [ ] D+3 复习（2026-07-05）
- [ ] D+7 复习（2026-07-09）
- [ ] D+21 复习（2026-07-23）
## 🔗 相关笔记
- [[04-闭包与 useState]]
- [[01-函数作为值与回调函数]]
