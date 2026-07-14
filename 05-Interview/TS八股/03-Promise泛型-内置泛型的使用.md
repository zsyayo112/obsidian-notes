---
tier: Tier-1
tech: TypeScript
topic: 泛型基础
knowledge-point: 内置泛型的使用（Promise/Array）
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
  - promise
---

# 📌 内置泛型的使用：Promise<T> / Array<T>

> Topic: [[TS 类型系统够用版]] · Tier: 1 · 难度: ⭐⭐

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Built-in generic | 内置泛型 | TS 自带的泛型类型（Promise/Array/Map…），作者已定义好 `T`，你调用时填 |
| Resolve | 兑现/敲定 | Promise 成功完成，交出它包裹的那个值 |
| Type erasure | 类型擦除 | 泛型只在编译期存在，编译后消失 |

## 📝 1. 实例代码

```typescript
// 代码读法说明：
// Promise<T> 和 Array<T> 都是"内置泛型"——你一直在用尖括号，其实就是在填别人留的占位符 T。

// Promise<T> 里的 T = 这个 Promise 成功后兑现出来的值的类型
async function createOrder(dto: CreateOrderRequest): Promise<OrderSubmitResult> {
  // ...
  return result; // result 是 OrderSubmitResult
  // 因为标了 async，实际返回被自动包成 Promise<OrderSubmitResult>
}

// await 就是"拆壳"，拆掉 Promise 外层，拿到里面的 T
const result = await createOrder(dto); // result: OrderSubmitResult（就是 T）

// Array<T> 里的 T = 数组元素的类型
const names: Array<string> = ["a", "b"]; // 等价于 string[]
const nums: number[] = [1, 2, 3];        // 两种写法等价

// 类比：Promise<T> 是"取餐号"，尖括号里写明"这号最终会取到什么"
// Promise<string>  → 最终取到一个字符串
// Promise<void>    → 取到"啥也没有"（只通知你完成）
```

⚠️ 易错点：
1. `Promise<X>` 里的 X 是"**成功后兑现的值的类型**"，不是"Promise 本身"。`await` 后拿到的才是 X。
2. `async` 函数**永远返回 Promise**——哪怕你 `return` 一个普通对象，也会被自动包进 Promise 壳。所以返回类型写 `Promise<你真正要返回的类型>`。
3. 调用 async 函数不 `await`，你拿到的是 Promise 壳（未拆），不是里面的值。

## 🎯 2. 使用场景

1. **SaaS 场景**：Service 层方法几乎全是 `async xxx(): Promise<Dto>`。调用方 `await` 拆壳拿到 DTO，类型全程精确，IDE 有提示。
2. **Fintech 场景**：`Promise<Transaction[]>` 表示"异步查一批交易"。`await` 后直接得到 `Transaction[]`，金额、状态字段都有类型保护。
3. **E-commerce 场景**：`fetch` 封装 `getProduct(id): Promise<Product>`，前端组件 `await` 后渲染，Product 的字段拼错立刻报错。

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| `Promise<any>` | 不用想返回类型 | 拆壳后丢类型，退化成 JS | 几乎不选 |
| 回调函数 callback | 老代码兼容 | 回调地狱、错误处理难 | 维护老库 |
| **`Promise<具体类型>`** | 类型精确 + async/await 可读 | 无 | **默认** |

**选型决策树**：
- 如果 异步操作 → `Promise<具体返回类型>` + async/await
- 如果 只需通知完成、无返回值 → `Promise<void>`

## 💼 4. 面试题

### 题 1（🟡 进阶）
**Q**: `Promise<OrderSubmitResult>` 里的尖括号是什么意思？`async` 函数的返回类型为什么必须是 Promise？
**核心 points**: Promise 是内置泛型，T = resolve 出来的值类型；async 函数强制返回 Promise，是语言规则；await 拆壳拿到 T。
**追问方向**:
1. `Promise<void>` 和 `Promise<undefined>` 区别？（void 表示"不关心返回值"，语义更清晰）
2. 不 await 直接用返回值会怎样？（拿到的是 Promise 对象本身，不是里面的值）
**加分回答**: "async/await 是 Promise 的语法糖，但类型上 `Promise<T>` 让'未来才有的值'也能被静态检查。这是把异步的不确定性，收进了编译期类型系统——错误左移。"

## 🧪 5. 小测验

> `data` 是什么类型？`raw` 呢？

```typescript
async function getUser(): Promise<{ id: number; name: string }> {
  return { id: 1, name: "Alice" };
}
const data = await getUser();
const raw = getUser(); // 注意：没有 await
```

<details>
<summary>展开答案</summary>

- `data` 是 `{ id: number; name: string }`——await 拆壳拿到里面的值。
- `raw` 是 `Promise<{ id: number; name: string }>`——没 await，拿到的是壳本身。
- 想用 `raw.id` 会报错（Promise 上没有 id），必须先 await。
</details>

## 🌟 Senior 思维彩蛋

`Promise<T>`、`Array<T>`、`Map<K,V>`、`Record<K,V>`——你每天用的这些尖括号**全是泛型**。Mid 把它们当"固定语法"背下来，Senior 意识到它们是**同一套泛型机制的实例**，于是能自己设计 `Result<T, E>`、`Paginated<T>` 这类领域泛型，让整个代码库的类型语言统一。看懂"内置泛型也是泛型"，是从"用类型"到"设计类型"的转折点。

## 🔗 关联算法题

- 无直接算法关联

---

## 📅 复习记录
- [ ] D+3 复习（2026-07-05）
- [ ] D+7 复习（2026-07-09）
- [ ] D+21 复习（2026-07-23）

## 🔗 相关笔记
- [[01-泛型基础-为什么需要泛型]]
- [[06-类型对象class实例-四个概念区分]]
