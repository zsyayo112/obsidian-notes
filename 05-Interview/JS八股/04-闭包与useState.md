---
tier: Tier-1
tech: JavaScript
topic: JS 核心机制
knowledge-point: 闭包 Closure · useState 原理
difficulty: ⭐⭐⭐
status: learning
created: 2026-07-02
review-1: 2026-07-05
review-2: 2026-07-09
review-3: 2026-07-23
tags:
  - tech/javascript
  - tech/react
  - tier-1
  - closure
  - hooks
  - 面试必考
---
# 📌 闭包 Closure & useState 原理（⭐ 接 React Hooks）
> Topic: [[JS 核心机制]] · Tier: 1 · 难度: ⭐⭐⭐
> 💡 一句话：**闭包 = 函数被带出去后，依然记得并能访问它出生时的外层变量。**

## 📖 0. 术语扫盲
| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Closure | 闭包 | 函数 + 它记住的外层变量，打包在一起 |
| Lexical scope | 词法作用域 | 函数能访问「定义时」所在位置的变量 |
| Private variable | 私有变量 | 被闭包锁住、外界无法直接访问的变量 |
| Stale closure | 陈旧闭包 | 闭包记住了旧变量值（面试坑，待补） |

## 📝 1. 实例代码
```javascript
// 代码读法说明：
// - makeCounter：闭包留住 count，跨调用记住
// - 极简 useState：React 就是这么用闭包「记住」state 的

// ① 闭包留住变量（count 不随 makeCounter 结束而销毁）
function makeCounter() {
  let count = 0;              // 被闭包留住
  return function () {        // 这个内层函数 = 闭包
    count += 1;              // 记住并修改外层 count
    return count;
  };
}
const counter = makeCounter();
counter();  // 1
counter();  // 2   ← 记住了！count 没被重置

// ② 极简版 useState（React 内部原理）
function 创建state(初始值) {
  let 值 = 初始值;              // 被闭包留住，不随组件重新渲染而丢
  function setState(新值) {    // setState = 闭包，唯一能改「值」的门
    值 = 新值;
    重新渲染();
  }
  return [值, setState];       // 结构和 const [count, setCount] = useState(0) 一样
}
```
⚠️ 易错点：
1. **匿名函数可省名字**：`return function(){}` 不起名，因为生出来就被 return 走，内部不需要再用名字引用它。
2. **闭包记住的是「变量本身」不是「当时的值」**：循环里 `var i` + `setTimeout` 经典输出 `3 3 3`（stale closure，待专门补）。
3. **对象方法别用箭头函数**：箭头函数 this 会指向外层而非该对象（见 [[06-this 五种绑定]]）。

## 🎯 2. 使用场景
1. **SaaS 场景**：React 组件用 `useState` 保存表单/对话状态，跨渲染记住——底层就是闭包。
2. **Fintech 场景**：用闭包做私有余额，只暴露 存/取/查 三个受控接口，外界无法直接篡改余额。
3. **E-commerce 场景**：防抖/节流函数（debounce/throttle）用闭包保存定时器 id 和上次执行时间。

## 🔄 3. 可替代技术
| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| 闭包私有变量 | 真正私有、封装好 | 每个实例一份闭包，稍耗内存 | 需要私有状态 |
| `class` + `#私有字段` | 语义清晰、面向对象 | 语法较新 | 类结构下的私有 |
| 全局变量 | 简单 | 会被乱改/共享冲突 | ❌ 尽量避免 |

**选型决策树**：
- 如果要「函数级私有状态」→ 闭包
- 如果在 class 里要私有 → `#field`
- 需要跨组件共享 → 状态管理库（Context/Zustand）

## 💼 4. 面试题
### 题 1（🔴 高级）
**Q**: 函数组件每次渲染都重新执行，局部变量会重置，那 `useState` 是怎么「记住」上次的 state 的？
**核心 points**: React 用**闭包**把 state 存在组件函数**外部**一个持久结构里（配合 Hooks 调用顺序定位）；组件通过闭包引用到它，所以跨渲染记住。`setState` 改的是那个外部值并触发重渲染。
**追问方向**:
1. 为什么 Hooks 不能写在 if 里？（依赖固定调用顺序定位每个 state）
2. `const count` 为什么能变？（变的不是常量，是外部存储，重渲染重新读出新值）
**加分回答**: 这也是 stale closure 的根源——`useEffect` 里若闭包捕获了旧 state，会读到过期值，需正确设置依赖数组。

## 🧪 5. 小测验
> 点击展开答案前先思考
```javascript
const [count, setCount] = useState(0);
function 点击() {
  setCount(count + 1);
  setCount(count + 1);
  console.log(count);
}
// 点一次：1) count 最终变几？ 2) log 打印几？
```
<details>
<summary>展开答案</summary>

1. **变成 1**：两次都是 `setCount(0 + 1)`（本次渲染 count 被闭包定死为 0）。
2. **打印 0**：setState 异步，本次 count 不变。
想正确 +2 → 用函数式：`setCount(prev => prev + 1)` 两次。prev 是 React 塞的最新值。
</details>

## 🌟 Senior 思维彩蛋
`makeCounter` 和 `useState` 是同一个东西：count 被闭包留住(不重置) + counter/setCount 是唯一改它的门(封装)。你随手写的迷你计数器，本质就是迷你 useState。懂这个，Hooks 不再是黑盒——这是 mid→senior 的分水岭。

## 🔗 关联算法题
- 无直接算法题（语言机制）
- 「实现 debounce/throttle」是闭包高频手写题

---
## 📅 复习记录
- [ ] D+3 复习（2026-07-05）⚠️ 待补 stale closure（var/let 循环坑 + React 版）
- [ ] D+7 复习（2026-07-09）
- [ ] D+21 复习（2026-07-23）
## 🔗 相关笔记
- [[01-函数作为值与回调函数]]
- [[03-事件循环 Event Loop]]
- [[05-setState 用法]]
