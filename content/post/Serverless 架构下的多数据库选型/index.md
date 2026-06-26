---
title: Serverless 架构下的多数据库选型
description: ""
slug: 2024-04-11-Serverless 架构下的多数据库选型
date: 2024-04-11
image: cloudflare.png
categories:
  - 运维&云原生
  - 数据库&缓存
tags:
  - Serverless
  - D1
  - R2
  - KV
  - cloudflare
---
## 为什么需要三种数据库

RuiTool AI 是一个全栈 SaaS 平台，数据需求多种多样：

- **用户数据、订单、积分流水**：需要结构化查询、关联查询
- **Session、API 缓存、页面缓存**：需要极快读写、支持过期时间
- **用户上传的文件、AI 生成的图片**：需要存储大文件、支持 CDN 分发

如果用单一数据库，比如全用 D1，反过来 Session 和缓存会拖慢关系型查询；全用 KV，用户订单这种结构化数据没法做关联查询。

所以选择了 **D1 + KV + R2** 的组合。

## D1：关系型数据库

Cloudflare D1 是基于 SQLite 的 Serverless 关系型数据库，兼容 SQLite 语法。

### 在 RuiTool AI 中的使用

D1 存储所有结构化数据：

```
user                     # 用户表
session                  # Session 表（已迁移到 KV）
credit_transaction       # 积分流水表
generated_image          # 图片生成记录表
cms_entry                # CMS 内容表
cms_navigation_node      # CMS 导航表
cms_media                # CMS 媒体表
```

### 集成方式

使用 Drizzle ORM 操作 D1：

```typescript
import { drizzle } from "drizzle-orm/d1";
import { getDB } from "@/db";

const db = getDB();

// 查询
const user = await db.query.userTable.findFirst({
  where: eq(userTable.email, email),
});

// 插入
await db.insert(userTable).values({
  id: `usr_${createId()}`,
  email,
  passwordHash,
  role: "user",
});
```

### 局限性

- **不支持事务**：D1 不支持多语句事务，处理退款时要补偿式设计
- **单文件限制**：数据库文件最大 2GB（付费版 10GB）
- **并发限制**：写入并发有限，高并发场景需要设计重试

## KV：键值存储

Cloudflare KV 是分布式的键值存储，最终一致性模型，全球秒级同步。

### 在 RuiTool AI 中的使用

KV 主要用在两个场景：

**1. Session 存储**

```typescript
// 存储 Session
await env.NEXT_TAG_CACHE_KV.put(
  `session:${sessionId}`,
  JSON.stringify(sessionData),
  { expirationTtl: 7 * 24 * 60 * 60 } // 7 天过期
);

// 读取 Session
const raw = await env.NEXT_TAG_CACHE_KV.get(`session:${sessionId}`);
const session = raw ? JSON.parse(raw) : null;
```

**2. Vinext 页面缓存**

```typescript
// worker-entrypoint.ts
new KVCacheHandler(env.NEXT_INC_CACHE_KV, {
  appPrefix: VINEXT_CACHE_PREFIX
})
```

Vinext 框架自动用 KV 做 ISR 缓存，每个页面请求都会先查 KV 是否有缓存。

### 特点

- **极快读取**：全球边缘节点，毫秒级
- **最终一致性**：写入后最多 60 秒同步到全球
- **免费额度**：每天 10 万次读取，1GB 存储

## R2：对象存储

Cloudflare R2 是 S3 兼容的对象存储，最大特点是**不收出口流量费**。

### 在 RuiTool AI 中的使用

R2 存储所有用户生成的文件：

```typescript
// 存储 AI 生成的图片
await env.USER_UPLOADS_BUCKET.put(r2Key, imageBuffer, {
  httpMetadata: {
    contentType: "image/png",
    contentDisposition: `attachment; filename="${filename}"`,
  },
  customMetadata: {
    prompt: prompt.slice(0, 500),
    generatedBy: userId,
    expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000).toISOString(),
  },
});

// 读取文件
const object = await env.USER_UPLOADS_BUCKET.get(r2Key);
const buffer = await object.arrayBuffer();

// 删除文件
await env.USER_UPLOADS_BUCKET.delete(r2Key);
```

### 生命周期规则

R2 支持自动清理过期文件。在 RuiTool AI 中，设置了 7 天生命周期规则，配合 Cron 定时任务清理 DB 记录：

```
R2 规则：uploads/ 前缀，7 天后自动删除
Cron 任务：每天凌晨 3 点，删除 DB 中 expiresAt < now 的记录
```

## 选型对比表

| 特性 | D1 | KV | R2 |
|------|----|----|----|
| 数据类型 | 关系型（SQLite） | 键值对 | 文件/对象 |
| 读取速度 | 快（毫秒级） | 极快（毫秒级） | 取决于文件大小 |
| 一致性 | 强一致 | 最终一致 | 强一致 |
| 查询能力 | SQL 完整支持 | 仅 key 查询 | 前缀/列表 |
| 关联查询 | 支持 | 不支持 | 不支持 |
| 最大大小 | 2GB/10GB | 1GB（免费） | 无限 |
| 出口流量费 | 无 | 无 | **无（核心优势）** |
| 适合场景 | 用户数据、订单 | 缓存、Session | 文件存储 |

## 实际使用场景

### 场景一：用户注册

```
用户提交 → D1 INSERT user → KV PUT session → 返回
```

### 场景二：AI 图片生成

```
用户提交 → D1 UPDATE credits → D1 INSERT task → Atlas API → R2 PUT image → D1 UPDATE status
```

### 场景三：页面加载

```
用户访问 → KV GET 缓存 → 命中返回 / 未命中 → DB 查询 → 渲染
```

## 总结

多数据库不是越多越好，关键是根据数据的**访问模式**选型：

- 需要关联查询 → D1
- 需要极快读写 + 过期 → KV
- 需要存储大文件 + 免流量 → R2

Cloudflare 的这三件套覆盖了 SaaS 产品 90% 的数据需求，而且全部按量付费，非常适合独立开发者。

## 参考资源

[Cloudflare D1](https://developers.cloudflare.com/d1/) 
[Cloudflare KV](https://developers.cloudflare.com/kv/) 
[Cloudflare R2](https://developers.cloudflare.com/r2/) 
[Drizzle ORM](https://orm.drizzle.team/)