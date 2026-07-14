---
tier: Tier-1
tech: TypeScript
topic: 类型守卫
knowledge-point: typeof/in/instanceof/自定义守卫/is
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

# 📌 TypeScript 类型守卫（Type Narrowing）

> Topic: 类型守卫 · Tier: 1 · 难度: ⭐⭐

---

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Type Guard | 类型守卫 | 用判断语句让 TS 缩小联合类型的范围 |
| Narrowing | 类型收窄 | 经过判断后，TS 知道变量是更具体的类型 |
| `typeof` | 类型判断运算符 | 判断基本类型：string/number/boolean/object 等 |
| `in` | 字段存在判断 | 判断对象里有没有某个字段名 |
| `instanceof` | 实例判断 | 判断对象是不是某个类创建出来的 |
| Type Predicate | 类型谓词 | `value is SomeType`，自定义类型守卫的返回类型写法 |

---

## 📝 1. 实例代码

```typescript
// 代码读法说明：
// - 第一段：typeof 守卫，处理基本类型联合
// - 第二段：in 守卫，区分不同形状的对象
// - 第三段：自定义类型守卫，封装复用判断逻辑

// typeof 守卫
function describeValue(value: string | number | boolean): void {
  if (typeof value === "string") {
    console.log(`String: ${value}`)
  } else if (typeof value === "number") {
    console.log(`Number: ${value}`)
  } else {
    console.log(`Boolean: ${value}`)
  }
}

// in 守卫：通过字段名区分类型
interface Cat { name: string; meow: () => void }
interface Dog { name: string; bark: () => void }
type Animal = Cat | Dog

function describeAnimal(animal: Animal): void {
  if ("meow" in animal) {
    animal.meow()    // TS 知道是 Cat
  } else {
    animal.bark()    // TS 知道是 Dog
  }
}

const cat1 = {
  name: "Kitty",
  meow: () => { console.log(`Cat: ${cat1.name}`) }
}
const dog1 = {
  name: "Buddy",
  bark: () => { console.log(`Dog: ${dog1.name}`) }
}

describeAnimal(cat1)
describeAnimal(dog1)

// 自定义类型守卫：封装判断逻辑，返回 is 类型谓词
function isCat(animal: Animal): animal is Cat {
  return "meow" in animal
}

function describeAnimal2(animal: Animal): void {
  if (isCat(animal)) {
    animal.meow()   // TS 推断为 Cat
  } else {
    animal.bark()   // TS 推断为 Dog
  }
}
```

⚠️ 易错点：
1. `in` 守卫靠字段名区分，对象里的字段名必须和 interface 定义的完全一致
2. 自定义守卫返回 `animal is Cat` 而不是 `boolean`，TS 才能做类型推断
3. `typeof` 只能判断基本类型，对象类型要用 `in` 或 `instanceof`

---

## 🎯 2. 使用场景

1. **SaaS 场景**：处理 WebSocket 消息，消息类型是联合类型 `ChatMessage | SystemMessage`，用 `in` 守卫判断 `"content" in msg` 来区分，再分别处理。
2. **Fintech 场景**：API 返回 `SuccessResponse | ErrorResponse`，用自定义守卫 `isErrorResponse()` 封装判断逻辑，业务代码复用，不重复写判断条件。
3. **E-commerce 场景**：购物车里有普通商品和限时特价商品，用 `"salePrice" in item` 判断是否是特价商品，再决定显示哪个价格。

---

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| 直接 `as` 类型断言 | 写法简单 | 不安全，跳过类型检查，可能运行时崩溃 | 永远不推荐替代守卫 |
| 辨别联合类型（discriminated union）| TS 自动收窄，无需手动判断 | 需要提前设计好 tag 字段 | 类型结构可以设计时 |
| Zod schema parse | 运行时验证 + 类型推断 | 需要引入库 | 处理外部数据时 |

**选型决策树**：
- 如果判断基本类型（string/number）→ `typeof`
- 如果判断对象有没有某字段 → `in`
- 如果判断 class 实例 → `instanceof`
- 如果判断逻辑要复用 → 自定义类型守卫 `is`
- 如果类型可以提前设计 → 辨别联合类型（加 `kind` 字段）

---

## 💼 4. 面试题

### 题 1（🟢 基础）

**Q**: `typeof`、`in`、`instanceof` 三种类型守卫分别在什么场景下使用？

**核心 points**: typeof 基本类型、in 对象字段判断、instanceof 类实例判断、自定义 is 封装复用

**追问方向**:
1. 自定义类型守卫和普通返回 `boolean` 的函数有什么区别？
2. 什么是辨别联合类型（Discriminated Union）？

**加分回答**: "自定义类型守卫的价值在于封装复用——如果有 10 个地方都要判断 `'meow' in animal`，不如封装成 `isCat(animal)`，改判断逻辑时只改一处。而且 `animal is Cat` 这个返回类型让 TS 能在 if 块里自动推断类型，普通 `boolean` 做不到这一点。"

---

## 🧪 5. 小测验

定义两个类型：
```typescript
interface Circle { kind: "circle"; radius: number }
interface Rectangle { kind: "rectangle"; width: number; height: number }
type Shape = Circle | Rectangle
```

写一个函数 `getArea(shape: Shape): number`，计算面积：
- 圆形：`Math.PI * radius²`
- 矩形：`width * height`

用 `in` 守卫或 `kind` 字段来区分。

> 先思考，再展开答案 👇

<details>
<summary>展开答案</summary>

```typescript
// 方法 1：用 in 守卫
function getArea(shape: Shape): number {
  if ("radius" in shape) {
    return Math.PI * shape.radius ** 2
  } else {
    return shape.width * shape.height
  }
}

// 方法 2：用 kind 字段（辨别联合类型，更推荐）
function getArea2(shape: Shape): number {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius ** 2
  } else {
    return shape.width * shape.height
  }
}

console.log(getArea({ kind: "circle", radius: 5 }))           // 78.54
console.log(getArea({ kind: "rectangle", width: 4, height: 6 }))  // 24
```

</details>

---

## 🌟 Senior 思维彩蛋

Senior 设计联合类型时，会主动加一个 `kind` 或 `type` 字段，让 TS 可以自动收窄——这叫"辨别联合类型"（Discriminated Union）。比 `in` 守卫更安全，因为 `in` 守卫依赖字段名，字段名改了守卫就失效了；而 `kind` 字段是契约，改了会立刻报错。Redux action 的设计就是经典的辨别联合类型。

---

## 🔗 关联算法题

- LeetCode #704 Binary Search（判断边界条件，类似类型收窄思维）
- LeetCode #20 Valid Parentheses（条件分支处理）
- HackerRank: Problem Solving - Conditional Statements

---

## 📅 复习记录

- [ ] D+3 复习（2026-05-27）
- [ ] D+7 复习（2026-05-31）
- [ ] D+21 复习（2026-06-14）

## 🔗 相关笔记

- [[TS-Topic-03-联合类型与交叉类型]]
- [[TS-Topic-13-严格模式与生产级实践]]
- [[TS-Topic-11-条件类型与infer]]
