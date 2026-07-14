---
tier: Tier-1
tech: React
topic: useReducer
knowledge-point: 复杂状态管理、action、reducer、dispatch
difficulty: ⭐⭐⭐
status: done
created: 2026-05-25
tags:
  - tech/react
  - tier-1
  - useReducer
  - state-management
---

# 📌 Topic 11：useReducer — 管理复杂状态

> Topic: React Tier 1 · Tier: 1 · 难度: ⭐⭐⭐

---

## 📖 0. 术语扫盲

| 英文 | 中文 | 一句话解释 |
|---|---|---|
| Reducer | 归纳函数 | 接收当前状态和动作，返回新状态的纯函数 |
| Action | 动作 | 描述"发生了什么"的对象，必须有 type 字段 |
| Dispatch | 派发 | 发送一个 action，触发 reducer 更新状态 |
| Payload | 载荷 | action 里携带的数据，比如 `{ id: 1, name: '苹果' }` |
| Pure Function | 纯函数 | 同样输入永远返回同样输出，不改变外部数据 |
| Immutable | 不可变 | 不直接改原数据，返回新的数据 |

---

## 📝 1. 实例代码

```tsx
import { useReducer } from 'react';

// ============ 初始状态 ============
const initialState = {
  values: {
    name: '',
    email: '',
    password: '',
  },
  errors: {
    name: '',
    email: '',
    password: '',
  },
  isSubmitting: false,
};

// ============ Reducer 函数 ============
// 规则：纯函数，不能有副作用，不能直接改 state
function formReducer(state, action) {
  switch (action.type) {
    case 'SET_FIELD':
      return {
        ...state,
        values: {
          ...state.values,
          [action.field]: action.value, // 动态字段名！
        },
        errors: {
          ...state.errors,
          [action.field]: '', // 输入时清掉错误
        },
      };

    case 'SET_ERROR':
      return {
        ...state,
        errors: {
          ...state.errors,
          [action.field]: action.message,
        },
      };

    case 'SUBMIT_START':
      return { ...state, isSubmitting: true };

    case 'SUBMIT_END':
      return { ...state, isSubmitting: false };

    case 'RESET':
      return initialState;

    default:
      return state; // 不认识的 action 原样返回
  }
}

// ============ 组件 ============
function RegisterForm() {
  const [state, dispatch] = useReducer(formReducer, initialState);

  function validate() {
    let isValid = true;
    if (!state.values.name) {
      dispatch({ type: 'SET_ERROR', field: 'name', message: '姓名不能为空' });
      isValid = false;
    }
    if (!state.values.email.includes('@')) {
      dispatch({ type: 'SET_ERROR', field: 'email', message: '邮箱格式不对' });
      isValid = false;
    }
    return isValid;
  }

  async function handleSubmit() {
    if (!validate()) return;
    dispatch({ type: 'SUBMIT_START' });
    await new Promise(resolve => setTimeout(resolve, 1000));
    dispatch({ type: 'SUBMIT_END' });
    dispatch({ type: 'RESET' });
  }

  return (
    <div>
      <input
        value={state.values.name}
        onChange={e => dispatch({
          type: 'SET_FIELD',
          field: 'name',      // 告诉 reducer 改哪个字段
          value: e.target.value,
        })}
      />
      {state.errors.name && <p>{state.errors.name}</p>}

      <button onClick={handleSubmit} disabled={state.isSubmitting}>
        {state.isSubmitting ? '提交中...' : '注册'}
      </button>
    </div>
  );
}
```

⚠️ 易错点：
1. reducer 里直接改 state（`state.items.push(...)`）React 检测不到变化，必须返回新对象
2. 忘记 `default` 返回 state，不认识的 action 返回 undefined 导致崩溃
3. `[action.field]` 是动态字段名（JS 计算属性），不是数组，`action.field = 'name'` 就等于 `name: value`
4. reducer 里不能有副作用（不能发请求、改外部变量、用 Date.now()）

---

## 🎯 2. 使用场景

1. **复杂表单（注册/结账）**：多字段、验证逻辑、提交状态，用 useReducer 集中管理比 8 个 useState 清晰很多
2. **购物车**：加入、移除、修改数量、打折等多种操作，所有逻辑在 reducer 里
3. **多步骤向导（Wizard）**：上一步/下一步、跳转、验证，状态转换逻辑复杂

---

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| useState | 简单，代码少 | 字段多了很啰嗦 | 1-3 个简单状态 |
| Zustand | 全局状态，API 更简洁 | 额外依赖 | 多组件共享复杂状态 |
| React Hook Form | 专门针对表单，性能好 | 额外依赖 | 复杂表单（推荐） |

**选型决策树**：
- 1-3 个简单独立状态 → useState
- 多个相关字段 + 多种操作 → useReducer
- 表单（字段多）→ React Hook Form（生产）或 useReducer（学习）
- 多组件共享 → Zustand

---

## 💼 4. 面试题

### 题 1（🟡 进阶）

**Q**: useReducer 和 useState 有什么区别？什么时候用 useReducer？

**核心 points**:
- useState 直接设值，useReducer 通过 action 描述变化
- useReducer 把逻辑集中在 reducer，更容易维护和测试
- 复杂状态、多种操作时用 useReducer

**追问方向**:
1. reducer 函数有什么要求？
2. dispatch 的 action 必须有 type 吗？

**加分回答**: "useReducer 的 reducer 必须是纯函数——同样的输入永远返回同样的输出，不能有副作用。这让逻辑变得可预测、可测试。Redux 的核心思想就是这个，useReducer 是 React 内置的轻量版 Redux。"

---

## 🧪 5. 小测验

实现购物车的 `DECREASE_QUANTITY` case：当数量减到 0 时，自动从购物车移除：

```tsx
case 'DECREASE_QUANTITY':
  return {
    ...state,
    items: state.items
      // 👇 第一步：数量 -1
      .map(i => ???)
      // 👇 第二步：过滤掉数量为 0 的
      .filter(i => ???),
  };
```

> 点击展开答案前先思考

<details>
<summary>展开答案</summary>

```tsx
case 'DECREASE_QUANTITY':
  return {
    ...state,
    items: state.items
      .map(i =>
        i.id === action.payload.id
          ? { ...i, quantity: i.quantity - 1 }
          : i
      )
      .filter(i => i.quantity > 0), // 数量变 0 就移除
  };
```

两步合一：map 先 -1，filter 再过滤掉 quantity <= 0 的。
</details>

---

## 🌟 Senior 思维彩蛋

**`[action.field]` 动态字段名是表单 useReducer 的精髓。** 有了它，不管表单有多少字段，`SET_FIELD` 这一个 case 就能处理所有输入，不需要为每个字段写单独的 case。这是 JS 的计算属性名（Computed Property Name），区分初级和中级工程师的一个细节。

---

## 📅 复习记录

- [ ] D+3 复习
- [ ] D+7 复习
- [ ] D+21 复习

## 🔗 相关笔记

- [[Topic-03-useState]]
- [[Topic-08-useContext]]
