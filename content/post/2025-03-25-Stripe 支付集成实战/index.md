---
title: Stripe 支付集成实战
description: ""
slug: 2025-03-25-Stripe 支付集成实战
date: 2025-03-25
image: stripe.webp
categories:
  - 效率&工具
tags:
  - Stripe
  - RuiToolAI
---
## 整体流程

```
用户点击购买
  → 创建 Stripe Checkout Session
  → 跳转到 Stripe 支付页面
  → 用户完成支付
  → Stripe 发送 Webhook 到服务器
  → 服务器验证签名 + 发放积分
  → 用户跳回成功页面
```

## 创建 Checkout Session

```typescript
// src/app/(settings)/settings/billing/billing.actions.ts
export const createCheckoutSessionAction = actionClient
  .inputSchema(createCheckoutSchema)
  .action(async ({ parsedInput }) => {
    const session = await requireVerifiedEmail();
    const userId = session.user.id;

    const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);

    const checkoutSession = await stripe.checkout.sessions.create({
      mode: "payment",
      payment_method_types: ["card"],
      line_items: [
        {
          price: parsedInput.priceId,
          quantity: 1,
        },
      ],
      success_url: `${process.env.NEXT_PUBLIC_APP_URL}/dashboard/billing?success=true`,
      cancel_url: `${process.env.NEXT_PUBLIC_APP_URL}/dashboard/billing?canceled=true`,
      metadata: {
        userId,
        priceId: parsedInput.priceId,
      },
    });

    return { url: checkoutSession.url };
  });
```

前端跳转：

```typescript
const { execute } = useAction(createCheckoutSessionAction, {
  onSuccess: ({ data }) => {
    if (data?.url) {
      window.location.href = data.url;
    }
  },
});
```

## Webhook 处理

Stripe 支付完成后，会向你的服务器发送 Webhook 事件。

```typescript
// src/app/api/webhooks/stripe/route.ts
export async function POST(request: Request) {
  const body = await request.text();
  const signature = request.headers.get("stripe-signature")!;

  let event: Stripe.Event;

  try {
    // 验证签名，防止伪造请求
    event = stripe.webhooks.constructEvent(
      body,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET!
    );
  } catch (err) {
    return new Response("Invalid signature", { status: 400 });
  }

  switch (event.type) {
    case "checkout.session.completed": {
      const session = event.data.object as Stripe.CheckoutSession;
      await handleCheckoutCompleted(session);
      break;
    }
    case "charge.refunded": {
      const charge = event.data.object as Stripe.Charge;
      await handleRefund(charge);
      break;
    }
  }

  return new Response("OK", { status: 200 });
}

async function handleCheckoutCompleted(session: Stripe.CheckoutSession) {
  const userId = session.metadata?.userId;
  const priceId = session.metadata?.priceId;

  if (!userId || !priceId) return;

  // 根据 priceId 查找对应的积分数量
  const creditPlan = CREDIT_PLANS.find(p => p.priceId === priceId);
  if (!creditPlan) return;

  const db = getDB();

  // 发放积分
  await db.insert(creditTransactionTable).values({
    id: `txn_${createId()}`,
    userId,
    amount: creditPlan.credits,
    remainingAmount: creditPlan.credits,
    type: CREDIT_TRANSACTION_TYPE.PURCHASE,
    description: `Purchased ${creditPlan.credits} credits`,
    createdAt: new Date(),
    updatedAt: new Date(),
  });

  // 更新用户积分缓存
  await db
    .update(userTable)
    .set({ currentCredits: sql`currentCredits + ${creditPlan.credits}` })
    .where(eq(userTable.id, userId));
}
```

## 幂等性处理

Stripe 的 Webhook 可能会重复发送（网络超时重试）。需要保证同一个事件只处理一次。

```typescript
async function handleCheckoutCompleted(session: Stripe.CheckoutSession) {
  const db = getDB();

  // 检查是否已经处理过这个 session
  const existing = await db.query.creditTransactionTable.findFirst({
    where: eq(creditTransactionTable.stripeSessionId, session.id),
  });

  if (existing) {
    console.log(`Session ${session.id} already processed, skipping`);
    return;
  }

  // 正常处理...
  await db.insert(creditTransactionTable).values({
    id: `txn_${createId()}`,
    stripeSessionId: session.id, // 记录 session ID
    // ...
  });
}
```

## 退款自动化

用户在 Stripe 后台申请退款后，Stripe 会发送 `charge.refunded` 事件：

```typescript
async function handleRefund(charge: Stripe.Charge) {
  const sessionId = charge.payment_intent as string;

  // 找到对应的积分记录
  const transaction = await db.query.creditTransactionTable.findFirst({
    where: eq(creditTransactionTable.stripeSessionId, sessionId),
  });

  if (!transaction) return;

  // 扣除积分（不能超过剩余积分）
  const refundCredits = Math.min(
    transaction.remainingAmount,
    transaction.amount
  );

  await db
    .update(creditTransactionTable)
    .set({ remainingAmount: 0 })
    .where(eq(creditTransactionTable.id, transaction.id));

  // 更新用户积分缓存
  await db
    .update(userTable)
    .set({
      currentCredits: sql`MAX(0, currentCredits - ${refundCredits})`,
    })
    .where(eq(userTable.id, transaction.userId));
}
```

## 本地调试

本地调试 Webhook 需要用 Stripe CLI 转发事件：

```bash
# 安装 Stripe CLI
brew install stripe/stripe-cli/stripe

# 登录
stripe login

# 转发 Webhook 到本地
stripe listen --forward-to localhost:3000/api/webhooks/stripe

# 触发测试事件
stripe trigger checkout.session.completed
```

## 总结

Stripe 集成的关键点：

1. **签名验证**：每个 Webhook 都要验证 `stripe-signature`，防止伪造
2. **幂等性**：用 `stripeSessionId` 去重，防止重复发放积分
3. **退款联动**：监听 `charge.refunded` 事件，自动扣除积分
4. **本地调试**：用 Stripe CLI 转发 Webhook 到本地

## 参考资源

- [Stripe Checkout](https://stripe.com/docs/checkout) 
- [Stripe Webhooks](https://stripe.com/docs/webhooks) 
- [Stripe CLI](https://stripe.com/docs/stripe-cli)