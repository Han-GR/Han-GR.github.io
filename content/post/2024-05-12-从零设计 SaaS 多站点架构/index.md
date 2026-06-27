---
title: 从零设计 SaaS 多站点架构
description: ""
slug: 2024-05-12-从零设计 SaaS 多站点架构
date: 2024-05-12
image: SaaS.jpg
categories:
  - 架构设计
  - 前端
tags:
  - SaaS
  - RuiToolAI
---
## 问题背景

开发 RuiToolAI 的目标之一，是做一个**可复用的 SaaS 模板**。每次上线新站，只需要改品牌配置、首页文案、一个主功能模块，以及对应的第三方 API 配置。

这意味着架构需要支持：

- 多个站点共享同一套代码
- 每个站点有独立的品牌、文案、功能模块
- 第三方 API 配置可以按站点隔离

## 核心设计思路

核心思路是 **"配置驱动"**：把变化的抽成配置，不变的复用。

```
共享代码（框架、认证、支付、数据库）
    └── 站点配置（品牌、功能、API）
         └── 具体站点实例（RuiToolAI、xxxxx、...）
```

## 目录结构

```
src/
├── app/                    # Next.js App Router
│   ├── (marketing)/        # Landing 页面（共用）
│   ├── (auth)/             # 认证页面（共用）
│   ├── (settings)/         # 设置页面（共用）
│   ├── (admin)/            # 管理后台（共用）
│   └── (sites)/            # 站点功能页面
│       ├── image-gen/      # 图片生成站点
│       └── history/        # 历史记录（共用）
├── sites/                  # 站点模块定义
│   ├── image-gen/          # 图片生成模块
│   │   ├── site.config.ts  # 站点配置
│   │   ├── site.actions.ts # Server Actions
│   │   └── site.client.tsx # 客户端组件
│   ├── primary/            # 主站点（fallback）
│   └── types.ts            # 站点类型定义
├── site/                   # 全局站点配置
│   ├── config.ts           # 环境变量读取
│   └── marketing.ts        # 营销配置
├── components/             # 共享组件
│   ├── landing/            # Landing 组件
│   └── ui/                 # UI 组件
└── db/                     # 数据库
    └── schema.ts
```

## 站点配置系统

### 类型定义

```typescript
// src/sites/types.ts
export interface SiteConfig {
  name: string;
  tagline: string;
  description: string;
  path: string;
  billing: {
    creditsPerUse: number;
    usageDescription: string;
  };
}
```

### 站点注册

每个站点模块导出自己的配置：

```typescript
// src/sites/image-gen/site.config.ts
export const imageGenSiteConfig: SiteConfig = {
  name: "AI Image Generator",
  tagline: "Turn your ideas into stunning visuals",
  description: "Professional-grade AI image generation",
  path: "/image-gen",
  billing: {
    creditsPerUse: 10,
    usageDescription: "AI image generation",
  },
};
```

## 功能模块的插件化

每个站点模块是一套独立的文件，包含：

### site.config.ts

站点的元数据配置：

```typescript
export const imageGenSiteConfig = {
  name: "AI Image Generator",
  tagline: "Turn your ideas into stunning visuals",
  billing: {
    creditsPerUse: 10,
    usageDescription: "AI image generation",
  },
};
```

### site.actions.ts

站点的 Server Actions（后端逻辑）：

```typescript
"use server";

export const generateImageAction = actionClient
  .inputSchema(generateImageSchema)
  .action(async ({ parsedInput }) => {
    // 扣积分、调用 AI、返回结果
  });
```

### site.client.tsx

站点的前端组件：

```typescript
"use client";

export function ImageGenClient({ isLoggedIn }: { isLoggedIn: boolean }) {
  // 输入框、提交按钮、结果展示
}
```

### 页面渲染

```typescript
// src/app/(sites)/image-gen/page.tsx
export default async function ImageGenPage() {
  return (
    <section>
      <h1>{imageGenSiteConfig.name}</h1>
      <ImageGenClient isLoggedIn={Boolean(userId)} />
    </section>
  );
}
```

## 品牌配置

品牌相关的配置通过环境变量管理：

```typescript
// src/site/config.ts
export const siteConfig = {
  branding: {
    siteName: process.env.NEXT_PUBLIC_SITE_NAME ?? "",
    siteDescription: process.env.NEXT_PUBLIC_SITE_DESCRIPTION ?? "",
    socialXUrl: process.env.NEXT_PUBLIC_SOCIAL_X_URL ?? "",
    socialInstagramUrl: process.env.NEXT_PUBLIC_SOCIAL_INSTAGRAM_URL ?? "",
    contactEmail: process.env.NEXT_PUBLIC_CONTACT_EMAIL ?? "",
  },
};
```

### 营销页面配置

```typescript
// src/site/marketing.ts
export const siteMarketing = {
  homeSections: {
    hero: true,
    features: true,
    pricing: true,
    testimonials: true,
    faq: true,
  },
  hero: {
    titleText: "AI-Powered Creative Platform",
    descriptionText: "Generate professional images in seconds",
  },
  features: {
    items: [
      { title: "Multi-Model AI", description: "..." },
    ],
  },
  testimonials: {
    items: [
      { body: "Great tool!", author: { name: "Sarah", avatarUrl: "..." } },
    ],
  },
};
```

## 实际案例：从模板到新站点

假设要创建一个叫 "video-gen" 的新站点，用不同的 AI 模型：

**步骤 1：** 复制 `sites/image-gen/` 为 `sites/video-gen/`

**步骤 2：** 修改 `site.config.ts`：

```typescript
export const videoGenSiteConfig = {
  name: "video-gen",
  tagline: "Professional design at your fingertips",
  billing: {
    creditsPerUse: 5,
    usageDescription: "video generation",
  },
};
```

**步骤 3：** 修改 `site.actions.ts` 里的 AI 模型配置：

```typescript
const IMAGE_GEN_MODEL = "different-model-v2";
```

**步骤 4：** 添加页面路由 `src/app/(sites)/video-gen/page.tsx`

**步骤 5：** 修改环境变量，改品牌名和社交链接

**完成！** 不需要修改任何共享代码（认证、支付、数据库）。

## 总结

多站点架构的核心原则：

1. **配置驱动**：变的抽成配置，不变的复用
2. **模块化**：每个站点功能是独立的模块
3. **环境变量隔离**：品牌、API Key 通过环境变量注入
4. **共享基础设施**：认证、支付、数据库、CMS 全部共用

这样每次上线新站，只需要改配置 + 一个功能模块，其他全部复用。