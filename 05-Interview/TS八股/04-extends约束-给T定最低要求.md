---
tier: Tier-1
tech: TypeScript
topic: 泛型约束
knowledge-point: extends 泛型约束
difficulty: ⭐⭐
status: learning
created: 2026-07-02
review-1: 2026-07-05
review-2: 2026-07-09
review-3: 2026-07-23
tags:
  - tech/typescript
  - tier-1
  - generics
  - constraints
---

# 📌 extends 约束：给 T 定个"最低要求"

> Topic: [[TS 类型系统够用版]] · Tier: 1 · 难度: ⭐⭐

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Constraint | 约束 | 给泛型 T 设一个门槛，限制它必须"长什么样" |
| `extends`（约束语境） | 约束/满足 | 泛型里的 extends = "必须是……之一 / 必须满足……"，**不是继承** |

> ⚠️ **核心纠错**：泛型参数里的 `extends` **不是**面向对象的"子类继承父类"。它是"**约束**"——把 T 限制成"至少满足某形状"。

## 📝 1. 实例代码

```typescript
// 代码读法说明：
// 先撞墙（纯 T 不能用属性），再用 extends 定门槛救场。

// ❌ 撞墙：纯 <T>，T 可能是任何类型，TS 不敢让你碰 .id
function getIdBad<T>(item: T): number {
  return item.id; // ❌ 报错：Property 'id' does not exist on type 'T'
  // 因为 T 可能是 number/string/boolean，它们都没有 .id
}

// ✅ 救场：extends 给 T 定"最低要求"——至少得有 id: number
function getId<T extends { id: number }>(item: T): number {
  return item.id; // ✅ 你保证了 T 有 id，TS 放行
}

// T extends { id: number } = "T 可以是任何类型，但至少得有 id: number 字段"
getId({ id: 1, name: "Alice" }); // ✅ 有 id（多 name 无所谓）
getId({ id: 99 });               // ✅ 有 id
getId(42);                       // ❌ number 没 id，进不来
getId({ name: "Bob" });          // ❌ 没 id，进不来

// 关键：extends 是"至少满足"（下限），不是"正好等于"
// 就像招聘写"至少本科"——本硕博都能投，唯独没到本科不行
```

⚠️ 易错点：
1. `extends` 在泛型约束里 = "**约束/至少满足**"，**不是"继承"**。（面试说错这个词会露怯）
2. `extends {...}` 是"**至少**有这些字段"，多余字段无所谓；唯独缺了要求的字段才拦。
3. 门槛**前移**了：没达标的类型在**调用时**就被拦，能进来的**保证有那个字段**，函数体里随便用。

## 🎯 2. 使用场景

1. **SaaS 场景**：`sortById<T extends { id: number }>(list: T[])`——通用排序函数，约束"元素得有 id"，任何带 id 的实体（User/Order/Project）都能排。
2. **Fintech 场景**：`audit<T extends { createdAt: Date }>(record: T)`——通用审计函数，约束"记录必须有时间戳"，保证能安全读 `createdAt`。
3. **E-commerce 场景**：`applyDiscount<T extends { price: number }>(item: T)`——约束"有 price 字段"，商品、套餐、附加服务都能复用打折逻辑。

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| 参数写死具体类型 | 简单直接 | 不通用，每种类型抄一遍 | 只处理一种类型 |
| 参数用 `any` | 能编译 | 丢类型、拼错不报错 | 不推荐 |
| 用接口 + 普通函数 | 清晰 | 返回类型丢失关联（不能保留具体 T） | 不需要保留精确类型时 |
| **泛型 + extends 约束** | 通用 + 类型安全 + 保留精确类型 | 语法门槛 | **"通用但需要用到 T 某些属性"的默认选择** |

**选型决策树**：
- 如果 只处理一种类型 → 写死类型
- 如果 通用、但函数体要用 T 的某属性 → **泛型 + `extends` 约束**
- 如果 通用、且只是原样搬运（不碰属性）→ 纯 `<T>`

## 💼 4. 面试题

### 题 1（🟡 进阶）
**Q**: 为什么纯 `<T>` 时不能写 `item.id`？`extends` 怎么解决？
**核心 points**: T 是任意类型，可能没有 id → TS 拒绝；`extends { id }` 给 T 定门槛，没 id 的类型在调用时被拦，函数体内即可安全使用。
**追问方向**:
1. `extends` 在泛型里和在 class 里含义一样吗？（不一样：泛型里是"约束"，class 里是"继承"——同一关键字两职）
2. `T extends { id: number }` 传 `{ id: 1, name: "x" }` 行吗？（行，extends 是"至少满足"）
**加分回答**: "泛型约束的本质是**在保留具体类型的同时，声明对它的最小假设**。这比'参数写成接口类型'更强——接口会把返回类型也擦成接口，而 `extends` 约束让 T 依然是传入的**精确类型**，可以在返回值里继续用。"

## 🧪 5. 小测验

> 下面函数想取"任意有 name 字段对象"的 name，请补全约束，并说明哪些调用合法。

```typescript
function getName<T extends ___>(item: T): string {
  return item.name;
}
getName({ name: "Alice", age: 30 }); // ?
getName({ title: "Bob" });           // ?
getName("hello");                    // ?
```

<details>
<summary>展开答案</summary>

```typescript
function getName<T extends { name: string }>(item: T): string {
  return item.name;
}
getName({ name: "Alice", age: 30 }); // ✅ 有 name（多 age 无所谓）
getName({ title: "Bob" });           // ❌ 没 name
getName("hello");                    // ❌ string 没有 name 字段
```
</details>

## 🌟 Senior 思维彩蛋

`extends` 约束是"**契约设计**"的类型层体现：你不关心 T 具体是什么，只声明"我需要它至少提供什么"。这跟依赖注入里"面向接口编程"是同一种思维——依赖抽象、不依赖具体。Mid 常把参数类型写死或写 any，Senior 用约束表达"最小依赖"，让函数既通用又安全。约束写得越精准（不多不少），API 越好用。

## 🔗 关联算法题

- 无直接算法关联

---

## 📅 复习记录
- [ ] D+3 复习（2026-07-05）
- [ ] D+7 复习（2026-07-09）
- [ ] D+21 复习（2026-07-23）

## 🔗 相关笔记
- [[01-泛型基础-为什么需要泛型]]
- [[05-keyof与索引访问-类型安全取值]]
