---
tier: Tier-1
tech: React
topic: useMemo + useCallback
knowledge-point: 性能优化、缓存计算、缓存函数、memo
difficulty: ⭐⭐⭐
status: done
created: 2026-05-25
tags:
  - tech/react
  - tier-1
  - useMemo
  - useCallback
  - performance
---

# 📌 Topic 10：useMemo + useCallback — 性能优化双剑客

> Topic: React Tier 1 · Tier: 1 · 难度: ⭐⭐⭐

---

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Memoize | 记忆化 | 把计算结果缓存起来，下次同样输入直接返回缓存 |
| Reference | 引用 | JS 里函数是对象，每次创建的函数地址不同，即使内容一样 |
| Referential Equality | 引用相等 | 两个变量指向同一个地址才算相等 |
| Overhead | 开销 | useMemo/useCallback 本身也有成本，存缓存、比较依赖 |
| Premature Optimization | 过早优化 | 没发现真正的性能问题就优化，浪费时间 |

---

## 📝 1. 实例代码

```tsx
import { useState, useMemo, useCallback, memo } from 'react';

function JobList() {
  const [jobs, setJobs] = useState([/* 大量数据 */]);
  const [filterText, setFilterText] = useState('');
  const [theme, setTheme] = useState('light');

  // ============ useMemo：缓存计算结果 ============
  // 只有 jobs 或 filterText 变了才重新计算
  // theme 变了不重新计算（节省性能）
  const filteredJobs = useMemo(() => {
    console.log('重新计算！');
    return jobs
      .filter(job => job.title.includes(filterText))
      .sort((a, b) => b.salary - a.salary);
  }, [jobs, filterText]); // 依赖数组

  // ============ useCallback：缓存函数 ============
  // 保证函数引用稳定，配合 memo 子组件使用
  const handleDelete = useCallback((id) => {
    setJobs(prev => prev.filter(j => j.id !== id));
    // 用 prev 函数式更新，不需要把 jobs 放依赖数组
  }, []); // 空数组，永远不变

  return (
    <div>
      <input value={filterText} onChange={e => setFilterText(e.target.value)} />
      <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
        切换主题
      </button>
      {filteredJobs.map(job => (
        <JobCard key={job.id} job={job} onDelete={handleDelete} />
      ))}
    </div>
  );
}

// ============ memo：包住子组件 ============
// props 没变就不重新渲染
// 配合 useCallback 才有效果
const JobCard = memo(function JobCard({ job, onDelete }) {
  console.log('JobCard 渲染：', job.title);
  return (
    <div>
      <p>{job.title}</p>
      <button onClick={() => onDelete(job.id)}>删除</button>
    </div>
  );
});

export default JobList;
```

⚠️ 易错点：
1. `useCallback` 单独用没有意义，必须配合 `memo`（只有 useCallback，子组件还是每次渲染）
2. `memo` 单独用也没用，函数 props 每次都是新引用，`memo` 认为 props 变了（只有 memo，没有 useCallback）
3. 简单计算不需要 useMemo（`count * 2` 加 useMemo 反而更慢）
4. 依赖数组和 useEffect 一样，用了什么放什么

---

## 🎯 2. 使用场景

**useMemo 适合：**
1. **大列表过滤（Seek.com.au）**：5 万条职位 filter，只有筛选条件变了才重新算
2. **复杂统计（Fintech）**：总收入、平均值、最大最小，只有 transactions 变了才重算
3. **排序列表**：只有数据或排序方式变了才重新排序

**useCallback 适合：**
1. **传给 memo 子组件的回调**：`onDelete`、`onSubmit` 等
2. **useEffect 的依赖是函数**：用 useCallback 稳定函数引用，避免无限循环
3. **自定义 Hook 返回的函数**：保证返回的函数引用稳定

---

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| 不优化 | 代码简单 | 复杂场景卡顿 | 列表小、计算简单 |
| React.memo only | 简单 | 函数 props 每次新引用，memo 失效 | 只传基本类型 props |
| 重构组件结构 | 根本解决 | 需要重新设计 | 状态放错位置导致不必要渲染 |

**选型决策树**：
- 计算量大 + 依赖少 + 渲染频繁 → useMemo
- 函数传给 memo 子组件 → useCallback + memo
- 简单计算 → 直接算，不需要优化
- 不确定有没有问题 → 先不优化，用 Profiler 测量

---

## 💼 4. 面试题

### 题 1（🟡 进阶）

**Q**: useMemo 和 useCallback 有什么区别？

**核心 points**:
- useMemo：缓存计算结果（一个值）
- useCallback：缓存函数（一个函数）
- 本质：`useCallback(fn, deps)` = `useMemo(() => fn, deps)`

**追问方向**:
1. 什么时候用 useMemo，什么时候用 useCallback？
2. useMemo 能缓存函数吗？

### 题 2（🔴 高级）

**Q**: 什么情况下 useMemo 反而会让性能更差？

**核心 points**:
- useMemo 本身有开销（存缓存、比较依赖）
- 简单计算用 useMemo 得不偿失
- 依赖频繁变化，每次都重新算，缓存没用

**加分回答**: "useMemo 适合'计算贵、依赖稳'的场景。如果计算很简单（加减乘除），或者依赖每次都变，useMemo 的开销反而大于收益。Senior 会先用 React DevTools Profiler 测量，发现真正的瓶颈再用 useMemo，而不是到处加。过早优化是万恶之源。"

---

## 🧪 5. 小测验

找出下面代码的性能问题并修复：

```tsx
function Parent() {
  const [count, setCount] = useState(0);

  // 问题在哪？
  const double = useMemo(() => count * 2, [count]);
  const handleClick = () => console.log('clicked');

  return (
    <div>
      <button onClick={() => setCount(p => p + 1)}>+1</button>
      <Child value={double} onClick={handleClick} />
    </div>
  );
}

function Child({ value, onClick }) {
  console.log('Child 渲染');
  return <button onClick={onClick}>{value}</button>;
}
```

> 点击展开答案前先思考

<details>
<summary>展开答案</summary>

**问题 1**：`useMemo(() => count * 2, [count])` 完全没必要，简单乘法不需要缓存

**问题 2**：`handleClick` 每次渲染都是新函数，传给 Child 导致每次都重新渲染

**问题 3**：`Child` 没有 `memo`，即使 useCallback 也没用

**修复**：
```tsx
const double = count * 2; // 直接算，不需要 useMemo

const handleClick = useCallback(() => {
  console.log('clicked');
}, []); // 缓存函数

const Child = memo(function Child({ value, onClick }) {
  // 配合 memo
});
```
</details>

---

## 🌟 Senior 思维彩蛋

**性能优化的最高境界是"不需要优化"。** Senior 在设计组件结构时就会考虑：状态放在哪里？数据怎么流动？好的设计本来就不会有性能问题。比如把频繁变化的 state 放在离它近的组件里，而不是放在顶层，就能自然减少不必要的重新渲染。优化是最后手段，好的设计才是根本。

---

## 📅 复习记录

- [ ] D+3 复习
- [ ] D+7 复习
- [ ] D+21 复习

## 🔗 相关笔记

- [[Topic-13-渲染机制+性能调优]]
- [[Topic-09-自定义Hook]]
