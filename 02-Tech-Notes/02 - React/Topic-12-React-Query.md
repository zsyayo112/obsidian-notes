---
tier: Tier-1
tech: React
topic: React Query
knowledge-point: 服务端状态、useQuery、useMutation、缓存、自动刷新
difficulty: ⭐⭐⭐
status: done
created: 2026-05-25
tags:
  - tech/react
  - tier-1
  - react-query
  - tanstack-query
  - server-state
---

# 📌 Topic 12：React Query — 服务端状态管理

> Topic: React Tier 1 · Tier: 1 · 难度: ⭐⭐⭐

---

## 📖 0. 术语扫盲

| 英文                | 中文    | 一句话解释                          |
| ----------------- | ----- | ------------------------------ |
| Server State      | 服务端状态 | 存在后端数据库里的数据，需要发请求才能拿到          |
| Client State      | 客户端状态 | 存在前端的数据，如 isOpen、theme         |
| queryKey          | 查询键   | 请求的唯一标识，像缓存的 key，同样的 key 不重复请求 |
| queryFn           | 查询函数  | 真正发请求的函数                       |
| staleTime         | 过期时间  | 数据多久后变"过期"，过期才重新请求             |
| invalidateQueries | 使缓存失效 | 告诉 React Query 某个数据过期了，重新请求    |
| Mutation          | 变更    | 改变数据的操作（POST/PUT/DELETE）       |
| Optimistic Update | 乐观更新  | 还没等后端确认，先在页面上显示结果              |

---

## 📝 1. 实例代码

```tsx
import {
  QueryClient, QueryClientProvider,
  useQuery, useMutation, useQueryClient
} from '@tanstack/react-query';

// ============ 全局配置 ============
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5,  // 5分钟内不过期
      retry: 3,                   // 失败自动重试 3 次
      refetchOnWindowFocus: true, // 切回页面自动刷新
    },
  },
});

// ============ 入口：包住所有组件 ============
function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <TodoApp />
    </QueryClientProvider>
  );
}

// ============ useQuery：读数据 ============
function TodoApp() {
  const { data: todos, isLoading, error, isFetching } = useQuery({
    queryKey: ['todos'],          // 唯一标识
    queryFn: async () => {
      const res = await fetch('https://jsonplaceholder.typicode.com/todos?_limit=5');
      if (!res.ok) throw new Error('请求失败');
      return res.json();
    },
    staleTime: 1000 * 30,        // 30秒内不重新请求
  });

  // ============ useMutation：写数据 ============
  const qc = useQueryClient();

  const deleteMutation = useMutation({
    mutationFn: async (id) => {
      await fetch(`https://jsonplaceholder.typicode.com/todos/${id}`, {
        method: 'DELETE',
      });
    },
    onSuccess: () => {
      // 删除成功 → 让缓存失效 → 自动重新拉数据
      qc.invalidateQueries({ queryKey: ['todos'] });
    },
  });

  // ============ 状态处理 ============
  if (isLoading) return <p>加载中...</p>;
  if (error) return <p>{error.message}</p>;

  return (
    <div>
      {isFetching && <p>🔄 后台刷新中...</p>}
      {todos.map(todo => (
        <div key={todo.id}>
          <span>{todo.title}</span>
          <button
            onClick={() => deleteMutation.mutate(todo.id)}
            disabled={deleteMutation.isPending}
          >
            {deleteMutation.isPending ? '删除中...' : '删除'}
          </button>
        </div>
      ))}
    </div>
  );
}

export default App;
```

⚠️ 易错点：
1. 忘记用 `QueryClientProvider` 包住，useQuery 报错"No QueryClient set"
2. queryKey 设计不对：同样数据不同 key → 重复请求；不同数据同样 key → 数据混乱
3. useMutation 成功后忘记 `invalidateQueries`，页面显示旧数据
4. `isLoading` 和 `isFetching` 区别：isLoading 只在第一次（没缓存）时 true，isFetching 任何请求中都 true

---

## 🎯 2. 使用场景

**useQuery（读）：**
1. **列表页（Seek.com.au）**：`queryKey: ['jobs']`，自动缓存，用户返回不重新 loading
2. **详情页（依赖参数）**：`queryKey: ['job', jobId]`，jobId 变了自动重新请求
3. **搜索（依赖搜索词）**：`queryKey: ['jobs', { search: debouncedQuery }]`，搜索词变了重新搜索

**useMutation（写）：**
1. **新增**：POST 请求，成功后 invalidate 列表
2. **删除**：DELETE 请求，成功后 invalidate 列表
3. **更新**：PUT/PATCH 请求，成功后 invalidate 详情和列表

---

## 🔄 3. 可替代技术

| 替代方案 | 优势 | 劣势 | 何时用 |
|---|---|---|---|
| 手写 useEffect | 无额外依赖 | 要手写 loading/error/缓存/重试 | 简单一次性请求 |
| SWR | 轻量 | 功能比 React Query 少 | 简单的数据请求 |
| Apollo Client | GraphQL 专用，功能强 | 只适合 GraphQL | 用 GraphQL API 时 |

**选型决策树**：
- 从后端拿数据 → React Query（强烈推荐）
- 简单一次性请求，不需要缓存 → useEffect 手写
- GraphQL API → Apollo Client
- 改数据（POST/PUT/DELETE）→ useMutation

---

## 💼 4. 面试题

### 题 1（🟡 进阶）

**Q**: useQuery 的 queryKey 有什么作用？

**核心 points**:
- queryKey 是缓存的唯一标识
- 同样的 queryKey → 直接用缓存，不重新请求
- queryKey 变了 → 自动重新请求
- 两个组件用同样的 queryKey → 只发一次请求（自动去重）

**追问方向**:
1. queryKey 为什么用数组？
2. 两个组件用同样的 queryKey 会怎样？

### 题 2（🔴 高级）

**Q**: useQuery 和 useEffect 发请求有什么区别？

**加分回答**: "useEffect 发请求需要手写 loading/error state、缓存、竞态条件处理、重试逻辑。useQuery 把这些全部内置了，还多了缓存、自动刷新、窗口聚焦重新请求、请求去重等功能。代码量少一半，bug 也少很多。在真实项目里，凡是从后端拿数据，我都会用 React Query 而不是手写 useEffect。"

---

## 🧪 5. 小测验

**staleTime 理解测验**：

```
staleTime: 1000 * 10  // 10秒

第一次访问 → 发请求 → 数据回来
10秒内第二次访问 → ???
超过10秒第三次访问 → ???
```

> 点击展开答案前先思考

<details>
<summary>展开答案</summary>

**10秒内第二次访问**：
直接用缓存，不发请求，页面瞬间显示 ✅

**超过10秒第三次访问**：
先用缓存显示（不会 loading！），后台悄悄发新请求，新数据回来更新页面 ✅

这就是 stale-while-revalidate 策略：先显示旧数据，后台更新，用户体验好。

**isLoading vs isFetching**：
- isLoading：第三次访问时是 false（有缓存）
- isFetching：第三次访问时是 true（后台在请求）
</details>

---

## 🌟 Senior 思维彩蛋

**`staleTime: 0` 是 React Query 的默认值——数据拿到就立刻过期。** 这意味着每次窗口聚焦都会重新请求。对大部分应用来说太频繁了，Senior 会根据数据的实时性要求调整：用户信息 5 分钟，股价 30 秒，静态配置 Infinity。合理设置 `staleTime` 是 React Query 性能优化的第一步。

---

## 📅 复习记录

- [ ] D+3 复习
- [ ] D+7 复习
- [ ] D+21 复习

## 🔗 相关笔记

- [[Topic-06-useEffect]]
- [[Topic-09-自定义Hook]]
