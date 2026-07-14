# 📊 TypeScript 学习看板

> Tier 1 · 13 Topics · 全部完成 ✅
> 学习日期：2026-05-24

---

## 🎯 总览

| 区段 | Topic 数 | 完成数 | 状态 |
|---|---|---|---|
| 🟢 入门区 | 5 | 5 | ✅ 完成 |
| 🟡 进阶区 | 4 | 4 | ✅ 完成 |
| 🔴 高级区 | 4 | 4 | ✅ 完成 |
| **合计** | **13** | **13** | **🎉 TypeScript 完成** |

---

## 🟢 入门区（Topic 1–5）

| # | 文件 | 难度 | 状态 | 复习 D+3 | 复习 D+7 | 复习 D+21 |
|---|---|---|---|---|---|---|
| 1 | [[TS-Topic-01-基本类型标注]] | ⭐ | ✅ | 05-27 | 05-31 | 06-14 |
| 2 | [[TS-Topic-02-type-vs-interface]] | ⭐ | ✅ | 05-27 | 05-31 | 06-14 |
| 3 | [[TS-Topic-03-联合类型与交叉类型]] | ⭐ | ✅ | 05-27 | 05-31 | 06-14 |
| 4 | [[TS-Topic-04-函数类型]] | ⭐ | ✅ | 05-27 | 05-31 | 06-14 |
| 5 | [[TS-Topic-05-对象类型与readonly]] | ⭐ | ✅ | 05-27 | 05-31 | 06-14 |

---

## 🟡 进阶区（Topic 6–9）

| # | 文件 | 难度 | 状态 | 复习 D+3 | 复习 D+7 | 复习 D+21 |
|---|---|---|---|---|---|---|
| 6 | [[TS-Topic-06-类型守卫]] | ⭐⭐ | ✅ | 05-27 | 05-31 | 06-14 |
| 7 | [[TS-Topic-07-泛型基础]] | ⭐⭐ | ✅ | 05-27 | 05-31 | 06-14 |
| 8 | [[TS-Topic-08-枚举与字面量类型]] | ⭐⭐ | ✅ | 05-27 | 05-31 | 06-14 |
| 9 | [[TS-Topic-09-模块系统与tsconfig]] | ⭐⭐ | ✅ | 05-27 | 05-31 | 06-14 |

---

## 🔴 高级区（Topic 10–13）

| # | 文件 | 难度 | 状态 | 复习 D+3 | 复习 D+7 | 复习 D+21 |
|---|---|---|---|---|---|---|
| 10 | [[TS-Topic-10-工具类型]] | ⭐⭐⭐ | ✅ | 05-27 | 05-31 | 06-14 |
| 11 | [[TS-Topic-11-条件类型与infer]] | ⭐⭐⭐ | ✅ | 05-27 | 05-31 | 06-14 |
| 12 | [[TS-Topic-12-映射类型与模板字面量]] | ⭐⭐⭐ | ✅ | 05-27 | 05-31 | 06-14 |
| 13 | [[TS-Topic-13-严格模式与生产级实践]] | ⭐⭐⭐⭐ | ✅ | 05-27 | 05-31 | 06-14 |

---

## 🔥 易混淆点速查

| 概念 | 正确理解 |
|---|---|
| `type` vs `interface` | type 加 `=`；interface 不加；interface 可合并，type 不行 |
| `extends` 两种含义 | interface 继承 = 加东西；泛型约束 = 入场限制 |
| `void` vs `never` | void = 正常结束没返回值；never = 根本不会结束 |
| `readonly field` vs `readonly T[]` | 前者锁字段，后者锁数组内容，两个要一起用 |
| `any` vs `unknown` | any 关掉检查；unknown 保留检查，用前必须判断 |
| `!` 非空断言 | 强行说不是 null，危险，用 `if` 判断代替 |
| `as` 类型断言 | 强行转类型，危险，用 `instanceof` 守卫代替 |
| `import` vs `import type` | 纯类型用 `import type`，编译后消失 |
| `keyof T` vs `T[K]` | keyof 拿字段名，T[K] 拿字段的类型 |
| `infer` | 放在想捕获的位置，TS 帮你推断那个位置的类型 |

---

## 📚 LeetCode 进度

| # | 题目 | 难度 | 状态 |
|---|---|---|---|
| 1 | Two Sum | Easy | ⬜ |
| 20 | Valid Parentheses | Easy | ⬜ |
| 21 | Merge Two Sorted Lists | Easy | ⬜ |
| 206 | Reverse Linked List | Easy | ⬜ |
| 704 | Binary Search | Easy | ⬜ |

---

## 🚀 下一步

- [ ] 开始 **React + React Hooks + React Query**
- [ ] 完成 3 道 LeetCode（Two Sum / Valid Parentheses / Binary Search）
- [ ] 启动第一个 end-to-end 项目

---

## 🔗 Dataview 查询（需要在 Obsidian 里启用 Dataview 插件）

```dataview
TABLE topic, difficulty, status
FROM "02-Tech-Notes"
WHERE tech = "TypeScript"
SORT difficulty ASC
```
