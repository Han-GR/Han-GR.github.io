---
title: Feature Flags 运行时开关实践
description: ""
slug: 2024-06-02-Feature Flags 运行时开关实践
date: 2024-06-02
image: FeatureFlags.png
categories:
  - 运维&云原生
  - 效率&工具
tags:
  - FeatureFlags
  - RuiToolAI
---
## 为什么需要 Feature Flags

Feature Flags 解决几个问题：

1. **环境隔离**：开发环境不需要 Turnstile 验证码，生产环境需要
2. **渐进发布**：新功能先对部分用户开放
3. **快速回滚**：出问题时关闭 Flag，不需要回滚代码

## 实现方式

RuiToolAI 的 Feature Flags 基于环境变量：

```typescript
// src/flags.ts

import "server-only";
import { cache } from "react";
import { isTestMode } from "@/utils/is-test-mode";

export async function isGoogleSSOEnabled() {
  return Boolean(
    process.env.GOOGLE_CLIENT_ID && process.env.GOOGLE_CLIENT_SECRET
  );
}

export async function isTurnstileEnabled() {
  if (isTestMode()) {
    return false;
  }

  return Boolean(process.env.TURNSTILE_SECRET_KEY);
}

// 集中导出的配置对象
export const getConfig = cache(async () => {
  return {
    isGoogleSSOEnabled: await isGoogleSSOEnabled(),
    isTurnstileEnabled: await isTurnstileEnabled(),
  };
});
```

设计原则：
- **每个 Flag 一个函数**：语义清晰，调用方一眼就知道在检查什么
- **test mode 自动跳过**：测试环境不触发外部依赖的 Flag
- **React cache**：同一请求内多次调用只执行一次

## 项目中的 Flags

| Flag | 用途 | 条件 |
|------|------|------|
| `isTurnstileEnabled` | 验证码开关 | 有 `TURNSTILE_SECRET_KEY` 且非测试模式 |
| `isGoogleSSOEnabled` | Google 登录 | 有 `GOOGLE_CLIENT_ID` 和 `GOOGLE_CLIENT_SECRET` |
| `DISABLE_CREDIT_BILLING_SYSTEM` | 积分系统 | 配置项，可关闭整个积分/计费模块 |

### 使用示例

```typescript
// 在 Server Action 中
if (await isTurnstileEnabled()) {
  const success = await validateTurnstileToken(input.captchaToken);
  if (!success) {
    throw new ActionError("INPUT_PARSE_ERROR", "Please complete the captcha");
  }
}
```

```typescript
// 在前端组件中
export async function SignInPage() {
  const config = await getConfig();

  return (
    <div>
      {/* 密码登录 */}
      <SignInForm />

      {/* 只在启用 Google SSO 时显示 */}
      {config.isGoogleSSOEnabled && <GoogleSignInButton />}
    </div>
  );
}
```

## 配置缓存

`getConfig` 用 `React.cache()` 包裹，同一请求内多次调用不会重复读取环境变量：

```typescript
export const getConfig = cache(async () => {
  return {
    isGoogleSSOEnabled: await isGoogleSSOEnabled(),
    isTurnstileEnabled: await isTurnstileEnabled(),
  };
});
```

在 Server Component 中，可以在页面顶部获取一次 config，然后传给子组件：

```typescript
export default async function Page() {
  const config = await getConfig();

  return (
    <>
      <Header config={config} />
      <Content config={config} />
    </>
  );
}
```

## 总结

- 基于环境变量的 Feature Flags，简单可靠
- 每个 Flag 独立函数，语义清晰
- `React.cache()` 避免重复读取
- 测试模式自动跳过外部依赖的 Flag