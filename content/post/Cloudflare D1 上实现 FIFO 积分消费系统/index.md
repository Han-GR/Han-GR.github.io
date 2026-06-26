---
title: Cloudflare D1 上实现 FIFO 积分消费系统
description: ""
slug: 2024-05-27-Cloudflare D1 上实现 FIFO 积分消费系统
date: 2024-05-27
image: FIFO.jpg
categories:
  - 架构设计
  - 数据库&缓存
  - 算法&计算机基础
tags:
  - FIFO
---

## 为什么不用简单的余额字段

最简单的积分系统就是一个 `currentCredits` 字段：

```typescript
// 加积分
user.currentCredits += 100;

// 消费
user.currentCredits -= 10;

// 退款
user.currentCredits += 10;
```

这很简单，但有几个致命问题：

**1. 无法审计**

用户投诉"我明明有 100 积分怎么没了"，你没法追溯每笔积分的来源和去向。

**2. 无法处理积分有效期**

如果用户买了两次积分，一次 100（30 天有效期），一次 50（7 天有效期）。消费时应该先扣哪个？简单的余额字段做不到。

**3. 退款很难精确**

用户买了一次积分，分 5 次消费了，现在要退款——退多少？退哪次买的？

## FIFO 消费桶模型

FIFO（First In, First Out）消费桶模型：

```
购买 100 积分 → bucket_1: remaining=100
购买 50 积分  → bucket_2: remaining=50

消费 30 积分：
  → bucket_1: remaining = 100 - 30 = 70
  → bucket_2: 不变

消费 80 积分：
  → bucket_1: remaining = 0（用完 70）
  → bucket_2: remaining = 50 - 10 = 40
```

## 数据库表设计

```typescript
export const creditTransactionTable = sqliteTable("credit_transaction", {
  id: text().primaryKey(),
  userId: text().notNull().references(() => userTable.id),
  amount: integer().notNull(),          // 交易总额
  remainingAmount: integer().notNull(), // 剩余可用积分
  type: text({
    enum: ["PURCHASE", "USAGE", "MONTHLY_REFRESH"]
  }).notNull(),
  description: text(),
  createdAt: integer({ mode: "timestamp" }).notNull(),
  updatedAt: integer({ mode: "timestamp" }).notNull(),
});
```

**三种交易类型：**

| 类型 | 说明 | amount | remainingAmount |
|------|------|--------|----------------|
| PURCHASE | 购买积分 | 100 | 100（初始全可用） |
| MONTHLY_REFRESH | 月度刷新 | 50 | 50 |
| USAGE | 消费积分 | 10 | 0（记录已消费的量） |

## 消费流程

```typescript
export async function consumeCredits({
  userId,
  amount,
  description,
}: {
  userId: string;
  amount: number;
  description: string;
}) {
  const db = getDB();

  // 1. 查询所有还有剩余积分的桶（按创建时间升序 = FIFO）
  const buckets = await db.query.creditTransactionTable.findMany({
    where: and(
      eq(creditTransactionTable.userId, userId),
      gt(creditTransactionTable.remainingAmount, 0),
      inArray(creditTransactionTable.type, [
        CREDIT_TRANSACTION_TYPE.PURCHASE,
        CREDIT_TRANSACTION_TYPE.MONTHLY_REFRESH,
      ])
    ),
    orderBy: (table, { asc }) => [asc(table.createdAt)],
  });

  // 2. 检查可用积分是否足够
  const totalAvailable = buckets.reduce(
    (sum, b) => sum + b.remainingAmount, 0
  );
  if (totalAvailable < amount) {
    throw new Error("Insufficient credits");
  }

  // 3. 从最早的桶开始扣
  let remaining = amount;
  for (const bucket of buckets) {
    if (remaining <= 0) break;
    const deduct = Math.min(bucket.remainingAmount, remaining);

    await db
      .update(creditTransactionTable)
      .set({ remainingAmount: bucket.remainingAmount - deduct })
      .where(eq(creditTransactionTable.id, bucket.id));

    remaining -= deduct;
  }

  // 4. 记录消费日志
  await logTransaction({
    userId,
    amount,
    description,
    type: CREDIT_TRANSACTION_TYPE.USAGE,
    remainingAmount: 0,
  });
}
```

## 退款流程

退款时，按创建时间倒序找到最近被消费过的桶，把积分加回去：

```typescript
export async function addUserCredits(userId: string, amount: number) {
  const db = getDB();

  // 找最近的购买记录（remainingAmount < amount 表示被消费过）
  const latestBucket = await db.query.creditTransactionTable.findFirst({
    where: and(
      eq(creditTransactionTable.userId, userId),
      eq(creditTransactionTable.type, CREDIT_TRANSACTION_TYPE.PURCHASE),
      lt(creditTransactionTable.remainingAmount, creditTransactionTable.amount),
    ),
    orderBy: (table, { desc }) => [desc(table.createdAt)],
  });

  if (latestBucket) {
    await db
      .update(creditTransactionTable)
      .set({ remainingAmount: latestBucket.remainingAmount + amount })
      .where(eq(creditTransactionTable.id, latestBucket.id));
  }
}
```

## D1 不支持事务的补偿方案

Cloudflare D1 不支持多语句事务。这意味着"扣积分 + 记录日志"不能原子执行。

**补偿方案：先操作，后补偿**

```typescript
// 扣积分
await consumeCredits({ userId, amount, description });

// 执行可能失败的操作
try {
  await someExternalApiCall();
} catch (error) {
  // 补偿：退积分
  await addUserCredits(userId, amount);
  await logTransaction({
    userId,
    amount,
    description: "Refund: operation failed",
    type: CREDIT_TRANSACTION_TYPE.PURCHASE,
    remainingAmount: amount,
  });
  throw error;
}
```

极端情况下（退款也失败），需要人工介入。但实际使用中，结合 Stripe Webhook 的幂等性，风险很低。

## 总结

FIFO 消费桶模型比简单余额复杂，但提供了：

- 积分来源和去向的完全可追溯性
- 积分有效期管理
- 精确的退款能力
- 配合补偿方案，在 D1 无事务的环境下也能保证数据一致性

对于 SaaS 产品来说，积分就是钱，这个系统值得花时间好好设计。

## 参考资源

- [Drizzle ORM](https://orm.drizzle.team/) 
- [Cloudflare D1](https://developers.cloudflare.com/d1/) 
- [FIFO 算法](https://en.wikipedia.org/wiki/FIFO_(computing_and_electronics))