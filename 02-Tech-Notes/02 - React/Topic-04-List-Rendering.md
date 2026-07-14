---
tier: Tier-1
tech: React
topic: 列表渲染 + key
knowledge-point: map、filter、key的作用
difficulty: ⭐
status: done
created: 2026-05-25
review-1: 
review-2: 
review-3: 
tags:
  - tech/react
  - tier-1
  - list
  - map
  - key
---

# 📌 Topic 4：列表渲染 + key

> Topic: React Tier 1 · Tier: 1 · 难度: ⭐

---

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| .map() | 遍历转换 | 把数组每一项变成另一个东西，返回新数组 |
| .filter() | 过滤 | 留下符合条件的项，扔掉不符合的 |
| .reduce() | 归纳 | 把数组所有项合并成一个值（如求和） |
| key | 唯一标识 | 列表每一项的身份证，帮 React 识别谁是谁 |
| Unique | 唯一的 | 整个列表里没有重复的 key |
| Stable | 稳定的 | key 不会因为位置变化而改变 |

---

## 📝 1. 实例代码

```tsx
// 代码读法说明：
// 1. .map() 把数组变成 JSX 数组，React 自动渲染
// 2. key 放在最外层元素上，必须唯一且稳定
// 3. .filter() 过滤，.reduce() 求和
// 4. 增删操作：新增用 spread，删除用 filter

import { useState } from 'react';

function JobList() {
  const [jobs, setJobs] = useState([
    { id: 1, title: 'Senior Engineer', company: 'Canva', price: 180000 },
    { id: 2, title: 'Full Stack Dev', company: 'Atlassian', price: 160000 },
    { id: 3, title: 'Frontend Lead', company: 'Afterpay', price: 170000 },
  ]);
  const [newTitle, setNewTitle] = useState('');
  const [newCompany, setNewCompany] = useState('');

  // 删除：用 filter 过滤掉那一项
  function deleteJob(id) {
    setJobs(prev => prev.filter(job => job.id !== id));
  }

  // 新增：用 spread 展开原数组，加上新项
  function addJob() {
    if (!newTitle || !newCompany) return;
    setJobs(prev => [...prev, {
      id: Date.now(),
      title: newTitle,
      company: newCompany,
      price: 0,
    }]);
    setNewTitle('');
    setNewCompany('');
  }

  // 派生状态：直接算
  const total = jobs.reduce((sum, job) => sum + job.price, 0);
  const isEmpty = jobs.length === 0;

  return (
    <div>
      {/* 空状态 */}
      {isEmpty && <p>暂无职位 🚀</p>}

      {/* 列表渲染：key 用 job.id，不用 index */}
      {jobs.map(job => (
        <div key={job.id}>
          <h3>{job.title}</h3>
          <p>{job.company}</p>
          <button onClick={() => deleteJob(job.id)}>删除</button>
        </div>
      ))}

      <p>总薪资：${total.toLocaleString()}</p>

      {/* 新增表单 */}
      <input value={newTitle} onChange={e => setNewTitle(e.target.value)} placeholder="职位名" />
      <input value={newCompany} onChange={e => setNewCompany(e.target.value)} placeholder="公司名" />
      <button onClick={addJob}>+ 新增</button>
    </div>
  );
}

export default JobList;
```

⚠️ 易错点：
1. `.map()` 必须有 `key`，放在最外层元素上，否则 React 报警告
2. 动态列表不要用 `index` 当 key，删除/排序时会出现 bug
3. 直接 `push` 修改原数组，React 检测不到变化，页面不更新
4. 空列表时 `.map()` 返回空数组，React 什么都不画，需要单独处理空状态

---

## 🎯 2. 使用场景

1. **SaaS 场景（Jira）**：看板任务列表，用 `.map()` 渲染每张卡片，新增任务用 spread，删除用 filter
2. **E-commerce 场景（Catch.com.au）**：商品列表，`.reduce()` 计算购物车总价，`.filter()` 实现筛选
3. **HealthTech 场景**：预约列表，按日期 `.filter()` 筛选今日预约，`.map()` 渲染每条记录

---

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| for 循环 | 初学者熟悉 | 不能直接在 JSX 里用 | 几乎不用于渲染 |
| forEach | 看起来像 map | 没有返回值，不能用于渲染 | 绝对不要用于渲染 |
| 虚拟列表(react-window) | 只渲染可见项，性能好 | 额外依赖，配置复杂 | 列表超过 1000 条时 |

**选型决策树**：
- 渲染列表 → 永远用 `.map()`
- 数据量超 1000 条 → 考虑 react-window 虚拟列表
- 需要过滤 → `.filter()`
- 需要求和/合并 → `.reduce()`

---

## 💼 4. 面试题

### 题 1（🟢 基础）

**Q**: React 列表里的 key 有什么要求？可以用 index 当 key 吗？

**核心 points**:
- key 必须唯一且稳定
- 静态列表用 index 勉强可以
- 动态列表（有增删排序）用 index 会出 bug
- 最好用后端给的唯一 id

**追问方向**:
1. 用 index 当 key 会出什么具体的 bug？
2. 如果后端没给 id 怎么办？

**加分回答**: "用 index 当 key 最典型的 bug 是输入框错位。比如列表里每项有个 input，删了第一项后，React 以为 key=0 还在，就把第二项的 input 当成原来第一项的 input，导致输入内容显示在错误的地方。用唯一 id 就不会有这个问题。"

---

## 🧪 5. 小测验

找出下面代码的 key 问题，说明原因并修复：

```tsx
function PlayerList() {
  const players = [
    { name: '张三', score: 100 },
    { name: '李四', score: 200 },
    { name: '张三', score: 150 }, // 注意！同名
  ];

  return (
    <ul>
      {players.map((player, index) => (
        <li key={player.name}>
          {player.name}: {player.score}分
        </li>
      ))}
    </ul>
  );
}
```

> 点击展开答案前先思考

<details>
<summary>展开答案</summary>

**问题**：`key={player.name}` 用了名字当 key，但有两个"张三"，key 重复了！

**修复方案一**：加唯一 id
```tsx
const players = [
  { id: 1, name: '张三', score: 100 },
  { id: 2, name: '李四', score: 200 },
  { id: 3, name: '张三', score: 150 },
];
<li key={player.id}>
```

**修复方案二**：这是静态列表不会改变，用 index 勉强可以
```tsx
<li key={index}>
```
</details>

---

## 🌟 Senior 思维彩蛋

**`key` 只是给 React 看的，不会显示在页面上，也不会传给子组件。** 如果你需要在子组件里用这个 id，要单独传一个 prop：`<Card key={item.id} id={item.id} />`。这个坑很多人踩过——以为子组件能拿到 key，结果拿不到，`props.key` 永远是 `undefined`。

---

## 🔗 关联算法题

- LeetCode #1 Two Sum（练习 HashMap 思维）
- LeetCode #49 Group Anagrams（练习分组逻辑）

---

## 📅 复习记录

- [ ] D+3 复习
- [ ] D+7 复习
- [ ] D+21 复习

## 🔗 相关笔记

- [[Topic-03-useState]]
- [[Topic-05-条件渲染]]
