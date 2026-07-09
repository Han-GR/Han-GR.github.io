---
title: 一次性邮箱检测与注册拦截实战
description: ""
slug: 2024-06-23-一次性邮箱检测与注册拦截实战
date: 2024-06-23
image: disposable-email.webp
categories:
  - 效率&工具
tags:
  - disposable-email
  - debounce
  - mailcheck
  - RuiToolAI
---

> 注册时拦截掉一次性邮箱，既能减少垃圾用户，也能让邮件投递数据更干净。

## 为什么要拦截一次性邮箱

SaaS 项目上线后，你会发现注册用户里有一批用临时邮箱的——10分钟邮箱、一次性邮箱。这些用户：

- 不会验证邮箱（因为邮箱已经过期了）
- 浪费邮件发送配额
- 污染用户数据，影响分析

所以在注册时就拦截掉。

## 双服务冗余检测

RuiToolAI 用了两个免费的一次性邮箱检测服务，串行调用，任一返回"是临时邮箱"就拦截：

1. **debounce.io** — 免费的 disposable email 检测 API
2. **mailcheck.ai** — 同样是免费的，作为备选

为什么要两个？因为单个免费服务可能宕机，双服务冗余能保证可用性。逻辑是：

- 先调 debounce.io，如果返回"是临时邮箱" → 直接拦截
- 如果 debounce.io 失败或返回"不是" → 再调 mailcheck.ai
- 两个都失败 → 放行（不阻塞正常用户注册）

## 实现代码

```typescript
// src/utils/auth.ts

interface DisposableEmailResponse {
  disposable: string;
}

interface MailcheckResponse {
  email: string;
  domain: string;
  mx: boolean;
  disposable: boolean;
  public_domain: boolean;
  relay_domain: boolean;
  alias: boolean;
}

interface ValidatorResult {
  success: boolean;
  isDisposable: boolean;
}

async function checkWithDebounce(email: string): Promise<ValidatorResult> {
  try {
    const response = await fetch(
      `https://disposable.debounce.io/?email=${encodeURIComponent(email)}`
    );

    if (!response.ok) {
      console.error("Debounce.io API error:", response.status);
      return { success: false, isDisposable: false };
    }

    const data = await response.json() as DisposableEmailResponse;
    return { success: true, isDisposable: data.disposable === "true" };
  } catch (error) {
    console.error("Failed to check disposable email with debounce.io:", error);
    return { success: false, isDisposable: false };
  }
}

async function checkWithMailcheck(email: string): Promise<ValidatorResult> {
  try {
    const response = await fetch(
      `https://mailcheck.ai/api/check?email=${encodeURIComponent(email)}`
    );

    if (!response.ok) {
      console.error("Mailcheck.ai API error:", response.status);
      return { success: false, isDisposable: false };
    }

    const data = await response.json() as MailcheckResponse;
    return { success: true, isDisposable: data.disposable };
  } catch (error) {
    console.error("Failed to check disposable email with mailcheck.ai:", error);
    return { success: false, isDisposable: false };
  }
}

export async function canSignUp({
  email,
  skipDisposableEmailCheck = false,
}: {
  email: string;
  skipDisposableEmailCheck?: boolean;
}): Promise<void> {
  if (!isProd) {
    return;
  }

  if (skipDisposableEmailCheck) {
    return;
  }

  const checkers = [checkWithDebounce, checkWithMailcheck];

  for (const checker of checkers) {
    const result = await checker(email);

    if (!result.success) {
      continue;
    }

    if (result.isDisposable) {
      throw new ActionError(
        "PRECONDITION_FAILED",
        "Disposable email addresses are not allowed. Please use a permanent email address."
      );
    }

    return;
  }

  // 两个服务都失败了，放行
  console.warn("Both disposable email check services failed, allowing signup");
}
```

## 注册流程中的集成

在注册 Action 中，`canSignUp` 是第一个校验步骤：

```typescript
// src/app/(auth)/sign-up/sign-up.actions.ts

export const signUpAction = actionClient
  .inputSchema(signUpSchema)
  .action(async ({ parsedInput: input }) => {
    return withRateLimit(async () => {
      const db = getDB();

      // 1. 验证 Turnstile（如果启用）
      if (await isTurnstileEnabled()) {
        // ...
      }

      // 2. 检查是否一次性邮箱
      await canSignUp({ email: input.email });

      // 3. 检查邮箱是否已被注册
      const existingUser = await db.query.userTable.findFirst({
        where: eq(userTable.email, input.email),
      });

      if (existingUser) {
        throw new ActionError("CONFLICT", "Email already taken");
      }

      // 4. 创建用户
      // ...
    }, RATE_LIMITS.SIGN_UP);
  });
```

## 开发环境跳过检测

开发环境不需要调外部 API，直接跳过：

```typescript
if (!isProd) {
  return;
}
```

`isProd` 判断的是 `process.env.NODE_ENV === "production"`（在 Vinext 构建中这个值是正确的）。

## 总结

- 两个免费 API 冗余调用，任一命中就拦截
- 两个都失败时放行，不阻塞正常用户
- 放在注册校验的最前面，尽早拦截
- 开发环境自动跳过
