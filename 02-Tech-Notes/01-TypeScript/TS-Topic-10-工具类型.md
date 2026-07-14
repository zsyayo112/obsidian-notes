---
tier: Tier-1
tech: TypeScript
topic: 工具类型
knowledge-point: Partial/Required/Readonly/Pick/Omit/Record
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

# 📌 TypeScript 工具类型（Utility Types）

> Topic: 工具类型 · Tier: 1 · 难度: ⭐⭐⭐

---

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Utility Type | 工具类型 | TS 内置的泛型类型，用来对现有类型做变换 |
| `Partial<T>` | 部分类型 | 把 T 的所有字段变成可选 |
| `Required<T>` | 必填类型 | 把 T 的所有字段变成必填 |
| `Readonly<T>` | 只读类型 | 把 T 的所有字段变成只读 |
| `Pick<T, K>` | 挑选类型 | 从 T 里挑出指定字段组成新类型 |
| `Omit<T, K>` | 删除类型 | 从 T 里删掉指定字段组成新类型 |
| `Record<K, V>` | 记录类型 | 创建一个 key 是 K 类型、value 是 V 类型的对象类型 |
| DTO | 数据传输对象 | Data Transfer Object，用于 API 请求/响应的数据结构 |

---

## 📝 1. 实例代码

```typescript
// 代码读法说明：
// - 第一段：基础接口定义
// - 第二段：用工具类型派生 DTO（最常见的实际用法）
// - 第三段：Record 做权限映射表

interface User {
  readonly id: number
  name: string
  email: string
  role: "admin" | "editor" | "viewer"
  status: "active" | "inactive"
  createdAt?: string
}

// Partial：所有字段变可选（更新接口常用）
type UpdateUserDto = Partial<User>
// { id?: number; name?: string; email?: string; ... }

// Required：所有字段变必填
type FullUser = Required<User>
// { id: number; name: string; email: string; ...; createdAt: string }

// Pick：只保留指定字段（列表展示用）
type UserPreview = Pick<User, "id" | "name" | "role">
// { id: number; name: string; role: "admin" | "editor" | "viewer" }

// Omit：删掉指定字段（创建接口，不需要 id 和 createdAt）
type CreateUserDto = Omit<User, "id" | "createdAt">
// { name: string; email: string; role: ...; status: ... }

// 组合使用：创建 DTO + 更新 DTO
type UpdateUserPayload = Partial<Omit<User, "id" | "createdAt">>

// Record：角色权限映射表
type Role = "admin" | "customer" | "guest"
interface Permission {
  canRead: boolean
  canWrite: boolean
  canDelete: boolean
}

type RolePermissions = Record<Role, Permission>

const rolePermissions: RolePermissions = {
  admin:    { canRead: true,  canWrite: true,  canDelete: true  },
  customer: { canRead: true,  canWrite: false, canDelete: false },
  guest:    { canRead: false, canWrite: false, canDelete: false },
}

// 函数里用展开运算符 ... 组合字段
function createUser(value: CreateUserDto): User {
  return {
    ...value,              // 把 CreateUserDto 的所有字段展开
    id: 1,
    createdAt: "2026-01-01",
  }
}
```

⚠️ 易错点：
1. `Omit` 里多个字段用 `|` 不用 `&`：`Omit<User, "id" | "createdAt">` ✅
2. `UpdateUserDto` 通常基于 `CreateUserDto`（`Partial<CreateUserDto>`），不是直接 `Partial<User>`
3. `Record<K, V>` 的 K 必须是 `string | number | symbol`，不能是对象类型

---

## 🎯 2. 使用场景

1. **SaaS 场景**：多租户系统里，`Tenant` 和 `TenantCreateDto = Omit<Tenant, "id" | "createdAt" | "updatedAt">` 是标准模式，每个实体都有对应的 Create/Update DTO，用工具类型自动派生，保持同步。
2. **Fintech 场景**：`Record<Currency, ExchangeRate>` 做汇率映射表，key 是货币枚举，value 是汇率对象，类型安全地覆盖所有货币种类。
3. **E-commerce 场景**：商品列表页用 `Pick<Product, "id" | "name" | "price" | "thumbnail">` 只取需要的字段，减少不必要的数据传输。

---

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| 手动重新定义类型 | 不需要学工具类型 | 和原类型不同步，改一处要改两处 | 永远不推荐 |
| 映射类型手动实现 | 完全自定义 | 比工具类型写法复杂 | 需要内置工具类型不支持的变换时 |
| `interface extends` | 继承清晰 | 不能做删除字段、可选化等操作 | 单纯扩展字段时 |

**选型决策树**：
- 需要所有字段变可选（更新接口）→ `Partial<T>`
- 需要去掉某些字段（创建接口）→ `Omit<T, K>`
- 需要只保留某些字段（展示接口）→ `Pick<T, K>`
- 需要 key-value 映射 → `Record<K, V>`
- 需要深度 Partial（嵌套对象也变可选）→ 自定义递归映射类型

---

## 💼 4. 面试题

### 题 1（🟡 进阶）

**Q**: `Pick` 和 `Omit` 有什么区别？分别在什么场景下使用？

**核心 points**: Pick 白名单选字段、Omit 黑名单删字段、字段少用 Pick、字段多用 Omit、DTO 模式

**追问方向**:
1. 手动实现 `Partial<T>` 的原理是什么？（提示：映射类型 + `?`）
2. `Partial<Partial<T>>` 和 `Partial<T>` 有什么区别？

**加分回答**: "在设计 API 类型时，我们团队用 `Omit` 的多（因为创建接口通常只去掉 id 和时间戳字段），用 `Pick` 的少。但在做 GraphQL 的 resolver 类型时，`Pick` 更常用——因为每个 resolver 只返回部分字段，用 `Pick` 精确表达返回的数据形状，减少过量返回。"

---

## 🧪 5. 小测验

用工具类型实现以下需求：

```typescript
interface BlogPost {
  id: number
  title: string
  content: string
  authorId: number
  publishedAt?: string
  tags: string[]
  views: number
}
```

1. `CreatePostDto`：去掉 `id`、`publishedAt`、`views`
2. `UpdatePostDto`：基于 `CreatePostDto`，所有字段变可选
3. `PostSummary`：只保留 `id`、`title`、`authorId`、`publishedAt`
4. `PostStats`：只保留 `id`、`views`

> 先思考，再展开答案 👇

<details>
<summary>展开答案</summary>

```typescript
type CreatePostDto = Omit<BlogPost, "id" | "publishedAt" | "views">
// { title: string; content: string; authorId: number; tags: string[] }

type UpdatePostDto = Partial<CreatePostDto>
// { title?: string; content?: string; authorId?: number; tags?: string[] }

type PostSummary = Pick<BlogPost, "id" | "title" | "authorId" | "publishedAt">
// { id: number; title: string; authorId: number; publishedAt?: string }

type PostStats = Pick<BlogPost, "id" | "views">
// { id: number; views: number }
```

</details>

---

## 🌟 Senior 思维彩蛋

Senior 在做 API 设计时，会把工具类型的组合作为"单一事实来源"原则的体现——`User` 是主类型，所有 DTO 都从它派生，永远不手动重复定义字段。这样当 `User` 加了一个必填字段，`CreateUserDto` 会自动要求传入，`UpdateUserDto` 会自动支持更新，不需要找三个地方分别改。这是 TS 类型系统最有价值的地方之一。

---

## 🔗 关联算法题

- LeetCode #380 Insert Delete GetRandom O(1)（Record 映射思维）
- LeetCode #146 LRU Cache
- HackerRank: SQL - Aggregation（Record 类似 GROUP BY 的思维）

---

## 📅 复习记录

- [ ] D+3 复习（2026-05-27）
- [ ] D+7 复习（2026-05-31）
- [ ] D+21 复习（2026-06-14）

## 🔗 相关笔记

- [[TS-Topic-07-泛型基础]]
- [[TS-Topic-12-映射类型与模板字面量]]
- [[TS-Topic-11-条件类型与infer]]
