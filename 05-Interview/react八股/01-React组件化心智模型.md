---
tier: Tier-1
tech: React
topic: React 原理
knowledge-point: 组件化心智模型
difficulty: ⭐⭐
status: learning
created: 2026-07-02
review-1: 2026-07-05
review-2: 2026-07-09
review-3: 2026-07-23
tags:
  - tech/react
  - tier-1
  - mental-model
---

# 📌 React 组件化心智模型

> Topic: [[React 原理]] · Tier: 1 · 难度: ⭐⭐

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Component | 组件 | 一个返回 JSX（界面）的函数，可复用 |
| Props | 属性 | 父组件传给子组件的数据，**只读** |
| State | 状态 | 组件自己的数据，**可变** |
| Declarative | 声明式 | 只描述"界面长什么样"，不描述"怎么一步步改" |
| One-way data flow | 单向数据流 | 数据只能父→子流动，子不能反改父的 props |

## 📝 1. 实例代码

```jsx
// 代码读法说明：
// - LikeButton 是一个组件（返回 JSX 的函数），有自己的 state（count）
// - App 里用了三次 LikeButton，每个都独立计数，靠 props(label) 显示不同文字

function LikeButton({ label }) {          // 从 props 解构出 label（只读）
  const [count, setCount] = useState(0);  // state：自己的数据，可变

  return (
    <button onClick={() => setCount(count + 1)}>
      {label} 👍 {count}
    </button>
  );
}

function App() {
  return (
    <div>
      <LikeButton label="点赞文章" />   {/* 三个独立实例，各数各的 */}
      <LikeButton label="点赞评论" />
      <LikeButton label="点赞作者" />
    </div>
  );
}
```

⚠️ 易错点：
1. 组件名**首字母必须大写**（`LikeButton` 不是 `likeButton`），React 靠这个区分组件和普通标签
2. **props 只读**，子组件里 `label = '改掉'` 是禁止的——数据只能父→子单向流

## 🎯 2. 使用场景

1. **SaaS 场景**：后台管理系统里同一个「数据表格行」组件复用几十次，每行靠 props 传入不同数据，改一处样式全表统一变。
2. **Fintech 场景**：交易面板里「货币金额卡片」组件复用，props 传币种/金额，state 管本地展开/收起状态，逻辑写一次到处用。
3. **E-commerce 场景**：商品列表里「商品卡」组件 map 渲染上千次，props 传商品数据，组件化让"改卡片布局"只需改一个文件。

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| 原生 JS + DOM 操作 | 无框架依赖、包极小 | 手动同步数据到 DOM，复用靠复制粘贴 | 极简静态页、无复杂交互 |
| Web Components | 浏览器原生标准、跨框架 | API 繁琐、生态弱 | 需跨框架复用的设计系统 |
| Vue 组件 | 模板直观、上手快 | 生态/招聘不如 React | 团队偏 Vue、要平缓曲线 |

**选型决策树**：
- 如果 有复杂交互 + 要复用 + 团队/招聘看 React → 选 React 组件
- 如果 纯静态展示、几乎无交互 → 原生 JS 就够
- 如果 要做跨框架的 UI 库 → Web Components

## 💼 4. 面试题

### 题 1（🟢 基础）
**Q**: props 和 state 有什么区别？
**核心 points**: props = 父传入、只读；state = 组件自己的、可变（用 setState 改）。props 变 or state 变都会触发重新渲染。
**追问方向**:
1. 子组件能修改 props 吗？→ 不能，单向数据流，只能通知父组件去改
2. 什么时候用 state，什么时候用 props？→ 自己管理的用 state，父级传下来的用 props
**加分回答**: 提"单向数据流让数据流可预测——出 bug 时往上游找源头即可"，体现对数据流架构的理解。

## 🧪 5. 小测验

> 点击展开答案前先思考

**Q**: 下面代码里，三个 LikeButton 点其中一个，另外两个的数字会变吗？为什么？
```jsx
<LikeButton label="A" /> <LikeButton label="B" /> <LikeButton label="C" />
```

<details>
<summary>展开答案</summary>

不会。每个 `<LikeButton />` 是**独立实例**，各自持有独立的 `count` state。点 A 只改 A 自己的 count，B/C 的 state 完全隔离。这就是组件化"每个积木管好自己数据"的体现。

</details>

## 🌟 Senior 思维彩蛋

Mid 会把一个巨型组件塞满逻辑；Senior 会问"这块能不能拆成独立组件"——拆分的标准不是"代码长短"，而是"**这块有没有独立的职责/可复用性/独立的 state**"。组件粒度是 code review 高频争论点：拆太碎增加心智负担，拆太粗难复用难测试。好的拆分让每个组件"只有一个改变的理由"（单一职责）。

## 🔗 关联算法题

- LeetCode #（组件树 ≈ N 叉树，见 Fiber 笔记）
- HackerRank: 无直接关联

---

## 📅 复习记录
- [ ] D+3 复习（2026-07-05）
- [ ] D+7 复习（2026-07-09）
- [ ] D+21 复习（2026-07-23）

## 🔗 相关笔记
- [[02-React-Fiber架构]]
- [[03-React-Diff算法与key]]
- [[04-React-Hooks底层原理]]
