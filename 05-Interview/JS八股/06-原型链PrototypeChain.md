---
tier: Tier-1
tech: JavaScript
topic: JS 核心机制
knowledge-point: 原型链 · new · prototype vs __proto__
difficulty: ⭐⭐⭐
status: learning
created: 2026-07-02
review-1: 2026-07-05
review-2: 2026-07-09
review-3: 2026-07-23
tags:
  - tech/javascript
  - tier-1
  - prototype
  - oop
  - 面试必考
---
# 📌 原型链 Prototype Chain
> Topic: [[JS 核心机制]] · Tier: 1 · 难度: ⭐⭐⭐
> 💡 一句话：**对象找不到的属性，就去「备用袋子」(__proto__) 找，一层层往上，直到 null。**

## 📖 0. 术语扫盲
| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Prototype chain | 原型链 | 沿 `__proto__` 逐层向上查找属性的链条 |
| `__proto__` | 原型引用 | **每个对象**都有，指向自己的「备用袋子」 |
| `prototype` | 原型对象 | **函数**才有，给 new 出来的实例当备用袋子 |
| Constructor | 构造函数 | 造对象的「模板函数」，配合 `new` 用 |
| Method / Property | 方法 / 属性 | 字段值是函数=方法；是数据=属性 |

## 📝 1. 实例代码
```javascript
// 代码读法说明：
// - 对象=一袋子属性；找不到就去备用袋子(__proto__)找
// - 构造函数 + new 批量造对象；方法挂 prototype 共享
// - class 是这套的语法糖

// ① 属性查找沿原型链
const 备用袋子 = { eat() { console.log("吃东西"); } };
const dog = { bark() { console.log("汪"); } };
Object.setPrototypeOf(dog, 备用袋子);   // dog 的备用袋子 = 备用袋子
dog.bark();  // "汪"     ← dog 自己有
dog.eat();   // "吃东西" ← dog 没有 → 去 __proto__ 找到

// ② 构造函数 + new
function Dog(name) {
  this.name = name;            // ③ new 让 this 指向新对象，填属性
}
Dog.prototype.bark = function () {   // 方法挂 prototype，所有实例共享一份
  console.log(this.name + ": 汪");
};
const d = new Dog("旺财");
// new 偷偷做 4 件事：
//   ① 造空对象 {}
//   ② 空对象.__proto__ = Dog.prototype（连上原型链）
//   ③ this 指向空对象，执行 Dog → this.name="旺财"
//   ④ 返回该对象
d.bark();   // "旺财: 汪"（d 自己没 bark → 沿 __proto__ 找到 Dog.prototype.bark）

console.log(d.__proto__ === Dog.prototype);   // true（同一个袋子！）
```
⚠️ 易错点：
1. **`prototype` vs `__proto__`**：`prototype` 是函数那头「存方法的袋子」；`__proto__` 是实例那头「指向袋子的线」；两者指向**同一个袋子**。`new` 自动接线，平时不用手写 `__proto__`。
2. **链的尽头**：找到 `null` 还没有 → 属性返回 `undefined`，方法报错。
3. **别给单个实例手改原型**：`setPrototypeOf` 运行时改原型会让引擎优化失效（慢）+ 难维护，用 `class extends` 代替。

## 🎯 2. 使用场景
1. **SaaS 场景**：NestJS 自定义异常 `class UserNotFound extends HttpException`——继承即原型链，白拿父类能力。
2. **Fintech 场景**：领域模型继承体系（基础账户 → 储蓄账户/信用账户），共享方法挂原型省内存。
3. **E-commerce 场景**：所有 Service 继承 BaseService（通用 CRUD/日志），`super()` 沿原型链调父类。

## 🔄 3. 可替代技术
| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| `class extends` | 语义清晰、规范 | 底层仍是原型 | 需要继承体系时（首选） |
| `Object.create(proto)` | 直接指定原型 | 不如 class 直观 | 需要精确控制原型时 |
| 组合优于继承（mixin/函数） | 灵活、避免深继承 | 需自己组织 | 继承层级过深时 |

**选型决策树**：
- 如果是 is-a 关系、要继承能力 → `class extends`
- 如果继承层级快超过 2-3 层 → 考虑组合（mixin/依赖注入）

## 💼 4. 面试题
### 题 1（🟡 进阶）
**Q**: `d.bark()` 是怎么找到 bark 方法的？如果 d 自己没有会怎样？
**核心 points**: 先在实例 d 自身找 → 没有 → 沿 `__proto__` 到 `Dog.prototype` → 再没有到 `Object.prototype` → 到 null 返回 undefined/报错。这就是原型链查找。
**追问方向**:
1. `prototype` 和 `__proto__` 区别？（函数的袋子 vs 实例指向袋子的线，同一个袋子）
2. 方法为什么挂 prototype 不挂实例？（所有实例共享一份，省内存）
**加分回答**: JS 是基于原型的语言，不是基于类的；`class` 只是原型机制的语法糖——理解这点才能看懂框架源码（NestJS/React class 组件）。

## 🧪 5. 小测验
> 点击展开答案前先思考
```javascript
function Animal() {}
Animal.prototype.eat = function () { return "eating"; };
const a = new Animal();
console.log(a.eat());              // ?
console.log(a.hasOwnProperty("eat")); // ?
```
<details>
<summary>展开答案</summary>

- `a.eat()` → `"eating"`：a 自己没 eat，沿原型链到 Animal.prototype 找到。
- `a.hasOwnProperty("eat")` → `false`：eat 在原型上，不在 a 自身。（hasOwnProperty 只查自身）
</details>

## 🌟 Senior 思维彩蛋
运行时用 `Object.setPrototypeOf` 改已有对象的原型，会让 V8 的隐藏类优化失效，热路径上是性能杀手。MDN 明确警告避免。要「让某些实例有额外能力」，用 class extends 或组合，别手改单个实例原型——能做 ≠ 该做。

## 🔗 关联算法题
- 树/链表遍历思路相通（原型链本质是单向链表向上查找）
- LeetCode #206 反转链表（理解链式结构）

---
## 📅 复习记录
- [ ] D+3 复习（2026-07-05）
- [ ] D+7 复习（2026-07-09）
- [ ] D+21 复习（2026-07-23）
## 🔗 相关笔记
- [[07-this 五种绑定]]（new 绑定接这里）
- [[04-闭包与 useState]]
