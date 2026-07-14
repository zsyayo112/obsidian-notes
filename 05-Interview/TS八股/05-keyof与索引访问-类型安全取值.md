---
tier: Tier-1
tech: TypeScript
topic: keyof 与索引访问类型
knowledge-point: keyof + T[K] + getProperty 经典招式
difficulty: ⭐⭐⭐
status: learning
created: 2026-07-02
review-1: 2026-07-05
review-2: 2026-07-09
review-3: 2026-07-23
tags:
  - tech/typescript
  - tier-1
  - generics
  - keyof
---

# 📌 keyof + T[K]：类型安全的取值函数

> Topic: [[TS 类型系统够用版]] · Tier: 1 · 难度: ⭐⭐⭐

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| `keyof` | 取键名 | `keyof User` = 把 User 的所有**键名**收集成联合类型 `"id" \| "name"` |
| Union type | 联合类型 | `"id" \| "name"` 表示"这几个之一" |
| Indexed access type | 索引访问类型 | `T[K]` = 从类型 T 里挖出 K 这个键对应的**值的类型** |

## 📝 1. 实例代码

```typescript
// 代码读法说明：
// 先认识 keyof 和 T[K] 两个零件，再组合成经典的 getProperty 招式。

type User = { id: number; name: string; age: number };

// keyof：取出所有"键名"（不是键值！），组成联合类型
type UserKeys = keyof User; // "id" | "name" | "age"

// T[K]：索引访问，挖出某个键对应的"值的类型"
type A = User["id"];   // number
type B = User["name"]; // string
// 注意：类型层面取字段只能用 [] 不能用 .（T.K 是语法错误）

// ── 经典招式：类型安全的取值函数 ──
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
// <T>              = 对象类型
// K extends keyof T = K 必须是 T 的键名之一（约束）→ 拼错报错
// : T[K]           = 返回那个键对应的值类型 → 返回精确，不是 any

const user = { id: 1, name: "Alice" };
const n = getProperty(user, "id");   // n: number（精确！）
const s = getProperty(user, "name"); // s: string（精确！）
getProperty(user, "naem");           // ❌ 拼错当场标红

// 联动机制：K 跟着你传的 key 变，T[K] 跟着 K 变
// 传 "id"   → K = "id"   → T[K] = User["id"]   = number
// 传 "name" → K = "name" → T[K] = User["name"] = string
```

⚠️ 易错点：
1. `keyof` 取的是**键名**（`"id" | "name"`），不是键值类型（`number | string`）。
2. 类型层面取字段**必须用 `T[K]`，不能用 `T.K`**——因为 K 是**变量**（类型参数），点只能跟死名字，方括号才能跟变量。（JS 值层面同理：`user[key]` 能用变量，`user.key` 是找字面的 key 字段）
3. `getProperty(user, "id")` 能返回 number、`getProperty(user, "name")` 能返回 string，是因为返回类型 `T[K]` 是**跟输入 K 联动计算的表达式**，不是写死的。

## 🎯 2. 使用场景

1. **SaaS 场景**：通用表格组件 `<Column field={key} />`，用 `keyof T` 约束 `field` 只能是数据行的真实字段，配错列名编译期就报错。
2. **Fintech 场景**：`pick<T, K extends keyof T>(obj, keys)` 从交易对象里挑字段做脱敏，保证挑的键都存在、类型精确。
3. **E-commerce 场景**：`sortBy(products, "price")` 排序工具，用 `keyof Product` 约束排序字段合法，且 `T[K]` 保证比较的是正确类型。

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| `obj[key: string]` 普通取值 | 简单 | key 可拼错、返回 any | 快速脚本 |
| 写死每个 getter | 类型精确 | 字段一多就爆炸 | 字段极少 |
| lodash `get` | 现成、支持深路径 | 深路径类型支持弱 | 需要深层取值 |
| **`keyof` + `T[K]` 泛型** | 防拼错 + 返回精确类型 | 语法门槛 | **通用类型安全取值的默认选择** |

**选型决策树**：
- 如果 需要通用取值 + 类型安全 → `<T, K extends keyof T>(obj, key): T[K]`
- 如果 需要深层路径取值 → lodash get（或 ts-toolbelt 类型工具）
- 如果 一次性脚本、不在乎类型 → 直接 `obj[key]`

## 💼 4. 面试题

### 题 1（🔴 高级）
**Q**: 逐段解释 `getProperty<T, K extends keyof T>(obj: T, key: K): T[K]`，说明它为什么"同一函数能对不同 key 返回不同精确类型"。
**核心 points**: T=对象类型；K extends keyof T=key 约束为真实键名（防拼错）；T[K]=返回对应值类型；调用时 TS 推断 K，T[K] 联动算出返回类型。
**追问方向**:
1. 为什么用 `T[K]` 不用 `T.K`？（类型层面无点语法；K 是变量只能用 `[]`）
2. `keyof` 得到的是键名还是键值？（键名，联合类型）
**加分回答**: "这个签名的精髓是**输入 key 和输出类型被 `T[K]` 锁在一起联动**。它不是'一个函数处理多类型'那么简单，而是用类型参数**表达了数据在函数里怎么被索引和取出**。React 更新器、Prisma、lodash 的类型魔法全建在这个模式上。"

## 🧪 5. 小测验

> 补全 `pluck`，从对象数组里取某个字段组成新数组，保持类型精确。

```typescript
function pluck<T, K extends ___>(arr: T[], key: K): ___[] {
  return arr.map(item => item[key]);
}
const users = [{ id: 1, name: "A" }, { id: 2, name: "B" }];
const ids = pluck(users, "id");     // 期望 number[]
const names = pluck(users, "name"); // 期望 string[]
```

<details>
<summary>展开答案</summary>

```typescript
function pluck<T, K extends keyof T>(arr: T[], key: K): T[K][] {
  return arr.map(item => item[key]);
}
const ids = pluck(users, "id");     // number[]
const names = pluck(users, "name"); // string[]
```
- `K extends keyof T`：key 必须是 T 的真实键名。
- 返回 `T[K][]`：K 对应值类型组成的数组，跟着 key 联动。
</details>

## 🌟 Senior 思维彩蛋

`keyof` + `T[K]` 是 TS "类型编程"的入门砖。Mid 觉得类型只是"给变量贴标签"，Senior 意识到类型本身可以**计算**——从一个类型推导出另一个类型（键名→值类型）。这是 `Pick`、`Omit`、`Record` 这些工具类型的底层原理。掌握它，你就能读懂库里那些"魔法类型"，也能自己写出让调用方零心智负担的类型安全 API。

## 🔗 关联算法题

- 无直接算法关联（但"用 key 索引"的思路和哈希表 [[LeetCode-1-Two-Sum]] 一脉相承）

---

## 📅 复习记录
- [ ] D+3 复习（2026-07-05）
- [ ] D+7 复习（2026-07-09）
- [ ] D+21 复习（2026-07-23）

## 🔗 相关笔记
- [[04-extends约束-给T定最低要求]]
- [[06-类型对象class实例-四个概念区分]]
