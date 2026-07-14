---
tier: Tier-1
tech: JavaScript
topic: JS 核心机制
knowledge-point: this 五种绑定 · 箭头函数 this
difficulty: ⭐⭐⭐
status: learning
created: 2026-07-02
review-1: 2026-07-05
review-2: 2026-07-09
review-3: 2026-07-23
tags:
  - tech/javascript
  - tier-1
  - this
  - 面试必考
---
# 📌 this 五种绑定
> Topic: [[JS 核心机制]] · Tier: 1 · 难度: ⭐⭐⭐
> 💡 一句话核心：**this 指向「调用这个函数时，点号前面的那个对象」——调用时决定，不是定义时。**

## 📖 0. 术语扫盲
| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Implicit binding | 隐式绑定 | `obj.fn()` → this = 点号前的 obj |
| Default binding | 默认绑定 | `fn()` 直接调用 → this = 全局/undefined |
| Explicit binding | 显式绑定 | `fn.call(obj)`/`bind(obj)` → this 由你指定 |
| new binding | new 绑定 | `new Fn()` → this = 新创建的对象 |
| Lexical this | 词法 this | 箭头函数：this = 定义时外层的 this（锁死） |

## 📝 1. 实例代码
```javascript
// 代码读法说明：五种情况判断 this 指向；重点看「丢失」和箭头函数救场

// ① 隐式绑定：看点号前面是谁
const dog = { name: "旺财", bark() { console.log(this.name); } };
dog.bark();   // "旺财"（点号前是 dog）

// ② 默认绑定 + this 丢失（经典坑）
const f = dog.bark;   // 只取出函数本身，dog 没跟着
f();                  // undefined（f() 直接调用，点号前空了 → this 丢）
setTimeout(dog.bark, 0);  // undefined（同理，传出去时 dog 被甩掉）

// ③ 箭头函数救场：没有自己的 this，借用「外层」的 this
const user = {
  name: "Hal",
  greetLater() {                       // 外层函数：user.greetLater() → this=user
    setTimeout(() => {                  // 内层箭头函数：借外层 this=user
      console.log(this.name);          // "Hal"（不丢！）
    }, 1000);
  }
};
user.greetLater();

// ④ 对比：内层用普通函数就丢
// setTimeout(function () { console.log(this.name); }, 1000);  // undefined
```
⚠️ 易错点：
1. **this 看调用时，不看定义时**：`dog.bark()` this=dog；`const f=dog.bark; f()` this 丢（点号前空了）。
2. **传方法当回调 = this 丢**：`setTimeout(user.greet)`、`onClick={this.handleClick}` 都只传函数本身，对象没跟着。
3. **对象方法别用箭头函数**：箭头 this 指向外层作用域（模块/全局），不是该对象 → 拿不到对象属性。

## 🎯 2. 使用场景
1. **SaaS 场景**：React class 组件事件处理，早期需 `bind(this)` 或箭头函数，避免 onClick 里 this 丢。
2. **Fintech 场景**：定时/异步回调里访问实例状态，用箭头函数锁定 this，避免读到 undefined 崩溃。
3. **E-commerce 场景**：工具库的 `this.config`，用 bind 显式绑定确保回调里 this 正确。

## 🔄 3. 可替代技术（解决 this 丢失的三种方式）
| 方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| 箭头函数定义方法 | 一劳永逸、简洁 | 不能用于对象方法/原型方法 | React 组件方法（现代首选） |
| `.bind(this)` | 显式、可读 | constructor 里要写一堆 | class 里绑定回调 |
| 调用时箭头包一层 `()=>this.fn()` | 就地解决 | 每次渲染新建函数 | 临时/一次性 |

**选型决策树**：
- 如果是 React 组件方法 → 箭头函数定义
- 如果是 class 里要传出去的回调 → constructor 里 bind
- 如果对象方法/原型方法 → **必须**普通函数（不能箭头）

## 💼 4. 面试题
### 题 1（🔴 高级）
**Q**: 箭头函数和普通函数在 this 上有什么区别？
**核心 points**: 普通函数 this 由**调用时**动态决定（看谁调用，会丢）；箭头函数**没有自己的 this**，用定义时外层作用域的 this，锁死不可变。
**追问方向**:
1. 为什么箭头函数适合回调、不适合对象方法？（回调里 this 不丢是优点；对象方法里 this 不指向对象是缺点）
2. React 为什么从 bind 演进到箭头函数？（根治回调里 this 丢失）
**加分回答**: `setTimeout(user.greet)` 打印 undefined 的本质——传参时只传了函数本身，`user.` 只用于「找到」函数，不会粘在函数上；调用时点号前空了，this 丢。

## 🧪 5. 小测验
> 点击展开答案前先思考
```javascript
const obj = {
  name: "A",
  regular: function () { return this.name; },
  arrow: () => { return this?.name; }
};
console.log(obj.regular());  // ?
console.log(obj.arrow());    // ?
```
<details>
<summary>展开答案</summary>

- `obj.regular()` → `"A"`：普通函数，点号前是 obj，this=obj。
- `obj.arrow()` → `undefined`：箭头函数 this 指向定义时外层（模块/全局），不是 obj。
教训：对象方法别用箭头函数。
</details>

## 🌟 Senior 思维彩蛋
现代 React 函数组件几乎不用 this——状态靠闭包(useState)，事件靠箭头函数。this 主要是 class 和对象方法的事。但看框架源码（NestJS、React class 组件、老库）时必须懂 this 绑定，否则读不懂 `.bind(this)` / `.call()` 在干嘛。

## 🔗 关联算法题
- 无直接算法题（语言机制）

---
## 📅 复习记录
- [ ] D+3 复习（2026-07-05）
- [ ] D+7 复习（2026-07-09）
- [ ] D+21 复习（2026-07-23）
## 🔗 相关笔记
- [[06-原型链 PrototypeChain]]（new 绑定）
- [[01-函数作为值与回调函数]]（回调里 this 丢）
- [[04-闭包与 useState]]
