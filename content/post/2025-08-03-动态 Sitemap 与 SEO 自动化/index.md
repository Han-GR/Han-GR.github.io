---
title: 动态 Sitemap 与 SEO 自动化
description: ""
slug: 2025-08-03-动态 Sitemap 与 SEO 自动化
date: 2025-08-03
image: sitemap.jpg
categories:
  - AI&数据
  - 效率&工具
tags:
  - sitemap
  - RuiToolAI
  - SEO
---

> 一个新博客发布后，自动进入 sitemap，自动配置 SEO 元数据。这些都是代码生成的，不是手写的。

## Sitemap 自动生成

RuiToolAI 的 sitemap 是全自动的，每次 CMS 内容变更后重新生成：

```typescript
// src/app/sitemap.ts

import { getCmsEntriesForSitemap } from "@/lib/cms/cms-repository";
import { SITE_URL } from "@/constants";
import { withKVCache, CACHE_KEYS } from "@/utils/with-kv-cache";

export async function GET() {
  const sitemap = await withKVCache(
    async () => {
      const entries = await getCmsEntriesForSitemap();

      const urls = entries.map((entry) => ({
        url: `${SITE_URL}/${entry.collection}/${entry.slug}`,
        lastModified: entry.updatedAt || entry.createdAt,
        changeFrequency: entry.collection === "blog" ? "weekly" : "monthly",
        priority: entry.collection === "docs" ? 0.9 : 0.7,
      }));

      // 静态页面
      urls.push(
        { url: SITE_URL, lastModified: new Date(), changeFrequency: "daily", priority: 1.0 },
        { url: `${SITE_URL}/blog`, lastModified: new Date(), changeFrequency: "daily", priority: 0.8 },
        { url: `${SITE_URL}/docs`, lastModified: new Date(), changeFrequency: "weekly", priority: 0.9 },
      );

      return urls;
    },
    { key: CACHE_KEYS.SITEMAP, ttl: "1h" }
  );

  // 生成 XML
  const xml = generateSitemapXml(sitemap);
  return new Response(xml, {
    headers: {
      "Content-Type": "application/xml",
      "Cache-Control": "public, max-age=3600, s-maxage=3600",
    },
  });
}
```

关键设计：
- **动态生成**：从数据库读取所有发布的 CMS 内容
- **KV 缓存**：1 小时缓存，避免每次请求都查数据库
- **优先级区分**：文档 > 博客 > 其他
- **自动更新频率**：博客 `weekly`，文档 `monthly`

## Robots.txt 配置

```typescript
// src/app/robots.ts

import { SITE_URL } from "@/constants";

export default function robots() {
  return {
    rules: {
      userAgent: "*",
      allow: "/",
      disallow: ["/admin/", "/api/", "/dashboard/"],
    },
    sitemap: `${SITE_URL}/sitemap.xml`,
  };
}
```

搜索引擎可以爬取公开页面，但不允许爬取管理后台、API 和用户面板。

## SEO 元数据自动化

每个 CMS 内容页面的 SEO 元数据都是自动生成的：

```typescript
// src/app/(marketing)/blog/[slug]/page.tsx

export async function generateMetadata({
  params,
}: {
  params: { slug: string };
}): Promise<Metadata> {
  const entry = await getCmsEntryBySlug("blog", params.slug);

  if (!entry) {
    return { title: "Not Found" };
  }

  return {
    title: entry.seoTitle || entry.title,
    description: entry.seoDescription || entry.excerpt,
    openGraph: {
      title: entry.seoTitle || entry.title,
      description: entry.seoDescription || entry.excerpt,
      type: "article",
      publishedTime: entry.createdAt?.toISOString(),
      modifiedTime: entry.updatedAt?.toISOString(),
      images: entry.featuredImage
        ? [{ url: entry.featuredImage }]
        : undefined,
    },
    alternates: {
      canonical: `${SITE_URL}/blog/${entry.slug}`,
    },
  };
}
```

字段优先级：`seoTitle` > `title` > 默认值。作者可以在 CMS 中手动覆盖 SEO 字段。

## CMS 内容自动收录

CMS 配置中定义了哪些集合需要被收录到 sitemap：

```typescript
// cms.config.ts

const blogCollection = {
  slug: "blog",
  includeInSitemap: true, // 自动收录
  previewUrl: (slug: string) => `/blog/${slug}`,
} satisfies DefineCmsCollection;

const docsCollection = {
  slug: "docs",
  includeInSitemap: true,
  enableSearch: true, // 启用文档搜索
} satisfies DefineCmsCollection;
```

只有 `includeInSitemap: true` 的集合才会出现在 sitemap 中。

## 总结

- Sitemap 从数据库动态生成，1 小时缓存
- Robots.txt 屏蔽管理后台、API 等非公开页面
- SEO 元数据从 CMS 字段自动生成，支持手动覆盖
- CMS 配置中控制哪些内容被收录