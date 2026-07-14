---
tier: Tier-1
tech: React
topic: useContext
knowledge-point: 跨层级传数据、Provider、性能陷阱
difficulty: ⭐⭐
status: done
created: 2026-05-25
tags:
  - tech/react
  - tier-1
  - useContext
  - context
---

# 📌 Topic 8：useContext — 跨层级传数据

> Topic: React Tier 1 · Tier: 1 · 难度: ⭐⭐

---

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Props Drilling | 属性穿透 | 数据从最外层一层层传到最里层，中间层不需要但也要传 |
| createContext | 创建上下文 | 创建一个"全公司群"，用来存共享数据 |
| Provider | 提供者 | 把数据放进群里，包住需要用这个数据的组件 |
| useContext | 使用上下文 | 从群里取数据，任何被 Provider 包住的组件都能用 |
| Consumer | 消费者 | 取数据的组件 |

---

## 📝 1. 实例代码

```tsx
// 代码读法说明：
// 三步口诀：创建群 → 放数据 → 取数据

import { createContext, useContext, useState } from 'react';

// 第一步：创建 Context（建群）
const UserContext = createContext(null);
const ThemeContext = createContext(null);

// 第二步：Provider 提供数据（放数据进群）
function App() {
  const [user, setUser] = useState({ name: '张三', role: 'admin' });
  const [theme, setTheme] = useState('light');

  return (
    // 两个 Context 分开，互不影响
    <UserContext.Provider value={{ user, setUser }}>
      <ThemeContext.Provider value={{ theme, setTheme }}>
        <Layout />  {/* 不需要传任何 props！ */}
      </ThemeContext.Provider>
    </UserContext.Provider>
  );
}

// 中间层：完全不需要管这些数据
function Layout() {
  return <Sidebar />;
}
function Sidebar() {
  return <UserMenu />;
}

// 第三步：useContext 取数据（从群里拿）
function UserMenu() {
  const { user } = useContext(UserContext); // 直接取，不需要 props
  return <p>你好，{user.name}！</p>;
}

function ThemeButton() {
  const { theme, setTheme } = useContext(ThemeContext);
  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      切换主题（{theme}）
    </button>
  );
}

export default App;
```

⚠️ 易错点：
1. 忘记用 Provider 包住组件，useContext 拿到的是默认值 null
2. createContext 要在组件外面定义，否则每次渲染都创建新的 Context
3. value 传对象时取出来要解构：`const { theme } = useContext(ThemeContext)`，不是 `const theme = useContext(...)`
4. 把频繁变化的数据放进 Context，导致所有消费者频繁重新渲染

---

## 🎯 2. 使用场景

1. **SaaS 场景（Atlassian）**：用户登录信息放 AuthContext，导航栏、设置页、个人主页都直接取，不层层传 props
2. **E-commerce 场景**：购物车主题切换放 ThemeContext，任何地方的按钮都能切换，所有组件同步变色
3. **HealthTech 场景**：语言设置放 i18nContext，所有需要显示文字的组件直接取当前语言

---

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| Props | 显式，数据流清晰 | 层级深了啰嗦 | 2-3 层内的数据传递 |
| Zustand | 全局状态，细粒度订阅 | 额外依赖 | 频繁变化的全局状态 |
| Redux | 功能最全，DevTools 强大 | 学习成本高，代码多 | 超大型应用 |

**选型决策树**：
- 只有 2-3 层 → Props
- 跨多层，变化少 → useContext
- 频繁变化，多组件共享 → Zustand
- 超大型应用，需要 DevTools → Redux

---

## 💼 4. 面试题

### 题 1（🟡 进阶）

**Q**: useContext 解决了什么问题？怎么用？

**核心 points**:
- 解决 Props Drilling（层层传 props）
- 三步：createContext → Provider → useContext
- Provider 包住的组件都能直接取数据

**追问方向**:
1. createContext 的默认值是什么时候用到的？
2. Provider 的 value 变了，子组件会重新渲染吗？

### 题 2（🟡 进阶）

**Q**: Context 有什么性能问题？怎么解决？

**核心 points**:
- Context 值变了，所有消费者都重新渲染
- 解决：拆分 Context，按变化频率分开
- 解决：用 memo 包住不需要频繁渲染的组件

**加分回答**: "Context 本质是依赖注入，不是状态管理。它解决的是数据传递问题，不是状态管理问题。频繁变化的状态用 Zustand 更合适，它有更细粒度的订阅机制，只有用到的那部分数据变了才重新渲染。"

---

## 🧪 5. 小测验

做一个主题切换，要求：
- App 里有 `theme` state
- 把 `theme` 和 `setTheme` 放进 Context
- `ThemeButton` 直接从 Context 取，切换主题
- `Display` 直接从 Context 取，显示当前主题

> 点击展开答案前先思考

<details>
<summary>展开答案</summary>

```tsx
const ThemeContext = createContext(null);

function App() {
  const [theme, setTheme] = useState('light');
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <Display />
      <ThemeButton />
    </ThemeContext.Provider>
  );
}

function Display() {
  const { theme } = useContext(ThemeContext); // 解构！
  return <p>当前主题：{theme}</p>;
}

function ThemeButton() {
  const { theme, setTheme } = useContext(ThemeContext);
  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      切换主题
    </button>
  );
}
```
</details>

---

## 🌟 Senior 思维彩蛋

**Context 不是越多越好，也不是越少越好。** 什么数据适合放 Context：变化少、全局需要（用户信息、主题、语言）。什么不适合：频繁变化（计数器、表单数据）、只有少数组件用（直接传 props 更清晰）。滥用 Context 让数据流向不清晰，难以调试。

---

## 📅 复习记录

- [ ] D+3 复习
- [ ] D+7 复习
- [ ] D+21 复习

## 🔗 相关笔记

- [[Topic-02-Props]]
- [[Topic-09-自定义Hook]]
