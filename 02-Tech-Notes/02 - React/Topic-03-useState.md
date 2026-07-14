---
tier: Tier-1
tech: React
topic: useState
knowledge-point: 状态管理、异步更新、函数式更新
difficulty: ⭐
status: done
created: 2026-05-25
review-1: 
review-2: 
review-3: 
tags:
  - tech/react
  - tier-1
  - useState
  - state
---

# 📌 Topic 3：useState — 让页面"动起来"

> Topic: React Tier 1 · Tier: 1 · 难度: ⭐

---

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| State | 状态 | 组件自己记住的数据，变了页面会自动更新 |
| useState | 状态钩子 | React 提供的工具，让函数组件能"记住"数据 |
| Re-render | 重新渲染 | state 变了，React 重新画这个组件，页面更新 |
| Batch | 批处理 | React 把多次 setState 合并成一次更新，减少渲染次数 |
| Functional Update | 函数式更新 | 传函数给 setXxx，参数是上一次的值 |
| Derived State | 派生状态 | 能从现有 state 算出来的值，不需要单独存 |

---

## 📝 1. 实例代码

```tsx
// 代码读法说明：
// 1. useState(初始值) 返回 [当前值, 更新函数]
// 2. 更新函数被调用后，React 重新渲染组件
// 3. setState 是异步的，不是立刻更新
// 4. 新值依赖旧值时，用函数式更新 prev => ...

import { useState } from 'react';

function ShoppingCart() {
  // 基本用法
  const [count, setCount] = useState(0);
  const [color, setColor] = useState('red');

  // 对象 state
  const [newJob, setNewJob] = useState({
    title: '',
    company: '',
    type: 'fulltime',
  });

  // ✅ 新值依赖旧值 → 用函数式更新
  function addThree() {
    setCount(prev => prev + 1); // prev 是最新值
    setCount(prev => prev + 1);
    setCount(prev => prev + 1); // 最终 +3
  }

  // ❌ 错误：三次都基于旧值，最终只 +1
  function addThreeWrong() {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
  }

  // 更新对象 state：必须展开旧值
  function updateTitle(newTitle) {
    setNewJob(prev => ({
      ...prev,           // 保留其他字段
      title: newTitle,   // 只更新 title
    }));
  }

  // 派生状态：直接算，不用 useState
  const isEmpty = count === 0;       // ✅ 派生
  const double = count * 2;          // ✅ 派生

  return (
    <div>
      <p>数量：{count}</p>
      <p>是否为空：{isEmpty ? '是' : '否'}</p>
      <button onClick={() => setCount(prev => prev + 1)}>+1</button>
      <button onClick={() => setCount(0)}>重置</button>
      <button onClick={() => setColor('blue')}>变蓝</button>
    </div>
  );
}

export default ShoppingCart;
```

⚠️ 易错点：
1. `setState` 之后立刻读 state 拿到的是旧值（异步）
2. 新值依赖旧值时必须用函数式更新，否则批处理会导致结果错误
3. 更新对象 state 忘记 `...prev`，其他字段会消失
4. 能算出来的值（派生状态）不要存进 state，越少越好

---

## 🎯 2. 使用场景

1. **E-commerce 场景（Catch.com.au）**：购物车数量，用 `prev => prev + 1` 保证快速点击时计算正确
2. **SaaS 场景（Atlassian Jira）**：筛选面板开关 `isOpen`，控制侧边栏显示/隐藏
3. **HealthTech 场景**：预约表单多字段，用对象 state 集中管理，提交时直接读 state

---

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| useReducer | 复杂状态逻辑集中管理 | 写法更啰嗦 | 多字段表单、购物车等复杂场景 |
| Zustand | 全局状态，组件间共享 | 多引入一个依赖 | 多组件共享同一个 state |
| useRef | 改变时不触发渲染 | 页面不会更新 | 存不需要显示的内部数据 |

**选型决策树**：
- 数据需要显示在页面，改了要更新 UI → `useState`
- 多字段、更新逻辑复杂 → `useReducer`（Topic 11）
- 多组件共享 → `useContext` 或 Zustand
- 不影响 UI → `useRef`（Topic 7）

---

## 💼 4. 面试题

### 题 1（🟢 基础）

**Q**: React 里为什么要用 useState，直接用普通变量不行吗？

**核心 points**:
- 普通变量改了 React 不知道，不会触发重新渲染
- `useState` 改了会通知 React，React 重新执行组件函数，页面更新
- React 的渲染机制是"数据驱动 UI"，state 是驱动的源头

**追问方向**:
1. 每次重新渲染，`useState(0)` 会不会把值重置回 0？
2. 什么情况下组件会重新渲染？

**加分回答**: "普通变量在每次重新渲染时都会被重置，因为函数重新执行了。useState 的值由 React 在组件外部管理，重新渲染不会丢失。这也是为什么 Hook 只能在组件顶层调用——React 靠调用顺序来匹配每个 useState 对应的值。"

### 题 2（🟢 基础）

**Q**: `setState` 是同步还是异步的？

**核心 points**:
- 异步的——React 会批处理多次 setState，合并成一次渲染
- setState 后立刻读 state 拿到的是旧值
- 想基于旧值更新，用函数式 `prev => prev + 1`

**追问方向**:
1. 那怎么在 setState 之后拿到最新的值？
2. 什么情况下 setState 是同步的？

**加分回答**: "React 18 之后所有场景都是批处理，包括 setTimeout 和 Promise 里的 setState。之前只有事件处理函数里才批处理。这个变化让性能更好，但也意味着更多场景需要用函数式更新。"

---

## 🧪 5. 小测验

做一个红绿灯，点"切换"按顺序切换：红→绿→黄→红→...

```tsx
import { useState } from 'react';

function TrafficLight() {
  const [color, setColor] = useState('红');

  function handleSwitch() {
    // 👇 你来写
  }

  return (
    <div>
      <p>当前：{color}灯</p>
      <button onClick={handleSwitch}>切换</button>
    </div>
  );
}
```

> 点击展开答案前先思考

<details>
<summary>展开答案</summary>

```tsx
function handleSwitch() {
  const next = {
    '红': '绿',
    '绿': '黄',
    '黄': '红',
  };
  setColor(next[color]);
}
```

**为什么用对象查找表而不是 if/else？**
对象查找表比 if-else 更简洁，加新颜色只需要加一行，不需要改逻辑结构。这叫"开放封闭原则"。
</details>

---

## 🌟 Senior 思维彩蛋

**useState 的初始值只在第一次渲染时用一次**，后面重新渲染都忽略它。所以如果初始值需要复杂计算，别直接写 `useState(heavyCalculation())`——每次渲染都会白跑一次。正确写法是传函数：`useState(() => heavyCalculation())`。这叫"惰性初始化"，Topic 10 会详细讲。

---

## 🔗 关联算法题

- 本 Topic 无直接关联算法题

---

## 📅 复习记录

- [ ] D+3 复习
- [ ] D+7 复习
- [ ] D+21 复习

## 🔗 相关笔记

- [[Topic-02-Props]]
- [[Topic-11-useReducer]]
- [[Topic-07-useRef]]
