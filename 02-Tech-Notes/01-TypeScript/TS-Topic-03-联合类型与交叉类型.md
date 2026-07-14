---
tier: Tier-1
tech: TypeScript
topic: 联合类型与交叉类型
knowledge-point: union/intersection/literal/typeof/编译期检查
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

# 📌 TypeScript 联合类型 & 交叉类型

> Topic: 联合类型与交叉类型 · Tier: 1 · 难度: ⭐

---

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Union Type | 联合类型 | `A \| B`：值可能是 A 或 B，二选一 |
| Intersection Type | 交叉类型 | `A & B`：值同时拥有 A 和 B 的所有属性 |
| Literal Type | 字面量类型 | 不只限定类型，连值也限定死，如 `"left" \| "right"` |
| Compile-time Check | 编译期检查 | TS 在转成 JS 之前的检查阶段，不影响运行时 |

---

## 📝 1. 实例代码

```typescript
// 代码读法说明：
// - 第一段：字面量联合类型限定方向值
// - 第二段：联合类型 + typeof 判断
// - 第三段：交叉类型合并两个对象类型

// 字面量联合类型：限定值只能是四个方向之一
type Direction = "left" | "right" | "up" | "down"

function move(direction: Direction) {
  console.log(`Moving: ${direction}`)
}

move("left")       // ✅
// move("diagonal") // ❌ 编译报错，但运行时还是会跑

// 联合类型 + typeof 判断
function formatId(id: number | string): string {
  if (typeof id === "number") {
    return `ID-${id}`
  } else {
    return `ID-${id.toUpperCase()}`  // string 才有 toUpperCase
  }
}

// 交叉类型：同时拥有两个类型的所有属性
type HasName = { name: string }
type HasAge = { age: number }
type Person = HasName & HasAge

function describePerson(person: Person) {
  console.log(`${person.name} is ${person.age} years old`)
}

const person1: Person = { name: "Alice", age: 25 }
describePerson(person1)
```

⚠️ 易错点：
1. TS 报红线但代码还能跑：编译检查 ≠ 运行时阻止，类型错误不会中断 JS 执行
2. 联合类型只能访问 A 和 B **共同有**的属性，独有属性必须先用 typeof/in 判断
3. 交叉类型 `&` 和联合类型 `|` 方向相反，`&` 是"且"，`|` 是"或"

---

## 🎯 2. 使用场景

1. **SaaS 场景**：用户 ID 可能是数字（旧系统）也可能是 UUID 字符串（新系统），用 `type UserId = number | string` 做兼容层，配合 `typeof` 判断走不同处理逻辑。
2. **Fintech 场景**：支付方式用字面量联合类型 `type PaymentMethod = "credit_card" | "debit_card" | "paypal"`，确保传入非法值时编译阶段就报错，不会进到支付流程。
3. **E-commerce 场景**：促销商品类型用交叉类型 `type SaleProduct = Product & { discount: number }`，复用 Product 的所有字段再加上折扣信息。

---

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| `enum` | 有命名，语义更清晰 | 写法稍复杂，编译产物多 | 值多且需要复用时 |
| `interface extends` | 继承语义清晰 | 只能用于对象类型 | 对象类型的"且"关系 |
| 运行时 if/switch | 兼容纯 JS | 无编译期保护 | 不用 TS 的项目 |

**选型决策树**：
- 如果值是"A 或 B" → 联合类型 `A | B`
- 如果对象是"A 且 B" → 交叉类型 `A & B`
- 如果值是固定几个字符串 → 字面量联合类型
- 如果字符串值多且要复用 → `enum`

---

## 💼 4. 面试题

### 题 1（🟢 基础）

**Q**: 联合类型和交叉类型有什么区别？各举一个实际使用场景。

**核心 points**: `|` 是或/二选一、`&` 是且/全部拥有、联合用于参数多态、交叉用于类型组合

**追问方向**:
1. 联合类型的变量，为什么不能直接调用某一个类型独有的方法？
2. `type A = { x: number } & { x: string }` 的结果是什么？（答：`x: never`，因为不存在既是 number 又是 string 的值）

**加分回答**: "交叉类型在做 Mixin 模式时特别有用，比如 `type AdminUser = User & AdminPermissions`，既保留了用户基础信息，又加上了管理员权限，不用重新定义一个大而全的 interface。"

---

## 🧪 5. 小测验

写一个函数 `processValue`，接收 `string | number | boolean`：
- 是字符串 → 返回字符串长度
- 是数字 → 返回数字的两倍
- 是布尔值 → 返回取反的结果

> 先思考，再展开答案 👇

<details>
<summary>展开答案</summary>

```typescript
function processValue(value: string | number | boolean): string | number | boolean {
  if (typeof value === "string") {
    return value.length
  } else if (typeof value === "number") {
    return value * 2
  } else {
    return !value
  }
}

console.log(processValue("hello"))  // 5
console.log(processValue(10))       // 20
console.log(processValue(true))     // false
```

</details>

---

## 🌟 Senior 思维彩蛋

Senior 看到联合类型的函数，第一反应是问："所有分支都处理了吗？"——这叫穷举检查。进阶做法是在 `switch` 的 `default` 分支加 `const _exhaustive: never = value`，如果漏掉某个联合类型分支，TS 编译时就会报错。这是 Topic 6 类型守卫里的 `never` 穷举，是 Senior 防止需求变更漏改代码的利器。

---

## 🔗 关联算法题

- LeetCode #1 Two Sum（输入输出类型明确）
- LeetCode #5 Longest Palindromic Substring（字符串处理）
- HackerRank: 10 Days of JavaScript - Day 2

---

## 📅 复习记录

- [ ] D+3 复习（2026-05-27）
- [ ] D+7 复习（2026-05-31）
- [ ] D+21 复习（2026-06-14）

## 🔗 相关笔记

- [[TS-Topic-02-type-vs-interface]]
- [[TS-Topic-06-类型守卫]]
- [[TS-Topic-08-枚举与字面量类型]]
