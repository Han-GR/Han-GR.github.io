---
title: Lucia Auth 认证体系搭建
description: ""
slug: 2024-05-08-Lucia Auth 认证体系搭建
date: 2024-05-08
image: Lucia-Auth.webp
categories:
  - 前端
tags:
  - Lucia-Auth
  - Google-OAuth
  - Passkey
  - Session
---
## 为什么选 Lucia Auth

市面上有很多认证方案：NextAuth、Clerk、Auth0、Supabase Auth……

选 Lucia Auth 的原因：

- **完全自托管**：数据存在自己的 D1，不依赖第三方服务
- **轻量**：只提供核心认证逻辑，不绑定特定数据库
- **灵活**：可以自定义 Session 存储（我们用 KV）
- **免费**：不像 Clerk 按 MAU 收费

## 邮箱密码登录

### 注册

```typescript
// src/app/(auth)/sign-up/sign-up.actions.ts
export const signUpAction = actionClient
  .inputSchema(signUpSchema)
  .action(async ({ parsedInput }) => {
    const { email, password } = parsedInput;
    const db = getDB();

    // 检查邮箱是否已注册
    const existing = await db.query.userTable.findFirst({
      where: eq(userTable.email, email.toLowerCase()),
    });
    if (existing) {
      throw new ActionError("CONFLICT", "Email already registered");
    }

    // 哈希密码
    const passwordHash = await hashPassword(password);

    // 创建用户
    const userId = `usr_${createId()}`;
    await db.insert(userTable).values({
      id: userId,
      email: email.toLowerCase(),
      passwordHash,
      role: "user",
      emailVerified: false,
    });

    // 发送验证邮件
    await sendVerificationEmail({ userId, email });

    // 创建 Session
    const sessionId = await createSession(userId);
    await setSessionCookie(sessionId);

    return { success: true };
  });
```

### 密码哈希

```typescript
// 使用 Web Crypto API（Workers 兼容）
export async function hashPassword(password: string): Promise<string> {
  const encoder = new TextEncoder();
  const salt = crypto.getRandomValues(new Uint8Array(16));
  const keyMaterial = await crypto.subtle.importKey(
    "raw",
    encoder.encode(password),
    "PBKDF2",
    false,
    ["deriveBits"]
  );
  const hash = await crypto.subtle.deriveBits(
    { name: "PBKDF2", salt, iterations: 100000, hash: "SHA-256" },
    keyMaterial,
    256
  );
  // 拼接 salt + hash，base64 编码存储
  const combined = new Uint8Array(salt.length + hash.byteLength);
  combined.set(salt);
  combined.set(new Uint8Array(hash), salt.length);
  return btoa(String.fromCharCode(...combined));
}
```

## Google OAuth

```typescript
// src/app/api/auth/google/route.ts
export async function GET() {
  const state = generateState();
  const codeVerifier = generateCodeVerifier();

  const url = google.createAuthorizationURL(state, codeVerifier, [
    "openid",
    "email",
    "profile",
  ]);

  // 存储 state 和 codeVerifier 到 Cookie
  const response = new Response(null, {
    status: 302,
    headers: { Location: url.toString() },
  });
  setCookie(response, "google_oauth_state", state, { httpOnly: true });
  setCookie(response, "google_code_verifier", codeVerifier, { httpOnly: true });

  return response;
}

// src/app/api/auth/google/callback/route.ts
export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const code = searchParams.get("code");
  const state = searchParams.get("state");

  // 验证 state
  const storedState = getCookie(request, "google_oauth_state");
  if (state !== storedState) {
    return new Response("Invalid state", { status: 400 });
  }

  // 换取 token
  const tokens = await google.validateAuthorizationCode(code, codeVerifier);
  const googleUser = await getGoogleUser(tokens.accessToken);

  // 查找或创建用户
  let user = await db.query.userTable.findFirst({
    where: eq(userTable.googleAccountId, googleUser.sub),
  });

  if (!user) {
    user = await createUserFromGoogle(googleUser);
  }

  // 创建 Session
  const sessionId = await createSession(user.id);
  await setSessionCookie(sessionId);

  return redirect("/dashboard");
}
```

## Passkey 无密码登录

Passkey 基于 WebAuthn 标准，用设备的生物识别（Face ID、指纹）替代密码。

### 注册 Passkey

```typescript
// 1. 服务器生成挑战
export const generatePasskeyRegistrationOptions = actionClient
  .action(async () => {
    const session = await requireVerifiedEmail();
    const options = await generateRegistrationOptions({
      rpName: "RuiTool AI",
      rpID: process.env.NEXT_PUBLIC_APP_DOMAIN!,
      userID: session.user.id,
      userName: session.user.email,
    });
    // 存储 challenge 到 KV
    await kv.put(`passkey_challenge:${session.user.id}`, options.challenge);
    return options;
  });

// 2. 前端调用 WebAuthn API
const credential = await startRegistration(options);

// 3. 服务器验证并保存
export const verifyPasskeyRegistration = actionClient
  .inputSchema(passkeyRegistrationSchema)
  .action(async ({ parsedInput }) => {
    const challenge = await kv.get(`passkey_challenge:${userId}`);
    const verification = await verifyRegistrationResponse({
      response: parsedInput.credential,
      expectedChallenge: challenge,
      expectedOrigin: process.env.NEXT_PUBLIC_APP_URL!,
    });
    if (verification.verified) {
      await db.insert(passkeyTable).values({
        userId,
        credentialId: verification.registrationInfo.credentialID,
        publicKey: verification.registrationInfo.credentialPublicKey,
      });
    }
  });
```

## Session 管理

Session 存储在 Cloudflare KV，而不是 D1，原因是：

- KV 读取速度更快（每次请求都要验证 Session）
- KV 原生支持 TTL（过期自动删除）
- 减少 D1 的读取压力

```typescript
// src/utils/kv-session.ts
export async function createSession(userId: string): Promise<string> {
  const sessionId = generateId(32);
  const { env } = await getCloudflareContext();

  await env.NEXT_TAG_CACHE_KV.put(
    `session:${sessionId}`,
    JSON.stringify({ userId, createdAt: Date.now() }),
    { expirationTtl: 7 * 24 * 60 * 60 } // 7 天
  );

  return sessionId;
}

export async function getSession(sessionId: string) {
  const { env } = await getCloudflareContext();
  const raw = await env.NEXT_TAG_CACHE_KV.get(`session:${sessionId}`);
  return raw ? JSON.parse(raw) : null;
}
```

## 邮箱验证

```typescript
export async function sendVerificationEmail({ userId, email }) {
  const token = generateId(32);
  const db = getDB();

  // 存储验证 token
  await db.insert(emailVerificationTable).values({
    id: `ev_${createId()}`,
    userId,
    token,
    expiresAt: new Date(Date.now() + 24 * 60 * 60 * 1000), // 24 小时
  });

  // 发送邮件
  await sendEmail({
    to: email,
    subject: "Verify your email",
    html: `<a href="${APP_URL}/verify-email?token=${token}">Verify Email</a>`,
  });
}
```

## 密码重置

```typescript
// 1. 用户提交邮箱
export const requestPasswordResetAction = actionClient
  .inputSchema(z.object({ email: z.string().email() }))
  .action(async ({ parsedInput }) => {
    const user = await db.query.userTable.findFirst({
      where: eq(userTable.email, parsedInput.email),
    });

    // 不管用户是否存在，都返回成功（防止枚举攻击）
    if (!user) return { success: true };

    const token = generateId(32);
    await db.insert(passwordResetTable).values({
      token,
      userId: user.id,
      expiresAt: new Date(Date.now() + 60 * 60 * 1000), // 1 小时
    });

    await sendPasswordResetEmail({ email: parsedInput.email, token });
    return { success: true };
  });

// 2. 用户点击链接，提交新密码
export const resetPasswordAction = actionClient
  .inputSchema(resetPasswordSchema)
  .action(async ({ parsedInput }) => {
    const { token, newPassword } = parsedInput;

    const resetRecord = await db.query.passwordResetTable.findFirst({
      where: and(
        eq(passwordResetTable.token, token),
        gt(passwordResetTable.expiresAt, new Date()),
      ),
    });

    if (!resetRecord) {
      throw new ActionError("INVALID_TOKEN", "Token is invalid or expired");
    }

    const passwordHash = await hashPassword(newPassword);
    await db.update(userTable)
      .set({ passwordHash })
      .where(eq(userTable.id, resetRecord.userId));

    // 删除 token，防止重复使用
    await db.delete(passwordResetTable)
      .where(eq(passwordResetTable.token, token));
  });
```

## 总结

Lucia Auth 的认证体系：

| 功能 | 实现方式 |
|------|---------|
| 邮箱密码 | PBKDF2 哈希 + D1 存储 |
| Google OAuth | OAuth 2.0 + PKCE |
| Passkey | WebAuthn + KV 存储 challenge |
| Session | KV 存储 + Cookie |
| 邮箱验证 | Token + 邮件发送 |
| 密码重置 | Token + 防枚举设计 |

## 参考资源

- [Lucia Auth](https://lucia-auth.com/) 
- [WebAuthn](https://webauthn.guide/) 
- [SimpleWebAuthn](https://simplewebauthn.dev/) 
- [Arctic OAuth](https://arcticjs.dev/)