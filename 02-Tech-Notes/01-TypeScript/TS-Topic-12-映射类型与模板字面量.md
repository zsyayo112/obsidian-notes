---
tier: Tier-1
tech: TypeScript
topic: 映射类型与模板字面量
knowledge-point: keyof/映射类型/模板字面量/Capitalize/as重映射
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

# 📌 TypeScript 映射类型 + 模板字面量类型

> Topic: 映射类型与模板字面量 · Tier: 1 · 难度: ⭐⭐⭐

---

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Mapped Type | 映射类型 | 遍历一个类型的所有字段，批量生成新类型 |
| `keyof` | 键提取运算符 | 拿到一个类型的所有字段名，结果是联合类型 |
| `T[K]` | 索引访问类型 | 拿到类型 T 中字段 K 的类型，类似对象取值 |
| `in` | 遍历运算符 | 在映射类型里遍历联合类型的每个成员 |
| Template Literal Type | 模板字面量类型 | 用模板字符串语法生成新的字符串类型 |
| `Capitalize` | 首字母大写 | TS 内置工具，把字符串类型的首字母变大写 |
| `as` 重映射 | key 重命名 | 在映射类型里用 `as` 改变输出的 key 名字 |

---

## 📝 1. 实例代码

```typescript
// 代码读法说明：
// - 第一段：keyof 拿到字段名联合类型
// - 第二段：映射类型批量处理字段
// - 第三段：模板字面量类型生成新字符串类型
// - 第四段：映射 + 模板字面量组合，自动生成 getter 类型

// keyof：拿到所有字段名
interface User {
  id: number
  name: string
  email: string
}

type UserKeys = keyof User  // "id" | "name" | "email"
type IdType = User["id"]    // number（索引访问类型）

// 映射类型：遍历所有字段，批量加 | null
type Nullable<T> = {
  [K in keyof T]: T[K] | null
}

type NullableUser = Nullable<User>
// { id: number | null; name: string | null; email: string | null }

// 映射类型：自定义 Partial（所有字段变可选）
type MyPartial<T> = {
  [K in keyof T]?: T[K]    // ? 让字段变可选
}

// 模板字面量类型
type Fields = "name" | "age" | "email"
type Getters<T extends string> = `get${Capitalize<T>}`
type GetterNames = Getters<Fields>
// "getName" | "getAge" | "getEmail"

// 映射类型 + 模板字面量 + as 重映射：自动生成 getter 对象类型
type Getterify<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K]
  //               ↑
  //    as 后面是新的 key 名字
}

type UserGetters = Getterify<User>
// {
//   getId: () => number
//   getName: () => string
//   getEmail: () => string
// }
```

⚠️ 易错点：
1. `keyof T` 拿的是**字段名**（`"id" | "name"`），`T[K]` 拿的是**字段类型**（`number | string`）
2. `string & K`：因为 `keyof T` 可能包含 `number | symbol`，`& string` 过滤掉非字符串 key
3. 映射类型里的 `in` 和类型守卫里的 `in` 含义不同：这里是"遍历"，守卫里是"字段存在判断"

---

## 🎯 2. 使用场景

1. **SaaS 场景**：表单验证错误类型 `type FormErrors<T> = { [K in keyof T]?: string }`，每个字段对应一个可选的错误信息字符串，和表单数据类型自动保持同步。
2. **Fintech 场景**：生成 API 事件名 `type EventName<T extends string> = `on${Capitalize<T>}``，配合 EventEmitter 类型，确保事件名和处理函数的类型一一对应。
3. **E-commerce 场景**：商品过滤条件 `type Filterable<T> = { [K in keyof T]?: T[K] | T[K][] }`，每个字段可以是单值或数组（多选过滤），从商品类型自动派生，不需要手写。

---

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| 内置工具类型（Partial/Required）| 直接用，不需要手写 | 只有固定的变换，无法自定义 | 标准变换场景 |
| 手动定义每个字段 | 最直观 | 和原类型不同步，改一处要改两处 | 永远不推荐用于派生类型 |
| 代码生成工具 | 运行时生成，动态 | 需要额外工具链，类型推断弱 | 需要运行时动态生成时 |

**选型决策树**：
- 需要批量修改字段属性（加可选、加 null）→ 映射类型
- 需要根据字段名生成新字符串类型 → 模板字面量类型
- 需要改变输出的 key 名字 → 映射类型 + `as` 重映射
- 内置工具类型够用 → 直接用，不要手写

---

## 💼 4. 面试题

### 题 1（🔴 高级）

**Q**: 手动实现 `Partial<T>`，并解释 `keyof`、`in`、`T[K]` 各自的作用。

**核心 points**: keyof 拿字段名联合类型、in 遍历、T[K] 拿字段类型、? 加可选、整体是映射类型

**追问方向**:
1. 如何实现深度 Partial（嵌套对象也变可选）？（提示：递归 + 条件类型）
2. 映射类型里的 `+?` 和 `-?` 分别是什么意思？

**加分回答**: "映射类型最大的价值是'单一事实来源'——比如 `FormErrors<T>` 直接从表单数据类型 `T` 派生，`T` 加了新字段，`FormErrors` 自动有对应的错误字段，不需要手动维护同步。这在大型表单或复杂数据模型里，能避免大量漏改的 bug。"

---

## 🧪 5. 小测验

实现一个 `Setterify<T>` 类型，和 `Getterify` 相反：
- key 变成 `set${Capitalize<字段名>}`
- value 变成接收该字段类型作为参数、返回 `void` 的函数

```typescript
interface User { id: number; name: string; email: string }

type UserSetters = Setterify<User>
// 期望结果：
// {
//   setId: (value: number) => void
//   setName: (value: string) => void
//   setEmail: (value: string) => void
// }
```

> 先思考，再展开答案 👇

<details>
<summary>展开答案</summary>

```typescript
type Setterify<T> = {
  [K in keyof T as `set${Capitalize<string & K>}`]: (value: T[K]) => void
}

interface User { id: number; name: string; email: string }
type UserSetters = Setterify<User>
// {
//   setId: (value: number) => void
//   setName: (value: string) => void
//   setEmail: (value: string) => void
// }
```

</details>

---

## 🌟 Senior 思维彩蛋

Senior 看到映射类型，会问："这个类型在运行时需要对应的实现吗？"比如 `Getterify<User>` 定义了类型，但真正的 getter 函数还需要手写或用 Proxy 动态生成。类型和实现要匹配——只有类型，没有实现，等于空头支票。高质量的工具函数会同时提供类型定义和运行时实现，比如 `function getterify<T>(obj: T): Getterify<T>`。

---

## 🔗 关联算法题

- LeetCode #49 Group Anagrams（keyof 思维：用字段/字符作为 key 做分组）
- LeetCode #347 Top K Frequent Elements
- HackerRank: SQL - Advanced Select

---

## 📅 复习记录

- [ ] D+3 复习（2026-05-27）
- [ ] D+7 复习（2026-05-31）
- [ ] D+21 复习（2026-06-14）

## 🔗 相关笔记

- [[TS-Topic-10-工具类型]]
- [[TS-Topic-11-条件类型与infer]]
- [[TS-Topic-07-泛型基础]]
