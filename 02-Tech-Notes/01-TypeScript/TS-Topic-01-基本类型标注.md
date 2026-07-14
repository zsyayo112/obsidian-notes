---
tier: Tier-1
tech: TypeScript
topic: 基本类型标注
knowledge-point: string/number/boolean/void/never/unknown/any
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

# 📌 TypeScript 基本类型标注

> Topic: 基本类型标注 · Tier: 1 · 难度: ⭐

---

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Type Annotation | 类型标注 | 手动告诉 TS 一个变量/参数是什么类型 |
| Type Inference | 类型推断 | 你不写类型，TS 自己猜出来 |
| `any` | 任意类型 | 关掉所有类型检查，Senior 不用 |
| `unknown` | 未知类型 | 安全版的 any，用前必须判断类型 |
| `void` | 空返回 | 函数正常执行完，没有返回值 |
| `never` | 永不返回 | 函数永远不会正常结束（抛错或死循环） |

---

## 📝 1. 实例代码

```typescript
// 代码读法说明：
// - 前两段展示变量标注 vs 类型推断
// - 中间展示 unknown 的安全用法
// - 最后展示 void 和 never 的区别

// 变量类型标注
let name: string = "Alice"
let age: number = 25
let isActive: boolean = true

// 类型推断（不写类型，TS 自己猜）
let score = 100        // TS 推断为 number
// score = "hello"     // ❌ 报错：string 不能赋给 number

// 函数类型标注
type GreetFn = (name: string, age: number) => string

const greetUser: GreetFn = (name, age) => {
  return `Hi, I'm ${name} and I'm ${age} years old.`
}
console.log(greetUser("Alice", 25))

// unknown vs any
function processInput(value: unknown) {
  if (typeof value === "string") {
    console.log(value.toUpperCase())  // ✅ 判断后才能用
  } else {
    console.log("not a string")
  }
}

// void vs never
function logError(message: string): void {
  console.log(message)               // 正常结束，没有返回值
}

function throwError(message: string): never {
  throw new Error(message)           // 永远不会正常结束
}
```

⚠️ 易错点：
1. 模板字符串里不要多加引号：`` `"Hi ${name}"` `` ❌ → `` `Hi ${name}` `` ✅
2. `any` 会关掉所有类型检查，运行时可能崩溃，Senior 不用
3. `type` 定义函数类型时参数不需要重复标注：`const fn: MyFn = (a, b) => ...` ✅

---

## 🎯 2. 使用场景

1. **SaaS 场景**：用户输入表单验证，用 `unknown` 接收用户输入，判断类型后再处理，防止 XSS 注入导致运行时崩溃。
2. **Fintech 场景**：金融计算函数严格标注参数为 `number`，避免字符串 `"100"` 和数字 `100` 混用导致计算错误。
3. **E-commerce 场景**：API 响应处理，用 `unknown` 接收服务端数据，类型校验通过后再操作，防止字段缺失导致页面白屏。

---

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| 纯 JavaScript | 无需类型标注，上手快 | 运行时才发现类型错误 | 小型脚本、原型验证 |
| JSDoc 注释类型 | 不用迁移到 TS，渐进式 | 类型检查能力弱，IDE 支持差 | 老项目改造过渡期 |
| Zod / io-ts | 运行时类型校验，前后端都能用 | 需要额外学习，bundle 变大 | 需要运行时校验 API 数据时 |

**选型决策树**：
- 如果是新项目全栈开发 → 选 TypeScript
- 如果是老 JS 项目渐进改造 → 先用 JSDoc，再迁移 TS
- 如果需要校验运行时数据（如 API 响应）→ TS + Zod 组合

---

## 💼 4. 面试题

### 题 1（🟢 基础）

**Q**: `any` 和 `unknown` 有什么区别？什么时候用 `unknown`？

**核心 points**: any 关掉检查、unknown 保留检查、用前必须 typeof/in/instanceof 判断、Senior 不用 any

**追问方向**:
1. 如果收到一个第三方 API 的响应，类型不确定，你怎么处理？
2. 什么情况下 `unknown` 比 `any` 更安全？

**加分回答**: "在处理外部数据时，我会用 `unknown` 配合 Zod 做运行时校验，这样不仅编译期安全，运行时也有保障。纯用 `any` 的话，类型错误会推迟到运行时，线上出问题很难排查。"

---

## 🧪 5. 小测验

下面这段代码有 2 个问题，找出并修复：

```typescript
function processData(data: any) {
  console.log(data.name.toUpperCase())
}

let score = 100
score = "one hundred"
```

> 先思考，再展开答案 👇

<details>
<summary>展开答案</summary>

**问题 1**：`data` 类型是 `any`，如果传入数字，`data.name` 是 `undefined`，`toUpperCase()` 会崩溃。
修复：改成 `unknown`，加类型判断：
```typescript
function processData(data: unknown) {
  if (typeof data === "object" && data !== null && "name" in data) {
    console.log((data as { name: string }).name.toUpperCase())
  }
}
```

**问题 2**：`score` 被推断为 `number`，不能赋值 `string`。
修复：初始化时就声明正确类型，或明确用联合类型：
```typescript
let score: number = 100       // 保持 number
// 或
let score: number | string = 100
score = "one hundred"         // ✅
```

</details>

---

## 🌟 Senior 思维彩蛋

Senior 在 code review 时，看到 `any` 会直接要求改掉——不是因为规则，而是因为他们见过太多 `any` 引起的生产事故：API 返回格式变了，字段从 `string` 变成 `number`，用了 `any` 的地方编译不报错，上线后页面直接白屏。用 `unknown` + 类型守卫，这类错误在 CI 阶段就能拦截。

---

## 🔗 关联算法题

- LeetCode #1 Two Sum（类型推断思维：确定输入输出类型再写逻辑）
- HackerRank: 10 Days of JavaScript - Day 0

---

## 📅 复习记录

- [ ] D+3 复习（2026-05-27）
- [ ] D+7 复习（2026-05-31）
- [ ] D+21 复习（2026-06-14）

## 🔗 相关笔记

- [[TS-Topic-02-type-vs-interface]]
- [[TS-Topic-04-函数类型]]
- [[TS-Topic-13-严格模式与生产级实践]]
