---
tier: Tier-1
tech: React
topic: 条件渲染
knowledge-point: &&、三元表达式、if提前返回
difficulty: ⭐
status: done
created: 2026-05-25
review-1: 
review-2: 
review-3: 
tags:
  - tech/react
  - tier-1
  - conditional-rendering
---

# 📌 Topic 5：条件渲染 — 什么时候显示什么

> Topic: React Tier 1 · Tier: 1 · 难度: ⭐

---

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Conditional Rendering | 条件渲染 | 根据条件决定显示什么 |
| Short Circuit | 短路 | `&&` 左边是 false，右边直接不执行 |
| Truthy | 真值 | JS 里被当成 true 的值（非空字符串、非零数字等） |
| Falsy | 假值 | JS 里被当成 false 的值（false、0、''、null、undefined） |
| Early Return | 提前返回 | 满足某个条件就直接 return，不往下走 |
| Guard Clause | 守卫语句 | 在函数开头用 if 把异常情况挡掉 |

---

## 📝 1. 实例代码

```tsx
// 代码读法说明：
// 三种写法，适用不同场景：
// 1. && 短路：只显示或隐藏
// 2. 三元表达式：二选一
// 3. if 提前返回：多种情况

import { useState } from 'react';

function JobList({ isLoading, error, jobs }) {

  // ============ if 提前返回（多种状态）============
  // 口诀：loading → error → 空 → 正常
  if (isLoading) return <p>加载中...</p>;

  if (error) {
    return (
      <div>
        <p>出错啦：{error}</p>
        <button>重试</button>
      </div>
    );
  }

  return (
    <div>
      {/* && 短路：只显示/隐藏 */}
      {jobs.length === 0 && <p>暂无职位 🚀</p>}

      {/* ⚠️ 数字 0 是 falsy，会显示数字 0！ */}
      {/* ❌ {jobs.length && <p>有数据</p>} */}
      {/* ✅ {jobs.length > 0 && <p>有数据</p>} */}

      {/* 三元表达式：二选一 */}
      {isLoading ? (
        <p>加载中...</p>
      ) : (
        <p>加载完成</p>
      )}

      {/* 列表渲染 */}
      {jobs.map(job => (
        <div key={job.id}>{job.title}</div>
      ))}
    </div>
  );
}

export default JobList;
```

⚠️ 易错点：
1. `{count && <p>有消息</p>}` — count 是 0 时页面显示数字 0，应该用 `count > 0 &&`
2. 三元表达式嵌套超过一层，立刻拆成函数或 if 提前返回
3. 空状态用 `&&` 不要用提前 return，否则表单等其他内容会消失

---

## 🎯 2. 使用场景

1. **SaaS 场景（Seek.com.au）**：搜索无结果显示"没有找到职位"，请求失败显示错误+重试按钮
2. **E-commerce 场景（Afterpay）**：订单已付款显示绿色"已完成"，未付款显示红色"待付款"
3. **HealthTech 场景**：加载用户信息时骨架屏，用户不存在显示 404，正常显示资料

---

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| && 短路 | 简洁 | 数字 0 有陷阱 | 只有显示/隐藏 |
| 三元表达式 | 直观 | 嵌套太深难读 | 二选一显示 |
| if 提前返回 | 清晰，适合多种情况 | 代码行数多 | 3种以上状态 |

**选型决策树**：
- 显示或不显示 → `&&`
- 显示 A 或显示 B → 三元表达式
- 三种以上情况 → `if` 提前返回

---

## 💼 4. 面试题

### 题 1（🟢 基础）

**Q**: 一个从后端拿数据的组件，你会怎么设计状态和条件渲染？

**核心 points**:
- 三个 state：`data`、`isLoading`、`error`
- 四种状态用 if 提前返回处理
- 顺序：loading → error → 空 → 正常

**追问方向**:
1. isLoading 和 error 同时为 true 可能吗？
2. 为什么要先判断 loading，再判断 error？

**加分回答**: "这个模式我会抽成自定义 Hook，比如 `useFetch(url)`，返回 `{ data, isLoading, error }`。这样每个需要请求数据的组件都能复用，不用重复写三个 state 和 fetch 逻辑。"

---

## 🧪 5. 小测验

补全 `UserProfile` 组件，处理四种状态：

```tsx
function UserProfile({ isLoading, error, user }) {
  // 👇 第一关：还在加载
  
  // 👇 第二关：出错了

  // 👇 第三关：没有用户

  // 正常显示
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

> 点击展开答案前先思考

<details>
<summary>展开答案</summary>

```tsx
function UserProfile({ isLoading, error, user }) {
  if (isLoading) return <p>加载中...</p>;
  if (error) return <p>加载失败，请重试</p>;
  if (!user) return <p>用户不存在</p>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```
</details>

---

## 🌟 Senior 思维彩蛋

**提前返回让代码"扁平化"**。嵌套越深，代码越难读；提前返回越多，主逻辑越清晰。Senior 写函数时习惯把所有"异常情况"放最前面挡掉，最后一个 return 才是"正常情况"——这叫 Guard Clause 模式，是写干净代码的基本功。

---

## 📅 复习记录

- [ ] D+3 复习
- [ ] D+7 复习
- [ ] D+21 复习

## 🔗 相关笔记

- [[Topic-04-列表渲染+key]]
- [[Topic-06-useEffect]]
