---
title: Cloudflare Cron 定时任务实战
description: ""
slug: 2025-06-09-Cloudflare Cron 定时任务实战
date: 2025-06-09
image: cloudflare-cron.png
categories:
  - 后端
  - 运维&云原生
tags:
  - cloudflare
  - Cron
  - Triggers
  - RuiToolAI
---

## 使用场景

RuiToolAI 的图片生成记录有 7 天有效期，R2 文件会自动删除，但 DB 里的记录不会自动清理。如果不清理，DB 会越来越大，而且历史页面会显示大量"已过期"的空白卡片。

解决方案：用 Cloudflare Cron Triggers 每天定时清理过期记录。

## 配置 Cron Trigger

在 `wrangler.jsonc` 中添加：

```jsonc
{
  "triggers": {
    "crons": ["0 3 * * *"]
  }
}
```

Cron 表达式格式：`分 时 日 月 周`

| 表达式 | 含义 |
|--------|------|
| `0 3 * * *` | 每天 UTC 凌晨 3 点 |
| `0 */6 * * *` | 每 6 小时 |
| `0 0 * * 0` | 每周日 UTC 0 点 |
| `*/5 * * * *` | 每 5 分钟 |

## 实现 scheduled 处理器

在 `worker-entrypoint.ts` 中添加 `scheduled` 方法：

```typescript
export default {
  ...worker,
  async scheduled(
    _event: ScheduledEvent,
    env: Env,
    ctx: ExecutionContext
  ): Promise<void> {
    ctx.waitUntil(cleanupExpiredImageRecords(env));
  },
};
```

注意：这里用 `ctx.waitUntil` 是正确的，因为 `scheduled` 处理器本身就是异步的，不像 HTTP 请求有 CPU 时间限制。

## 自动清理过期图片记录

```typescript
async function cleanupExpiredImageRecords(env: Env): Promise<void> {
  const { drizzle } = await import("drizzle-orm/d1");
  const schema = await import("./src/db/schema");
  const { generatedImageTable, IMAGE_GEN_STATUS } = schema;
  const { lt, eq, and } = await import("drizzle-orm");

  // 直接用 env.NEXT_TAG_CACHE_D1，不用 getDB()
  // 因为 scheduled 处理器里没有 cloudflare:workers 的全局 env
  const db = drizzle(env.NEXT_TAG_CACHE_D1, { schema });
  const now = new Date();

  const result = await db
    .delete(generatedImageTable)
    .where(
      and(
        eq(generatedImageTable.status, IMAGE_GEN_STATUS.COMPLETED),
        lt(generatedImageTable.expiresAt, now),
      ),
    )
    .returning({ id: generatedImageTable.id });

  console.log(`[cron] Cleaned up ${result.length} expired image records.`);
}
```

**为什么不用 `getDB()`？**

`getDB()` 内部用了 `cloudflare:workers` 的 `env`，这个 `env` 是通过 Cloudflare 的全局上下文注入的，只在 HTTP 请求处理中可用。`scheduled` 处理器里需要直接用参数传入的 `env`。

## R2 生命周期规则配合

R2 的生命周期规则负责删除文件，Cron 负责清理 DB 记录，两者配合：

**R2 设置（Cloudflare Dashboard）：**

```
对象生命周期规则
  规则名称：Delete user uploads after 7 days
  前缀：users/
  操作：7 天后删除对象
```

**时间线：**

```
Day 0：用户生成图片 → 存 R2 → DB 记录 expiresAt = Day 7
Day 7：R2 自动删除文件
Day 7 凌晨 3 点：Cron 清理 DB 中 expiresAt < now 的记录
```

这样 DB 和 R2 都保持干净。

## 本地调试

先 build 再用 wrangler dev：

```bash
# 方法一：部署到 Cloudflare 后在 Dashboard 手动触发
# Workers → 你的 Worker → Triggers → Cron Triggers → Run

# 方法二：本地 wrangler dev（需要先 build）
pnpm build
npx wrangler dev --port 8787

# 然后触发
curl "http://localhost:8787/__scheduled?cron=0+3+*+*+*"
```

**推荐方法一**，更简单，而且可以看到真实的 D1 数据。

## 总结

Cloudflare Cron Triggers 的使用要点：

1. 在 `wrangler.jsonc` 的 `triggers.crons` 中配置 Cron 表达式
2. 在 `worker-entrypoint.ts` 中实现 `scheduled` 方法
3. `scheduled` 处理器里用参数传入的 `env`，不用全局 `env`
4. 配合 R2 生命周期规则，实现文件和 DB 记录的双重清理

## 参考资源

- [Cloudflare Cron Triggers](https://developers.cloudflare.com/workers/configuration/cron-triggers/)
- [Cron 表达式](https://crontab.guru/) 
- [R2 生命周期规则](https://developers.cloudflare.com/r2/buckets/object-lifecycles/)