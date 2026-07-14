---
tier: Tier-1
tech: TypeScript
topic: 对象类型与readonly
knowledge-point: 可选属性/只读属性/嵌套对象/readonly数组
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

# 📌 TypeScript 对象类型 + `?` + `readonly`

> Topic: 对象类型与readonly · Tier: 1 · 难度: ⭐

---

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Optional Property | 可选属性 | 用 `?` 标注，这个字段可能不存在 |
| Readonly Property | 只读属性 | 用 `readonly` 标注，赋值后不能再改 |
| Nested Object | 嵌套对象 | 对象里面的字段也是一个对象 |
| Readonly Array | 只读数组 | `readonly T[]`，数组内容不能被修改 |

---

## 📝 1. 实例代码

```typescript
// 代码读法说明：
// - 第一段：嵌套对象类型 + readonly + 可选属性组合使用
// - 第二段：readonly 数组的两层锁定
// - 第三段：验证 readonly 的编译报错

// 嵌套对象 + readonly + 可选属性
interface Order {
  readonly id: number          // 只读：赋值后不能改
  product: {
    name: string
    price: number
  }
  quantity: number
  note?: string                // 可选：可以不填
}

const order1: Order = {
  id: 1,
  product: {
    name: "iPhone",
    price: 999,
  },
  quantity: 2,
  // note 不填也没事
}

// order1.id = 2  // ❌ 报错：Cannot assign to 'id' because it is read-only

function printOrder(order: Order): string {
  return `Order ${order.id}: ${order.product.name} x ${order.quantity}`
}

console.log(printOrder(order1))

// readonly 数组：两层锁定
interface Config {
  readonly appName: string
  readonly allowedRoles: readonly string[]  // 字段不能换，内容也不能改
}

const config: Config = {
  appName: "MyApp",
  allowedRoles: ["admin", "user"],
}

// config.appName = "NewApp"          // ❌ 字段只读
// config.allowedRoles.push("guest")  // ❌ 数组内容只读
// config.allowedRoles = ["admin"]    // ❌ 字段只读
```

⚠️ 易错点：
1. `readonly field` 只锁字段本身（不能换新数组），但数组内容还能 push
2. `readonly T[]` 才能同时锁住数组内容
3. 两个 `readonly` 要一起用才能完全锁死：`readonly allowedRoles: readonly string[]`

---

## 🎯 2. 使用场景

1. **SaaS 场景**：系统配置对象用 `readonly`，防止业务代码意外修改全局配置，导致所有用户受影响。
2. **Fintech 场景**：交易记录用 `readonly id` 和 `readonly amount`，保证核心字段不被业务逻辑篡改，审计时数据一致。
3. **E-commerce 场景**：订单详情页展示用 `readonly` 对象，明确这些数据只读展示，不做修改，配合 React 的 immutable 数据流。

---

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| `Object.freeze()` | 运行时真正冻结对象 | 浅冻结，嵌套对象不受保护 | 需要运行时保护时 |
| `Readonly<T>` 工具类型 | 批量给所有字段加 readonly | 不能精细控制哪些字段只读 | Topic 10 工具类型 |
| `const` 断言 `as const` | 把整个对象深度只读 | 推断类型太窄，不够灵活 | 配置常量对象 |

**选型决策树**：
- 如果单个字段只读 → `readonly field`
- 如果所有字段都只读 → `Readonly<T>`
- 如果整个对象是常量 → `as const`
- 如果需要运行时保护 → `Object.freeze()`

---

## 💼 4. 面试题

### 题 1（🟢 基础）

**Q**: `readonly` 和 `const` 有什么区别？

**核心 points**: const 是变量声明不可重新赋值、readonly 是对象属性不可修改、const 对象内容可变、readonly 字段不可变

**追问方向**:
1. `readonly string[]` 和 `string[]` 有什么区别？
2. 如何实现一个深度只读的对象类型？（提示：递归 + 映射类型）

**加分回答**: "`readonly` 是编译期保护，`Object.freeze()` 是运行时保护，生产环境里我会两个都用——`readonly` 在开发阶段给出错误提示，`freeze()` 在运行时防止第三方库或者动态代码绕过 TS 检查直接修改。"

---

## 🧪 5. 小测验

定义一个 `UserProfile` 类型，要求：
- `id` 只读
- `name` 和 `email` 普通字段
- `address` 可选，包含 `city` 和 `country` 两个字段
- `tags` 是只读字符串数组

然后创建一个对象，试着修改 `id` 和 `tags`，看报什么错。

> 先思考，再展开答案 👇

<details>
<summary>展开答案</summary>

```typescript
interface UserProfile {
  readonly id: number
  name: string
  email: string
  address?: {
    city: string
    country: string
  }
  readonly tags: readonly string[]
}

const profile: UserProfile = {
  id: 1,
  name: "Alice",
  email: "alice@example.com",
  address: {
    city: "Sydney",
    country: "Australia",
  },
  tags: ["admin", "verified"],
}

// profile.id = 2              // ❌ Cannot assign to 'id' because it is read-only
// profile.tags.push("new")    // ❌ Property 'push' does not exist on type 'readonly string[]'
// profile.tags = ["new"]      // ❌ Cannot assign to 'tags' because it is read-only
```

</details>

---

## 🌟 Senior 思维彩蛋

Senior 在设计 API 响应类型时，会把整个响应对象标注为 `Readonly<T>`——不是因为担心自己改，而是因为团队里总有人会"临时改一下"响应数据来修 bug，这种修改在 React 里会绕过 re-render 机制，导致 UI 和状态不同步，而且极难复现。`readonly` 是一种团队契约，写在类型里比写在注释里更有约束力。

---

## 🔗 关联算法题

- LeetCode #54 Spiral Matrix（只读二维数组遍历思维）
- LeetCode #448 Find All Numbers Disappeared in an Array
- HackerRank: Arrays - DS

---

## 📅 复习记录

- [ ] D+3 复习（2026-05-27）
- [ ] D+7 复习（2026-05-31）
- [ ] D+21 复习（2026-06-14）

## 🔗 相关笔记

- [[TS-Topic-02-type-vs-interface]]
- [[TS-Topic-10-工具类型]]
- [[TS-Topic-13-严格模式与生产级实践]]
