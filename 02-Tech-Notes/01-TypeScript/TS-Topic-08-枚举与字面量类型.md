---
tier: Tier-1
tech: TypeScript
topic: 枚举与字面量类型
knowledge-point: enum/const-enum/字面量联合/switch/选型
difficulty: ⭐⭐
status: done
created: 2026-05-24
review-1: 2026-05-27
review-2: 2026-05-31
review-3: 2026-06-14
tags:
  - tech/typescript
  - tier-1
---

# 📌 TypeScript 枚举（Enum）& 字面量类型

> Topic: 枚举与字面量类型 · Tier: 1 · 难度: ⭐⭐

---

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Enum | 枚举 | 给一组相关的值起名字，同时是类型也是值 |
| String Enum | 字符串枚举 | `enum Foo { Bar = "bar" }`，成员值是字符串 |
| Numeric Enum | 数字枚举 | `enum Foo { Bar }`，成员值默认从 0 开始自增 |
| Literal Type | 字面量类型 | 限定值只能是某些具体值，如 `"left" \| "right"` |
| `const enum` | 常量枚举 | 编译后内联值，不产生额外 JS 对象，性能更好 |

---

## 📝 1. 实例代码

```typescript
// 代码读法说明：
// - 第一段：字符串枚举 + switch 使用
// - 第二段：enum 配合 Record 做映射表
// - 第三段：enum vs 字面量联合类型对比

// 字符串枚举
enum OrderStatus {
  Pending = "pending",
  Shipped = "shipped",
  Delivered = "delivered",
  Cancelled = "cancelled",
}

function describeStatus(status: OrderStatus): void {
  switch (status) {
    case OrderStatus.Pending:
      console.log("待发货")
      break
    case OrderStatus.Shipped:
      console.log("已发货")
      break
    case OrderStatus.Delivered:
      console.log("已送达")
      break
    case OrderStatus.Cancelled:
      console.log("已取消")
      break
  }
}

describeStatus(OrderStatus.Pending)    // 待发货
describeStatus(OrderStatus.Shipped)    // 已发货
describeStatus(OrderStatus.Delivered)  // 已送达
describeStatus(OrderStatus.Cancelled)  // 已取消

// enum 配合 Record 做映射表
enum HttpMethod {
  Get = "GET",
  Post = "POST",
  Put = "PUT",
  Delete = "DELETE",
}

const apiConfig = {
  method: HttpMethod.Get,
  url: "/api/users"
}

// 字面量联合类型（简单场景用这个）
type YesNo = "yes" | "no"
function answer(value: YesNo) {
  console.log(value)
}
```

⚠️ 易错点：
1. `switch` 每个 `case` 必须加 `break`，否则会继续往下执行（fall-through）
2. `enum` 里用 `=` 赋值，对象字段用 `:`，不要混淆
3. 调用时用 `OrderStatus.Pending` 不能直接写 `"pending"`，TS 会报错

---

## 🎯 2. 使用场景

1. **SaaS 场景**：用户角色权限用 `enum Role { Admin = "admin", Editor = "editor" }`，配合 `Record<Role, Permission>` 做权限表，角色改动时只改 enum 一处，权限表自动跟着变。
2. **Fintech 场景**：交易状态机用 enum 定义所有状态，配合 switch 做状态转换逻辑，穷举检查确保所有状态都被处理，防止新增状态漏掉处理分支。
3. **E-commerce 场景**：HTTP 请求方法用 `enum HttpMethod`，配合 API 客户端封装，避免业务代码里出现 `"GET"` 魔法字符串，重构时统一修改。

---

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| 字面量联合类型 `"a" \| "b"` | 简单，无编译产物 | 不能复用枚举成员的引用 | 值少、只用一次 |
| `const` 对象 + `as const` | 类似 enum 但更灵活 | 写法稍繁琐 | 需要运行时访问值的场景 |
| `const enum` | 编译后内联，无 JS 对象 | 不能动态访问，不能反向映射 | 性能敏感的库开发 |

**选型决策树**：
- 如果值少、简单、只用一次 → 字面量联合类型
- 如果值多、语义重要、要复用 → `enum`
- 如果要在运行时遍历所有枚举值 → `enum`
- 如果要在库里用，减少编译产物 → `const enum`

---

## 💼 4. 面试题

### 题 1（🟡 进阶）

**Q**: `enum` 和字面量联合类型有什么区别？什么时候选哪个？

**核心 points**: enum 同时是类型和值、可以运行时访问、联合类型只在编译时存在、enum 有命名语义、联合类型简洁

**追问方向**:
1. `const enum` 和普通 `enum` 有什么区别？
2. 如何遍历一个 enum 的所有值？

**加分回答**: "我们团队的约定是：如果这组值需要在运行时遍历（比如生成下拉菜单选项），用 `enum`；如果只是编译期的类型约束，用字面量联合类型，更轻量，也不会在编译产物里留下额外的 JS 对象。"

---

## 🧪 5. 小测验

用 `enum` 定义一个 `Priority` 枚举（`Low`、`Medium`、`High`），然后：
1. 写一个函数 `getPriorityLabel` 用 switch 返回中文标签
2. 用 `Record<Priority, string>` 写同样的功能

对比两种写法，哪个更适合维护？

> 先思考，再展开答案 👇

<details>
<summary>展开答案</summary>

```typescript
enum Priority {
  Low = "low",
  Medium = "medium",
  High = "high",
}

// 写法 1：switch
function getPriorityLabel(priority: Priority): string {
  switch (priority) {
    case Priority.Low:    return "低优先级"
    case Priority.High:   return "高优先级"
    case Priority.Medium: return "中优先级"
  }
}

// 写法 2：Record（更推荐，扩展性更好）
const priorityLabel: Record<Priority, string> = {
  [Priority.Low]:    "低优先级",
  [Priority.Medium]: "中优先级",
  [Priority.High]:   "高优先级",
}

// Record 更好维护：新增枚举值时，TS 会提醒你补充 Record 里的映射
// switch 漏掉新 case 时，不会有编译错误（除非加 never 穷举检查）
```

</details>

---

## 🌟 Senior 思维彩蛋

Senior 更倾向用 `Record<Enum, Value>` 替代 `switch`——原因是 `Record` 是数据，`switch` 是逻辑。当枚举新增一个值，`Record` 会在编译时报错提醒你补充映射；`switch` 默默进 `default` 分支，静默失败。数据驱动的方式更安全，也更容易测试。

---

## 🔗 关联算法题

- LeetCode #169 Majority Element（枚举思维）
- LeetCode #268 Missing Number
- HackerRank: Problem Solving - Enumerations

---

## 📅 复习记录

- [ ] D+3 复习（2026-05-27）
- [ ] D+7 复习（2026-05-31）
- [ ] D+21 复习（2026-06-14）

## 🔗 相关笔记

- [[TS-Topic-03-联合类型与交叉类型]]
- [[TS-Topic-06-类型守卫]]
- [[TS-Topic-10-工具类型]]
