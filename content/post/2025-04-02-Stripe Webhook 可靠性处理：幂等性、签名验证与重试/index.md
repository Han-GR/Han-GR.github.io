---
title: Stripe Webhook 可靠性处理：幂等性、签名验证与重试
description: ""
slug: 2025-04-02-Stripe Webhook 可靠性处理：幂等性、签名验证与重试
date: 2025-04-02
image: stripe.webp
categories:
  - 效率&工具
tags:
  - Stripe
  - RuiToolAI
---

> Webhook 回调是支付系统的命脉。处理不好，用户付了钱却没到账，就是生产事故。

## Webhook 的三大挑战

1. **安全性**：如何确认请求确实来自 Stripe，而不是伪造的？
2. **幂等性**：Stripe 可能重发同一个事件，如何处理重复？
3. **可靠性**：处理失败时如何保证不丢事件？

## 签名验证

每个 Webhook 请求都带有 Stripe 的签名，用 `stripe.webhooks.constructEvent` 验证：

```typescript
// src/app/api/webhooks/stripe/route.ts

export async function POST(request: Request) {
  const body = await request.text();
  const signature = request.headers.get("stripe-signature");

  if (!signature) {
    return new Response("Missing stripe-signature header", { status: 400 });
  }

  try {
    const stripe = getStripe();
    const event = stripe.webhooks.constructEvent(
      body,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET!
    );

    // 处理事件
    await handleStripeEvent(event);

    return new Response(null, { status: 200 });
  } catch (error) {
    console.error("Webhook error:", error);
    return new Response("Webhook error", { status: 400 });
  }
}
```

签名验证失败直接返回 400，Stripe 会重试。

## 幂等性处理

Stripe 的事件对象有唯一的 `id`，可以用来做幂等：

```typescript
async function handleStripeEvent(event: Stripe.Event) {
  const db = getDB();

  // 检查是否已处理过
  const existing = await db.query.stripeEventTable.findFirst({
    where: eq(stripeEventTable.stripeEventId, event.id),
  });

  if (existing) {
    console.log(`Event ${event.id} already processed, skipping`);
    return;
  }

  // 处理事件
  switch (event.type) {
    case "checkout.session.completed":
      await handleCheckoutCompleted(event.data.object);
      break;
    case "customer.subscription.updated":
      await handleSubscriptionUpdated(event.data.object);
      break;
    case "customer.subscription.deleted":
      await handleSubscriptionDeleted(event.data.object);
      break;
    case "invoice.paid":
      await handleInvoicePaid(event.data.object);
      break;
  }

  // 记录已处理
  await db.insert(stripeEventTable).values({
    stripeEventId: event.id,
    type: event.type,
    processedAt: new Date(),
  });
}
```

关键点：
- 处理前先查 `stripeEventId` 是否已存在
- 处理后立即记录，防止重复处理
- 处理和记录不是原子操作（D1 不支持事务），但 Stripe 的重试间隔足够长，实际不会出问题

## 事件类型路由

Stripe Webhook 需要处理的事件类型：

| 事件 | 处理逻辑 |
|------|----------|
| `checkout.session.completed` | 订阅创建成功，创建本地记录 |
| `customer.subscription.updated` | 订阅状态变更（续费、升级等） |
| `customer.subscription.deleted` | 订阅取消，吊销用户权限 |
| `invoice.paid` | 发票支付成功，记录支付历史 |
| `invoice.payment_failed` | 支付失败，通知用户 |

## 错误处理与重试

Stripe 的重试策略是**指数退避**：

- 第一次失败后，立即重试
- 之后按指数增长间隔重试（几分钟 → 几小时）
- 最多重试 3 天

所以我们的处理逻辑要保证：

1. **2xx 响应**：即使处理失败（比如数据库错误），也要返回 2xx 避免 Stripe 无意义重试
2. **日志记录**：处理失败时记录详细日志，方便排查
3. **幂等保证**：即使 Stripe 重试，也不会重复处理

```typescript
async function handleStripeEvent(event: Stripe.Event) {
  try {
    // 幂等检查
    const existing = await db.query.stripeEventTable.findFirst({
      where: eq(stripeEventTable.stripeEventId, event.id),
    });
    if (existing) return;

    // 处理事件
    await processEvent(event);

    // 记录
    await db.insert(stripeEventTable).values({
      stripeEventId: event.id,
      type: event.type,
      processedAt: new Date(),
    });
  } catch (error) {
    // 记录错误但不返回 4xx/5xx（避免 Stripe 无意义重试）
    console.error("Failed to process webhook event:", event.id, error);
    // 仍然返回 200 给 Stripe
  }
}
```

## 总结

- 签名验证防止伪造请求
- `stripeEventId` 做幂等，重复事件直接跳过
- 处理失败也返回 200，避免 Stripe 无意义重试
- 记录详细日志，方便排查问题
