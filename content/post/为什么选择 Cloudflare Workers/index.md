---
title: 为什么选择 Cloudflare Workers 而不是传统 VPS
description: ""
slug: 2024-03-26-为什么选择 Cloudflare Workers 而不是传统 VPS
date: 2024-03-26
image: cloudflare.png
categories:
  - 效率&工具
tags:
  - cloudflare
  - workers
---
## 背景

在开发 RuiTool AI 这个全栈 SaaS 项目时，面临一个核心问题：**部署在哪里**？

传统方案是买一台 VPS（如阿里云、AWS EC2），装 Docker，跑 Node.js 服务。但作为一个独立开发者，我不希望把时间花在运维上。于是我调研了 Cloudflare Workers 这个 Serverless 方案，最终决定把整个项目跑在它上面。

这篇文章就来聊聊为什么做出这个选择，以及实际使用中的经验和教训。

## 方案对比

### 传统 VPS 方案

**架构：**

```
用户 → Nginx → Node.js 服务 → 数据库
```

**优点：**

- 成熟稳定，生态丰富
- 无运行时限制，可以跑长时间任务
- 完全控制服务器环境

**缺点：**

- 需要自己管理服务器（安全补丁、系统更新、证书续期）
- 需要自己配置负载均衡和扩容
- 单地域部署，国内访问国外 VPS 延迟高，反之亦然
- 费用固定，即使没流量也要付月费

### Cloudflare Workers 方案

**架构：**

```
用户 → Cloudflare 全球边缘节点 → Workers → D1/KV/R2
```

**优点：**

- 全球 330+ 边缘节点，就近响应
- 零运维，不用管服务器
- 按量付费，没流量不花钱
- D1、KV、R2 原生集成，不需要额外配置连接

**缺点：**

- CPU 时间限制（免费版 10ms，付费版 30s）
- 不支持 WebSocket 长连接
- 生态相对较新，部分 npm 包不兼容

## Cloudflare Workers 的优势

### 1. 全球边缘部署，零冷启动

Workers 部署在 Cloudflare 的全球边缘网络上，用户请求会被路由到最近的节点处理。我测试过从国内访问部署在 Workers 上的 RuiTool AI，响应时间在 100-200ms 左右，而同样的应用部署在美西 VPS 上，延迟在 300-500ms。

### 2. 原生集成 D1/KV/R2

这一点非常重要。传统方案里，你需要：

- 配置数据库连接字符串
- 处理连接池
- 管理文件存储（如 S3 SDK）

而在 Workers 里，这些都是原生 binding：

```typescript
const { env } = await getCloudflareContext();

// D1 数据库
const db = getDB();

// KV 缓存
await env.NEXT_INC_CACHE_KV.put(key, value);

// R2 文件存储
await env.USER_UPLOADS_BUCKET.put(key, buffer);
```

### 3. GitHub Actions 自动化部署

推送代码到 GitHub → 自动 lint → typecheck → e2e 测试 → 构建 → 部署到 Workers → 执行 D1 迁移。整个过程全自动，不需要手动操作服务器。

## 成本分析

以 RuiTool AI 为例，目前的使用情况：

| 服务 | 免费额度 | 当前用量 | 费用 |
|------|---------|---------|------|
| Workers | 10 万请求/天 | ~5000/天 | 免费 |
| D1 | 5GB 存储 + 500 万读/天 | <1% | 免费 |
| KV | 1GB 存储 + 10 万读/天 | ~50% | 免费 |
| R2 | 10GB 存储 | <1GB | 免费 |

对比传统 VPS：
- 最低配置 VPS：$5-10/月
- Cloudflare Workers：**$0/月**（当前用量），流量增长后 $5/月 起步

## 踩过的坑

### 1. CPU 时间限制导致异步任务失败

最开始我用 `waitUntil` 在后台轮询 AI 生成结果，但 Workers 的 `setTimeout` 不会真正 sleep，导致轮询瞬间跑完。

**解决方案：** 改用前端轮询 Server Action。每次轮询是一个独立请求，不受 30 秒限制。

### 2. D1 不支持事务

SQLite 在 D1 上不支持多语句事务，这在处理积分退款时很麻烦。

**解决方案：** 补偿式设计。先扣积分，失败时插入退款记录补充余额。

### 3. 某些 npm 包不兼容

Workers 不支持 Node.js 原生模块（如 `fs`、`net`、`crypto` 的部分 API）。需要用 Web Crypto API 替代，或者找 Workers 兼容的替代包。

## 总结

| 场景 | 推荐方案 |
|------|---------|
| SaaS 产品、API 服务、静态网站 | Cloudflare Workers |
| 长时间任务（视频处理、模型训练） | 传统 VPS |
| 需要 WebSocket 长连接 | 传统 VPS |
| 低预算、低流量项目 | Cloudflare Workers |

对于 RuiTool AI 这种 SaaS 产品，Workers 是完美的选择。零运维、按量付费、全球低延迟，让独立开发者可以专注于产品本身。

## 参考资源

[Cloudflare Workers](https://developers.cloudflare.com/workers/) 
[Vinext](https://vinext.io/) 
[D1](https://developers.cloudflare.com/d1/) 
[R2](https://developers.cloudflare.com/r2/)