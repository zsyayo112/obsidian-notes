---
tier: Tier-1
tech: TypeScript
topic: 严格模式与生产级实践
knowledge-point: strict/unknown/非空断言/as断言/instanceof守卫
difficulty: ⭐⭐⭐⭐
status: done
created: 2026-05-24
review-1: 2026-05-27
review-2: 2026-05-31
review-3: 2026-06-14
tags:
  - tech/typescript
  - tier-1
---

# 📌 TypeScript 严格模式 + 生产级实践

> Topic: 严格模式与生产级实践 · Tier: 1 · 难度: ⭐⭐⭐⭐

---

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Strict Mode | 严格模式 | `strict: true` 一键开启 6 个严格检查 |
| Non-null Assertion | 非空断言 | `value!`，告诉 TS "这个值不是 null/undefined"，危险 |
| Type Assertion | 类型断言 | `value as SomeType`，强行告诉 TS 值是某个类型，危险 |
| `strictNullChecks` | 严格空值检查 | 开启后 `null/undefined` 不能赋给其他类型，必须单独处理 |
| `noImplicitAny` | 禁止隐式 any | 开启后没有类型的变量不能隐式推断为 any |
| Defensive Programming | 防御性编程 | 假设最坏情况，先判断后使用，防止运行时崩溃 |

---

## 📝 1. 实例代码

```typescript
// 代码读法说明：
// - 每段都是"❌ 危险写法 → ✅ 正确写法"的对比
// - 三个核心问题：any / ! 非空断言 / as 类型断言

// ===== 问题 1：any 的危险 =====

// ❌ 危险：any 关掉所有检查，运行时可能崩溃
function processData_bad(data: any) {
  console.log(data.name.toUpperCase())  // 传入数字时：undefined.toUpperCase() 崩溃
}

// ✅ 正确：unknown + 类型判断，编译期就能发现问题
function processData_good(data: unknown) {
  if (typeof data === "object" && data !== null && "name" in data) {
    console.log((data as { name: string }).name.toUpperCase())
  }
}

// ===== 问题 2：! 非空断言的危险 =====

// ❌ 危险：! 强行说"不是 undefined"，找不到时崩溃
function getUser_bad(id: number) {
  const users = [{ id: 1, name: "Alice" }]
  const user = users.find(u => u.id === id)
  return user!.name   // id=999 时：undefined.name 崩溃
}

// ✅ 正确：先判断再用
function getUser_good(id: number): string {
  const users = [{ id: 1, name: "Alice" }]
  const user = users.find(u => u.id === id)
  if (!user) {
    throw new Error(`User ${id} not found`)
  }
  return user.name    // TS 知道这里 user 一定存在
}

// ===== 问题 3：as 类型断言的危险 =====

// ❌ 危险：as 强行断言，元素不存在时 null.value 崩溃
const input_bad = document.getElementById("username") as HTMLInputElement
console.log(input_bad.value)  // 页面没有这个 id 时崩溃

// ✅ 正确：instanceof 守卫，先确认类型再使用
const input_good = document.getElementById("username")
if (input_good instanceof HTMLInputElement) {
  console.log(input_good.value)   // TS 确认是 HTMLInputElement 才让用
}
```

⚠️ 易错点：
1. `!` 非空断言和 `?` 可选链不一样：`user?.name` 是安全的（找不到返回 undefined），`user!.name` 是危险的（找不到直接崩）
2. `as` 断言不做运行时检查，只是告诉 TS"相信我"——但 TS 不会，也不能帮你验证
3. `strict: true` 开启的是编译期检查，不能防止所有运行时错误，还需要结合运行时校验（如 Zod）

---

## 🎯 2. 使用场景

1. **SaaS 场景**：处理第三方 webhook 回调数据，类型完全未知，用 `unknown` 接收后配合 Zod schema 校验，既有编译期类型安全，又有运行时数据验证。
2. **Fintech 场景**：查询数据库返回的记录可能是 `undefined`（记录不存在），用 `if (!record) throw new Error(...)` 明确处理空值，防止后续操作在 `undefined` 上执行金融计算。
3. **E-commerce 场景**：DOM 操作（表单提交、按钮点击）用 `instanceof` 守卫确认元素类型，避免用户修改 DOM 或浏览器兼容问题导致类型不匹配的运行时崩溃。

---

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| Zod / io-ts 运行时校验 | 同时处理编译期和运行时 | 需要额外库，bundle 变大 | 处理外部数据时组合使用 |
| `Optional Chaining ?.` | 写法简洁，不崩溃 | 静默返回 undefined，可能掩盖 bug | 允许值不存在的场景 |
| `Nullish Coalescing ??` | 提供默认值 | 只处理 null/undefined，不抛错 | 有合理默认值的场景 |

**选型决策树**：
- 参数类型不确定 → `unknown` + 类型判断，不用 `any`
- 变量可能是 null/undefined，必须存在 → `if (!val) throw Error`
- 变量可能是 null/undefined，可以不存在 → `?.` 可选链
- 判断 DOM 元素类型 → `instanceof`
- 对外部数据做校验 → TS 类型 + Zod 运行时校验双保险

---

## 💼 4. 面试题

### 题 1（🔴 高级）

**Q**: 为什么 Senior 工程师不用 `any`？如果必须处理类型不确定的数据，你怎么做？

**核心 points**: any 关掉编译检查、运行时崩溃难排查、unknown 强制判断、类型守卫收窄、Zod 运行时校验

**追问方向**:
1. `unknown` 和 `any` 在类型系统里有什么本质区别？
2. 如果团队里有人写了 `as any`，你在 code review 里怎么评论？

**加分回答**: "在处理第三方 API 响应时，我会用 `unknown` 接收，然后用 Zod 定义 schema 做运行时校验，`parse()` 成功后 TS 自动推断出正确的类型。这样既有编译期类型安全（`unknown` 强制检查），又有运行时保护（Zod 校验），API 格式变了会立刻报错而不是静默失败。"

### 题 2（🔴 高级）

**Q**: `!` 非空断言和 `?.` 可选链有什么区别？什么时候该用哪个？

**核心 points**: ! 强行断言不为空（危险、跳过检查）、?. 安全访问返回 undefined（安全）、! 只在确定不为空时用、生产代码尽量用 ?. 或 if 判断

**追问方向**:
1. 有哪些场景下 `!` 是合理的？
2. `??` 和 `||` 有什么区别？

**加分回答**: "`!` 在测试代码里是合理的，因为测试数据是你控制的，你确定不为 null；在生产代码里几乎不应该出现，因为你永远不能确定运行时数据——总有用户会用你没预料到的方式触发边界情况。"

---

## 🧪 5. 小测验

下面这段代码有 3 个生产级问题，找出来并修复：

```typescript
function getUserName(id: any) {
  const users = [
    { id: 1, name: "Alice" },
    { id: 2, name: "Bob" }
  ]
  const user = users.find(u => u.id === id)
  return user!.name.toUpperCase()
}

const form = document.getElementById("loginForm") as HTMLFormElement
form.submit()
```

> 先思考，再展开答案 👇

<details>
<summary>展开答案</summary>

**问题 1**：参数 `id` 类型是 `any`，失去类型检查。
```typescript
function getUserName(id: number): string {  // ✅ 明确类型
```

**问题 2**：`user!.name` 用了非空断言，找不到 id 时崩溃。
```typescript
if (!user) {
  throw new Error(`User ${id} not found`)
}
return user.name.toUpperCase()
```

**问题 3**：`as HTMLFormElement` 强制断言，元素不存在时 `null.submit()` 崩溃。
```typescript
const form = document.getElementById("loginForm")
if (form instanceof HTMLFormElement) {
  form.submit()
} else {
  console.error("Login form not found")
}
```

</details>

---

## 🌟 Senior 思维彩蛋

Senior 在做 code review 时有一个"三不用"原则：不用 `any`、不用 `!`、不用无理由的 `as`。这三个东西都是在告诉 TS "相信我"——但 Senior 知道，代码里最不能相信的就是"相信我"。每一个 `!` 和 `as` 都是潜在的运行时炸弹，只是还没到触发的时机。真正的 Senior 代码是让 TS 自己推断，不需要你"相信我"。

---

## 🔗 关联算法题

- LeetCode #3 Longest Substring Without Repeating（边界检查思维：类似 null check）
- LeetCode #21 Merge Two Sorted Lists（空指针处理）
- HackerRank: Problem Solving - Defensive Coding

---

## 📅 复习记录

- [ ] D+3 复习（2026-05-27）
- [ ] D+7 复习（2026-05-31）
- [ ] D+21 复习（2026-06-14）

## 🔗 相关笔记

- [[TS-Topic-01-基本类型标注]]
- [[TS-Topic-06-类型守卫]]
- [[TS-Topic-09-模块系统与tsconfig]]
