---
tier: Tier-1
tech: React
topic: useEffect
knowledge-point: 副作用、依赖数组、cleanup、发请求
difficulty: ⭐⭐
status: done
created: 2026-05-25
review-1: 
review-2: 
review-3: 
tags:
  - tech/react
  - tier-1
  - useEffect
  - side-effect
---

# 📌 Topic 6：useEffect — 处理"副作用"

> Topic: React Tier 1 · Tier: 1 · 难度: ⭐⭐

---

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Side Effect | 副作用 | 和画界面无关的操作，比如发请求、改网页标题、设定计时器 |
| Dependency Array | 依赖数组 | useEffect 第二个参数，告诉 React 什么变了才重新执行 |
| Cleanup | 清理函数 | useEffect return 的函数，组件卸载或下次执行前调用 |
| Memory Leak | 内存泄漏 | 组件消失了，但某些东西还在后台跑，占用内存 |
| Mount | 挂载 | 组件第一次出现在页面上 |
| Unmount | 卸载 | 组件从页面上消失 |

---

## 📝 1. 实例代码

```tsx
// 代码读法说明：
// 三种依赖数组写法：
// 1. 没有 [] → 每次渲染都执行
// 2. [] → 只执行一次（挂载时）
// 3. [dep] → dep 变了才执行

import { useState, useEffect } from 'react';

function JobList() {
  const [jobs, setJobs] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);
  const [searchQuery, setSearchQuery] = useState('');

  // ✅ 页面加载时发一次请求（空依赖数组）
  useEffect(() => {
    async function load() {
      try {
        const res = await fetch('https://api.example.com/jobs');
        const data = await res.json();
        setJobs(data);
      } catch (err) {
        setError('请求失败');
      } finally {
        setIsLoading(false);
      }
    }
    load();
  }, []); // 只执行一次

  // ✅ searchQuery 变了重新搜索
  useEffect(() => {
    document.title = `搜索：${searchQuery}`;
  }, [searchQuery]); // searchQuery 变了才执行

  // ✅ 定时器 + cleanup（防止内存泄漏）
  useEffect(() => {
    const timer = setInterval(() => {
      console.log('tick');
    }, 1000);

    // cleanup：组件消失时停掉定时器
    return () => clearInterval(timer);
  }, []);

  // ✅ 事件监听 + cleanup
  useEffect(() => {
    function handleResize() {
      console.log(window.innerWidth);
    }
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  if (isLoading) return <p>加载中...</p>;
  if (error) return <p>{error}</p>;
  return <ul>{jobs.map(j => <li key={j.id}>{j.title}</li>)}</ul>;
}

export default JobList;
```

⚠️ 易错点：
1. useEffect 不能直接写 `async`，要在里面定义 async 函数再调用
2. 依赖数组漏写了用到的变量，会拿到旧值（stale closure）
3. 有开就要有关：setInterval → clearInterval，addEventListener → removeEventListener
4. `finally` 里关 loading，不要只在 try 里关，否则报错时永远 loading

---

## 🎯 2. 使用场景

1. **SaaS 场景（Jira）**：进入看板页面，`[]` 依赖自动发请求拿任务数据，不需要用户手动触发
2. **E-commerce 场景（Seek.com.au）**：搜索词变了重新搜索，`[searchQuery]` 监听变化自动发新请求
3. **HealthTech 场景**：预约页面实时刷新，`refetchInterval: 30000` 每 30 秒更新状态

---

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| React Query | 内置缓存、重试、竞态处理 | 额外依赖 | 所有数据请求场景（推荐） |
| SWR | 轻量，stale-while-revalidate | 功能比 React Query 少 | 简单数据请求 |
| 手写 useEffect | 无额外依赖 | 要手写 loading/error/缓存 | 非请求类副作用（改标题、监听事件） |

**选型决策树**：
- 发网络请求 → 用 React Query（Topic 12）
- 改 document.title / 监听事件 / 定时器 → 用 useEffect

---

## 💼 4. 面试题

### 题 1（🟡 进阶）

**Q**: useEffect 的依赖数组是什么？三种写法分别什么时候执行？

**核心 points**:
- 没有数组：每次渲染都执行
- 空数组：只执行一次（挂载时）
- 有依赖：依赖变了才执行

**追问方向**:
1. 如果依赖数组写错了会怎样？
2. 为什么 useEffect 里用了某个变量，要放进依赖数组？

**加分回答**: "React 18 的 StrictMode 在开发环境下会故意执行 useEffect 两次，就是为了帮你发现副作用问题。生产环境只执行一次，所以上线前要在 StrictMode 下测试。"

### 题 2（🟡 进阶）

**Q**: useEffect 的 cleanup 函数是什么？什么时候执行？

**核心 points**:
- cleanup = useEffect return 的函数
- 执行时机：组件卸载时 + 下一次 effect 执行前
- 作用：清理定时器、移除监听、取消请求

**追问方向**:
1. 如果不写 cleanup 会怎样？
2. cleanup 是组件消失时才执行吗？

**加分回答**: "用户快速切换页面时，旧请求可能在新请求之后回来覆盖数据（竞态条件）。用 AbortController 在 cleanup 里取消请求可以解决这个问题。更好的方案是用 React Query，它内置了竞态处理。"

---

## 🧪 5. 小测验

做一个秒表，有开始和停止按钮，补全 cleanup：

```tsx
function Stopwatch() {
  const [seconds, setSeconds] = useState(0);
  const [isRunning, setIsRunning] = useState(false);

  useEffect(() => {
    if (!isRunning) return;

    const timer = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);

    // 👇 你来写 cleanup
    return () => {
      // ???
    };
  }, [isRunning]);

  return (
    <div>
      <p>⏱️ {seconds} 秒</p>
      <button onClick={() => setIsRunning(true)}>开始</button>
      <button onClick={() => setIsRunning(false)}>停止</button>
      <button onClick={() => { setIsRunning(false); setSeconds(0); }}>重置</button>
    </div>
  );
}
```

> 点击展开答案前先思考

<details>
<summary>展开答案</summary>

```tsx
return () => {
  clearInterval(timer);
};
```

**为什么需要 cleanup？**
点"停止" → `isRunning` 变 false → useEffect 重新执行前调用 cleanup → clearInterval 停掉定时器 → 秒数停止增加

如果不写 cleanup：点停止后上一个 timer 还在跑，再点开始又开一个新 timer，两个同时跑，秒数每秒 +2！
</details>

---

## 🌟 Senior 思维彩蛋

口诀：**开了什么，cleanup 里关什么。** Senior 写 useEffect 时，脑子里会同时想"这个 effect 需要 cleanup 吗？"——这是防止内存泄漏的第一道防线。生产环境里内存泄漏很难发现，但用户会感觉到页面越用越慢，最后崩溃。

---

## 🔗 关联算法题

- 无直接关联

---

## 📅 复习记录

- [ ] D+3 复习
- [ ] D+7 复习
- [ ] D+21 复习

## 🔗 相关笔记

- [[Topic-09-自定义Hook]]
- [[Topic-12-React-Query]]
