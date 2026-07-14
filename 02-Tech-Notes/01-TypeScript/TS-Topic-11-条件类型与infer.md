---
tier: Tier-1
tech: TypeScript
topic: 条件类型与infer
knowledge-point: 条件类型/infer/ReturnType/参数类型提取
difficulty: ⭐⭐⭐
status: done
created: 2026-05-24
review-1: 2026-05-27
review-2: 2026-05-31
review-3: 2026-06-14
tags:
  - tech/typescript
  - tier-1
---

# 📌 TypeScript 条件类型 + `infer`

> Topic: 条件类型与infer · Tier: 1 · 难度: ⭐⭐⭐

---

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Conditional Type | 条件类型 | 类型版的三元运算符：`T extends X ? A : B` |
| `infer` | 推断关键字 | 在条件类型里捕获未知类型，类似正则捕获组 |
| Type-level | 类型层面 | 只在编译时存在，运行时消失，不是值 |
| `ReturnType<T>` | 返回值类型 | TS 内置工具类型，提取函数的返回值类型 |
| `Parameters<T>` | 参数类型 | TS 内置工具类型，提取函数的参数类型元组 |
| Distributive | 分发 | 联合类型在条件类型里会自动分发，逐个处理每个成员 |

---

## 📝 1. 实例代码

```typescript
// 代码读法说明：
// - 第一段：条件类型基础，类型版三元运算符
// - 第二段：infer 捕获数组元素类型
// - 第三段：infer 捕获函数返回值类型（手动实现 ReturnType）
// - 第四段：infer 捕获函数参数类型

// 条件类型基础
type IsString<T> = T extends string ? "yes" : "no"
type A = IsString<string>   // "yes"
type B = IsString<number>   // "no"

// IsArray：判断是不是数组
type IsArray<T> = T extends any[] ? "array" : "not array"
type C = IsArray<string[]>  // "array"
type D = IsArray<number>    // "not array"

// infer 捕获数组元素类型
// 类比正则捕获组：(infer U) 就是 (\d+)，捕获那个位置的内容
type GetElement<T> = T extends (infer U)[] ? U : never
type E = GetElement<string[]>   // string
type F = GetElement<number[]>   // number
type G = GetElement<boolean>    // never（不是数组）

// infer 捕获函数返回值类型（手动实现 ReturnType）
type GetReturnType<T> = T extends (...args: any[]) => infer R ? R : never
type H = GetReturnType<() => string>    // string
type I = GetReturnType<() => number[]>  // number[]
type J = GetReturnType<string>          // never（不是函数）

// infer 捕获函数参数类型
type GetParamType<T> = T extends (infer R) => any ? R : never
type K = GetParamType<(name: string) => void>  // string
type L = GetParamType<(age: number) => void>   // number
```

⚠️ 易错点：
1. `type` 定义的东西是类型层面的，不能 `console.log`，用 VS Code 悬停验证
2. `infer U` 放在你想**捕获**的位置，TS 帮你推断那个位置的类型
3. `(...args: any[])` 表示"任意参数，我不在乎"，目的是捕获返回值而不限制参数

---

## 🎯 2. 使用场景

1. **SaaS 场景**：提取异步函数的返回值类型，`type UserData = Awaited<ReturnType<typeof fetchUser>>`，不需要手动定义返回类型，函数改了类型自动跟着变。
2. **Fintech 场景**：用条件类型实现类型安全的 API 路由映射，根据路由字符串的格式自动推断返回的数据类型。
3. **E-commerce 场景**：提取组件 props 类型 `type ButtonProps = Parameters<typeof Button>[0]`，复用组件类型而不重复定义。

---

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| TS 内置 `ReturnType<T>` | 不用手写，直接用 | 只处理返回值，无法自定义捕获位置 | 直接提取函数返回值时 |
| TS 内置 `Parameters<T>` | 不用手写 | 返回元组，需要用索引取具体参数 | 提取函数参数类型时 |
| 手动声明类型 | 最简单，不需要学高级语法 | 和原类型不同步，维护成本高 | 临时方案 |

**选型决策树**：
- 提取函数返回值类型 → `ReturnType<typeof fn>`（内置，直接用）
- 提取函数参数类型 → `Parameters<typeof fn>`（内置，直接用）
- 需要自定义捕获逻辑 → 手写 `infer`
- 根据类型做不同处理 → 条件类型

---

## 💼 4. 面试题

### 题 1（🔴 高级）

**Q**: 什么是 `infer`？举例说明它解决了什么问题。

**核心 points**: infer 在条件类型里捕获未知类型、类比正则捕获组、解决"我知道形状但不知道具体类型"的问题、ReturnType 是典型应用

**追问方向**:
1. `infer` 只能在 `extends` 子句里用，为什么？
2. 如果函数有多个参数，怎么用 `infer` 捕获第一个参数的类型？

**加分回答**: "我在项目里用 `infer` 最多的场景是提取异步函数的 resolved 值类型：`type Unwrap<T> = T extends Promise<infer U> ? U : T`，这样 `Unwrap<Promise<string>>` 得到 `string`，避免到处写 `Awaited<ReturnType<...>>` 的嵌套。"

---

## 🧪 5. 小测验

实现以下三个工具类型：

```typescript
// 1. UnpackArray<T>：如果 T 是数组，返回元素类型；否则返回 T 本身
type UnpackArray<T> = ???
type R1 = UnpackArray<string[]>   // string
type R2 = UnpackArray<number>     // number（不是数组，返回本身）

// 2. UnpackPromise<T>：如果 T 是 Promise，返回 resolved 类型；否则返回 T
type UnpackPromise<T> = ???
type R3 = UnpackPromise<Promise<string>>  // string
type R4 = UnpackPromise<number>           // number

// 3. FunctionReturnType<T>：提取函数返回值，不是函数返回 never
type FunctionReturnType<T> = ???
type R5 = FunctionReturnType<() => boolean>  // boolean
type R6 = FunctionReturnType<string>          // never
```

> 先思考，再展开答案 👇

<details>
<summary>展开答案</summary>

```typescript
type UnpackArray<T> = T extends (infer U)[] ? U : T

type UnpackPromise<T> = T extends Promise<infer U> ? U : T

type FunctionReturnType<T> = T extends (...args: any[]) => infer R ? R : never
```

</details>

---

## 🌟 Senior 思维彩蛋

Senior 看到 `infer` 的第一反应是："这里真的需要 `infer` 吗？还是直接用 `ReturnType` / `Parameters` 内置工具类型就够了？"`infer` 是强力工具，但也让代码变复杂。能用内置工具类型解决的，不用手写 `infer`。只有内置工具类型覆盖不了的场景（比如提取嵌套类型、条件分支里的类型捕获），才值得上 `infer`。

---

## 🔗 关联算法题

- LeetCode #70 Climbing Stairs（条件递推思维，类似条件类型的递归）
- LeetCode #509 Fibonacci Number
- HackerRank: Functional Programming - Higher Order Functions

---

## 📅 复习记录

- [ ] D+3 复习（2026-05-27）
- [ ] D+7 复习（2026-05-31）
- [ ] D+21 复习（2026-06-14）

## 🔗 相关笔记

- [[TS-Topic-07-泛型基础]]
- [[TS-Topic-10-工具类型]]
- [[TS-Topic-12-映射类型与模板字面量]]
