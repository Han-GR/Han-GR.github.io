---
title: Rate Limiting 通用封装：从 Action 到 API Route
description: ""
slug: 2025-05-25-Rate Limiting 通用封装：从 Action 到 API Route
date: 2025-05-25
image: Rate-Limiting.png
categories:
  - 架构设计
  - 效率&工具
  - 运维&云原生
tags:
  - Rate-Limiting
  - RuiToolAI
---

> 限流是防止滥用的第一道防线。聊聊如何封装一个通用的 Rate Limiting 方案。

## 为什么需要 Rate Limiting

没有限流的 SaaS 是脆弱的：
- 注册接口可以被脚本暴力注册
- 登录接口可以被撞库
- 邮件发送接口可以被滥用

RuiToolAI 在所有敏感操作上都加了限流。

## 底层：KV 实现滑动窗口

```typescript
// src/utils/rate-limit.ts

export async function checkRateLimit({
  key,
  options,
}: {
  key: string;
  options: RateLimitOptions;
}): Promise<RateLimitResult> {
  const { env } = await getCloudflareContext();
  const now = Math.floor(Date.now() / 1000);

  // 归一化 IP（IPv6 用 /64 子网）
  const normalizedKey = ipaddr.isValid(key) ? normalizeIP(key) : key;

  // 按时间窗口分组
  const windowKey = `rate-limit:${options.identifier}:${normalizedKey}:${Math.floor(
    now / options.windowInSeconds
  )}`;

  // 获取当前计数
  const currentCount = parseInt(
    (await env.NEXT_INC_CACHE_KV.get(windowKey)) || "0"
  );
  const reset = (Math.floor(now / options.windowInSeconds) + 1) * options.windowInSeconds;

  // 达到上限
  if (currentCount >= options.limit) {
    return {
      success: false,
      remaining: 0,
      reset,
      limit: options.limit,
    };
  }

  // 递增计数
  await env.NEXT_INC_CACHE_KV.put(windowKey, (currentCount + 1).toString(), {
    expirationTtl: options.windowInSeconds,
  });

  return {
    success: true,
    remaining: options.limit - (currentCount + 1),
    reset,
    limit: options.limit,
  };
}
```

核心设计：
- **滑动窗口**：用 `Math.floor(now / windowInSeconds)` 做时间窗口分组
- **KV 自动过期**：`expirationTtl` 让窗口自动清理
- **返回 `reset` 时间**：前端可以告诉用户"请 X 分钟后重试"

## 封装层：withRateLimit

```typescript
// src/utils/with-rate-limit.ts

export async function withRateLimit<T>(
  action: () => Promise<T>,
  config: RateLimitConfig
): Promise<T> {
  // 非生产环境或测试模式跳过
  if (!isProd || isTestMode()) {
    return action();
  }

  const ip = await getIP();
  const key = config.userIdentifier || ip || UNKNOWN_IP_RATE_LIMIT_KEY;

  const rateLimitResult = await checkRateLimit({
    key,
    options: {
      identifier: config.identifier,
      limit: config.limit,
      windowInSeconds: config.windowInSeconds,
    },
  });

  if (!rateLimitResult.success) {
    throw new RateLimitError(
      Math.max(0, Math.ceil(rateLimitResult.reset - Date.now() / 1000))
    );
  }

  return action();
}
```

`RateLimitError` 包含 `retryAfterSeconds`，前端可以展示剩余等待时间。

## 预定义限流配置

```typescript
export const RATE_LIMITS = {
  SIGN_IN: {
    identifier: "sign-in",
    limit: 15,           // 每 60 分钟最多 15 次
    windowInSeconds: Math.floor(ms("60 minutes") / 1000),
  },
  SIGN_UP: {
    identifier: "sign-up",
    limit: 3,            // 每 1 小时最多 3 次注册
    windowInSeconds: Math.floor(ms("1 hour") / 1000),
  },
  EMAIL: {
    identifier: "email",
    limit: 10,           // 每 1 小时最多 10 封邮件
    windowInSeconds: Math.floor(ms("1 hour") / 1000),
  },
  FORGOT_PASSWORD: {
    identifier: "forgot-password",
    limit: 4,            // 每 1 小时最多 4 次
    windowInSeconds: Math.floor(ms("1 hour") / 1000),
  },
  SETTINGS: {
    identifier: "settings",
    limit: 15,           // 每 5 分钟最多 15 次
    windowInSeconds: Math.floor(ms("5 minutes") / 1000),
  },
  // ... 更多配置
};
```

每个业务场景有独立的限流配置，互不影响。

## IPv6 归一化

IPv6 地址一个用户可能有多个（因为隐私扩展），直接用原始 IP 做限流 key 会导致绕过。解决方法是无视后 64 位：

```typescript
function normalizeIP(ip: string): string {
  try {
    const addr = ipaddr.parse(ip);

    if (addr.kind() === "ipv6") {
      const ipv6 = addr as ipaddr.IPv6;
      const bytes = ipv6.toByteArray();
      // 后 64 位清零
      for (let i = 8; i < 16; i++) {
        bytes[i] = 0;
      }
      return `${ipaddr.fromByteArray(bytes).toString()}/64`;
    } else {
      return addr.toString();
    }
  } catch {
    return ip;
  }
}
```

## 总结

- 基于 KV 的滑动窗口限流，无需额外基础设施
- `withRateLimit` 封装统一入口，调用方只需传配置
- 每个业务场景独立限流，互不影响
- IPv6 归一化防止绕过
- 开发/测试环境自动跳过，不干扰日常开发
