---
title: Vinext：用 Vite 替代 Next.js 构建
description: ""
slug: 2026-03-21-Vinext：用 Vite 替代 Next.js 构建
date: 2026-03-21
image: vinext.png
categories:
  - 运维&云原生
  - 效率&工具
tags:
  - Vinext
  - cloudflare
  - RuiToolAI
---
## 为什么要迁移到 Vinext

RuiToolAI 最初是用标准 Next.js 开发的。但部署到 Cloudflare Workers 时遇到了一个根本问题：**Next.js 需要 Node.js 运行时，Workers 基于 V8 引擎，它们不兼容。**

当时只有一个选择：

**OpenNext + SST**：用 OpenNext 把 Next.js 编译成 Workers 兼容代码

但是，现在有了新的选择：Vinext

## Vinext 是什么

2026年2月24日，Cloudflare 正式发布了 Vinext。这也是从 OpenNext 迁移过来的直接契机——官方工具出来了，没理由继续用绕弯子的方案。

Vinext 是 Cloudflare 的一个实验性项目，核心思路是：**用 Vite 实现 Next.js 的公开 API 接口**。

它保留了 Next.js 的 App Router、React Server Components、Server Actions、Route Handlers 等核心功能，但底层构建换成了 Vite。

```bash
# 开发 — 和 Next.js 一样的体验
pnpm dev

# 构建
pnpm build

# 部署
pnpm deploy
```

选择 Vinext，因为：

- Cloudflare 官方维护，长期支持更有保障
- Vite 构建速度更快
- 对 Workers 的原生支持更好

## 迁移过程

### 1. 安装 Vinext

```bash
pnpm add vinext
```

### 2. 替换 package.json 里的脚本

```json
{
  "scripts": {
    "dev": "vinext dev",
    "build": "vinext build",
    "start": "vinext start",
    "deploy": "vinext deploy"
  }
}
```

### 3. 创建 worker-entrypoint.ts

Vinext 需要这个文件作为 Workers 的入口：

```typescript
import { getCloudflareContext } from "./src/utils/cloudflare-context";
import { createHandler } from "vinext";

const handler = await createHandler({
  getCloudflareContext,
  // 配置 KV 缓存
  cache: {
    handler: new KVCacheHandler(env.NEXT_INC_CACHE_KV, {
      appPrefix: "ruitoolai",
    }),
  },
});

export default {
  fetch: handler.fetch,
  scheduled: async (event, env, ctx) => {
    ctx.waitUntil(cleanupExpiredImageRecords(env));
  },
};
```

### 4. 运行兼容性检查

```bash
pnpm run check:vinext
```

这个命令会扫描项目，找出不兼容的代码。比如：

- 使用了 Node.js 原生模块（`fs`, `net`, `crypto` 等）
- 使用了 Workers 不支持的 API

### 5. 修复兼容性问题

主要修复了三类问题：

**a) 替换 `crypto` 模块**

```typescript
// 之前
import crypto from "crypto";
const hash = crypto.createHash("sha256").update(data).digest("hex");

// 之后
const encoder = new TextEncoder();
const data = encoder.encode(input);
const hashBuffer = await crypto.subtle.digest("SHA-256", data);
const hash = Array.from(new Uint8Array(hashBuffer))
  .map(b => b.toString(16).padStart(2, "0"))
  .join("");
```

**b) 替换文件系统操作**

Workers 没有 `fs` 模块，文件操作改为 R2：

```typescript
// 之前：本地文件
import fs from "fs";
const buffer = fs.readFileSync("./template.html");

// 之后：R2 存储
const { env } = await getCloudflareContext();
const object = await env.USER_UPLOADS_BUCKET.get("template.html");
const buffer = await object.arrayBuffer();
```

**c) 处理 Edge Runtime 差异**

一些中间件和库需要适配 Edge Runtime：

```typescript
// vite.config.ts
export default defineConfig({
  ssr: {
    noExternal: ["@some-edge-incompatible-lib"],
  },
});
```

## 遇到的坑和解决方案

### 坑 1：`waitUntil` 里的 setTimeout 不生效

Workers 的 `setTimeout` 不会真正 sleep，导致后台轮询任务瞬间跑完。

**解决方案：** 改用前端轮询 Server Action。

### 坑 2：某些 npm 包在构建时报错

部分包在 Vite 构建时报 `dynamic require` 相关的错误。

**解决方案：** 在 `vite.config.ts` 中配置 `ssr.noExternal` 或 `optimizeDeps.exclude`。

### 坑 3：热更新偶尔失效

Vinext 的 dev server 热更新偶尔不生效，需要手动刷新。

**解决方案：** 这是已知问题，社区正在修复。临时方案是 `pnpm dev --force`。

## 性能对比

| 指标 | Next.js + OpenNext | Vinext |
|------|-------------------|--------|
| 开发启动 | 8-15s | 3-5s |
| 构建时间 | 45-60s | 20-30s |
| 热更新 | 200-500ms | 100-300ms |
| 生产包大小 | 2.5MB | 1.8MB |
| 冷启动 | 500ms | 300ms |

## 总结

Vinext 目前还处于实验阶段，但对于 RuiToolAI 这种项目来说，它已经足够稳定。迁移带来的好处是：

- 构建速度提升 50%
- 包体积减少 30%
- 与 Cloudflare Workers 的原生集成

如果你也在用 Next.js + Cloudflare Workers，Vinext 值得一试。但建议先在测试环境跑通完整流程，确认兼容性后再上生产。

## 参考资源

- [Vinext 官方文档](https://vinext.io/)  
- [Vinext GitHub](https://github.com/cloudflare/vinext) 
- [Cloudflare Workers](https://developers.cloudflare.com/workers/) 
- [Vite](https://vite.dev/)