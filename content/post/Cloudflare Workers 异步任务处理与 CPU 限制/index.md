---
title: Cloudflare Workers 异步任务处理与 CPU 限制
description: ""
slug: 2024-06-26-Cloudflare Workers 异步任务处理与 CPU 限制
date: 2024-06-26
image: Server-Action.jpg
categories:
  - 前端
tags:
  - workers
  - CPU
  - Server-Action
---

## 问题背景

RuiTool AI 的图片生成流程是：

```
用户提交 prompt → 调用大模型 → 返回预测 ID
  → 轮询获取结果 → 下载图片 → 存到 R2 → 返回给用户
```

图片生成需要 10-30 秒，这就需要一个异步任务处理机制。

## 最初的方案：waitUntil + 后台轮询

最开始，我想用 Cloudflare Workers 的 `waitUntil` API 在后台轮询：

```typescript
// 提交任务
const predictionId = await atlasSubmitImage({ model, prompt });

// 保存任务到 DB
await db.insert(generatedImageTable).values({
  id: taskId,
  userId,
  status: IMAGE_GEN_STATUS.PROCESSING,
  predictionId,
});

// 启动后台轮询
waitUntil(
  runBackgroundPoll({ taskId, userId, predictionId })
);

// 立即返回给前端
return { taskId };
```

后台轮询函数：

```typescript
async function runBackgroundPoll({ taskId, predictionId }) {
  for (let attempt = 1; attempt <= 60; attempt++) {
    await new Promise(r => setTimeout(r, 3000));

    const prediction = await atlasCheckPrediction(predictionId);

    if (prediction.status === "completed") {
      // 下载 + 存 R2 + 更新 DB
      return;
    }
  }
}
```

## 为什么失败了

部署上线后，发现任务永远停留在 `processing` 状态。

**根本原因：Cloudflare Workers 的 `setTimeout` 不会真正 sleep。**

Workers 的运行时是 V8 引擎，不是 Node.js。在 Workers 中：

```typescript
await new Promise(r => setTimeout(r, 3000));
```

这行代码**不会等待 3 秒**。它几乎立即继续执行，导致 60 次轮询瞬间跑完，Workers 进程结束，Atlas 那边还没完成。

此外，`waitUntil` 在 Workers 中也有 CPU 时间限制：

| 计划 | CPU 限制 |
|------|---------|
| 免费 | 10ms |
| 付费 | 30s（wall clock） |
| `waitUntil` | 30s（wall clock） |

即使 `setTimeout` 能正常 sleep，30 秒的 wall clock 限制也不够轮询完 60 次 × 3 秒 = 3 分钟。

## 最终方案：前端轮询 Server Action

**核心思路：** 不在后台轮询，而是让前端每次请求时去问一下状态。

### 1. 提交任务（同步返回）

```typescript
export const generateImageAction = actionClient
  .inputSchema(generateImageSchema)
  .action(async ({ parsedInput }) => {
    // 扣积分
    await ensureBillingAccess({ userId, creditsRequired: 10 });

    // 提交到 Atlas
    const predictionId = await atlasSubmitImage({ model, prompt });

    // 存 DB（状态 = processing）
    await db.insert(generatedImageTable).values({
      id: taskId,
      userId,
      status: IMAGE_GEN_STATUS.PROCESSING,
      predictionId,
    });

    // 立即返回，不等待结果
    return { taskId };
  });
```

### 2. 前端轮询

```typescript
// 每 3 秒调用一次
useEffect(() => {
  if (!pollingTaskId) return;

  const interval = setInterval(() => {
    executePoll({ taskId: pollingTaskId });
  }, 3000);

  return () => clearInterval(interval);
}, [pollingTaskId]);
```

### 3. 状态查询 Action（关键）

```typescript
export const checkImageStatusAction = actionClient
  .inputSchema(checkImageStatusSchema)
  .action(async ({ parsedInput }) => {
    const task = await db.query.generatedImageTable.findFirst({
      where: eq(generatedImageTable.id, parsedInput.taskId),
    });

    // 已经完成或失败 → 直接返回
    if (task.status === IMAGE_GEN_STATUS.COMPLETED) {
      return { status: "completed", r2Key: task.r2Key };
    }
    if (task.status === IMAGE_GEN_STATUS.FAILED) {
      return { status: "failed" };
    }

    // 还是在 processing → 去问 Atlas 现在状态
    const prediction = await atlasCheckPrediction(task.predictionId);

    if (prediction.status === "completed") {
      // 下载图片 + 存 R2 + 更新 DB
      const imageRes = await fetch(prediction.url);
      const buffer = await imageRes.arrayBuffer();
      await env.USER_UPLOADS_BUCKET.put(r2Key, buffer);

      await db.update(generatedImageTable)
        .set({ status: "completed", r2Key })
        .where(eq(generatedImageTable.id, task.id));

      return { status: "completed", r2Key };
    }

    if (prediction.status === "failed") {
      // 退款 + 更新 DB
      await db.update(generatedImageTable)
        .set({ status: "failed" })
        .where(eq(generatedImageTable.id, task.id));

      await addUserCredits(userId, task.creditsCharged);
      return { status: "failed" };
    }

    // 还在处理中
    return { status: "processing" };
  });
```

## 用户关闭浏览器后的恢复机制

用户关闭浏览器，再打开时，怎么恢复任务状态？

**页面加载时查 DB 里有没有 processing 的任务：**

```typescript
// page.tsx (Server Component)
const processingTask = await db.query.generatedImageTable.findFirst({
  where: and(
    eq(generatedImageTable.userId, userId),
    eq(generatedImageTable.status, IMAGE_GEN_STATUS.PROCESSING),
  ),
  orderBy: (table, { desc }) => [desc(table.createdAt)],
});

// 传给客户端组件
return <ImageGenClient processingTaskId={processingTask?.id} />;
```

```typescript
// site.client.tsx
useEffect(() => {
  if (processingTaskId) {
    setPollingTaskId(processingTaskId);
  }
}, []);
```

**历史页面也自动轮询：**

```typescript
// 找到所有 processing 状态的任务
const processingIds = items
  .filter(item => item.status === IMAGE_GEN_STATUS.PROCESSING)
  .map(item => item.id);

// 每 4 秒轮询一次
useEffect(() => {
  if (processingIds.length === 0) return;
  const interval = setInterval(() => {
    for (const id of processingIds) {
      pollStatus({ taskId: id });
    }
  }, 4000);
  return () => clearInterval(interval);
}, [processingIds.length]);
```

## 总结

**方案演进：**

| 版本 | 方案 | 问题 |
|------|------|------|
| v1 | waitUntil + setTimeout | setTimeout 不 sleep |
| v2 | 前端轮询 + checkImageStatusAction | 每个请求独立，无时间限制 |

**关键设计原则：**

1. 每次轮询是一个独立的 Worker 请求，不受 CPU 时间限制
2. 用户关闭浏览器后，DB 保留任务状态，回来后可恢复
3. 历史页面自动轮询所有 processing 任务，不需要手动刷新

## 参考资源

[Cloudflare Workers waitUntil](https://developers.cloudflare.com/workers/runtime-apis/context/) 
[Next.js Server Actions](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations) 