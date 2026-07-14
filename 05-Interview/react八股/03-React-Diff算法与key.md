---
tier: Tier-1
tech: React
topic: React 原理
knowledge-point: Diff 算法与 key
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
  - interview-must
---

# 📌 React Diff 算法与 key

> Topic: [[React 原理]] · Tier: 1 · 难度: ⭐⭐⭐
> ⭐ **澳洲 Senior 面试必考**："为什么列表要写 key？为什么不能用 index？"

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Diff / Reconciliation | 差异对比 / 协调 | React 算新旧界面差在哪、用最小代价更新 DOM |
| key | 键 | 你给 React 的"组件身份证"，认出"换了位置的还是它" |
| Heuristic | 启发式 | 用几条合理假设把复杂问题简化（把 O(n³) 降到 O(n)） |

## 📝 1. 实例代码

```jsx
// 代码读法说明：
// - 上半段是错误示范（index 当 key），下半段是正确做法（稳定 id）
// - 场景：往列表头部插入一条，观察 React 怎么认组件

// ❌ 错误：index 当 key —— 头插时 index 全错位
const [todos, setTodos] = useState(['buy milk', 'do homework', 'run']);
todos.map((item, index) => <TodoItem key={index} text={item} />)
// 头插 'wake up' 后：milk 的 index 从 0 变 1，React 认不出它还是它
// → 无谓重建 + 组件 state（如 checkbox 勾选）跟错行

// ✅ 正确：稳定唯一 id
const [todos, setTodos] = useState([
  { id: 'a1', text: 'buy milk' },     // id 跟着数据走，位置怎么变都不变
  { id: 'b2', text: 'do homework' },
  { id: 'c3', text: 'run' },
]);
todos.map((todo) => <TodoItem key={todo.id} text={todo.text} />)
```

⚠️ 易错点：
1. **index 当 key**：只有列表"纯静态、永不增删排序"时安全；一旦头插/中插/删除/排序 → 已有元素 index 变 → 认错组件 → **轻则性能浪费，重则 state 跟错行**。
2. **`Math.random()` 当 key 更坑**：每次渲染 key 都变 → React 认为整个列表全删全建 → 性能爆炸 + state 全丢 + 焦点/输入全没。
3. **key 只需同层兄弟唯一**，不用全局唯一（diff 只在同一个父节点的孩子间比较）。

## 🎯 2. 使用场景

1. **SaaS 场景**：可拖拽排序的看板（Trello 式），每张卡必须用稳定 id 当 key，否则拖动后卡片内的编辑状态会错乱到别的卡上。
2. **Fintech 场景**：可增删的持仓列表，删除中间一行时，稳定 id 保证其余行的组件状态（展开的明细）不跟错行。
3. **E-commerce 场景**：购物车增删商品，稳定 id 确保数量输入框、选中状态跟着正确的商品走。

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| 稳定唯一 id（后端主键/UUID） | 增删排序全正确、状态不错乱 | 需数据里带 id | ✅ 默认首选 |
| index 当 key | 无需 id、写着省事 | 动态列表出错 | 仅纯静态、永不变序的列表 |
| `Math.random()` | —— | 每次渲染全列表重建 | ❌ 永远别用 |

**选型决策树**：
- 如果 列表会增删/排序 → 必须稳定 id（后端主键优先，没有就创建时 `crypto.randomUUID()` 存进数据）
- 如果 列表纯静态展示、永不变序 → index 勉强可用（但业界仍推荐直接上 id，防未来加功能）
- 任何情况 → 不用 Math.random()

## 💼 4. 面试题

### 题 1（🔴 高级 · 必考）
**Q**: 为什么列表要写 key？为什么不能用数组 index？
**核心 points**:
- key = 组件身份证，让 React 在 diff 时认出"换了位置的还是同一个组件"，复用而非重建
- 没 key → React 按**位置对号入座**，头插/中插时误判"每格都变了"
- index 就是位置，会随增删排序变化 = 等于没给身份证
- 后果：轻则无谓 DOM 更新（性能），重则组件 state 跟错行（功能 bug）
**追问方向**:
1. 什么情况用 index 才安全？→ 已有元素 index 永不变化时（如纯尾部追加、纯静态列表）
2. 用 Math.random() 行不行？→ 更糟，每次渲染 key 变 → 整个列表销毁重建
**加分回答**: 提"key 只需同层兄弟唯一，非全局唯一，因为 diff 是同层比较"，展示对 diff 三大规则的理解；再补一句用 checkbox/输入框状态错乱的具体例子。

## 🧪 5. 小测验

> 点击展开答案前先思考

**Q**: 下面两个 `<ul>` 里都用了 `key={1}` 和 `key={2}`，会冲突吗？为什么？
```jsx
<div>
  <ul><li key={1}>苹果</li><li key={2}>香蕉</li></ul>
  <ul><li key={1}>牛奶</li><li key={2}>面包</li></ul>
</div>
```

<details>
<summary>展开答案</summary>

**不冲突。** key 只在"同一个父节点下的兄弟之间"比较。两个 `<ul>` 的孩子分属不同父节点——`li苹果` 的爹是第一个 ul，`li牛奶` 的爹是第二个 ul，它俩**永远不会被放一起比 key**。React diff 是**同层比较**，隔壁父节点的孩子用什么 key 互不相干。所以 key 只需"亲兄弟之间唯一"，不需全局唯一。

</details>

## 🌟 Senior 思维彩蛋

判断"能不能用 index 当 key"的通用规律：**只要一个操作会让"已有元素的 index 发生变化"，就不能用 index**。尾部追加是唯一安全例外（不改已有元素 index）。但 Senior 的判断是——**别赌"我这列表永远只尾插"**：今天纯尾插没事，产品明天加个删除/拖拽，bug 就来，而且这种 state 错乱 bug **不报错、极隐蔽**。所以直接上稳定 id 一劳永逸，是防御性设计。

## 🔗 关联算法题

- LeetCode #（diff 的树对比思想，可关联树的遍历，见 Fiber 笔记）
- HackerRank: 无直接关联

---

## 📅 复习记录
- [ ] D+3 复习（2026-07-05）
- [ ] D+7 复习（2026-07-09）
- [ ] D+21 复习（2026-07-23）

## 🔗 相关笔记
- [[01-React组件化心智模型]]
- [[02-React-Fiber架构]]
- [[04-React-Hooks底层原理]]
