---
tier: Tier-1
tech: TypeScript
topic: 函数类型
knowledge-point: 可选参数/默认参数/回调函数/void/never
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

# 📌 TypeScript 函数类型

> Topic: 函数类型 · Tier: 1 · 难度: ⭐

---

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Optional Parameter | 可选参数 | 用 `?` 标注，调用时可以不传 |
| Default Parameter | 默认参数 | 不传时使用默认值，只能写在实现里 |
| Callback | 回调函数 | 把函数作为参数传给另一个函数 |
| Higher-order Function | 高阶函数 | 接收函数作为参数，或返回函数的函数 |
| `void` | 空返回 | 函数正常执行完，不返回任何值 |
| `never` | 永不返回 | 函数抛出错误或死循环，永远不会正常结束 |

---

## 📝 1. 实例代码

```typescript
// 代码读法说明：
// - 第一段：type 定义函数形状（不含默认值）
// - 第二段：实现时才写默认值
// - 第三段：回调函数类型的定义和使用

// type 只管形状，不能写默认值
type CreateGreeting = (name: string, greeting?: string) => string

// 实现时写默认值
const createGreeting: CreateGreeting = (name, greeting = "Hello") => {
  return `${greeting}, ${name}!`
}

console.log(createGreeting("Alice"))          // Hello, Alice!
console.log(createGreeting("Bob", "Hi"))      // Hi, Bob!
console.log(createGreeting("Charlie", "Hey")) // Hey, Charlie!

// 回调函数类型
type Transformer = (value: string) => string

function processName(name: string, transform: Transformer): string {
  return transform(name)
}

console.log(processName("alice", (name) => name.toUpperCase()))  // ALICE
console.log(processName("BOB", (name) => name.toLowerCase()))    // bob

// void vs never
function logMessage(message: string): void {
  console.log(message)    // 正常结束，不返回
}

function throwError(message: string): never {
  throw new Error(message)  // 永远不会正常结束
}
```

⚠️ 易错点：
1. `type` 里不能写默认值：`(name: string, greeting?: string = "Hello")` ❌
2. 已在 `type` 定义类型，实现时参数不需要重复标注：`(name, greeting = "Hello")` ✅
3. 箭头函数没有参数时必须写括号：`() => void` ✅，不能省略

---

## 🎯 2. 使用场景

1. **SaaS 场景**：事件处理回调，`onClick: (event: MouseEvent) => void`，明确告诉使用者这个回调不需要返回值，避免误用返回值。
2. **Fintech 场景**：数据格式化函数，`type Formatter = (value: number) => string`，不同货币用不同的 Formatter 传入，统一接口多态实现。
3. **E-commerce 场景**：价格计算策略，`type PriceStrategy = (price: number, quantity: number) => number`，促销、折扣、会员价用不同策略函数传入。

---

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| `interface` 定义函数类型 | 可以被 extends | 语法稍繁琐 | 函数类型需要被继承时 |
| 直接内联类型 | 简洁，不用起名 | 无法复用 | 只用一次的简单回调 |
| `Function` 类型 | 接受任何函数 | 等于 any，失去类型检查 | 永远不推荐 |

**选型决策树**：
- 如果函数类型要复用多次 → `type Fn = (...) => ...`
- 如果只用一次 → 直接内联 `(value: string) => string`
- 如果需要继承 → `interface Fn { (value: string): string }`
- 如果用 `Function` → ❌ 不要用，和 `any` 一样危险

---

## 💼 4. 面试题

### 题 1（🟢 基础）

**Q**: `void` 和 `never` 有什么区别？各在什么场景下使用？

**核心 points**: void 正常结束无返回值、never 永远不结束、throw/死循环用 never、事件处理用 void

**追问方向**:
1. 一个返回 `void` 的函数，能不能有 `return` 语句？（答：可以，`return` 不带值是合法的）
2. 什么时候 TS 会自动推断返回类型为 `never`？

**加分回答**: "`never` 在穷举检查里特别有用，在 `switch` 的 `default` 分支写 `const x: never = value`，如果联合类型新增了一个值但忘记处理，编译时就会报错，这是防御性编程的重要手段。"

---

## 🧪 5. 小测验

补全这个高阶函数：

```typescript
type Validator = (value: string) => boolean

function validateInput(
  input: string,
  // 补全这里的参数类型
  validators
): boolean {
  // 补全：所有 validator 都通过才返回 true
}

// 调用示例
validateInput("hello@email.com", [
  (v) => v.length > 0,
  (v) => v.includes("@"),
])
```

> 先思考，再展开答案 👇

<details>
<summary>展开答案</summary>

```typescript
type Validator = (value: string) => boolean

function validateInput(input: string, validators: Validator[]): boolean {
  return validators.every((validator) => validator(input))
}

console.log(validateInput("hello@email.com", [
  (v) => v.length > 0,
  (v) => v.includes("@"),
]))  // true

console.log(validateInput("hello", [
  (v) => v.length > 0,
  (v) => v.includes("@"),
]))  // false
```

</details>

---

## 🌟 Senior 思维彩蛋

Senior 看到回调函数类型，会问："这个回调会不会被异步调用？"如果是异步的，返回类型应该是 `() => Promise<void>` 而不是 `() => void`。混用会导致调用方没有 `await`，异步错误被吞掉，生产环境里极难排查。

---

## 🔗 关联算法题

- LeetCode #912 Sort an Array（理解高阶函数：comparator 就是回调函数）
- LeetCode #217 Contains Duplicate
- HackerRank: Functional Programming - Functions

---

## 📅 复习记录

- [ ] D+3 复习（2026-05-27）
- [ ] D+7 复习（2026-05-31）
- [ ] D+21 复习（2026-06-14）

## 🔗 相关笔记

- [[TS-Topic-01-基本类型标注]]
- [[TS-Topic-03-联合类型与交叉类型]]
- [[TS-Topic-07-泛型基础]]
