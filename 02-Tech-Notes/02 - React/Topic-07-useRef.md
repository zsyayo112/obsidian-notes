---
tier: Tier-1
tech: React
topic: useRef
knowledge-point: 不触发渲染的记忆、DOM操作、计时器ID
difficulty: ⭐⭐
status: done
created: 2026-05-25
tags:
  - tech/react
  - tier-1
  - useRef
---

# 📌 Topic 7：useRef — 不触发渲染的"记忆"

> Topic: React Tier 1 · Tier: 1 · 难度: ⭐⭐

---

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| ref | 引用 | 一个盒子，里面放着一个值，改了不会触发重新渲染 |
| .current | 当前值 | ref 里存的值，通过 `ref.current` 读取和修改 |
| Mutable | 可变的 | 可以直接改，不需要通过 setState |
| focus | 聚焦 | 输入框被选中，可以直接打字（自动 focus = 省掉用户手动点击） |
| DOM | 文档对象模型 | 浏览器里每个 HTML 元素都是一个对象 |

---

## 📝 1. 实例代码

```tsx
import { useState, useRef, useEffect } from 'react';

// ============ 用途一：操作 DOM ============
function SearchBar() {
  const inputRef = useRef(null);

  useEffect(() => {
    inputRef.current.focus(); // 页面加载后自动 focus
  }, []);

  function handleClear() {
    setQuery('');
    inputRef.current.focus(); // 清空后重新 focus
  }

  return <input ref={inputRef} placeholder="搜索..." />;
}

// ============ 用途二：存计时器 ID ============
function Stopwatch() {
  const [seconds, setSeconds] = useState(0);
  const timerRef = useRef(null); // 存 setInterval 的 ID

  function start() {
    if (timerRef.current) return; // 防止重复启动
    timerRef.current = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);
  }

  function stop() {
    clearInterval(timerRef.current); // 用 ID 停掉定时器
    timerRef.current = null;
  }

  return (
    <div>
      <p>{seconds} 秒</p>
      <button onClick={start}>开始</button>
      <button onClick={stop}>停止</button>
    </div>
  );
}

// ============ 用途三：记录上一次的值 ============
function CounterWithPrev() {
  const [count, setCount] = useState(0);
  const prevCount = useRef(0);

  useEffect(() => {
    prevCount.current = count; // 每次渲染后更新
  });

  return (
    <div>
      <p>现在：{count}</p>
      <p>上一次：{prevCount.current}</p>
      <button onClick={() => setCount(p => p + 1)}>+1</button>
    </div>
  );
}

export default SearchBar;
```

⚠️ 易错点：
1. ref 改了页面不会更新，想让页面更新要用 useState
2. 读写 ref 必须用 `.current`，直接用 `ref` 拿到的是整个对象 `{ current: ... }`
3. 操作 DOM 必须在 useEffect 里（渲染完成后），不能在组件函数里直接调用
4. 普通变量存计时器 ID 会在重新渲染时丢失，必须用 useRef

---

## 🎯 2. 使用场景

1. **SaaS 场景（搜索页）**：搜索框自动 focus，用户进入页面直接打字，不需要手动点
2. **E-commerce 场景**：防抖搜索，用 useRef 存 setTimeout ID，每次输入先 clearTimeout 再开新的
3. **HealthTech 场景**：计时器控制，用 useRef 存 interval ID，可以精确启动/停止

---

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| useState | 改了页面会更新 | 多余渲染 | 数据需要显示在页面上 |
| 普通变量 | 最简单 | 重新渲染后丢失 | 只在一次函数执行内用 |
| document.getElementById | 直接 | 绕过 React 可能出 bug | 避免在 React 里用 |

**选型决策树**：
- 数据需要显示在页面 → useState
- 数据不需要显示，但要跨渲染保持 → useRef
- 需要操作 DOM 元素 → useRef

---

## 💼 4. 面试题

### 题 1（🟡 进阶）

**Q**: useRef 和 useState 有什么区别？

**核心 points**:
- useState 变了触发重新渲染，useRef 不会
- useRef 通过 `.current` 读写值
- useState 适合页面上显示的数据，useRef 适合内部逻辑用的数据

**追问方向**:
1. 为什么不全用 useState？
2. useRef 的值在重新渲染后会丢失吗？

**加分回答**: "useRef 的值在重新渲染之间是持久的，不会丢失——这点和普通变量不一样。普通变量每次渲染都重新初始化，useRef 的值一直保持到组件卸载。这就是为什么存计时器 ID 要用 useRef 而不是普通变量。"

---

## 🧪 5. 小测验

做一个带自动 focus 的搜索框：
1. 页面加载时输入框自动 focus
2. 一个"清空"按钮，点击后清空输入框并重新 focus

```tsx
function SearchBar() {
  const [query, setQuery] = useState('');
  const inputRef = useRef(null);

  // 👇 任务一：页面加载时自动 focus
  useEffect(() => {
    // ???
  }, []);

  // 👇 任务二：清空并重新 focus
  function handleClear() {
    // ???
  }

  return (
    <div>
      <input ref={inputRef} value={query}
        onChange={e => setQuery(e.target.value)} />
      <button onClick={handleClear}>清空</button>
    </div>
  );
}
```

> 点击展开答案前先思考

<details>
<summary>展开答案</summary>

```tsx
useEffect(() => {
  inputRef.current.focus();
}, []);

function handleClear() {
  setQuery('');
  inputRef.current.focus();
}
```
</details>

---

## 🌟 Senior 思维彩蛋

**能用 state 控制的绝对不用 ref 直接操作 DOM。** 比如控制输入框的值，用 `value={query}` + `onChange`，不要用 `inputRef.current.value = ''`。后者绕过了 React，会导致 React 的状态和页面不同步，出现奇怪 bug。ref 操作 DOM 只用在 React 真的做不到的事情上：focus、scroll、读取尺寸。

---

## 📅 复习记录

- [ ] D+3 复习
- [ ] D+7 复习
- [ ] D+21 复习

## 🔗 相关笔记

- [[Topic-03-useState]]
- [[Topic-06-useEffect]]
