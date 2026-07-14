---
tier: Tier-1
tech: React
topic: JSX + 组件基本写法
knowledge-point: JSX语法、函数组件
difficulty: ⭐
status: done
created: 2026-05-25
review-1: 
review-2: 
review-3: 
tags:
  - tech/react
  - tier-1
  - jsx
  - component
---

# 📌 Topic 1：JSX + 组件基本写法

> Topic: React Tier 1 · Tier: 1 · 难度: ⭐

---

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| JSX | JSX | 在 JS 文件里写"看起来像 HTML"的语法 |
| Component | 组件 | 一块可复用的 UI 积木，有自己的样子和行为 |
| Function Component | 函数组件 | 用普通 JS 函数写的组件，现代 React 全部用这种 |
| Return | 返回 | 函数组件必须 return 一段 JSX，React 才知道要画什么 |
| Export | 导出 | 让这个组件能被其他文件使用 |
| Compile | 编译 | 把 JSX 翻译成浏览器能看懂的代码，自动完成 |

---

## 📝 1. 实例代码

```tsx
// 代码读法说明：
// 1. JSX 看起来像 HTML，但其实是 JS
// 2. Babel 自动把 JSX 翻译成 React.createElement()
// 3. 函数组件名必须大写开头
// 4. 必须 return 一个根元素

import React from 'react';

// 最简单的函数组件
function Greeting() {
  return (
    <h1>你好，Sydney！</h1>
  );
}

// 组件里可以先写 JS 逻辑，再 return JSX
function UserCard() {
  const name = '张三';
  const role = 'Senior Engineer';

  return (
    <div>
      <h2>{name}</h2>
      <p>{role}</p>
    </div>
  );
}

// 导出组件
export default function WelcomePage() {
  return (
    <div>
      <Greeting />
      <UserCard />
    </div>
  );
}
```

⚠️ 易错点：
1. JSX 里用 `className` 不用 `class`，用 `htmlFor` 不用 `for`
2. JSX 里注释用 `{/* */}` 不用 `<!-- -->`
3. `return` 后面换行必须加括号，否则 JS 自动加分号返回 undefined
4. 组件名必须大写开头，小写会被当成 HTML 标签

---

## 🎯 2. 使用场景

1. **SaaS 场景（Atlassian Jira）**：每张任务卡片是一个组件，写一次到处复用，团队可以同时开发不同组件互不干扰
2. **E-commerce 场景（Kmart AU）**：商品列表页每个商品格子是一个组件，改一次样式所有地方同步更新
3. **HealthTech 场景（Telstra Health）**：病人信息卡、预约日历、报告图表各自是独立组件，方便维护

---

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| Vue Template | 更接近 HTML，新手友好 | 和 JS 逻辑分离，灵活性差 | 团队 HTML/CSS 背景强 |
| Angular Template | 企业级功能内置 | 语法复杂，学习曲线陡 | 大型企业项目 |
| 纯 HTML + JS | 零依赖，加载最快 | 手动操作 DOM 痛苦 | 极简静态页面 |

**选型决策树**：
- 如果公司用 React → 用 JSX，别想了
- 如果新项目团队都是新人 → Vue 上手更快
- 如果静态展示页无复杂交互 → 考虑纯 HTML

---

## 💼 4. 面试题

### 题 1（🟢 基础）

**Q**: JSX 是什么？它和 HTML 有什么区别？

**核心 points**:
- JSX 不是 HTML，是 JS 的语法扩展
- 浏览器不认识 JSX，需要 Babel 编译成 `React.createElement()`
- JSX 里用 `className` 不用 `class`，用 `htmlFor` 不用 `for`

**追问方向**:
1. JSX 能写在 `.js` 文件里吗？和 `.tsx` 有什么区别？
2. 为什么 JSX 必须有一个根元素？

**加分回答**: "JSX 编译后是 `React.createElement(type, props, children)` 的调用。根元素的限制是因为函数只能返回一个值，用 Fragment `<>...</>` 可以避免多余的 DOM 节点。"

---

## 🧪 5. 小测验

下面这段代码有 2 个错误，找出来并修复：

```tsx
function ProfileCard() {
  return (
    <div class="card">
      <!-- 用户信息 -->
      <h2>张三</h2>
      <p>Senior Engineer @ Canva</p>
    </div>
  );
}
```

> 点击展开答案前先思考

<details>
<summary>展开答案</summary>

**错误 1**：`class` 应该改成 `className`

**错误 2**：`<!-- 用户信息 -->` 是 HTML 注释写法，JSX 里要用 `{/* 用户信息 */}`

```tsx
function ProfileCard() {
  return (
    <div className="card">
      {/* 用户信息 */}
      <h2>张三</h2>
      <p>Senior Engineer @ Canva</p>
    </div>
  );
}
```
</details>

---

## 🌟 Senior 思维彩蛋

Senior 在 Code Review 时看到有人用 `<>...</>` 空标签（Fragment）而不是多余的 `<div>`，会给个👍——因为少一个 DOM 节点，页面结构更干净。Topic 13 会讲为什么减少不必要的 DOM 节点在性能优化里很重要。

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
- [[Topic-03-useState]]
