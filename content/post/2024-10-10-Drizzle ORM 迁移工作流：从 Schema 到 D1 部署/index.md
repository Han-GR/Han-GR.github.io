---
title: Drizzle ORM 迁移工作流：从 Schema 到 D1 部署
description: ""
slug: 2024-10-10-Drizzle ORM 迁移工作流：从 Schema 到 D1 部署
date: 2024-10-10
image: drizzle_orm.png
categories:
  - 数据库&缓存
  - 效率&工具
  - 运维&云原生
tags:
  - Drizzle
  - ORM
  - RuiToolAI
---

> Schema 改一行，一条命令生成 SQL，再一条命令部署到 D1。Drizzle 的迁移工作流让数据库变更从未如此简单。

## Drizzle 的迁移哲学

Drizzle 的迁移方式是 **声明式 Schema → 自动生成 SQL**。你只需要定义 Schema，Drizzle 对比新旧 Schema 的差异，自动生成迁移 SQL。

```typescript
// src/db/schema.ts
export const users = sqliteTable("users", {
  id: text("id").primaryKey(),
  email: text("email").notNull().unique(),
  firstName: text("first_name"),
  lastName: text("last_name"),
  passwordHash: text("password_hash"),
  emailVerified: integer("email_verified", { mode: "timestamp" }),
  createdAt: integer("created_at", { mode: "timestamp" })
    .notNull()
    .$defaultFn(() => new Date()),
});
```

## 项目中的迁移历史

RuiToolAI 从零开始经历了 12 次迁移：

| 迁移 | 内容 |
|------|------|
| 0000 | 初始化：users、sessions 表 |
| 0001 | 添加 emailVerified 字段 |
| 0003 | 添加 Google SSO 支持 |
| 0004 | 添加 Passkey 凭证表 |
| 0005 | OpenNext 缓存表 |
| 0006 | 积分计费系统 |
| 0007 | 支付意图 ID 和更新计数器 |
| 0008 | 多租户架构 |
| 0009 | CMS 系统表 |
| 0010 | 定价计划表 |
| 0011 | 移除旧的团队 Schema |
| 0012 | 添加 AI 图片生成表 |

每次迁移都是增量式的，从不回退。

## 工作流：三步走

### 第一步：修改 Schema

```typescript
// 在 src/db/schema.ts 中添加新字段
export const users = sqliteTable("users", {
  // ... 已有字段
  avatar: text("avatar"), // 新增
});
```

### 第二步：生成迁移文件

```bash
pnpm db:generate add-user-avatar
```

Drizzle Kit 会对比当前 Schema 和上一次迁移的快照，生成 `.sql` 文件：

```sql
-- 000X_add_user_avatar.sql
ALTER TABLE `users` ADD COLUMN `avatar` text;
```

### 第三步：部署到 D1

```bash
pnpm wrangler d1 migrations apply ruitoolai-db --remote
```

D1 会按顺序执行所有未应用的迁移。

## Schema 定义最佳实践

### 1. 使用 Drizzle 的枚举约束

```typescript
export const generatedImages = sqliteTable("generated_images", {
  status: text("status", {
    enum: ["pending", "processing", "completed", "failed"],
  }).notNull().default("pending"),
});
```

TypeScript 类型自动推导为 `"pending" | "processing" | "completed" | "failed"`。

### 2. 外键用 references

```typescript
export const generatedImages = sqliteTable("generated_images", {
  userId: text("user_id")
    .notNull()
    .references(() => users.id),
});
```

### 3. 时间戳统一用 `integer` + `{ mode: "timestamp" }`

D1 底层是 SQLite，没有原生 DateTime 类型。用整数时间戳 + `{ mode: "timestamp" }` 让 Drizzle 自动帮你做 Date 和数字的转换：

```typescript
createdAt: integer("created_at", { mode: "timestamp" })
  .notNull()
  .$defaultFn(() => new Date()),
```

### 4. 不要手动编辑迁移文件

迁移文件是 Drizzle Kit 自动生成的，手改会破坏快照一致性。如果迁移有问题，回退 Schema 修改，重新生成。

## D1 的特殊限制

D1 基于 SQLite，有一些需要注意的限制：

1. **不支持事务**：Drizzle 的事务 API 在 D1 上不可用。每条语句独立执行。
2. **ALTER TABLE 受限**：SQLite 的 ALTER TABLE 只能加列、重命名列，不能删除列或修改列类型。如果需要删列，得创建新表、迁移数据、删除旧表。
3. **并发写入**：D1 是单写入者，高并发写入需要做好重试。

## 总结

- Schema 即文档，修改后一条命令生成 SQL
- 迁移文件自动生成，不要手写
- 12 次增量迁移，每次只改一小部分
- 注意 D1 的 SQLite 限制，特别是事务和 ALTER TABLE