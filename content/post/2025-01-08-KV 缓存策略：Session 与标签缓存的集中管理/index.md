---
title: KV 缓存策略：Session 与标签缓存的集中管理
description: ""
slug: 2025-01-08-KV 缓存策略：Session 与标签缓存的集中管理
date: 2025-01-08
image: KV.png
categories:
  - 数据库&缓存
  - 架构设计
tags:
  - cloudflare
  - KV
  - RuiToolAI
---

> Cloudflare KV 是 Workers 的"Redis"。聊聊如何用它做 Session 存储、数据缓存和 Rate Limiting。

## KV 在项目中的角色

RuiToolAI 用一个 KV Namespace 承担了多种职责：

| 用途 | Key 前缀 | 过期策略 |
|------|----------|----------|
| Session | `session:{userId}:{sessionId}` | 30天后过期 |
| 验证 Token | `verification-token:{token}` | 24小时后过期 |
| Rate Limit | `rate-limit:{identifier}:{key}:{window}` | 窗口结束后过期 |
| 数据缓存 | `stats:*`, `cms:*`, `sitemap` | 按需设置 TTL |

## 通用缓存封装：withKVCache

```typescript
// src/utils/with-kv-cache.ts

export async function withKVCache<T>(
  fn: () => Promise<T>,
  { key, ttl }: CacheOptions
): Promise<T> {
  if (!isProd) {
    return fn(); // 开发环境跳过缓存
  }

  const { env } = await getCloudflareContext();
  const kv = env.NEXT_INC_CACHE_KV;

  if (!kv) {
    throw new Error("Can't connect to KV store");
  }

  // 尝试读取缓存
  const cached = await kv.get(key, "text");
  if (cached !== null) {
    return superjson.parse<T>(cached);
  }

  // 缓存未命中，执行函数并缓存结果
  const result = await fn();
  await kv.put(key, superjson.stringify(result), {
    expirationTtl: Math.floor(ms(ttl) / 1000),
  });

  return result;
}
```

核心设计：
- **Cache-Aside 模式**：先查缓存，未命中再执行函数
- **superjson 序列化**：支持 Date、Map、Set 等原生 JSON 不支持的类型
- **开发环境自动跳过**：用 `isProd` 判断，本地开发时总是获取最新数据

## 缓存 Key 集中管理

所有缓存 Key 定义在一个地方，避免散落各处导致冲突：

```typescript
export const CACHE_KEYS = {
  SITEMAP: "sitemap",
  TOTAL_USERS: "stats:total-users",
  GITHUB_STARS: "stats:github-stars",
  CMS_ENTRY: "cms:entry",
  CMS_COLLECTION: "cms:collection",
  CMS_NAVIGATION: "cms:navigation",
  CMS_REDIRECT: "cms:redirect",
  CMS_SEARCH: "cms:search",
  CMS_TAGS: "cms:tags",
} as const;
```

使用示例：

```typescript
const entries = await withKVCache(
  () => db.query.cmsEntryTable.findMany({ ... }),
  {
    key: `${CACHE_KEYS.CMS_COLLECTION}:${collection}:${page}`,
    ttl: "5m",
  }
);
```

## Session 缓存

Session 不是用 `withKVCache`，而是直接读写 KV：

```typescript
// 读取 Session
const sessionStr = await kv.get(getSessionKey(userId, sessionId));
const session = JSON.parse(sessionStr) as KVSession;

// 写入 Session
await kv.put(
  getSessionKey(userId, sessionId),
  JSON.stringify(session),
  { expirationTtl: Math.floor((expiresAt.getTime() - Date.now()) / 1000) }
);
```

Session 的 TTL 和 Session 过期时间一致（30天），KV 会自动清理过期数据。

## 过期 Token 管理

邮箱验证、密码重置等场景需要有时效性的 Token：

```typescript
// src/utils/kv-token.ts

export async function createExpiringToken({
  key, expiresInSeconds, payload,
}: CreateExpiringTokenParams): Promise<string> {
  const token = createId(); // cuid2
  const expiresAt = new Date(Date.now() + expiresInSeconds * 1000);

  await kv.put(
    key(token),
    JSON.stringify({ ...payload, expiresAt: expiresAt.toISOString() }),
    { expirationTtl: expiresInSeconds }
  );

  return token;
}
```

KV 的 `expirationTtl` 让 Token 自动过期，不需要定时任务清理。

## 开发环境跳过缓存

```typescript
if (!isProd) {
  return fn();
}
```

这条规则对 Session 也同样适用——开发环境每次请求都重新从数据库加载用户数据，保证修改立即可见。

## 总结

- 一个 KV Namespace 承担 Session、缓存、Rate Limit、Token 多种职责
- `withKVCache` 封装了通用的 Cache-Aside 模式
- Key 集中管理，避免冲突
- 开发环境自动跳过缓存，生产环境开启