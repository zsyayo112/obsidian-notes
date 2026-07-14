---
tier: Tier-1
tech: TypeScript
topic: 泛型基础
knowledge-point: 类型推断
difficulty: ⭐
status: learning
created: 2026-07-02
review-1: 2026-07-05
review-2: 2026-07-09
review-3: 2026-07-23
tags:
  - tech/typescript
  - tier-1
  - generics
  - type-inference
---

# 📌 类型推断：TS 怎么自己猜出 T

> Topic: [[TS 类型系统够用版]] · Tier: 1 · 难度: ⭐

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Type inference | 类型推断 | TS 不用你写类型，看你传的值自己猜出来 |
| Type argument | 类型实参 | 调用时手动传进去的具体类型，如 `identity<string>` 里的 string |

## 📝 1. 实例代码

```typescript
// 代码读法说明：
// 调用泛型函数有两种写法——手动指定 T，或让 TS 自己猜。日常首选"让它猜"。

function identity<T>(arg: T): T {
  return arg;
}

// ── 写法一：手动指定 T（啰嗦，但有时需要）──
const a = identity<string>("hello"); // 明确告诉 TS：这次 T = string，a: string

// ── 写法二：让 TS 自己猜（更常用）──
const b = identity("hello"); // 没写 <string>，TS 看到字符串，自己猜 T = string，b: string
// a 和 b 类型完全一样，写法二省事

// TS 怎么猜：看你传的值是什么类型，就把 T 猜成那个类型
identity("hello"); // 传字符串 → T = string
identity(42);      // 传数字   → T = number
identity(true);    // 传布尔   → T = boolean

// ── 什么时候要手动指定？TS 猜不准时 ──
const arr = identity([]);            // 传空数组，TS 猜成 never[]/any[]，不是你要的
const arr2 = identity<number[]>([]); // 手动指定 T = number[]，明确
```

⚠️ 易错点：
1. 能让 TS 猜就让它猜（写法二）；只有它**猜不准**（如空数组、空对象）或你要**覆盖它的猜测**时，才手写 `<T>`。
2. 手动指定的尖括号写在**函数名后、参数括号前**：`identity<string>("hi")`。

## 🎯 2. 使用场景

1. **SaaS 场景**：`useState([])` 时 TS 猜不出数组元素类型（never[]），得手动 `useState<Todo[]>([])`——空初始值是"必须手写泛型"的高频场景。
2. **Fintech 场景**：`axios.get<Transaction[]>('/api/txns')` 手动指定响应类型，让后续 `.data` 精确成 `Transaction[]`。
3. **E-commerce 场景**：`items.map(x => x.price)` 这种，TS 从 `items` 的类型一路推断，全程不用手写类型——推断替你省掉大量样板。

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| 全部手写类型 | 显式、无歧义 | 极其啰嗦，样板爆炸 | 几乎不需要 |
| 依赖类型推断 | 简洁，代码干净 | 偶尔猜不准/猜太宽 | **日常默认** |
| 手动指定关键处 | 精确控制 | 需判断哪里该写 | 空初始值、想收窄类型时 |

**选型决策树**：
- 如果 传的值能看出类型 → 让 TS 推断（不写尖括号）
- 如果 空数组/空对象/想覆盖推断 → 手动指定 `<T>`

## 💼 4. 面试题

### 题 1（🟢 基础）
**Q**: 调用 `identity(42)` 时没写 `<number>`，TS 怎么知道 T 是 number？
**核心 points**: 类型推断；TS 看实参的类型倒推出类型参数；大多数情况能自动推断。
**追问方向**:
1. 什么情况下推断会失败或不准？（空数组 never[]、空对象、联合类型场景）
2. 推断和手动指定冲突时以谁为准？（手动指定优先，会覆盖推断）
**加分回答**: "推断让代码简洁，但空初始值这类场景推断会退化成 never[]，这时显式标注反而是防御性写法。知道'何时该信任推断、何时该接管'本身是一种类型素养。"

## 🧪 5. 小测验

> 下面两行，`a` 和 `b` 分别是什么类型？为什么不一样？

```typescript
function wrap<T>(x: T): T[] {
  return [x];
}
const a = wrap("hi");
const b = wrap<number>(42);
```

<details>
<summary>展开答案</summary>

- `a` 是 `string[]`：TS 从 `"hi"` 推断 T = string，返回 `T[]` = `string[]`。
- `b` 是 `number[]`：手动指定 T = number，返回 `number[]`。
- 一个靠推断、一个靠手动指定，但都正确得到 `T[]`。
</details>

## 🌟 Senior 思维彩蛋

类型推断是 TS "无感"的核心。Mid 要么到处手写类型（啰嗦），要么全靠 any（放弃）。Senior 懂得**让推断承担 90% 的工作，只在推断失效的边界（空集合、复杂联合、库的公共 API）显式标注**。一个好的类型签名应该让调用方"什么都不用写就得到精确类型"——这是设计泛型 API 的品味。

## 🔗 关联算法题

- 无直接算法关联

---

## 📅 复习记录
- [ ] D+3 复习（2026-07-05）
- [ ] D+7 复习（2026-07-09）
- [ ] D+21 复习（2026-07-23）

## 🔗 相关笔记
- [[01-泛型基础-为什么需要泛型]]
- [[03-Promise泛型-内置泛型的使用]]
