---
tier: Tier-1
tech: React
topic: Props
knowledge-point: 父子组件通信
difficulty: ⭐
status: done
created: 2026-05-25
review-1: 
review-2: 
review-3: 
tags:
  - tech/react
  - tier-1
  - props
---

# 📌 Topic 2：Props — 怎么给组件传数据

> Topic: React Tier 1 · Tier: 1 · 难度: ⭐

---

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Props | 属性 | 父组件传给子组件的数据，子组件只能读不能改 |
| Parent | 父组件 | 包含其他组件的组件 |
| Child | 子组件 | 被包含在其他组件里的组件 |
| Destructuring | 解构 | 从对象里一次拿出多个值 `const { name } = props` |
| Default Props | 默认属性 | props 没传时使用的默认值 |
| Children | 子内容 | 组件标签之间的内容，通过 `props.children` 访问 |

---

## 📝 1. 实例代码

```tsx
// 代码读法说明：
// 1. 父组件通过属性传数据给子组件
// 2. 子组件通过 props 接收数据
// 3. props 是只读的，不能在子组件里修改

// 子组件：接收 props
function JobCard({ title, company, salary = 0 }) {
  // 解构 props，salary 有默认值 0
  return (
    <div>
      <h3>{title}</h3>
      <p>{company}</p>
      <p>${salary.toLocaleString()}/年</p>
    </div>
  );
}

// 父组件：传 props
function JobList() {
  return (
    <div>
      <JobCard
        title="Senior Engineer"
        company="Canva"
        salary={180000}
      />
      <JobCard
        title="Full Stack Dev"
        company="Atlassian"
        salary={160000}
      />
    </div>
  );
}

// props.children 用法
function Card({ children, title }) {
  return (
    <div className="card">
      <h2>{title}</h2>
      {children}  {/* 标签之间的内容 */}
    </div>
  );
}

// 使用 children
function App() {
  return (
    <Card title="职位信息">
      <p>这里是卡片内容</p>  {/* 这就是 children */}
    </Card>
  );
}

export default JobList;
```

⚠️ 易错点：
1. props 是只读的，不能在子组件里 `props.title = '新标题'`（会报错）
2. 传数字/boolean/对象要用 `{}`，传字符串可以直接用引号
3. 忘记解构，直接用 `props.xxx` 也可以但更啰嗦

---

## 🎯 2. 使用场景

1. **SaaS 场景（Canva）**：设计模板卡片组件接收 `title`、`thumbnail`、`author` props，首页、搜索页、个人页都复用这一个组件
2. **E-commerce 场景（Catch.com.au）**：商品卡片接收 `name`、`price`、`image`、`discount` props，不同页面展示不同商品
3. **HealthTech 场景**：预约卡片接收 `doctorName`、`time`、`status` props，不同状态显示不同样式

---

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| useContext | 跨层级传数据，不用层层传 | 隐式依赖，调试难 | 全局数据（主题、用户信息） |
| Redux/Zustand | 全局状态管理 | 额外依赖，学习成本 | 大型应用复杂状态 |
| Props Drilling | 简单直接 | 层级深了很啰嗦 | 2-3 层内的数据传递 |

**选型决策树**：
- 如果父子直接传 → 用 Props
- 如果需要跨多层 → 用 useContext（Topic 8）
- 如果全局共享且频繁变化 → 用 Zustand

---

## 💼 4. 面试题

### 题 1（🟢 基础）

**Q**: Props 和 State 有什么区别？

**核心 points**:
- Props 由父组件传入，子组件只读；State 是组件自己的数据，可以修改
- Props 变了组件重新渲染；State 变了组件也重新渲染
- Props 是"外部数据"，State 是"内部数据"

**追问方向**:
1. 子组件可以修改 props 吗？
2. 什么情况下把数据放 props，什么时候放 state？

**加分回答**: "React 的数据流是单向的——从父到子通过 props，子组件想改父组件的数据要通过回调函数（父传给子一个函数，子调用这个函数）。这个设计让数据流向清晰，容易调试。"

---

## 🧪 5. 小测验

补全下面的 `UserBadge` 组件，让它接收 `name`、`role`、`isOnline` 三个 props，并显示出来：

```tsx
function UserBadge(???) {
  return (
    <div>
      <p>???</p>
      <p>???</p>
      {/* isOnline 是 true 显示"在线"，false 显示"离线" */}
      <p>???</p>
    </div>
  );
}

// 使用方式：
<UserBadge name="张三" role="Senior Engineer" isOnline={true} />
```

> 点击展开答案前先思考

<details>
<summary>展开答案</summary>

```tsx
function UserBadge({ name, role, isOnline }) {
  return (
    <div>
      <p>{name}</p>
      <p>{role}</p>
      <p>{isOnline ? '在线' : '离线'}</p>
    </div>
  );
}
```
</details>

---

## 🌟 Senior 思维彩蛋

Senior 在设计组件时会问："这个数据是谁的？"——如果是外部传入的就用 props，如果是组件自己管理的就用 state。另一个原则：**props 越少越好**，一个组件接收超过 5-6 个 props，通常意味着组件承担了太多职责，应该拆分。

---

## 🔗 关联算法题

- 本 Topic 无直接关联算法题

---

## 📅 复习记录

- [ ] D+3 复习
- [ ] D+7 复习
- [ ] D+21 复习

## 🔗 相关笔记

- [[Topic-01-JSX-组件基本写法]]
- [[Topic-03-useState]]
- [[Topic-08-useContext]]
