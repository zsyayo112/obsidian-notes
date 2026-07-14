---
tier: Tier-1
tech: React
topic: React渲染机制+性能调优
knowledge-point: Virtual DOM、Reconciliation、memo、lazy、Suspense、Profiler
difficulty: ⭐⭐⭐⭐
status: done
created: 2026-05-25
tags:
  - tech/react
  - tier-1
  - performance
  - virtual-dom
  - code-splitting
  - senior-must
---

# 📌 Topic 13：React 渲染机制 + 性能调优

> Topic: React Tier 1 · Tier: 1 · 难度: ⭐⭐⭐⭐ Senior 必备

---

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Virtual DOM | 虚拟 DOM | React 在内存里维护的"假页面"，用来和真实页面对比 |
| Reconciliation | 协调 | React 对比新旧虚拟 DOM，找出差异，只更新变化的部分 |
| Diffing | 差异比较 | 对比新旧虚拟 DOM 的算法，时间复杂度 O(n) |
| Code Splitting | 代码分割 | 把大的 JS 文件拆成小块，需要时才加载 |
| Lazy Loading | 懒加载 | 不是一开始全部加载，用到时才加载 |
| Suspense | 悬停 | 等待懒加载组件时，显示 fallback 内容 |
| Profiler | 性能分析器 | React DevTools 里的工具，记录每次渲染时间 |
| Flame Graph | 火焰图 | 可视化展示每个组件渲染时间 |

---

## 📝 1. 实例代码

```tsx
import { useState, useMemo, useCallback, memo, lazy, Suspense } from 'react';

// ============ Virtual DOM 工作原理 ============
// React 只更新变化的部分
function App() {
  const [count, setCount] = useState(0);
  const [name] = useState('张三');

  return (
    <div>
      <p>count: {count}</p>  {/* count 变了 → 只有这个 p 更新 */}
      <p>name: {name}</p>    {/* name 没变 → 这个 p 不更新 */}
      <button onClick={() => setCount(p => p + 1)}>+1</button>
    </div>
  );
}

// ============ memo + useCallback：阻止不必要渲染 ============
function JobList() {
  const [jobs, setJobs] = useState([]);
  const [theme, setTheme] = useState('light');

  const filteredJobs = useMemo(() =>
    jobs.filter(j => j.salary > 100000),
  [jobs]);

  const handleDelete = useCallback((id) => {
    setJobs(prev => prev.filter(j => j.id !== id));
  }, []);

  return (
    <div>
      <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
        切换主题（{theme}）
      </button>
      {filteredJobs.map(job => (
        <JobCard key={job.id} job={job} onDelete={handleDelete} />
      ))}
    </div>
  );
}

// memo：props 没变不重新渲染，配合 useCallback 才有效果
const JobCard = memo(function JobCard({ job, onDelete }) {
  console.log('JobCard 渲染：', job.title);
  return (
    <div>
      <p>{job.title}</p>
      <button onClick={() => onDelete(job.id)}>删除</button>
    </div>
  );
});

// ============ 代码分割：lazy + Suspense ============
// ❌ 普通 import：一开始就加载所有页面代码
// import Dashboard from './Dashboard';

// ✅ lazy import：用到时才加载
const Dashboard = lazy(() => import('./Dashboard'));
const Settings = lazy(() => import('./Settings'));

function AppWithRoutes() {
  const [page, setPage] = useState('home');

  return (
    <div>
      <nav>
        <button onClick={() => setPage('dashboard')}>仪表盘</button>
        <button onClick={() => setPage('settings')}>设置</button>
      </nav>

      {/* Suspense：等待懒加载时显示 fallback */}
      <Suspense fallback={<p>页面加载中...</p>}>
        {page === 'dashboard' && <Dashboard />}
        {page === 'settings' && <Settings />}
      </Suspense>
    </div>
  );
}
```

⚠️ 易错点：
1. lazy 组件必须用 Suspense 包住，否则报错
2. lazy 只能用于默认导出（`export default`），命名导出不行
3. memo 失效的常见原因：传了内联对象（`style={{ color: 'red' }}`）每次都是新引用
4. 不是到处加 useMemo/useCallback/memo 就好，滥用反而更慢

---

## 🎯 2. 使用场景

**Virtual DOM + Reconciliation**：
- React 自动处理，开发者理解原理即可
- key 的正确使用影响 Reconciliation 效率

**memo + useCallback**：
1. **大列表（1000+ 条）**：每个 item 用 memo，删除时只重渲染被删的那一条
2. **复杂子组件**：图表、富文本编辑器，props 没变不重新渲染

**代码分割**：
1. **路由级别**：每个页面单独 chunk，首屏只下载首页代码
2. **大组件懒加载**：富文本编辑器、图表库，用到时才下载
3. **管理员后台**：99% 用户不需要，懒加载节省所有普通用户的流量

---

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| 不优化 | 代码简单 | 复杂场景卡顿 | 小应用、列表短 |
| react-window | 虚拟列表，只渲染可见项 | 额外依赖，配置复杂 | 列表 1000+ 条 |
| React.PureComponent | 类组件版的 memo | 已过时 | 维护老代码 |

**性能调优决策树**：
```
页面感觉卡？
  ↓ 用 React DevTools Profiler 找瓶颈
  ↓
父组件渲染，子组件跟着渲染（props 没变）
  → memo + useCallback

组件内有复杂计算
  → useMemo

初始加载 JS 太大
  → lazy + Suspense 代码分割

列表 1000+ 条
  → react-window 虚拟列表
```

---

## 💼 4. 面试题

### 题 1（🟡 进阶）

**Q**: Virtual DOM 是什么？React 为什么用 Virtual DOM？

**核心 points**:
- Virtual DOM 是内存里的 JS 对象，模拟真实 DOM
- 直接操作真实 DOM 慢，操作 JS 对象快
- React 对比新旧 Virtual DOM（Diffing），只更新变化的部分

**追问方向**:
1. Virtual DOM 一定比直接操作 DOM 快吗？
2. Reconciliation 的 Diffing 算法时间复杂度是多少？

**加分回答**: "React 用 Diffing 算法对比新旧 Virtual DOM，时间复杂度是 O(n)，比传统树对比 O(n³) 快很多。关键是两个假设：同层对比、key 唯一标识，让算法可以线性时间完成。"

### 题 2（🔴 高级）

**Q**: 1000 条数据的列表很卡，怎么优化？

**加分回答**: "首先用 react-window 做虚拟列表，1000 条变成只渲染 10-20 条可见项。同时用 memo 包住每个列表项，用 useCallback 稳定传入的函数。如果数据是从后端来的，用 React Query 配合分页 API，每次只请求当前页的数据。三个层面：渲染优化、计算优化、网络优化，组合使用。"

---

## 🧪 5. 小测验

找出下面代码所有性能问题：

```tsx
function App() {
  const [jobs, setJobs] = useState([/* 500条 */]);
  const [count, setCount] = useState(0);
  const [filter, setFilter] = useState('');

  const result = jobs
    .filter(j => j.title.includes(filter))
    .sort((a, b) => b.salary - a.salary);

  const handleDelete = (id) => {
    setJobs(prev => prev.filter(j => j.id !== id));
  };

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>count: {count}</button>
      <input value={filter} onChange={e => setFilter(e.target.value)} />
      {result.map(job => (
        <JobCard key={job.id} job={job} onDelete={handleDelete} />
      ))}
    </div>
  );
}

function JobCard({ job, onDelete }) {
  return <div><p>{job.title}</p><button onClick={() => onDelete(job.id)}>删除</button></div>;
}
```

> 点击展开答案前先思考

<details>
<summary>展开答案</summary>

**问题 1**：复杂计算没缓存，count 变了也重新算
```tsx
const result = useMemo(() =>
  jobs.filter(j => j.title.includes(filter)).sort((a, b) => b.salary - a.salary),
[jobs, filter]);
```

**问题 2**：handleDelete 每次渲染都是新函数
```tsx
const handleDelete = useCallback((id) => {
  setJobs(prev => prev.filter(j => j.id !== id));
}, []);
```

**问题 3**：JobCard 没有 memo，父组件任何变化都触发重渲染
```tsx
const JobCard = memo(function JobCard({ job, onDelete }) { ... });
```
</details>

---

## 🌟 Senior 思维彩蛋

**性能优化的正确顺序：先写正确，再写清晰，最后才是优化。** Senior 不会看到渲染就优化，而是先用 React DevTools Profiler 找到真正的性能瓶颈，再针对性优化。过早优化是万恶之源（Donald Knuth）。说出"先测量，再优化"这句话，面试官会对你印象深刻。

---

## 🔗 关联算法题

- LeetCode #146 LRU Cache（理解缓存机制）

---

## 📅 复习记录

- [ ] D+3 复习
- [ ] D+7 复习
- [ ] D+21 复习

## 🔗 相关笔记

- [[Topic-10-useMemo+useCallback]]
- [[Topic-12-React-Query]]
