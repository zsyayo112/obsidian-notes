---
tier: Tier-1
tech: React
topic: 自定义Hook
knowledge-point: 逻辑复用、useFetch、useDebounce、useLocalStorage
difficulty: ⭐⭐
status: done
created: 2026-05-25
tags:
  - tech/react
  - tier-1
  - custom-hook
---

# 📌 Topic 9：自定义 Hook — 把逻辑变成可复用的函数

> Topic: React Tier 1 · Tier: 1 · 难度: ⭐⭐

---

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Custom Hook | 自定义 Hook | 你自己写的以 `use` 开头的函数，里面可以用其他 Hook |
| Reusable | 可复用 | 写一次，多个组件都能用 |
| Separation of Concerns | 关注点分离 | 数据逻辑和 UI 逻辑分开写 |
| Abstraction | 抽象 | 把复杂细节隐藏起来，对外只暴露简单接口 |

---

## 📝 1. 实例代码

```tsx
import { useState, useEffect, useRef } from 'react';

// ============ useFetch：拿数据 ============
function useFetch(url) {
  const [data, setData] = useState(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    async function load() {
      try {
        setIsLoading(true);
        const res = await fetch(url);
        if (!res.ok) throw new Error('请求失败');
        const json = await res.json();
        setData(json);
      } catch (err) {
        setError(err.message);
      } finally {
        setIsLoading(false);
      }
    }
    load();
  }, [url]); // url 变了重新请求

  return { data, isLoading, error }; // 返回数据，不是 JSX！
}

// ============ useDebounce：防抖 ============
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(timer); // cleanup 取消上一个定时器
  }, [value, delay]);

  return debouncedValue;
}

// ============ useCounter：计数器 ============
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  const increment = () => setCount(prev => prev + 1);
  const decrement = () => setCount(prev => prev - 1);
  const reset = () => setCount(initialValue);

  return { count, increment, decrement, reset };
}

// ============ 组件里使用（非常简洁！）============
function JobList() {
  const { data: jobs, isLoading, error } = useFetch('/api/jobs');
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebounce(query, 500);
  const { count, increment } = useCounter(0);

  if (isLoading) return <p>加载中...</p>;
  if (error) return <p>{error}</p>;

  return (
    <div>
      <input value={query} onChange={e => setQuery(e.target.value)} />
      <p>防抖搜索词：{debouncedQuery}</p>
      <p>计数：{count}</p>
      <button onClick={increment}>+1</button>
    </div>
  );
}
```

⚠️ 易错点：
1. 名字必须以 `use` 开头，否则里面不能用 useState/useEffect 等 Hook
2. 返回数据不是 JSX——自定义 Hook return `{ data, loading }`，组件 return `<div>...</div>`
3. 自定义 Hook 和普通函数的区别：Hook 里可以用其他 Hook，普通函数不行
4. 依赖数组和 useEffect 一样，用了什么放什么

---

## 🎯 2. 使用场景

1. **数据请求（useFetch）**：任何需要发请求的组件都用 `useFetch(url)`，不需要每次都写三个 state + useEffect
2. **防抖搜索（useDebounce）**：搜索框输入停止 500ms 后才发请求，避免每个字母都发一次
3. **本地存储（useLocalStorage）**：`const [theme, setTheme] = useLocalStorage('theme', 'light')`，刷新页面数据还在

---

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| 复制粘贴逻辑 | 简单 | 改一处要改所有地方 | 逻辑只用一次时 |
| HOC（高阶组件） | 老 React 的复用方式 | 包装层级多，调试难 | 维护老代码时 |
| Render Props | 灵活 | 回调地狱，代码可读性差 | 几乎不用了 |

**选型决策树**：
- 多个组件有重复的 Hook 逻辑 → 抽成自定义 Hook
- 逻辑只用一次 → 直接在组件里写
- 需要复用 UI + 逻辑 → 抽成组件

---

## 💼 4. 面试题

### 题 1（🟡 进阶）

**Q**: 什么是自定义 Hook？为什么要用它？

**核心 points**:
- 自定义 Hook = 以 `use` 开头的函数，里面可以用其他 Hook
- 解决：逻辑复用，避免复制粘贴
- 好处：组件更简洁，逻辑集中管理

**追问方向**:
1. 自定义 Hook 和普通函数有什么区别？
2. 自定义 Hook 和组件有什么区别？

**加分回答**: "自定义 Hook 是 React 逻辑复用的最佳方式。和普通函数的区别是：自定义 Hook 里可以用 useState、useEffect 等 Hook。和组件的区别是：自定义 Hook 返回数据和函数，组件返回 JSX。组件负责'长什么样'，自定义 Hook 负责'怎么运作'。"

---

## 🧪 5. 小测验

写一个 `useCounter` Hook：

```tsx
function useCounter(initialValue = 0) {
  // 👇 你来写
}

// 用起来：
function Counter() {
  const { count, increment, decrement, reset } = useCounter(0);
  return (
    <div>
      <p>{count}</p>
      <button onClick={increment}>+1</button>
      <button onClick={decrement}>-1</button>
      <button onClick={reset}>重置</button>
    </div>
  );
}
```

> 点击展开答案前先思考

<details>
<summary>展开答案</summary>

```tsx
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);

  function increment() { setCount(prev => prev + 1); }
  function decrement() { setCount(prev => prev - 1); }
  function reset() { setCount(initialValue); }

  return { count, increment, decrement, reset };
}
```
</details>

---

## 🌟 Senior 思维彩蛋

**组件负责"长什么样"，自定义 Hook 负责"怎么运作"。** Senior 写组件时，如果发现组件里有超过 3-4 个 useState 或 useEffect，就会考虑把相关逻辑抽成自定义 Hook。组件越干净越好，理想的组件只有 JSX 和几行调用 Hook 的代码。

---

## 📅 复习记录

- [ ] D+3 复习
- [ ] D+7 复习
- [ ] D+21 复习

## 🔗 相关笔记

- [[Topic-06-useEffect]]
- [[Topic-12-React-Query]]
