---
title: 积分过期机制设计：从 FIFO 到年度过期
description: ""
slug: 2025-05-08-积分过期机制设计：从 FIFO 到年度过期
date: 2025-05-08
image: FIFO.jpg
categories:
  - 架构设计
  - 数据库&缓存
tags:
  - FIFO
  - 并发
  - RuiToolAI
---

> 积分不是永久的。设计一个既公平又简单的过期机制，比想象中复杂。


## 设计目标

1. **FIFO 消费**：先购买的积分优先消费
2. **年度过期**：积分有有效期，过期后自动清零
3. **并发安全**：积分消费和过期处理不能出现数据不一致

## 数据模型

```typescript
// src/db/schema.ts

export const creditTransactionTable = sqliteTable("credit_transactions", {
  id: text("id").primaryKey(),
  userId: text("user_id").notNull().references(() => userTable.id),
  amount: integer("amount").notNull(),           // 原始金额
  remainingAmount: integer("remaining_amount").notNull(), // 剩余金额
  type: text("type", {
    enum: ["purchase", "consumption", "expiration", "refund"],
  }).notNull(),
  description: text("description"),
  expirationDate: integer("expiration_date", { mode: "timestamp" }),
  // 过期处理标记
  expirationDateProcessedAt: integer("expiration_date_processed_at",
    { mode: "timestamp" }),
  createdAt: integer("created_at", { mode: "timestamp" })
    .notNull().$defaultFn(() => new Date()),
});

export const userTable = sqliteTable("users", {
  // ...
  currentCredits: integer("current_credits").notNull().default(0),
  lastCreditRefreshAt: integer("last_credit_refresh_at",
    { mode: "timestamp" }),
});
```

核心字段：
- `remainingAmount`：每笔积分交易都记录剩余金额，消费时扣减
- `expirationDate`：积分的过期时间
- `expirationDateProcessedAt`：过期处理的时间戳，null 表示未处理

## FIFO 消费：先买的先花

```typescript
// src/utils/credits.ts

// 实际函数名：consumeUserCredits，参数为对象
export async function consumeUserCredits({
  userId,
  amount,
  description,
}: {
  userId: string;
  amount: number;
  description: string;
}) {
  const db = getDB();
  const currentTime = new Date();

  // 先处理过期积分
  await processExpiredCredits(userId, currentTime);

  // 查询可用积分桶，排序：先按过期时间（最快过期的先消费），再按创建时间
  const sourceTransactions = await db.query.creditTransactionTable.findMany({
    where: and(
      eq(creditTransactionTable.userId, userId),
      inArray(creditTransactionTable.type, [
        CREDIT_TRANSACTION_TYPE.PURCHASE,
        CREDIT_TRANSACTION_TYPE.MONTHLY_REFRESH,
      ]),
      gt(creditTransactionTable.remainingAmount, 0),
      isNull(creditTransactionTable.expirationDateProcessedAt),
      or(
        isNull(creditTransactionTable.expirationDate),
        gte(creditTransactionTable.expirationDate, currentTime),
      ),
    ),
    orderBy: [
      asc(creditTransactionTable.expirationDate), // 最快过期的先消费
      asc(creditTransactionTable.createdAt),
    ],
    columns: { id: true, remainingAmount: true },
  });

  const availableFromBuckets = sourceTransactions.reduce(
    (total, t) => total + t.remainingAmount, 0
  );

  if (availableFromBuckets < amount) {
    throw new InsufficientCreditsError({
      requiredCredits: amount,
      availableCredits: availableFromBuckets,
    });
  }

  // 从桶中扣减（FIFO，按过期时间优先）
  // ...（逐桶扣减逻辑）

  // 更新用户总积分
  await db
    .update(userTable)
    .set({ currentCredits: sql`${userTable.currentCredits} - ${amount}` })
    .where(and(
      eq(userTable.id, userId),
      gte(userTable.currentCredits, amount),
    ));
}
```

FIFO 的关键：排序是 `asc(expirationDate), asc(createdAt)`——**最快过期的积分优先消费**，而不是单纯按创建时间。这样能最大化积分利用率，避免积分在过期前没用完。

## 过期处理：原子更新防并发

```typescript
async function processExpiredCredits(userId: string, currentTime: Date) {
  const db = getDB();

  // 查找所有已过期但未处理的积分
  const expiredTransactions = await db.query.creditTransactionTable.findMany({
    where: and(
      eq(creditTransactionTable.userId, userId),
      lt(creditTransactionTable.expirationDate, currentTime),
      isNull(creditTransactionTable.expirationDateProcessedAt),
      gt(creditTransactionTable.remainingAmount, 0),
    ),
    orderBy: [asc(creditTransactionTable.createdAt)],
  });

  for (const transaction of expiredTransactions) {
    // 原子更新：只有 remainingAmount 没变才更新
    const updateResult = await db
      .update(creditTransactionTable)
      .set({
        expirationDateProcessedAt: currentTime,
        remainingAmount: 0,
      })
      .where(and(
        eq(creditTransactionTable.id, transaction.id),
        isNull(creditTransactionTable.expirationDateProcessedAt),
        // 乐观锁：只在 remainingAmount 没变时才更新
        eq(creditTransactionTable.remainingAmount, transaction.remainingAmount)
      ))
      .returning({ id: creditTransactionTable.id });

    // 如果没更新成功，说明被其他请求处理了
    if (!updateResult || updateResult.length === 0) {
      continue;
    }

    // 从用户总积分中扣除
    await db
      .update(userTable)
      .set({ currentCredits: sql`current_credits - ${transaction.remainingAmount}` })
      .where(eq(userTable.id, userId));
  }
}
```

并发安全的保证：
- 用 `remainingAmount` 做乐观锁：只有值没变时才更新
- 如果被其他请求抢先处理了，`returning` 会返回空，直接跳过
- 不需要事务（D1 也不支持）

## 积分同步到 Session

User 的 `currentCredits` 存在数据库里，但 Session 里有缓存。需要定期同步：

```typescript
// src/utils/auth.ts

async function validateSessionToken(token: string, userId: string) {
  const session = await getKVSession(sessionId, userId);

  // 同步积分
  const currentCredits = await syncCreditsIfNeeded(session);

  if (
    session?.user?.currentCredits &&
    currentCredits !== session.user.currentCredits
  ) {
    session.user.currentCredits = currentCredits;
  }

  return session;
}
```

## 总结

- `remainingAmount` 字段实现 FIFO 消费
- 过期处理用乐观锁防并发
- Session 中的积分定期同步
- 消费前先处理过期积分，确保数据一致性
