---
type: index
tier: Tier-1
tech: React
created: 2026-05-25
tags:
  - tech/react
  - tier-1
  - index
---

# ⚛️ React Tier 1 学习笔记索引

> 基于澳洲 50 个 Senior Software Engineer JD 分析的核心技术栈

---

## 📊 学习进度

| # | Topic | 难度 | 状态 | 笔记链接 |
|---|---|---|---|---|
| 01 | JSX + 组件基本写法 | ⭐ | ✅ | [[Topic-01-JSX-组件基本写法]] |
| 02 | Props — 怎么给组件传数据 | ⭐ | ✅ | [[Topic-02-Props]] |
| 03 | useState — 让页面动起来 | ⭐ | ✅ | [[Topic-03-useState]] |
| 04 | 列表渲染 + key | ⭐ | ✅ | [[Topic-04-列表渲染+key]] |
| 05 | 条件渲染 | ⭐ | ✅ | [[Topic-05-条件渲染]] |
| 06 | useEffect — 处理副作用 | ⭐⭐ | ✅ | [[Topic-06-useEffect]] |
| 07 | useRef — 不触发渲染的记忆 | ⭐⭐ | ✅ | [[Topic-07-useRef]] |
| 08 | useContext — 跨层级传数据 | ⭐⭐ | ✅ | [[Topic-08-useContext]] |
| 09 | 自定义 Hook | ⭐⭐ | ✅ | [[Topic-09-自定义Hook]] |
| 10 | useMemo + useCallback | ⭐⭐⭐ | ✅ | [[Topic-10-useMemo+useCallback]] |
| 11 | useReducer | ⭐⭐⭐ | ✅ | [[Topic-11-useReducer]] |
| 12 | React Query | ⭐⭐⭐ | ✅ | [[Topic-12-React-Query]] |
| 13 | React 渲染机制 + 性能调优 | ⭐⭐⭐⭐ | ✅ | [[Topic-13-渲染机制+性能调优]] |

---

## 🗺️ 知识图谱

```
基础层（Topic 1-5）
  JSX + 组件
    └── Props（父传子）
    └── useState（状态管理）
        └── 列表渲染（.map + key）
        └── 条件渲染（&&、三元、if）

进阶层（Topic 6-9）
  useEffect（副作用）
    └── 发请求（async/await）
    └── cleanup（防内存泄漏）
  useRef（不触发渲染）
    └── DOM 操作（focus、scroll）
    └── 计时器 ID 存储
  useContext（跨层级）
    └── 解决 Props Drilling
    └── 性能陷阱（分拆 Context）
  自定义 Hook（逻辑复用）
    └── useFetch
    └── useDebounce
    └── useCounter

高级层（Topic 10-13）
  useMemo + useCallback（性能优化）
    └── 配合 React.memo
  useReducer（复杂状态）
    └── action + reducer 模式
    └── 复杂表单管理
  React Query（服务端状态）
    └── useQuery（读）
    └── useMutation（写）
    └── 缓存 + 自动刷新
  渲染机制 + 性能调优
    └── Virtual DOM + Reconciliation
    └── 代码分割（lazy + Suspense）
    └── React DevTools Profiler
```

---

## 🔑 核心决策框架

### 状态管理选型

```
数据需要显示在页面 + 会变化 → useState
多字段相关联 + 多种操作 → useReducer
从后端拿的数据 → React Query
多组件共享 + 变化少 → useContext
多组件共享 + 变化频繁 → Zustand
不影响 UI 的内部数据 → useRef
```

### 性能优化决策树

```
发现性能问题？
  ↓ 先用 Profiler 找瓶颈
  ↓
父组件渲染带动子组件 → memo + useCallback
复杂计算每次都跑 → useMemo
初始 JS 太大 → lazy + Suspense
列表 1000+ 条 → react-window
```

### 条件渲染选型

```
显示/隐藏 → &&（注意数字 0 的坑）
二选一 → 三元表达式
三种以上 → if 提前返回（loading/error/空/正常）
```

---

## 📚 综合 Case 记录

| Case | 涵盖 Topic | 笔记 |
|---|---|---|
| 职位板（JobBoard）| 1-5 | [[React_JobBoard_笔记]] |
| Canva 用户系统 | 6-9 | [[React_综合Case2_笔记]] |

---

## 💼 高频面试题汇总

### 必答题（几乎每场必问）

1. Props 和 State 的区别？
2. useState 的 setState 是同步还是异步的？
3. useEffect 的依赖数组三种写法？
4. useRef 和 useState 的区别？
5. Virtual DOM 是什么？为什么用？

### 进阶题（Senior 面试常问）

1. useCallback 为什么要配合 memo？
2. Context 的性能问题怎么解决？
3. React Query 和 useEffect 发请求有什么区别？
4. 1000 条列表很卡怎么优化？
5. 自定义 Hook 和普通函数的区别？

---

## 🔗 外部资源

- [React 官方文档](https://react.dev)
- [TanStack Query 文档](https://tanstack.com/query)
- [React DevTools](https://react.dev/learn/react-developer-tools)
- [Neetcode 150](https://neetcode.io/practice)

---

**创建时间**：2026-05-25
**覆盖范围**：React Tier 1 全部（13 个 Topic）
**下一步**：开始 Tier 2（Next.js、GraphQL、Redis...）
