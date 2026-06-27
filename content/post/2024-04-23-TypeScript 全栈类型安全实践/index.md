---
title: TypeScript 全栈类型安全实践
description: ""
slug: 2024-04-23-TypeScript 全栈类型安全实践
date: 2024-04-23
image: TypeScript.jpg
categories:
  - 前端
  - 编程语言
tags:
  - TypeScript
  - RuiToolAI
---

> 类型安全不只是"不报红线"。这篇聊聊在 RuiToolAI 里如何让类型从数据库一路贯穿到前端 UI。

## 为什么要全栈类型安全

没有类型安全时，一个字段改名会引发连锁问题：

```
数据库 user.email_address 改名为 user.email
    ↓
API 返回 { emailAddress: string }
    ↓
前端用 user.email 访问 → undefined
    ↓
运行时报错，用户看到空白
```

有了全栈类型安全，改名时 TypeScript 会在编译时报错，强制你修改所有引用。

---

## 数据库层：Drizzle ORM 的类型推导

Drizzle 的最大优势是类型推导。Schema 定义即类型定义：

```typescript
// src/db/schema.ts
import { sqliteTable, text, integer, real } from "drizzle-orm/sqlite-core";

export const users = sqliteTable("users", {
  id: text("id").primaryKey(),
  email: text("email").notNull().unique(),
  createdAt: integer("created_at", { mode: "timestamp" })
    .notNull()
    .$defaultFn(() => new Date()),
});

export const generatedImages = sqliteTable("generated_images", {
  id: text("id").primaryKey(),
  userId: text("user_id")
    .notNull()
    .references(() => users.id),
  prompt: text("prompt").notNull(),
  imageUrl: text("image_url"),
  status: text("status", {
    enum: ["pending", "processing", "completed", "failed"],
  })
    .notNull()
    .default("pending"),
  creditsUsed: integer("credits_used").notNull().default(1),
  createdAt: integer("created_at", { mode: "timestamp" })
    .notNull()
    .$defaultFn(() => new Date()),
});
```

从 Schema 推导类型：

```typescript
import { InferSelectModel, InferInsertModel } from "drizzle-orm";
import * as schema from "@/db/schema";

// 查询结果类型
export type User = InferSelectModel<typeof schema.users>;
export type GeneratedImage = InferSelectModel<typeof schema.generatedImages>;

// 插入类型（id 和 createdAt 是可选的，因为有默认值）
export type NewGeneratedImage = InferInsertModel<typeof schema.generatedImages>;
```

查询时类型自动推导：

```typescript
const images = await db
  .select()
  .from(schema.generatedImages)
  .where(eq(schema.generatedImages.userId, userId));

// images 的类型是 GeneratedImage[]，不需要手动标注
images[0].status; // 类型是 "pending" | "processing" | "completed" | "failed"
images[0].nonExistent; // TypeScript 报错：Property 'nonExistent' does not exist
```

---

## API 层：Zod Schema 作为契约

Zod Schema 同时承担两个职责：**运行时验证** 和 **类型定义**。

```typescript
// src/schemas/image-gen.schema.ts
import { z } from "zod";

export const generateImageSchema = z.object({
  prompt: z.string().min(1, "提示词不能为空").max(500, "提示词最多 500 字"),
  style: z.enum(["realistic", "anime", "oil-painting"]).optional(),
  size: z.enum(["512x512", "1024x1024"]).default("1024x1024"),
});

// 从 Schema 推导类型，不需要手动写 interface
export type GenerateImageInput = z.infer<typeof generateImageSchema>;
```

这样 Schema 和类型永远同步，不会出现"类型说可以，运行时报错"的情况。

---

## Server Actions：next-safe-action 的类型传递

next-safe-action 让 Server Action 的输入输出都有类型：

```typescript
// src/app/(sites)/image-gen/site.actions.ts
import "server-only";
import { actionClient } from "@/lib/safe-action";
import { generateImageSchema } from "@/schemas/image-gen.schema";

export const generateImageAction = actionClient
  .inputSchema(generateImageSchema)
  .action(async ({ parsedInput }) => {
    // parsedInput 的类型是 GenerateImageInput，自动推导
    const { prompt, style, size } = parsedInput;

    // 返回值类型也会被推导
    return { taskId: "xxx", status: "processing" as const };
  });
```

客户端调用时，输入输出都有类型检查：

```typescript
// src/app/(sites)/image-gen/site.client.tsx
"use client";
import { useAction } from "next-safe-action/hooks";
import { generateImageAction } from "./site.actions";

function ImageGenForm() {
  const { execute, result, status } = useAction(generateImageAction, {
    onSuccess: ({ data }) => {
      // data 的类型是 { taskId: string; status: "processing" }
      console.log(data.taskId); // 有类型提示
      console.log(data.nonExistent); // TypeScript 报错
    },
  });

  return (
    <button
      onClick={() =>
        execute({
          prompt: "a beautiful sunset",
          // size 有默认值，可以不传
          // nonExistent: "xxx" // TypeScript 报错
        })
      }
    >
      生成
    </button>
  );
}
```

---

## 前端：类型从服务端流向客户端

Server Component 查询数据，把结果传给 Client Component：

```typescript
// src/app/(sites)/history/page.tsx（Server Component）
import type { GeneratedImage } from "@/db/schema";

export default async function HistoryPage() {
  const items: GeneratedImage[] = await db
    .select()
    .from(schema.generatedImages)
    .where(eq(schema.generatedImages.userId, userId));

  // 类型从这里传递给客户端组件
  return <HistoryClient initialItems={items} />;
}
```

```typescript
// src/app/(sites)/history/client.tsx（Client Component）
"use client";
import type { GeneratedImage } from "@/db/schema";

interface HistoryClientProps {
  initialItems: GeneratedImage[];
}

export function HistoryClient({ initialItems }: HistoryClientProps) {
  const [items, setItems] = useState<GeneratedImage[]>(initialItems);

  // items[0].status 的类型是 "pending" | "processing" | "completed" | "failed"
  // 不是 string，可以做精确的条件判断
  const processingItems = items.filter(
    (item) => item.status === "processing" || item.status === "pending"
  );

  return <div>{/* 渲染 */}</div>;
}
```

---

## 环境变量类型安全

环境变量默认是 `string | undefined`，用之前需要检查。更好的做法是用 Zod 在启动时验证：

```typescript
// src/env.ts
import { z } from "zod";

const envSchema = z.object({
  // 服务端环境变量
  MODEL_API_KEY: z.string().min(1),
  STRIPE_SECRET_KEY: z.string().startsWith("sk_"),
  STRIPE_WEBHOOK_SECRET: z.string().startsWith("whsec_"),

  // 客户端环境变量（NEXT_PUBLIC_ 前缀）
  NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY: z.string().startsWith("pk_"),
});

// 启动时验证，缺少必要变量直接报错
export const env = envSchema.parse(process.env);
```

```typescript
// 使用时有类型，不需要 ! 断言
import { env } from "@/env";

const stripe = new Stripe(env.STRIPE_SECRET_KEY); // string，不是 string | undefined
```

---

## Cloudflare Bindings 类型生成

Cloudflare Workers 的 Bindings（D1、KV、R2）需要类型定义。不要手写，用 `cf-typegen` 自动生成：

```bash
pnpm run cf-typegen
```

这会根据 `wrangler.jsonc` 生成 `worker-configuration.d.ts`：

```typescript
// worker-configuration.d.ts（自动生成，不要手动编辑）
interface Env {
  NEXT_TAG_CACHE_D1: D1Database;
  NEXT_TAG_CACHE_KV: KVNamespace;
  IMAGES_R2: R2Bucket;
}
```

在代码里使用时有完整的类型提示：

```typescript
import { getCloudflareContext } from "@opennextjs/cloudflare";

const { env } = await getCloudflareContext();

// env.NEXT_TAG_CACHE_D1 的类型是 D1Database
// env.nonExistent 会报错
const db = drizzle(env.NEXT_TAG_CACHE_D1, { schema });
```

---

## 常用类型工具

### 从联合类型中提取特定成员

```typescript
type Status = "pending" | "processing" | "completed" | "failed";
type ActiveStatus = Extract<Status, "pending" | "processing">;
// 等价于 "pending" | "processing"
```

### 让某些字段可选

```typescript
type GeneratedImage = {
  id: string;
  prompt: string;
  imageUrl: string | null;
  status: Status;
};

// 创建时 id 由数据库生成，不需要传
type CreateImageInput = Omit<GeneratedImage, "id">;

// 更新时所有字段都是可选的
type UpdateImageInput = Partial<Pick<GeneratedImage, "imageUrl" | "status">>;
```

### 函数参数用命名对象

```typescript
// ❌ 参数多了容易搞混顺序
function updateImage(id: string, url: string, status: Status) {}

// ✅ 命名对象，顺序无关，可读性好
function updateImage({
  id,
  url,
  status,
}: {
  id: string;
  url: string;
  status: Status;
}) {}
```

### 类型守卫

```typescript
function isCompleted(
  image: GeneratedImage
): image is GeneratedImage & { imageUrl: string } {
  return image.status === "completed" && image.imageUrl !== null;
}

// 使用
if (isCompleted(image)) {
  // 这里 image.imageUrl 的类型是 string，不是 string | null
  console.log(image.imageUrl.toUpperCase());
}
```

---

## 总结

全栈类型安全的关键链路：

```
Drizzle Schema → InferSelectModel → 查询结果类型
    ↓
Server Component → props → Client Component
    ↓
Zod Schema → infer → Server Action 输入类型
    ↓
next-safe-action → useAction → 客户端调用类型
```

核心原则：

1. **Schema 即类型**：用 Drizzle 和 Zod 推导类型，不手写 interface
2. **类型向下流动**：从数据库 → 服务端 → 客户端，类型一路传递
3. **边界验证**：在系统边界（用户输入、外部 API）用 Zod 验证
4. **自动生成**：Cloudflare Bindings 类型用 `cf-typegen` 生成，不手写

---

## 参考资源

- [Drizzle ORM 类型文档](https://orm.drizzle.team/docs/column-types/sqlite)
- [Zod 文档](https://zod.dev/)
- [next-safe-action 文档](https://next-safe-action.dev/)
- [TypeScript Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html)
- [Cloudflare Workers TypeScript](https://developers.cloudflare.com/workers/languages/typescript/)

