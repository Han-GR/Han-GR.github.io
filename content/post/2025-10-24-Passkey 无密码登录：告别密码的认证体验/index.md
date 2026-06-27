---
title: Passkey 无密码登录：告别密码的认证体验
description: ""
slug: 2025-10-24-Passkey 无密码登录：告别密码的认证体验
date: 2025-10-24
image: Passkey.webp
categories:
  - 效率&工具
tags:
  - Passkey
  - WebAuthn
  - RuiToolAI
---

> 比密码更安全，比短信验证码更方便。聊聊 Passkey 在 RuiTool AI 中的落地。

## 什么是 Passkey

Passkey 是基于 WebAuthn 标准的无密码认证方式。用户用指纹、面部识别或设备 PIN 来验证身份，私钥存在设备的安全芯片里，服务器只存公钥。

对用户来说，体验就是：点一下按钮 → 指纹/面容验证 → 登录成功。没有密码，没有验证码。

## 为什么选 Passkey

- **安全性更高**：没有密码就不会被撞库、钓鱼
- **用户体验好**：指纹/面容一键登录，比输入密码快得多
- **天然防机器人**：需要用户物理交互，自动化脚本无法绕过
- **WebAuthn 是标准**：所有主流浏览器和设备都支持

## 注册 Passkey

注册流程分两步：服务端生成注册选项 → 客户端调用浏览器 API → 服务端验证。

### 第一步：生成注册选项

```typescript
// src/app/(settings)/settings/security/passkey-settings.actions.ts

export const generateRegistrationOptionsAction = actionClient
  .inputSchema(generateRegistrationOptionsSchema)
  .action(async ({ parsedInput: input }) => {
    return withRateLimit(async () => {
      const session = await requireVerifiedEmail();
      const db = getDB();

      const user = await db.query.userTable.findFirst({
        where: eq(userTable.email, input.email),
      });

      if (!user) {
        throw new ActionError("NOT_FOUND", "User not found");
      }

      if (user.id !== session?.user?.id) {
        throw new ActionError("FORBIDDEN",
          "You can only register passkeys for your own account");
      }

      // 限制每个用户最多 5 个 Passkey
      const existingPasskeys = await db
        .select()
        .from(passKeyCredentialTable)
        .where(eq(passKeyCredentialTable.userId, user.id));

      if (existingPasskeys.length >= 5) {
        throw new ActionError("FORBIDDEN",
          "You have reached the maximum limit of 5 passkeys");
      }

      const options = await generatePasskeyRegistrationOptions(
        user.id,
        input.email
      );

      // 把 challenge 存到 cookie，验证时比对
      const cookieStore = await cookies();
      cookieStore.set(
        PASSKEY_REGISTRATION_CHALLENGE_COOKIE_NAME,
        options.challenge,
        {
          httpOnly: true,
          secure: isProd,
          sameSite: "strict",
          path: "/",
          maxAge: PASSKEY_CHALLENGE_TTL_SECONDS,
        }
      );

      return options;
    }, RATE_LIMITS.SETTINGS);
  });
```

### 第二步：客户端创建凭证

```typescript
// 客户端代码简化为：
const credential = await navigator.credentials.create({
  publicKey: options,
});
```

### 第三步：服务端验证注册

```typescript
export const verifyRegistrationAction = actionClient
  .inputSchema(/* ... */)
  .action(async ({ parsedInput: input }) => {
    const session = await requireVerifiedEmail();

    // 从 cookie 中取出 challenge 验证
    const cookieStore = await cookies();
    const challenge = cookieStore.get(
      PASSKEY_REGISTRATION_CHALLENGE_COOKIE_NAME
    )?.value;

    // verifyPasskeyRegistration 内部完成验证并写入数据库
    await verifyPasskeyRegistration({
      userId: session.user.id,
      response: input.registrationResponse,
      challenge: challenge!,
      userAgent: request.headers.get("user-agent"),
      ipAddress: await getIP(),
    });

    return { success: true };
  });
```

## 用 Passkey 登录

登录流程类似，但方向相反：

```typescript
export const generateAuthenticationOptionsAction = actionClient
  .inputSchema(passkeyAuthenticationOptionsSchema)
  .action(async ({ parsedInput: input }) => {
    return withRateLimit(async () => {
      const db = getDB();
      const user = await db.query.userTable.findFirst({
        where: eq(userTable.email, input.email),
      });

      if (!user) {
        throw new ActionError("NOT_FOUND", "User not found");
      }

      const options = await generatePasskeyAuthenticationOptions(user.id);

      // 存 challenge 和 userId 到 cookie
      const cookieStore = await cookies();
      cookieStore.set(
        PASSKEY_AUTHENTICATION_CHALLENGE_COOKIE_NAME,
        options.challenge,
        { httpOnly: true, secure: isProd, sameSite: "strict",
          path: "/", maxAge: PASSKEY_CHALLENGE_TTL_SECONDS }
      );
      cookieStore.set(
        PASSKEY_AUTHENTICATION_USER_ID_COOKIE_NAME,
        user.id,
        { httpOnly: true, secure: isProd, sameSite: "strict",
          path: "/", maxAge: PASSKEY_CHALLENGE_TTL_SECONDS }
      );

      return options;
    }, RATE_LIMITS.SIGN_IN);
  });
```

客户端调用 `navigator.credentials.get()`，服务端用 `verifyPasskeyAuthentication` 验证签名，成功后创建 Session。

## 管理已注册的 Passkey

在用户设置页面，可以查看、删除已注册的 Passkey：

- 列出所有已注册的 Passkey（显示设备名、注册时间）
- 支持删除单个 Passkey
- 每个用户最多 5 个，防止滥用

## 限制与安全

- **Challenge 存在 Cookie 中**：httpOnly + secure + strict sameSite，防止 XSS 窃取
- **Challenge 有效期 10 分钟**：过期后需要重新生成
- **Rate Limit**：注册和登录都受 `RATE_LIMITS.SETTINGS` 和 `RATE_LIMITS.SIGN_IN` 保护
- **最多 5 个 Passkey**：防止无限创建

## 总结

Passkey 是目前最安全的认证方式之一。RuiTool AI 同时支持密码登录和 Passkey 登录，用户可以在设置中自行添加。实现上依赖 `@simplewebauthn/server` 库，核心代码不到 200 行。