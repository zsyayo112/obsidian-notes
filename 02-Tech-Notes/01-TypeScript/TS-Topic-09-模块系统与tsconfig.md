---
tier: Tier-1
tech: TypeScript
topic: 模块系统与tsconfig
knowledge-point: export/import/import-type/tsconfig/strict
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

# 📌 TypeScript 模块系统 + tsconfig

> Topic: 模块系统与tsconfig · Tier: 1 · 难度: ⭐⭐

---

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Module | 模块 | 一个独立的文件，有自己的作用域 |
| `export` | 导出 | 让其他文件可以用这个文件里的内容 |
| `import` | 导入 | 从其他文件引入内容 |
| `import type` | 类型导入 | 只导入类型，编译后这行代码消失 |
| `tsconfig.json` | TS 配置文件 | 告诉 TS 编译器怎么工作 |
| `strict` 模式 | 严格模式 | 开启后 TS 做更严格的类型检查，Senior 必开 |
| Path Alias | 路径别名 | 用 `@/` 代替 `../../` 这样的长路径 |

---

## 📝 1. 实例代码

```typescript
// 代码读法说明：
// - types.ts：定义并导出类型
// - index.ts：导入并使用类型
// - tsconfig.json：项目配置

// ---- types.ts ----
export interface User {
  id: number
  name: string
  email: string
}

export interface Order {
  id: number
  detail: string
  user: User
}

// ---- index.ts ----
import type { User, Order } from "./types"  // 纯类型用 import type

const user1: User = {
  id: 1,
  name: "syz",
  email: "syz@example.com",
}

console.log(`${user1.name}`)

// ---- 两种 export 写法 ----

// 写法 1：定义时直接 export（推荐）
export interface Product {
  id: number
  name: string
}

// 写法 2：先定义，最后统一 export
interface Cart {
  items: Product[]
}
export { Cart }
```

```json
// tsconfig.json 核心配置
{
  "compilerOptions": {
    "target": "ES2020",        // 编译成哪个版本的 JS
    "strict": true,            // ⭐ 必须开！开启 6 个严格检查
    "outDir": "./dist",        // 编译后文件放哪里
    "rootDir": "./src",        // 源码在哪里
    "baseUrl": ".",            // 路径别名的根目录
    "paths": {
      "@/*": ["src/*"]         // 用 @/ 代替 src/
    },
    "moduleResolution": "node" // 模块解析策略
  }
}
```

⚠️ 易错点：
1. `type` 加 `=`，`interface` 不加 `=`，对象字段内部永远用 `:`
2. `import type` 用于纯类型导入，编译后自动消失；普通函数/变量用普通 `import`
3. `strict: true` 关掉后很多检查失效，等于白用 TS

---

## 🎯 2. 使用场景

1. **SaaS 场景**：把所有 API 响应类型统一放在 `src/types/api.ts`，用 `export` 导出，业务模块用 `import type` 引入，保持类型定义集中管理，改一处全部生效。
2. **Fintech 场景**：用路径别名 `@/types/transaction` 代替 `../../../types/transaction`，重构目录结构时只改 `tsconfig paths`，不用改所有 import 路径。
3. **E-commerce 场景**：`strict: true` 强制处理 `null` 检查，防止用户信息为空时直接读取 `user.name` 导致白屏，编译期就能发现问题。

---

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| CommonJS `require` | 兼容老 Node.js | 不支持 tree-shaking，TS 类型支持差 | 老 Node.js 项目 |
| 全局类型声明 `.d.ts` | 不需要 import | 全局污染，难以追踪来源 | 扩展第三方库类型时 |
| `verbatimModuleSyntax` | 强制区分类型和值导入 | 需要到处加 `import type` | 严格的库开发 |

**选型决策树**：
- 导入的是 `interface` / `type` → `import type`
- 导入的是函数 / 变量 / class → 普通 `import`
- 路径超过两层 `../../` → 配置路径别名 `@/`
- 新项目 → `strict: true` 必开

---

## 💼 4. 面试题

### 题 1（🟡 进阶）

**Q**: `import` 和 `import type` 有什么区别？什么时候用 `import type`？

**核心 points**: import type 只导入类型、编译后消失、不产生运行时依赖、verbatimModuleSyntax 强制要求

**追问方向**:
1. `tsconfig.json` 里的 `strict` 选项具体开启了哪些检查？
2. 如果第三方库没有类型定义，怎么处理？

**加分回答**: "`import type` 不只是语义上的区分——在某些打包工具（如 esbuild）里，它能帮助 tree-shaking 更准确地删掉未使用的代码，因为打包工具不需要分析这个 import 是否有副作用，直接删掉就行。对于大型项目，这能减少编译产物的体积。"

---

## 🧪 5. 小测验

**场景题**：

你接手了一个老项目，`tsconfig.json` 里 `strict: false`，代码里到处是 `any`。现在要升级到严格模式，你会怎么做？说出至少 3 个步骤。

> 先思考，再展开答案 👇

<details>
<summary>展开答案</summary>

**步骤**：
1. **先开 `noImplicitAny: true`**（不要一次全开 strict），只报 `any` 问题，逐个修复
2. **用 `// @ts-ignore` 临时忽略**无法立刻修复的报错，标注 TODO，不要让 CI 卡死
3. **开 `strictNullChecks: true`**，处理所有 `null/undefined` 可能导致崩溃的地方
4. **最后开完整 `strict: true`**，处理剩余报错
5. **每个 PR 只处理一个模块**，避免改动范围太大导致 review 困难

**关键原则**：渐进迁移，不要一次全改，每步都要让 CI 通过。

</details>

---

## 🌟 Senior 思维彩蛋

Senior 接手老项目第一件事，就是看 `tsconfig.json` 里 `strict` 是不是 `true`。如果是 `false`，就说明这个项目的类型安全是有水分的——很可能有大量 `any` 和未处理的 `null`，是 bug 的温床。开启 `strict` 不只是规范，是生产安全的基线。

---

## 🔗 关联算法题

- LeetCode #1 Two Sum（模块化思维：把解题逻辑封装成函数导出）
- HackerRank: 10 Days of JavaScript - Day 8 Create a Rectangle Object

---

## 📅 复习记录

- [ ] D+3 复习（2026-05-27）
- [ ] D+7 复习（2026-05-31）
- [ ] D+21 复习（2026-06-14）

## 🔗 相关笔记

- [[TS-Topic-01-基本类型标注]]
- [[TS-Topic-13-严格模式与生产级实践]]
