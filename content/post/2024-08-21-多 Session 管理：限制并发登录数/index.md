---
title: 多 Session 管理：限制并发登录数
description: ""
slug: 2024-08-21-多 Session 管理：限制并发登录数
date: 2024-08-21
image: session.png
categories:
  - 架构设计
  - 数据库&缓存
tags:
  - Session
  - RuiToolAI
---

> 一个用户最多 5 个 Session，超出自动踢掉最早的。聊聊 Session 管理的实现细节。

## 为什么限制 Session 数量

不限制的话，一个用户可以在无数设备上登录，Session 数据无限膨胀。另外还有安全考虑：如果账号被盗，攻击者可以无限创建 Session。

RuiToolAI 设置为 **每个用户最多 5 个 Session**。

## Session 存储设计

Session 存储在 Cloudflare KV 中，key 格式为：

```
session:{userId}:{sessionId}
```

每个 Session 对象包含：

```typescript
// src/utils/kv-session.ts

export interface KVSession {
  id: string;           // sessionId
  userId: string;
  expiresAt: number;    // 过期时间戳
  createdAt: number;    // 创建时间戳
  user: KVSessionUser;  // 用户信息快照
  country?: string;     // 登录地（从 Cloudflare cf 对象获取）
  city?: string;
  continent?: string;
  ip?: string | null;
  userAgent?: string | null;
  authenticationType?: "passkey" | "password" | "google-oauth";
  version?: number;      // 用于版本升级
}
```

## 创建 Session 时自动淘汰

创建新 Session 时，如果已达到上限，自动淘汰最早的 Session：

```typescript
// src/utils/kv-session.ts

export async function createKVSession({
  sessionId, userId, expiresAt, user,
  authenticationType, passkeyCredentialId,
}: CreateKVSessionParams): Promise<KVSession> {
  const { cf } = await getCloudflareContext();
  const kv = await getKV();

  // 先构建 Session 对象
  const session: KVSession = {
    id: sessionId,
    userId,
    expiresAt: expiresAt.getTime(),
    createdAt: Date.now(),
    country: cf?.country,
    city: cf?.city,
    continent: cf?.continent,
    ip: await getIP(),
    userAgent: headersList.get("user-agent"),
    user,
    authenticationType,
    passkeyCredentialId,
    version: CURRENT_SESSION_VERSION,
  };

  // 再检查是否超出上限，超出则淘汰最早的
  const existingSessions = await getAllSessionIdsOfUser(userId);

  if (existingSessions.length >= MAX_SESSIONS_PER_USER) {
    const sessionsToDelete =
      existingSessions.length - MAX_SESSIONS_PER_USER + 1;

    // 按过期时间排序，最早的在前面
    const sortedSessions = [...existingSessions].sort((a, b) => {
      if (!a.absoluteExpiration) return -1;
      if (!b.absoluteExpiration) return 1;
      return a.absoluteExpiration.getTime() - b.absoluteExpiration.getTime();
    });

    for (let i = 0; i < sessionsToDelete; i++) {
      const sessionKey = sortedSessions[i]?.key;
      if (!sessionKey) continue;

      const oldSessionId = sessionKey.split(":")[2];
      if (!oldSessionId) continue;

      await deleteKVSession(oldSessionId, userId);
    }
  }

  // 写入新 Session
  await kv.put(
    getSessionKey(userId, sessionId),
    JSON.stringify(session),
    { expirationTtl: Math.floor((expiresAt.getTime() - Date.now()) / 1000) }
  );

  return session;
}
```

淘汰策略：
1. 按过期时间排序，淘汰最早的
2. 删除数量 = 当前数量 - 上限 + 1（为新 Session 腾位置）
3. 直接删除 KV 中的记录

## Session 版本管理

Session 结构可能随产品迭代变化。如果改了 Session 的字段，老的 Session 数据就和新的代码不兼容了。

解决方案：**版本号**

```typescript
export const CURRENT_SESSION_VERSION = 5;
```

验证 Session 时，如果版本号不匹配，自动更新：

```typescript
// src/utils/auth.ts

async function validateSessionToken(token: string, userId: string) {
  const session = await getKVSession(sessionId, userId);

  if (!session) return null;

  // 版本不匹配 → 自动更新 Session
  if (!session.version || session.version !== CURRENT_SESSION_VERSION) {
    const updatedSession = await updateKVSession(
      sessionId, userId, new Date(session.expiresAt)
    );
    return updatedSession;
  }

  return session;
}
```

这样改了 Session 结构后，只需要递增 `CURRENT_SESSION_VERSION`，用户下次请求时自动迁移。

## Session 元数据

每个 Session 记录了一些有用的元数据：

- **登录方式**：密码 / Passkey / Google OAuth
- **地理位置**：从 Cloudflare 的 `cf` 对象获取 country、city、continent
- **IP 地址**：从信任的请求头获取
- **User Agent**：浏览器和设备信息

这些数据在"管理 Session"页面上展示给用户，如果发现异常登录（陌生地点、陌生设备），用户可以手动下线。

## 用户主动管理 Session

用户在设置页面可以看到所有 Session 并手动删除：

```typescript
// 列出所有 Session（实际函数名：getSessionsAction）
// 返回每个 Session 的完整信息，包含 parsedUserAgent（浏览器/设备/OS）
// 以及 isCurrentSession 标记当前 Session
export const getSessionsAction = actionClient
  .inputSchema(v.void())
  .action(async () => {
    return withRateLimit(async () => {
      const session = await getSessionFromCookie();
      if (!session?.user?.id) {
        throw new ActionError("NOT_AUTHORIZED", "Unauthorized");
      }

      const sessionIds = await getAllSessionIdsOfUser(session.user.id);
      const sessions = await Promise.all(
        sessionIds.map(async ({ key, absoluteExpiration }) => {
          const sessionId = key.split(":")[2];
          const sessionData = await getKVSession(sessionId, session.user.id);
          if (!sessionData) return null;

          const result = new UAParser(sessionData.userAgent ?? "").getResult();
          return {
            ...sessionData,
            isCurrentSession: sessionId === session.id,
            expiration: absoluteExpiration,
            parsedUserAgent: { browser: result.browser, device: result.device, os: result.os },
          };
        })
      );

      return sessions.filter(Boolean);
    }, RATE_LIMITS.SETTINGS);
  });

// 删除指定 Session
export const deleteSessionAction = actionClient
  .inputSchema(/* sessionId */)
  .action(async ({ parsedInput }) => {
    return withRateLimit(async () => {
      const session = await getSessionFromCookie();
      if (!session?.user?.id) {
        throw new ActionError("NOT_AUTHORIZED", "Unauthorized");
      }
      await deleteKVSession(parsedInput.sessionId, session.user.id);
      return { success: true };
    }, RATE_LIMITS.DELETE_SESSION);
  });
```

## 总结

- 上限 5 个 Session，超出自动淘汰最早的
- 版本号机制让 Session 结构变更零停机
- 记录登录地、设备信息，方便用户识别异常
- 支持用户主动查看和删除 Session