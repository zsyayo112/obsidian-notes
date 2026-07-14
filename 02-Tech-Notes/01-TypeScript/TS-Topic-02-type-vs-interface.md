---
tier: Tier-1
tech: TypeScript
topic: type vs interface
knowledge-point: interface/type/extends/合并/选型
difficulty: ⭐
status: done
created: 2026-05-24
review-1: 2026-05-27
review-2: 2026-05-31
review-3: 2026-06-14
tags:
  - tech/typescript
  - tier-1
---

# 📌 TypeScript type vs interface

> Topic: type vs interface · Tier: 1 · 难度: ⭐

---

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Interface | 接口 | 定义对象形状的关键字，可继承可合并 |
| Type Alias | 类型别名 | 用 `type` 给一个类型起名字 |
| extends | 继承 | interface 用 extends 继承另一个 interface 的所有字段 |
| Declaration Merging | 声明合并 | 同名 interface 自动合并，type 不支持 |
| Intersection | 交叉类型 | 用 `&` 把两个类型合并，type 用这个代替 extends |

---

## 📝 1. 实例代码

```typescript
// 代码读法说明：
// - 前两段分别用 interface 和 type 定义同样的对象
// - 中间展示 interface extends 继承
// - 最后展示 type & 交叉（等效继承）

// interface 定义对象
interface User {
  id: number
  name: string
  email: string
  age?: number       // 可选属性
}

// type 定义对象（效果一样，多一个 =）
type Product = {
  id: number
  name: string
  price: number
}

// interface extends 继承
interface Admin extends User {
  role: string       // Admin 拥有 User 全部字段 + role
}

const admin1: Admin = {
  id: 1,
  name: "Alice",
  email: "alice@example.com",
  role: "manager",
}

// type & 交叉（等效继承）
type HasName = { name: string }
type HasAge = { age: number }
type Person = HasName & HasAge   // 同时拥有两者的字段

// interface 同名自动合并（type 不支持）
interface Window {
  title: string
}
interface Window {
  width: number      // 自动合并，不报错
}
// 最终 Window = { title: string; width: number }
```

⚠️ 易错点：
1. `type` 加 `=`，`interface` 不加 `=`，最常见笔误
2. `type` 重复定义直接报错，`interface` 同名自动合并
3. `interface extends type` 是合法的，两者可以混用

---

## 🎯 2. 使用场景

1. **SaaS 场景**：定义多租户系统的用户模型，用 `interface User` + `interface AdminUser extends User` 表达权限层级，继承链清晰。
2. **Fintech 场景**：用 `type` 定义联合类型表达支付状态 `type PaymentStatus = "pending" | "success" | "failed"`，比 interface 更简洁。
3. **E-commerce 场景**：购物车 Item 类型用 `interface`，方便后续 `extends` 拓展出促销 Item、Gift Item 等变体。

---

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| 只用 `interface` | 统一风格，继承语义清晰 | 无法表达联合类型 | 团队约定对象类型统一用 interface |
| 只用 `type` | 更灵活，联合/交叉都支持 | 不能声明合并 | 函数式风格项目 |
| class | 既是类型又是值，可实例化 | 比 interface 重，编译产物多 | 需要实例化和方法的场景 |

**选型决策树**：
- 如果是普通对象类型定义 → `interface`（约定俗成）
- 如果是联合类型 `A | B` → `type`
- 如果是交叉类型 `A & B` → `type`
- 如果需要继承 → `interface extends`
- 如果需要声明合并（扩展第三方库类型）→ `interface`

---

## 💼 4. 面试题

### 题 1（🟢 基础）

**Q**: `type` 和 `interface` 有什么区别？什么时候选哪个？

**核心 points**: interface 可继承/合并、type 支持联合交叉、对象用 interface 约定、其他用 type

**追问方向**:
1. 如果要扩展第三方库的类型定义，用 type 还是 interface？为什么？
2. `interface A extends B` 和 `type A = B & {...}` 有什么区别？

**加分回答**: "我们团队的约定是：对象结构用 `interface`，因为可以被继承和合并，方便扩展；联合类型、工具类型、函数类型用 `type`。扩展第三方库时必须用 `interface` 的声明合并，`type` 做不到。"

---

## 🧪 5. 小测验

下面代码有 2 个问题，找出并修复：

```typescript
type Status = "active" | "inactive"
type Status = "pending"

interface Product {
  id: number
  name: string
}

const p: Product = {
  id: 1
}
```

> 先思考，再展开答案 👇

<details>
<summary>展开答案</summary>

**问题 1**：`type` 不能重复定义，报错。修复：合并成一个。
```typescript
type Status = "active" | "inactive" | "pending"
```

**问题 2**：`Product` 有 `name` 字段但对象里没填，报错。修复：补上。
```typescript
const p: Product = {
  id: 1,
  name: "iPhone"
}
```

</details>

---

## 🌟 Senior 思维彩蛋

Senior 在 code review 时看到 `type User = {...}` 会问："这个类型后面会被继承吗？会扩展吗？"如果答案是会，就会建议改成 `interface`。不是因为 `type` 不能用 `&` 交叉，而是 `extends` 的继承语义更清晰，IDE 的错误提示也更友好——`interface` 报错会告诉你哪个字段缺了，`type &` 的报错信息有时候很难读。

---

## 🔗 关联算法题

- LeetCode #146 LRU Cache（class 实现，interface 定义缓存结构）
- HackerRank: 10 Days of JavaScript - Day 5 Inheritance

---

## 📅 复习记录

- [ ] D+3 复习（2026-05-27）
- [ ] D+7 复习（2026-05-31）
- [ ] D+21 复习（2026-06-14）

## 🔗 相关笔记

- [[TS-Topic-01-基本类型标注]]
- [[TS-Topic-03-联合类型与交叉类型]]
- [[TS-Topic-05-对象类型与readonly]]
