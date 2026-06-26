---
title: Next.js Server Actions 最佳实践
description: ""
slug: 2024-08-13-Next.js Server Actions 最佳实践
date: 2024-08-13
image: Server-Action.jpg
categories:
  - 前端
tags:
  - Server-Action
---
## 为什么用 Server Actions 而不是 API Route

| 对比项 | Server Actions | API Route |
|--------|---------------|-----------|
| 类型安全 | 全栈类型推断 | 需要手动定义 |
| 调用方式 | 直接调用函数 | fetch + JSON |
| 错误处理 | 统一封装 | 每个 route 单独处理 |
| 认证 | 在 action 内部处理 | 在 route 内部处理 |
| 代码量 | 少 | 多 |

Server Actions 的最大优势是**类型安全**：前端调用时，TypeScript 直接知道返回值的类型，不需要手动定义接口。

## next-safe-action 封装

直接用 Next.js 原生 Server Actions 缺少统一的错误处理和输入校验。`next-safe-action` 提供了一个更好的封装：

```typescript
// src/lib/safe-action.ts
import { createSafeActionClient } from "next-safe-action";

export const actionClient = createSafeActionClient({
  handleServerError(error) {
    if (error instanceof ActionError) {
      return {
        code: error.code,
        message: error.message,
        details: error.details,
      };
    }
    console.error("Unexpected error:", error);
    return {
      code: "INTERNAL_SERVER_ERROR",
      message: "An unexpected error occurred",
    };
  },
});
```

## 输入校验

所有 Server Actions 都用 Zod 做输入校验：

```typescript
// src/schemas/image-gen.schema.ts
import { z } from "zod";

export const generateImageSchema = z.object({
  prompt: z
    .string()
    .min(3, "Prompt must be at least 3 characters")
    .max(1000, "Prompt must be at most 1000 characters"),
});

export type GenerateImageSchema = z.infer<typeof generateImageSchema>;
```

```typescript
// src/sites/image-gen/site.actions.ts
export const generateImageAction = actionClient
  .inputSchema(generateImageSchema)
  .action(async ({ parsedInput }) => {
    // parsedInput 已经过 Zod 校验，类型安全
    const { prompt } = parsedInput;
    // ...
  });
```

## 错误处理

统一的错误类型：

```typescript
// src/lib/action-error.ts
export class ActionError extends Error {
  constructor(
    public code: string,
    message: string,
    public details?: unknown
  ) {
    super(message);
  }
}
```

在 action 中抛出：

```typescript
export const generateImageAction = actionClient
  .inputSchema(generateImageSchema)
  .action(async ({ parsedInput }) => {
    const session = await requireVerifiedEmail();
    if (!session) {
      throw new ActionError("NOT_AUTHORIZED", "You must be signed in");
    }

    const hasCredits = await checkCredits(session.user.id, 10);
    if (!hasCredits) {
      throw new ActionError("PAYMENT_REQUIRED", "Insufficient credits", {
        billingPath: "/dashboard/billing",
      });
    }
    // ...
  });
```

## 客户端调用

```typescript
"use client";

import { useAction } from "next-safe-action/hooks";
import { generateImageAction } from "./site.actions";

export function ImageGenClient() {
  const { execute, status } = useAction(generateImageAction, {
    onSuccess: ({ data }) => {
      toast.success("Generation started!");
    },
    onError: ({ error }) => {
      const serverError = error.serverError;

      if (serverError?.code === "PAYMENT_REQUIRED") {
        toast.error(serverError.message);
        const billingPath = serverError.details?.billingPath;
        if (billingPath) router.push(billingPath);
        return;
      }

      toast.error(serverError?.message ?? "Something went wrong");
    },
  });

  return (
    <Button
      onClick={() => execute({ prompt })}
      disabled={status === "executing"}
    >
      {status === "executing" ? "Generating..." : "Generate"}
    </Button>
  );
}
```

## AI 任务轮询模式

AI 生成任务需要轮询状态，这是一个常见的异步模式：

### 提交任务

```typescript
export const generateImageAction = actionClient
  .inputSchema(generateImageSchema)
  .action(async ({ parsedInput }) => {
    // 扣积分 + 提交到 Atlas + 存 DB
    return { taskId };
  });
```

### 轮询状态

```typescript
export const checkImageStatusAction = actionClient
  .inputSchema(z.object({ taskId: z.string() }))
  .action(async ({ parsedInput }) => {
    const task = await db.query.generatedImageTable.findFirst({
      where: eq(generatedImageTable.id, parsedInput.taskId),
    });

    if (task.status === "completed") {
      return { status: "completed", r2Key: task.r2Key };
    }

    if (task.status === "processing") {
      // 去问 Atlas 最新状态
      const prediction = await atlasCheckPrediction(task.predictionId);
      if (prediction.status === "completed") {
        // 存 R2 + 更新 DB
        return { status: "completed", r2Key };
      }
    }

    return { status: task.status };
  });
```

### 前端轮询

```typescript
const { execute: pollStatus } = useAction(checkImageStatusAction, {
  onSuccess: ({ data }) => {
    if (data?.status === "completed") {
      setResult(data);
      clearInterval(intervalRef.current);
    }
  },
});

useEffect(() => {
  if (!taskId) return;
  const interval = setInterval(() => {
    pollStatus({ taskId });
  }, 3000);
  return () => clearInterval(interval);
}, [taskId]);
```

## 总结

Server Actions 的最佳实践：

1. 用 `next-safe-action` 统一封装，避免重复的错误处理代码
2. 所有输入用 Zod 校验，Schema 前后端共享
3. 用 `ActionError` 统一错误类型，前端可以根据 `code` 做不同处理
4. 异步任务用"提交 + 轮询"模式，不要在 action 里等待结果

## 参考资源

- [next-safe-action](https://next-safe-action.dev/)  
- [Zod](https://zod.dev/) 
- [Next.js Server Actions](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations)