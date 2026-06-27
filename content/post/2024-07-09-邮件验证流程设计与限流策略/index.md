---
title: 邮件验证流程设计与限流策略
description: ""
slug: 2024-07-09-邮件验证流程设计与限流策略
date: 2024-07-09
image: email.png
categories:
  - 架构设计
tags:
  - email
  - 邮件
  - token
  - Cloudflare
  - RuiToolAI
---

> 注册后发验证邮件看似简单，但 token 过期、重发限制、验证状态管理这些细节才是关键。

## 整体流程

1. 用户注册成功 → 自动发送验证邮件
2. 邮件包含验证链接：`/verify-email?token=xxx`
3. 用户点击链接 → 验证 token → 更新 `emailVerified` 字段
4. 未验证时可以重发邮件（有限流）

## KV 存储过期 Token

验证 token 存储在 Cloudflare KV 中，利用 KV 的 `expirationTtl` 自动过期：

```typescript
// src/utils/kv-token.ts

export async function createExpiringToken({
  key,
  expiresInSeconds,
  payload,
  createToken = createId,
}: CreateExpiringTokenParams): Promise<string> {
  const kv = await getTokenKV();
  const token = createToken();
  const expiresAt = new Date(Date.now() + expiresInSeconds * 1000);

  await kv.put(
    key(token),
    JSON.stringify({
      ...payload,
      expiresAt: expiresAt.toISOString(),
    }),
    {
      expirationTtl: expiresInSeconds, // KV 自动过期，无需手动清理
    }
  );

  return token;
}
```

Token 的核心设计：
- 用 `cuid2` 生成随机 token，不可预测
- 存储在 KV 中，key 格式为 `verification-token:{token}`
- 24 小时过期（`EMAIL_VERIFICATION_TOKEN_EXPIRATION_SECONDS`）
- 过期后 KV 自动删除，查询时返回 null

## 发送验证邮件

```typescript
// src/utils/email-verification.ts

export async function sendUserVerificationEmail({
  userId,
  email,
  username,
}: SendUserVerificationEmailParams) {
  const verificationToken = await createExpiringToken({
    key: getVerificationTokenKey,
    expiresInSeconds: EMAIL_VERIFICATION_TOKEN_EXPIRATION_SECONDS,
    payload: { userId },
    createToken: createId,
  });

  await sendVerificationEmail({
    email,
    verificationToken,
    username,
  });
}
```

`sendVerificationEmail` 通过 Cloudflare 的邮件服务发送，包含验证链接：

```
https://yourapp.com/verify-email?token=xxx
```

## 验证 Token

```typescript
// src/app/(auth)/verify-email/verify-email.action.ts

export const verifyEmailAction = actionClient
  .inputSchema(/* ... */)
  .action(async ({ parsedInput: input }) => {
    const { token } = input;

    // 1. 从 KV 获取 token
    const payload = await getValidExpiringToken({
      token,
      key: getVerificationTokenKey,
      notFoundError: {
        code: "NOT_FOUND",
        message: "Invalid or expired verification token",
      },
    });

    // 2. 更新用户的 emailVerified 字段
    const db = getDB();
    await db
      .update(userTable)
      .set({ emailVerified: new Date() })
      .where(eq(userTable.id, payload.userId));

    // 3. 删除已使用的 token
    await deleteExpiringToken({
      token,
      key: getVerificationTokenKey,
    });

    return { success: true };
  });
```

关键点：
- 验证成功后**立即删除 token**，防止重复使用
- 如果 token 不存在或已过期，返回明确错误
- 不依赖 token 中的 userId 做身份验证（只是更新 emailVerified）

## 重发限制

在用户设置页面可以重发验证邮件，但需要限流：

```typescript
export const sendVerificationAction = actionClient
  .action(async () => {
    return withRateLimit(async () => {
      const session = await requireVerifiedEmail({
        requireVerified: false, // 允许未验证用户访问
      });

      if (session.user.emailVerified) {
        throw new ActionError("CONFLICT", "Email already verified");
      }

      await sendUserVerificationEmail({
        userId: session.user.id,
        email: session.user.email!,
        username: session.user.firstName || session.user.email!,
      });

      return { success: true };
    }, RATE_LIMITS.EMAIL); // 每小时最多 10 次
  });
```

## 前端验证状态管理

用户登录后，如果邮箱未验证，会弹出一个验证提示对话框：

```typescript
// src/components/email-verification-dialog.tsx
// 检查 session.user.emailVerified 是否为 null
// 如果未验证 → 显示对话框，提供"重发邮件"按钮
// 如果已验证 → 不显示
```

## 总结

- KV 存储 token，利用 `expirationTtl` 自动过期，零维护成本
- 验证后立即删除 token，防止重复使用
- 重发邮件有限流保护（每小时 10 次）
- 前端通过 `emailVerified` 字段判断是否显示验证提示