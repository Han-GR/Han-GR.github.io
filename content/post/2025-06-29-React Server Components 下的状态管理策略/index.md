---
title: React Server Components 下的状态管理策略
description: ""
slug: 2025-06-29-React Server Components 下的状态管理策略
date: 2025-06-29
image: rsc.png
categories:
  - 前端
tags:
  - React
  - RSC
  - Zustand
  - nuqs
  - RuiToolAI
---

> 用 RSC 之后，我扔掉了大半个 Redux。这篇聊聊在 Server Components 主导的架构里，状态到底该放哪。

## 背景：RSC 改变了什么

传统 React SPA 里，所有数据都要先 fetch 到客户端，再用状态管理库（Redux、Zustand、React Query）缓存和分发。

RSC 出现后，服务端组件可以直接 `await` 数据库查询，数据天然在服务端，根本不需要传到客户端再管理。

```
传统 SPA：
Browser → API → Redux Store → Component

RSC 架构：
Server Component → DB → HTML → Browser
```

这意味着大量"服务端数据的客户端缓存"需求直接消失了。

---

## 状态分类模型

在 RuiToolAI 项目里，我把状态分成四类：

| 类型     | 存放位置              | 工具              |
| ------ | ----------------- | --------------- |
| 服务端数据  | Server Component  | 直接 await        |
| URL 参数 | URL search params | nuqs            |
| 客户端全局  | 内存                | Zustand         |
| 表单     | 组件局部              | react-hook-form |

原则很简单：**能放服务端就放服务端，能放 URL 就放 URL，实在不行才用 Zustand。**

---

## 服务端状态：直接读，不缓存

Server Component 里直接查数据库，不需要任何状态管理库。

```typescript
// src/app/(sites)/history/page.tsx
import { getCloudflareContext } from "@opennextjs/cloudflare";
import { drizzle } from "drizzle-orm/d1";
import * as schema from "@/db/schema";
import { eq, desc } from "drizzle-orm";

export default async function HistoryPage() {
  const { env } = await getCloudflareContext();
  const db = drizzle(env.NEXT_TAG_CACHE_D1, { schema });

  // 直接查，不需要 useEffect、不需要 loading state
  const session = await getSessionFromCookie();
  const items = await db
    .select()
    .from(schema.generatedImages)
    .where(eq(schema.generatedImages.userId, session.user.id))
    .orderBy(desc(schema.generatedImages.createdAt))
    .limit(50);

  return <HistoryClient initialItems={items} />;
}
```

**关键点**：数据作为 `initialItems` prop 传给客户端组件，客户端组件只负责交互逻辑（删除、轮询状态更新）。

### 何时需要 revalidate

Server Component 的数据在请求时是新鲜的，但如果用了 Next.js 缓存，需要手动 revalidate：

```typescript
// Server Action 里操作完数据后
import { revalidatePath } from "next/cache";

export const deleteImageAction = actionClient
  .inputSchema(deleteImageSchema)
  .action(async ({ parsedInput }) => {
    // ... 删除逻辑
    revalidatePath("/history");
  });
```

---

## URL 状态：nuqs 管理搜索参数

URL 状态的好处：刷新不丢失、可分享、浏览器前进后退正常工作。

在 RuiToolAI 里，管理后台的筛选条件就放在 URL 里：

```typescript
// src/app/(admin)/admin/users/page.tsx
import { parseAsString, parseAsInteger, useQueryStates } from "nuqs";

// Server Component 读取 URL 参数
export default async function UsersPage({
  searchParams,
}: {
  searchParams: { page?: string; search?: string };
}) {
  const page = Number(searchParams.page ?? 1);
  const search = searchParams.search ?? "";

  const users = await queryUsers({ page, search });
  return <UsersTable users={users} />;
}
```

```typescript
// Client Component 更新 URL 参数
"use client";
import { useQueryState } from "nuqs";

function SearchInput() {
  const [search, setSearch] = useQueryState("search", {
    defaultValue: "",
    shallow: false, // 触发服务端重新渲染
  });

  return (
    <input
      value={search}
      onChange={(e) => setSearch(e.target.value)}
      placeholder="搜索用户..."
    />
  );
}
```

`shallow: false` 会触发完整的服务端重新渲染，数据自动更新，不需要任何额外的状态管理。

---

## 客户端全局状态：Zustand 最小化

Zustand 只用于真正需要跨组件共享的客户端状态。在这个项目里主要是两个：

### Session Store

```typescript
// src/state/session.ts
import { create } from "zustand";

interface SessionStore {
  user: User | null;
  credits: number;
  setUser: (user: User | null) => void;
  setCredits: (credits: number) => void;
}

export const useSessionStore = create<SessionStore>((set) => ({
  user: null,
  credits: 0,
  setUser: (user) => set({ user }),
  setCredits: (credits) => set({ credits }),
}));
```

Session 信息在 Layout 里初始化，之后客户端组件直接读取，不需要每次都请求服务端：

```typescript
// src/components/session-provider.tsx
"use client";
import { useEffect } from "react";
import { useSessionStore } from "@/state/session";

export function SessionProvider({
  user,
  credits,
}: {
  user: User;
  credits: number;
}) {
  const { setUser, setCredits } = useSessionStore();

  useEffect(() => {
    setUser(user);
    setCredits(credits);
  }, [user, credits]);

  return null;
}
```

### 生成任务状态

多任务并发时，需要在生成页面和历史页面之间共享"进行中的任务数量"：

```typescript
// src/state/generation.ts
import { create } from "zustand";

interface GenerationStore {
  pendingCount: number;
  increment: () => void;
  decrement: () => void;
}

export const useGenerationStore = create<GenerationStore>((set) => ({
  pendingCount: 0,
  increment: () => set((s) => ({ pendingCount: s.pendingCount + 1 })),
  decrement: () =>
    set((s) => ({ pendingCount: Math.max(0, s.pendingCount - 1) })),
}));
```

---

## 表单状态：react-hook-form 局部管理

表单状态是最局部的，用 react-hook-form 管理，不需要提升到全局：

```typescript
// src/app/(auth)/sign-up/sign-up.client.tsx
"use client";
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { useAction } from "next-safe-action/hooks";
import { signUpSchema, type SignUpSchema } from "@/schemas/signup.schema";
import { signUpAction } from "./sign-up.actions";

export function SignUpForm() {
  const form = useForm<SignUpSchema>({
    resolver: zodResolver(signUpSchema),
    defaultValues: { email: "", password: "" },
  });

  const { execute, status } = useAction(signUpAction, {
    onSuccess: () => {
      toast.success("注册成功！");
    },
    onError: ({ error }) => {
      toast.error(error.serverError ?? "注册失败");
    },
  });

  return (
    <form onSubmit={form.handleSubmit(execute)}>
      {/* 表单字段 */}
    </form>
  );
}
```

---

## 实战：生成历史页面的状态设计

历史页面是这个项目里状态最复杂的页面，综合用了所有四种状态：

```
服务端状态（initialItems）
    ↓ 传给客户端
客户端局部状态（items: useState）
    ↓ 轮询更新
Server Action（checkImageStatusAction）
    ↓ 完成后
revalidatePath（可选，刷新服务端缓存）
```

```typescript
// src/app/(sites)/history/client.tsx
"use client";

export function HistoryClient({
  initialItems,
}: {
  initialItems: GeneratedImage[];
}) {
  // 客户端局部状态：从服务端初始值开始
  const [items, setItems] = useState(initialItems);
  const [failedExpanded, setFailedExpanded] = useState(false);

  // 找出所有 processing 的任务
  const processingIds = items
    .filter((item) => item.status === "processing" || item.status === "pending")
    .map((item) => item.id);

  // 轮询逻辑
  const pollAll = useCallback(async () => {
    for (const id of processingIds) {
      const result = await checkImageStatusAction({ id });
      if (result?.data) {
        setItems((prev) =>
          prev.map((item) => (item.id === id ? result.data! : item))
        );
      }
    }
  }, [processingIds]);

  useEffect(() => {
    if (processingIds.length === 0) return;
    pollAll();
    const interval = setInterval(pollAll, 4000);
    // tab 切换回来立即轮询
    const handleVisibility = () => {
      if (document.visibilityState === "visible") pollAll();
    };
    document.addEventListener("visibilitychange", handleVisibility);
    return () => {
      clearInterval(interval);
      document.removeEventListener("visibilitychange", handleVisibility);
    };
  }, [processingIds.length, pollAll]);

  // 删除
  const { execute: deleteImage } = useAction(deleteGeneratedImageAction, {
    onSuccess: ({ input }) => {
      setItems((prev) => prev.filter((item) => item.id !== input.id));
    },
  });

  // 渲染...
}
```

---

## 常见误区

### 误区 1：在 Server Component 里用 useState

```typescript
// 错误：Server Component 不能用 hooks
export default async function Page() {
  const [data, setData] = useState(null); // 报错
}

// 正确：直接 await
export default async function Page() {
  const data = await fetchData();
}
```

### 误区 2：把服务端数据放进 Zustand

```typescript
// 不必要：用 Zustand 缓存服务端数据
const useUserStore = create((set) => ({
  users: [],
  fetchUsers: async () => {
    const users = await fetch("/api/users").then((r) => r.json());
    set({ users });
  },
}));

// 直接在 Server Component 查询
export default async function UsersPage() {
  const users = await db.select().from(schema.users);
  return <UsersList users={users} />;
}
```

### 误区 3：URL 状态用 useState 管理

```typescript
// 刷新后丢失
const [filter, setFilter] = useState("all");

// 放 URL 里，刷新不丢失，可分享
const [filter, setFilter] = useQueryState("filter", {
  defaultValue: "all",
});
```

---

## 总结

RSC 架构下的状态管理原则：

1. **服务端数据** → Server Component 直接 await，不需要任何状态库
2. **URL 参数** → nuqs，刷新不丢失，支持分享
3. **客户端全局** → Zustand，只放真正需要跨组件共享的少量状态
4. **表单** → react-hook-form，局部管理

这套分层策略让代码更简单，性能更好，也更容易维护。

---

## 参考资源

- [Next.js Server Components 文档](https://nextjs.org/docs/app/building-your-application/rendering/server-components)
- [nuqs 文档](https://nuqs.47ng.com/)
- [Zustand 文档](https://zustand-demo.pmnd.rs/)
- [react-hook-form 文档](https://react-hook-form.com/)

