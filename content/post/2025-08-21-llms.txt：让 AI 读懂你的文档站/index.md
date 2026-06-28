---
title: llms.txt：让 AI 读懂你的文档站
description: ""
slug: 2025-08-21-llms.txt：让 AI 读懂你的文档站
date: 2025-08-21
image: llms.png
categories:
  - 效率&工具
  - AI&数据
tags:
  - LLM
  - llms
  - SEO
  - sitemap
  - RuiToolAI
---

> llms.txt 是给 AI 看的 sitemap。一行代码，让 LLM 能结构化地理解你的整个文档站。


## 什么是 llms.txt

`llms.txt` 是一个新兴的 Web 标准，类似于 `robots.txt` 但面向 AI/LLM。

它的作用：**让 AI 模型（如 ChatGPT、Claude）能高效地爬取和理解你的文档站结构**。

当用户问 AI "这个产品怎么用"时，AI 可以先读 `llms.txt` 了解文档结构，再精准获取需要的页面。

## 为什么需要它

传统方式下，AI 需要爬取整个文档站来确定结构，效率低，token 消耗大。`llms.txt` 提供一个结构化的索引：

1. 列出所有文档页面的标题和 URL
2. 提供简要描述
3. 按层级组织

## 实现方式

```typescript
// src/app/(marketing)/docs/llms.txt/route.ts

import { buildDocsLlmsTxtContent } from "@/lib/cms/build-docs-llms-txt";
import { getCmsNavigationTree } from "@/lib/cms/cms-navigation-repository";
import { DOCS_SLUG } from "@/lib/cms/docs-config";
import { SITE_URL } from "@/constants";

export async function GET() {
  const tree = await getCmsNavigationTree({
    navigationKey: DOCS_SLUG,
  });

  if (tree.length === 0) {
    return Response.redirect(SITE_URL, 302);
  }

  const body = buildDocsLlmsTxtContent(tree);

  return new Response(body, {
    headers: {
      "content-type": "text/plain; charset=utf-8",
      "cache-control": "public, s-maxage=3600, stale-while-revalidate=86400",
    },
  });
}
```

核心逻辑：
1. 从 CMS 导航树获取文档结构
2. 构建 `llms.txt` 格式的内容
3. 返回纯文本响应

## 内容格式

`llms.txt` 的内容基于导航树生成：

```typescript
// src/lib/cms/build-docs-llms-txt.ts

export function buildDocsLlmsTxtContent(
  tree: CmsNavigationNode[]
): string {
  const lines: string[] = [];

  lines.push(`# ${SITE_NAME} Documentation`);
  lines.push("");

  for (const section of tree) {
    // 一级标题
    lines.push(`## ${section.title}`);
    if (section.description) {
      lines.push(`> ${section.description}`);
    }
    lines.push("");

    // 子页面
    if (section.children) {
      for (const child of section.children) {
        const url = buildDocUrl(child.slug);
        lines.push(`- [${child.title}](${url}): ${child.description || ""}`);
      }
      lines.push("");
    }
  }

  lines.push("---");
  lines.push(`Generated at: ${new Date().toISOString()}`);
  lines.push(`Base URL: ${SITE_URL}`);

  return lines.join("\n");
}
```

生成的 `llms.txt` 类似：

```
# RuiToolAI Documentation

## Getting Started
> Quick start guide for new users

- [Installation](https://ruitool.ai/docs/installation): How to install and set up
- [Quick Start](https://ruitool.ai/docs/quick-start): Build your first app in 5 minutes

## API Reference
> Complete API documentation

- [Authentication](https://ruitool.ai/docs/api/auth): API authentication methods
- [Endpoints](https://ruitool.ai/docs/api/endpoints): Available API endpoints
```

## 缓存策略

```typescript
headers: {
  "cache-control": "public, s-maxage=3600, stale-while-revalidate=86400",
}
```

- `s-maxage=3600`：CDN 缓存 1 小时
- `stale-while-revalidate=86400`：缓存过期后仍可使用旧内容，后台异步更新

不需要实时更新，1 小时刷新一次足够。

## 总结

- `llms.txt` 是 AI 的文档索引，类似 `robots.txt` 对搜索引擎的作用
- 从 CMS 导航树自动生成，文档结构变了自动更新
- 纯文本格式，一个请求就能让 AI 理解整个文档站
- CDN 缓存 1 小时，几乎零成本
