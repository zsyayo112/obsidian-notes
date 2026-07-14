---
tier: Tier-1
tech: React
topic: React 原理
knowledge-point: Fiber 架构
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
  - interview-core
---

# 📌 React Fiber 架构

> Topic: [[React 原理]] · Tier: 1 · 难度: ⭐⭐⭐

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Fiber | 纤程 | React 内部的工作单元，一个存着组件信息+进度的 JS 对象 |
| Reconciliation | 协调 | React 算"新旧界面差在哪"的过程（Render 阶段做的事） |
| Render Phase | 渲染阶段 | 在内存里打草稿算差异，**可中断** |
| Commit Phase | 提交阶段 | 把差异刷到真实 DOM，**不可中断** |
| Concurrent | 并发 | Fiber 之上盖的能力：可中断带来的"边渲染边响应用户" |

## 📝 1. 实例代码

```typescript
// 代码读法说明：
// - 这不是你要写的代码，是 React 内部长什么样的示意
// - 帮你把"Fiber 节点是个对象""靠三指针遍历"落到实处
// - 日常写 React 永远不碰这层，但面试要讲得出

// ① 一个 Fiber 节点，本质就是个 JS 对象
interface FiberNode {
  type: Function | string;       // 组件函数 或 'li'/'div' 标签
  key: string | number | null;   // 你在 map 里写的 key，Diff 时用

  // ② 三个指针，把节点织成一张能遍历的网
  child: FiberNode | null;       // 下：第一个孩子
  sibling: FiberNode | null;     // 右：下一个兄弟
  return: FiberNode | null;      // 上：父节点

  stateNode: HTMLElement | null; // 对应的真实 DOM
  memoizedState: any;            // 组件的 state（Hooks 数据存这）
  pendingProps: any;             // 待处理的新 props
}

// ③ 可中断循环的骨架（React 内部简化版）
function workLoop(deadline: IdleDeadline) {
  while (nextUnitOfWork && deadline.timeRemaining() > 1) {
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    // ↑ 每轮检查：还有时间吗？没时间就跳出，把主线程还给浏览器
  }
  requestIdleCallback(workLoop);  // 浏览器空了再回来接着干
}
```

⚠️ 易错点：
1. **"Fiber 让 React 更快"是错的**——单次渲染 Fiber 甚至略慢（有调度开销）。它换来的是**不卡顿**（响应性），不是**更快**（吞吐量）。
2. **可中断的只有 Render 阶段**，Commit 阶段绝不中断。说"Fiber 让渲染可中断"要补一句"仅 Render 阶段"。
3. **Fiber ≠ 并发特性本身**。Fiber 是地基；`useTransition`/`Suspense` 是盖在它上面的楼。

## 🎯 2. 使用场景

1. **SaaS 场景**：大型仪表盘几千个数据节点，用户在搜索框打字时，Fiber 让"渲染大表格"能被"用户输入"打断，输入保持流畅。
2. **Fintech 场景**：实时行情列表持续重渲染，高优先级的用户交互（点股票看详情）能插队到低优先级的后台刷新前面。
3. **E-commerce 场景**：搜索联想实时过滤上千商品，`useDeferredValue`（建在 Fiber 上）让输入优先、结果列表重算延后，打字不掉帧。

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| Svelte（编译时） | 运行时无 VDOM 开销、包极小 | 生态较小、极端动态场景受限 | 性能/包体积敏感、团队愿尝新 |
| SolidJS（细粒度响应式） | 精准更新到 DOM、不做树 diff、极快 | 生态小、招人难 | 追极致性能的新项目 |
| Vue（响应式+VDOM） | 依赖追踪精确、天然知道谁变了 | 大列表仍可能有开销 | 中大型项目、要平缓曲线 |
| 老 React（Stack Reconciler） | 实现简单 | 递归不可中断、大树必卡 | 已被取代，不选 |

**选型决策树**：
- 如果 已在 React 生态 → 用 React 18+，Fiber 白送，无需操心
- 如果 追极致运行时性能 + 新项目 + 团队肯学 → SolidJS / Svelte
- 如果 要平缓曲线 + 精确更新 → Vue

> 务实结论：99% 场景"选不选 Fiber"是伪命题（选 React 18 就自动有）。真正选型在"选不选 React"那层，而那层性能几乎不是决定因素（团队/生态/招聘才是）。

## 💼 4. 面试题

### 题 1（🟡 进阶）
**Q**: Fiber 架构解决了什么问题？
**核心 points**: 老 Stack Reconciler 递归渲染（链式调用）不可中断 → 大组件树长时间占主线程 → 掉帧。Fiber 把递归改成可中断的循环 + 用节点存进度 → 渲染可暂停/恢复/丢弃。
**追问方向**:
1. "可中断"具体中断哪个阶段？→ 只有 Render 阶段，Commit 不可中断
2. 进度为什么能存下来？→ 存在 Fiber 节点字段里（child/sibling/return + memoizedState），不在调用栈
**加分回答**: 主动区分"响应性 vs 吞吐量"——Fiber 不让渲染更快，是让主线程能被高优先级任务插队，本质是**调度**。

### 题 2（🔴 高级）
**Q**: Fiber 是"更快"吗？
**核心 points**: 不是。单次渲染因调度开销可能更慢。优化的是"感知流畅度/响应性"，不是"总渲染耗时"。
**追问方向**:
1. 什么场景反而更慢？→ 小而简单的组件树，调度开销占比高
2. 怎么衡量收益？→ 看 INP / long task 指标，不是总渲染时间
**加分回答**: 能说"过度优化陷阱"——小应用上并发特性收益微乎其微，别为用而用。

## 🧪 5. 小测验

> 点击展开答案前先思考

**Q**: 同事说"我们页面卡，把 React 升到 18 开并发模式，Fiber 就能让渲染变快，卡顿就解决了"。这句话有**两个概念错误**，指出来。

<details>
<summary>展开答案</summary>

**错误 1**：**"Fiber 让渲染变快"**——Fiber 不让渲染更快（单次甚至更慢），它让渲染**可被打断**，从而不阻塞用户交互。解决的是卡顿感不是渲染速度。

**错误 2**：**"升到 18 就自动解决"**——Fiber 从 React 16 就有；18 加的是**并发特性 API**（useTransition 等）。光升级不主动用这些 API + 找出真正瓶颈（没做 memo、渲染了不该渲染的东西），卡顿不会自动消失。**先 profile 定位瓶颈，再谈优化。**

</details>

## 🌟 Senior 思维彩蛋

Mid 看到"页面卡"第一反应是"上并发特性/加 memo"；Senior 第一反应是**打开 React DevTools Profiler 先定位**——是渲染太多次、单次太重、还是被别的脚本占了主线程。Fiber 和并发特性是"让不得不做的大渲染不阻塞交互"，不是"让本不该发生的渲染消失"。生产事故里 90% 的卡顿真因是"渲染了不该渲染的东西"（依赖数组写错、没拆组件），根本轮不到 Fiber 出场。**先测量，再优化。**

## 🔗 关联算法题

Fiber 的 child/sibling/return 遍历 = **N 叉树的非递归遍历**（用循环+指针代替递归）
- LeetCode #589 N 叉树前序遍历（M，用迭代做）
- LeetCode #429 N 叉树层序遍历（M）
- LeetCode #105 从前序+中序构造二叉树（M，呼应 reconciliation）
- HackerRank: Trees 主题下的 non-recursive traversal

---

## 📅 复习记录
- [ ] D+3 复习（2026-07-05）
- [ ] D+7 复习（2026-07-09）
- [ ] D+21 复习（2026-07-23）

## 🔗 相关笔记
- [[01-React组件化心智模型]]
- [[03-React-Diff算法与key]]
- [[04-React-Hooks底层原理]]
