---
tier: Tier-1
tech: TypeScript
topic: 泛型基础
knowledge-point: 泛型函数/泛型接口/泛型约束/多泛型参数
difficulty: ⭐⭐
status: done
created: 2026-05-24
review-1: 2026-05-27
review-2: 2026-05-31
review-3: 2026-06-14
tags:
  - tech/typescript
  - tier-1
---

# 📌 TypeScript 泛型基础（Generics）

> Topic: 泛型基础 · Tier: 1 · 难度: ⭐⭐

---

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Generic | 泛型 | 类型的占位符，用 `T` 表示"调用时再决定是什么类型" |
| Type Parameter | 类型参数 | 泛型里的 `T`、`U`、`K`、`V` 等占位符名字 |
| Generic Constraint | 泛型约束 | `T extends X`：限制 T 必须满足某个条件，是入场限制 |
| Generic Interface | 泛型接口 | interface 里用 `<T>` 占位，使用时再填入真实类型 |
| Type Inference | 类型推断 | 调用泛型函数时，TS 自动推断 T 是什么，不需要手动写 |

---

## 📝 1. 实例代码

```typescript
// 代码读法说明：
// - 第一段：泛型函数，类型到调用时才决定
// - 第二段：泛型接口，API 响应的通用结构
// - 第三段：泛型约束，限制 T 必须有特定字段

// 泛型函数：返回数组第一个元素
function getFirst<T>(item: T[]): T | null {
  if (item.length !== 0) {
    return item[0]
  } else {
    return null
  }
}

console.log(getFirst([1, 2, 3]))         // 1，T 自动推断为 number
console.log(getFirst(["a", "b", "c"]))   // "a"，T 自动推断为 string
console.log(getFirst([]))                // null

// 泛型接口：API 响应通用结构
interface ApiResponse<T> {
  success: boolean
  data: T           // T 是"空白"，使用时填入真实类型
  error?: string
}

interface User { id: number; name: string }

const userResponse: ApiResponse<User> = {
  success: true,
  data: { id: 1, name: "Alice" }
}

const numberResponse: ApiResponse<number> = {
  success: true,
  data: 42
}

// 泛型约束：T 必须有 id 和 name 字段
function getNameAndId<T extends { id: number; name: string }>(item: T): void {
  console.log(`ID: ${item.id}, Name: ${item.name}`)
}

getNameAndId({ id: 1, name: "Alice", role: "admin" })  // ✅ 有 id 和 name
getNameAndId({ id: 2, name: "Bob", age: 25 })          // ✅ 有 id 和 name
// getNameAndId({ age: 25 })                            // ❌ 缺少 id 和 name
```

⚠️ 易错点：
1. `interface extends` = 继承（加东西）；`<T extends ...>` = 约束（入场限制），两个含义完全不同
2. `T[]` 表示 T 类型的数组，不是 `Array<T>` 的写法问题，两者等价
3. 泛型接口使用时 `<T>` 里填的是类型，不是值：`ApiResponse<User>` ✅，`ApiResponse<user1>` ❌

---

## 🎯 2. 使用场景

1. **SaaS 场景**：所有 API 请求封装成 `async function request<T>(url: string): Promise<ApiResponse<T>>`，调用方传入期望的返回类型，一套函数处理所有接口。
2. **Fintech 场景**：分页查询结果 `interface Paginated<T> { items: T[]; total: number; page: number }`，订单列表、交易记录、用户列表复用同一个分页结构。
3. **E-commerce 场景**：购物车操作 `function updateItem<T extends CartItem>(items: T[], id: number, update: Partial<T>): T[]`，泛型约束确保只能传 CartItem 的子类型。

---

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| `any` 类型 | 写起来简单 | 失去所有类型检查，等于没用 TS | 永远不推荐 |
| 函数重载 | 精确定义每种参数组合 | 代码冗余，维护成本高 | 参数组合固定且少时 |
| 联合类型 | 不需要学泛型语法 | 每种类型都要写一遍，不可扩展 | 类型数量固定且少时 |

**选型决策树**：
- 如果函数/接口要处理多种类型，且逻辑相同 → 泛型
- 如果类型固定只有 2-3 种 → 函数重载或联合类型
- 如果用 `any` → ❌ 改用泛型

---

## 💼 4. 面试题

### 题 1（🟡 进阶）

**Q**: 什么是泛型约束？`<T extends object>` 和 `<T extends { name: string }>` 有什么区别？

**核心 points**: extends 在泛型里是限制不是继承、object 约束只排除基本类型、具体字段约束更精确、约束让 TS 知道 T 一定有哪些属性

**追问方向**:
1. 泛型约束和 interface extends 继承有什么本质区别？
2. 如果我想约束 T 必须是一个构造函数，怎么写？

**加分回答**: "泛型约束在设计工具函数时特别重要，比如 `function pick<T, K extends keyof T>(obj: T, keys: K[]): Pick<T, K>`，这里 `K extends keyof T` 确保只能传 T 实际存在的字段名，传不存在的字段名编译就报错，这是 TS 工具类型的核心模式。"

---

## 🧪 5. 小测验

实现一个泛型函数 `filterBy`：
- 接收一个数组 `items: T[]`
- 接收一个 `key`，类型约束为 `keyof T`
- 接收一个 `value`，类型为 `T[K]`
- 返回数组中所有 `item[key] === value` 的元素

```typescript
// 调用示例
const users = [
  { id: 1, name: "Alice", role: "admin" },
  { id: 2, name: "Bob", role: "user" },
  { id: 3, name: "Charlie", role: "admin" },
]

filterBy(users, "role", "admin")
// 返回 Alice 和 Charlie
```

> 先思考，再展开答案 👇

<details>
<summary>展开答案</summary>

```typescript
function filterBy<T, K extends keyof T>(items: T[], key: K, value: T[K]): T[] {
  return items.filter(item => item[key] === value)
}

const users = [
  { id: 1, name: "Alice", role: "admin" },
  { id: 2, name: "Bob", role: "user" },
  { id: 3, name: "Charlie", role: "admin" },
]

console.log(filterBy(users, "role", "admin"))
// [{ id: 1, name: "Alice", role: "admin" }, { id: 3, name: "Charlie", role: "admin" }]
```

</details>

---

## 🌟 Senior 思维彩蛋

Senior 写泛型时会问："T 的约束是否足够精确？"约束太宽（`T extends object`）意味着函数内部只能用 object 共有的方法，很受限；约束太窄意味着复用性差。好的泛型约束像好的函数签名——精确表达"我需要什么，不多也不少"。`K extends keyof T` 这个模式是最常见的高质量约束，几乎所有 TS 工具类型都用到了它。

---

## 🔗 关联算法题

- LeetCode #146 LRU Cache（泛型缓存设计思维）
- LeetCode #208 Implement Trie（泛型数据结构）
- HackerRank: Data Structures - Linked Lists

---

## 📅 复习记录

- [ ] D+3 复习（2026-05-27）
- [ ] D+7 复习（2026-05-31）
- [ ] D+21 复习（2026-06-14）

## 🔗 相关笔记

- [[TS-Topic-04-函数类型]]
- [[TS-Topic-10-工具类型]]
- [[TS-Topic-11-条件类型与infer]]
