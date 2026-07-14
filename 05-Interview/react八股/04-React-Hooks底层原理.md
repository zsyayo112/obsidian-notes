---
tier: Tier-1
tech: React
topic: React 原理
knowledge-point: Hooks 底层原理
difficulty: ⭐⭐⭐
status: learning
created: 2026-07-02
review-1: 2026-07-05
review-2: 2026-07-09
review-3: 2026-07-23
tags:
  - tech/react
  - tier-1
  - react-internals
  - closure
  - interview-core
---

# 📌 React Hooks 底层原理

> Topic: [[React 原理]] · Tier: 1 · 难度: ⭐⭐⭐
> 接 [[JS 闭包]]（Day 1）。核心三问：① 靠什么认 state ② 为什么不能写 if 里 ③ stale closure

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Hook | 钩子 | useState/useEffect 等，让函数组件拥有 state 和生命周期 |
| Stale Closure | 陈旧闭包 | 闭包锁住了旧渲染的变量快照，读到的是过期值 |
| Dependency Array | 依赖数组 | useEffect 第二个参数 `[]`，决定 effect 何时重跑 |
| Functional Update | 函数式更新 | `setCount(prev => prev+1)`，让 React 喂最新值 |

## 📝 1. 实例代码

```jsx
// 代码读法说明：
// - 上半段演示"永远打印 0"的 stale closure bug
// - 下半段给三种修法，注意 console.log(读) 和 setCount(写) 用不同解法

// ❌ Bug：setInterval 永远打印 0
function Counter() {
  const [count, setCount] = useState(0);
  useEffect(() => {
    const timer = setInterval(() => {
      console.log(count);   // 永远打印 0：闭包锁死首次渲染的 count 变量
    }, 1000);
    return () => clearInterval(timer);
  }, []);                    // 空依赖 → 定时器只建一次，看不到新 count
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}

// ✅ 修法 A（针对"读"count）：依赖数组加 [count]
useEffect(() => {
  const timer = setInterval(() => console.log(count), 1000);
  return () => clearInterval(timer);
}, [count]);   // count 变 → effect 重跑 → 新定时器闭包看见新 count

// ✅ 修法 B（针对"写"count）：函数式更新
setCount(prev => prev + 1);   // prev 是 React 喂的最新值，不读闭包里的 count

// ✅ 修法 C（要定时器不重建又读最新值）：useRef
const countRef = useRef(count);
countRef.current = count;       // 每次渲染同步最新值
useEffect(() => {
  const timer = setInterval(() => console.log(countRef.current), 1000);
  return () => clearInterval(timer);
}, []);   // 定时器只建一次，ref 永远指向最新值
```

⚠️ 易错点：
1. **Hooks 必须写在组件函数最顶层**，不能放 if/for/嵌套函数/return 之后——否则调用顺序变，React 内部指针数错格子，state 全错位。
2. **"读"和"写"用不同解法**：`console.log(count)` 是读 → 只能靠**依赖数组**；`setCount(基于旧值算)` 是写 → 用**函数式更新**。函数式更新救不了 console.log。
3. **闭包是"看见"不是"传参"**：每次渲染生成新 count 变量，老回调没重建就锁死在旧变量上。

## 🎯 2. 使用场景

1. **SaaS 场景**：实时协作编辑器里的自动保存定时器，要读最新文档内容 → 用 useRef 存最新值，避免定时器频繁重建又能拿到新数据。
2. **Fintech 场景**：行情轮询 setInterval 里要用最新的筛选条件，典型 stale closure 场景，用依赖数组或 ref 修。
3. **E-commerce 场景**：秒杀倒计时 + 实时库存，函数式更新 `setCount(prev=>prev-1)` 保证并发点击下计数正确。

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| 依赖数组 `[count]` | 简单直接，能读最新值 | count 变就重建 effect（如重建定时器） | 重建代价小、逻辑简单 |
| 函数式更新 `prev=>...` | 绕过闭包、并发安全 | 只能用于"写"，不能读 | 基于旧 state 算新 state |
| useRef 存最新值 | 定时器不重建 + 读最新值 | 需手动同步 ref，代码稍绕 | 定时器/事件回调要稳定又读新值 |

**选型决策树**：
- 如果 要基于旧 state 算新 state（写）→ 函数式更新 `setCount(prev=>...)`
- 如果 要在 effect 里读最新 state 且重建代价小 → 依赖数组 `[state]`
- 如果 要定时器/订阅稳定不重建又读最新值 → useRef

## 💼 4. 面试题

### 题 1（🔴 高级）
**Q**: 为什么 React Hooks 不能写在条件语句（if）里？
**核心 points**: React 靠**调用顺序**认 state——内部一个数组/链表 + 从 0 数的指针，按 useState 调用顺序分配格子。if 让某个 Hook 时有时无 → 每次渲染调用顺序不一致 → 指针数错格 → state 张冠李戴。
**追问方向**:
1. React 内部怎么存 state？→ 按顺序存进链表节点（Fiber 的 memoizedState）
2. 那 useEffect 拿到旧 state 怎么办？→ 依赖数组 or 函数式更新 or useRef
**加分回答**: 画出"每次渲染是独立闭包快照，指针每次从 0 重数"的心智模型，说明为什么顺序稳定是铁律。

### 题 2（🔴 高级）
**Q**: 一个 setInterval 里 console.log(count) 永远打印 0，为什么？怎么修？
**核心 points**: 空依赖 `[]` → 定时器只建一次，其闭包锁死首次渲染的 count 变量（=0）；后续渲染生成新 count 变量但老定时器没重建，看不到。修法：依赖数组加 `[count]`（重建 effect）或 useRef（不重建读最新）。
**追问方向**:
1. 为什么不能用函数式更新修？→ 那是给"写"用的，这里是"读"，无 setCount 可用
2. 加 [count] 有什么副作用？→ 定时器每次 count 变都销毁重建
**加分回答**: 提 useRef 方案，在"定时器稳定"和"读最新值"间找平衡——mid 知道加依赖能修，senior 会权衡重建代价。

## 🧪 5. 小测验

> 点击展开答案前先思考

**Q**: `setCount(count + 1)` 和 `setCount(prev => prev + 1)` 有什么区别？在"1 秒内连续点 3 次按钮"时，哪个能正确加到 3？

<details>
<summary>展开答案</summary>

- `setCount(count + 1)`：`count` 从闭包读，是本次渲染的快照。连点 3 次可能都基于同一个旧 count 算，结果只加 1。
- `setCount(prev => prev + 1)`：`prev` 是 **React 喂的最新值**，不读闭包。连点 3 次会依次基于最新值累加，正确加到 3。

核心区别：写法 A 是"你把算好的数递给 React"（用了可能过期的 count）；写法 B 是"你把'怎么算'的规则递给 React"，让 React 用它手里最新的值执行。**基于旧 state 算新 state，永远用函数式更新。**

</details>

## 🌟 Senior 思维彩蛋

Hooks 的三个坑背后是**同一个心智模型**：每次渲染都是一次独立的函数执行，产生独立的变量快照和独立的闭包。Mid 把组件当"一个持续存在的对象"，于是被 stale closure 反复坑；Senior 把每次渲染当"一张独立的快照照片"——就自然理解了为什么 count 会过期、为什么函数式更新能救、为什么 useRef 是"跨快照的可变引用"。**理解了"渲染即快照"，Hooks 的所有诡异行为都变得可预测。**

## 🔗 关联算法题

- 无直接算法关联（属原理理解题）
- 概念关联：[[JS 闭包]] Day1 的 makeCounter 闭包练习是理解 stale closure 的地基

---

## 📅 复习记录
- [ ] D+3 复习（2026-07-05）
- [ ] D+7 复习（2026-07-09）
- [ ] D+21 复习（2026-07-23）

## 🔗 相关笔记
- [[01-React组件化心智模型]]
- [[02-React-Fiber架构]]
- [[03-React-Diff算法与key]]
- [[JS 闭包]]
