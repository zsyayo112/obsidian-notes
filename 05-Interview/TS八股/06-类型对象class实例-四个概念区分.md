---
tier: Tier-1
tech: TypeScript
topic: 基础概念辨析
knowledge-point: 类型/对象/class/实例 + type vs class
difficulty: ⭐
status: learning
created: 2026-07-02
review-1: 2026-07-05
review-2: 2026-07-09
review-3: 2026-07-23
tags:
  - tech/typescript
  - tier-1
  - fundamentals
---

# 📌 类型 / 对象 / class / 实例 —— 四个概念区分

> Topic: [[TS 类型系统够用版]] · Tier: 1 · 难度: ⭐

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Type | 类型 | 数据形状的蓝图，编译期存在、运行时消失 |
| Object | 对象 | `{}` 装的键值对**数据**（值），运行时真实存在 |
| Class | 类 | 造对象的"模具/工厂"，运行时存在，能 new、能装逻辑 |
| Instance | 实例 | 用 `new` 从 class 造出来的对象 |

## 📝 1. 实例代码

```typescript
// 代码读法说明：
// 四个概念串成一条线：类型是蓝图，对象是数据，class 是模具，实例是模具造出的对象。

// ① 类型 Type：数据形状的蓝图（编译期，运行时消失）
type User = { id: number; name: string };
// interface 在这个场景几乎等价：
interface IUser { id: number; name: string }

// ② 对象 Object：一坨真实数据（值）。这个对象"符合 User 类型"
const user: User = { id: 1, name: "Alice" };
// ⚠️ 对象是"数据/值"，不是 class！

// ③ class：造对象的模具（运行时真实存在，能 new、能装方法）
class UserClass {
  constructor(public id: number, public name: string) {}
  greet() { return `Hi, ${this.name}`; }
}

// ④ 实例 Instance：用模具 new 出来的对象
const instance = new UserClass(1, "Alice"); // instance 是 UserClass 的实例
```

⚠️ 易错点：
1. **对象 ≠ class**。对象是 `{}` 装的键值对（值/数据）；class 是"造对象的模具"。模具造出来的才是对象/实例。
2. **"实例"通常特指 `new` class 出来的对象**。`const user: User = {...}` 没有 new、没有 class，严格叫"符合 User 类型的对象"，不叫实例。
3. **类型运行时不存在**：`type User` 编译成 JS 后消失，不能 `new User()`、不能装方法。

## 🎯 2. 使用场景

1. **SaaS 场景**：从 API/数据库拿到的数据用 `type`/`interface` 描述形状（90% 情况），因为是纯数据、不需要方法。
2. **Fintech 场景**：需要封装带逻辑的领域模型（如 `Money` 类带 `add()`/`format()` 方法）时用 `class`——模具 + 内置行为。
3. **E-commerce 场景**：NestJS 的 Service/Controller 都是 `class`，因为要装业务逻辑 + 依赖注入；而 DTO 常用 class（配合装饰器校验）或 interface。

## 🔄 3. 可替代技术：type vs class 描述数据

| 方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| `type`/`interface` | 轻量、运行时零成本、纯描述形状 | 不能 new、不能装逻辑 | **描述纯数据（API/DB 返回），日常 90%** |
| `class` | 能 new、能装方法/逻辑、能被 DI | 运行时有成本、更重 | 需要"模具 + 行为"（Service、领域模型、DTO+装饰器） |

**选型决策树**：
- 如果 只是描述数据形状（无逻辑）→ `type`/`interface`
- 如果 需要 new 实例 + 内置方法/逻辑 → `class`
- 如果 NestJS Service/Controller/需要依赖注入 → `class`

## 💼 4. 面试题

### 题 1（🟢 基础）
**Q**: `type User = {...}` 和 `class UserClass {...}` 都能描述 `{id, name}`，区别在哪？
**核心 points**: type 编译期消失、不能 new、不装逻辑；class 运行时存在、能 new、能装方法、能被 DI。
**追问方向**:
1. 什么时候必须用 class 而不是 type？（需要实例化、内置方法、装饰器、依赖注入）
2. interface 和 type 区别？（interface 可声明合并、更适合对象形状；type 能表达联合/交叉/工具类型）
**加分回答**: "选 type 还是 class 的本质是'这坨东西只是数据，还是数据+行为'。DDD 里领域实体用 class（封装不变量），DTO/查询结果用 type（纯数据传输）。滥用 class 描述纯数据会带来无谓的运行时开销和序列化麻烦。"

## 🧪 5. 小测验

> 判断下列说法对错：
> 1. `const u = { id: 1 }` 里的 `u` 是一个 class。
> 2. `type User` 可以 `new User()`。
> 3. `new UserClass()` 造出来的东西叫"实例"。

<details>
<summary>展开答案</summary>

1. ❌ 错。`u` 是**对象**（数据/值），不是 class。class 是模具。
2. ❌ 错。`type` 运行时不存在，不能 new。只有 class 能 new。
3. ✅ 对。`new class` 造出来的对象就是"实例"。
</details>

## 🌟 Senior 思维彩蛋

"类型只在编译期、值在运行期"是 TS 的**分界线**，理解它能避开一大类困惑（如"为什么不能 `if (x instanceof MyType)`"——因为 type 运行时不存在，`instanceof` 只能用于 class）。Mid 常把 type 和 class 混用，Senior 清楚：**类型是编译期的契约，class 是运行期的实体**。这条界线也是"类型擦除"、装饰器元数据、`reflect-metadata`（NestJS DI 底层）等一系列高级话题的根。

## 🔗 关联算法题

- 无直接算法关联

---

## 📅 复习记录
- [ ] D+3 复习（2026-07-05）
- [ ] D+7 复习（2026-07-09）
- [ ] D+21 复习（2026-07-23）

## 🔗 相关笔记
- [[03-Promise泛型-内置泛型的使用]]
- [[05-keyof与索引访问-类型安全取值]]
