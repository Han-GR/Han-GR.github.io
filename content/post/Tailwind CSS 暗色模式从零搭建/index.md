---
title: Tailwind CSS 暗色模式从零搭建
description: ""
slug: 2024-11-23-Tailwind CSS 暗色模式从零搭建
date: 2024-11-23
image: Tailwind.jpg
categories:
  - 前端
tags:
  - Tailwind
  - CSS
  - Shadcn
---

> 暗色模式不只是加个 `dark:` 前缀那么简单。这篇记录我在 RuiTool AI 里搭建完整暗色模式体系的过程。

## 为什么暗色模式容易搞乱

暗色模式最常见的问题：

1. **颜色硬编码**：`text-gray-900` 在暗色下看不见
2. **闪烁（FOUC）**：页面加载时先显示亮色再切换到暗色
3. **图片/图标不适配**：亮色图标在暗色背景上消失
4. **第三方组件不跟随**：自己的组件暗色了，Shadcn 组件还是亮色

---

## Tailwind 暗色模式配置

Tailwind 支持两种暗色模式策略：

```javascript
// tailwind.config.ts
export default {
  // 方案一：跟随系统（media）
  darkMode: "media",

  // 方案二：手动切换（class）—— 推荐
  darkMode: "class",
};
```

推荐用 `class` 方案，因为：
- 用户可以手动切换，不受系统设置影响
- 可以持久化到 localStorage
- SSR 时可以从 cookie 读取，避免闪烁

---

## CSS 变量 + Tailwind 的组合方案

硬编码颜色是暗色模式的大敌。正确做法是用 CSS 变量定义语义化颜色：

```css
/* src/app/globals.css */
@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --card: 0 0% 100%;
    --card-foreground: 222.2 84% 4.9%;
    --primary: 221.2 83.2% 53.3%;
    --primary-foreground: 210 40% 98%;
    --muted: 210 40% 96.1%;
    --muted-foreground: 215.4 16.3% 46.9%;
    --border: 214.3 31.8% 91.4%;
    --radius: 0.5rem;
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    --card: 222.2 84% 4.9%;
    --card-foreground: 210 40% 98%;
    --primary: 217.2 91.2% 59.8%;
    --primary-foreground: 222.2 47.4% 11.2%;
    --muted: 217.2 32.6% 17.5%;
    --muted-foreground: 215 20.2% 65.1%;
    --border: 217.2 32.6% 17.5%;
  }
}
```

然后在 `tailwind.config.ts` 里把这些变量映射成 Tailwind 颜色：

```typescript
// tailwind.config.ts
import type { Config } from "tailwindcss";

const config: Config = {
  darkMode: "class",
  theme: {
    extend: {
      colors: {
        background: "hsl(var(--background))",
        foreground: "hsl(var(--foreground))",
        card: {
          DEFAULT: "hsl(var(--card))",
          foreground: "hsl(var(--card-foreground))",
        },
        primary: {
          DEFAULT: "hsl(var(--primary))",
          foreground: "hsl(var(--primary-foreground))",
        },
        muted: {
          DEFAULT: "hsl(var(--muted))",
          foreground: "hsl(var(--muted-foreground))",
        },
        border: "hsl(var(--border))",
      },
    },
  },
};
```

这样写组件时只需要用语义化类名，暗色模式自动切换：

```tsx
// 不需要写 dark: 前缀
<div className="bg-background text-foreground">
  <p className="text-muted-foreground">副标题</p>
</div>
```

---

## 主题切换实现

### 避免闪烁的关键

闪烁（FOUC）发生在：服务端渲染了亮色 HTML，客户端 JS 加载后才切换到暗色，用户看到一闪。

解决方案：在 `<head>` 里内联一段脚本，在页面渲染前就设置好 `class`：

```typescript
// src/app/layout.tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="zh" suppressHydrationWarning>
      <head>
        {/* 内联脚本，在 HTML 解析时立即执行，避免闪烁 */}
        <script
          dangerouslySetInnerHTML={{
            __html: `
              (function() {
                try {
                  var theme = localStorage.getItem('theme');
                  if (theme === 'dark' || (!theme && window.matchMedia('(prefers-color-scheme: dark)').matches)) {
                    document.documentElement.classList.add('dark');
                  }
                } catch (e) {}
              })();
            `,
          }}
        />
      </head>
      <body>{children}</body>
    </html>
  );
}
```

`suppressHydrationWarning` 是必须的，因为服务端和客户端的 `class` 可能不同。

### 主题切换组件

```typescript
// src/components/theme-toggle.tsx
"use client";
import { useEffect, useState } from "react";
import { SunIcon, MoonIcon } from "@heroicons/react/24/outline";

export function ThemeToggle() {
  const [isDark, setIsDark] = useState(false);

  useEffect(() => {
    // 初始化时读取当前主题
    setIsDark(document.documentElement.classList.contains("dark"));
  }, []);

  function toggle() {
    const next = !isDark;
    setIsDark(next);
    if (next) {
      document.documentElement.classList.add("dark");
      localStorage.setItem("theme", "dark");
    } else {
      document.documentElement.classList.remove("dark");
      localStorage.setItem("theme", "light");
    }
  }

  return (
    <button
      onClick={toggle}
      className="rounded-md p-2 hover:bg-muted transition-colors"
      aria-label={isDark ? "切换到亮色模式" : "切换到暗色模式"}
    >
      {isDark ? (
        <SunIcon className="h-5 w-5" />
      ) : (
        <MoonIcon className="h-5 w-5" />
      )}
    </button>
  );
}
```

---

## Shadcn UI 的暗色模式集成

Shadcn UI 的组件本身已经支持暗色模式，因为它们内部用的就是 CSS 变量（`bg-background`、`text-foreground` 等）。

只要你的 CSS 变量配置正确，Shadcn 组件会自动跟随主题切换。

但有一个坑：**Shadcn 的 `cn()` 工具函数**。

```typescript
// src/lib/utils.ts
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

当你用 `dark:` 前缀覆盖 Shadcn 组件的默认样式时，`twMerge` 会正确处理冲突，不会出现样式叠加问题。

---

## 常见坑点

### 坑 1：图片在暗色下太亮

```tsx
// 给图片加暗色滤镜
<img
  src="/logo.png"
  className="dark:brightness-90 dark:invert"
  alt="Logo"
/>
```

### 坑 2：渐变在暗色下不好看

```tsx
// 亮色：浅蓝渐变；暗色：深蓝渐变
<div className="bg-gradient-to-r from-blue-50 to-indigo-50 dark:from-blue-950 dark:to-indigo-950">
```

### 坑 3：边框在暗色下消失

```tsx
// 用语义化 border 颜色
<div className="border border-border">
  {/* border 颜色会随主题切换 */}
</div>
```

### 坑 4：第三方库的弹窗/Portal 不跟随主题

有些第三方库把弹窗挂载到 `document.body`，不在 `.dark` 类的作用范围内。

解决方案：把 `.dark` 类加到 `<html>` 而不是 `<body>`，这样所有子元素都能继承。

---

## 实战：给 Testimonials 组件加暗色支持

```tsx
// src/components/landing/testimonials.tsx
export function Testimonials() {
  return (
    <section className="py-16 bg-background">
      <div className="max-w-7xl mx-auto px-4">
        <h2 className="text-3xl font-bold text-center text-foreground mb-12">
          用户评价
        </h2>

        {/* 滚动容器 */}
        <div className="relative overflow-hidden">
          {/* 左侧渐变遮罩 */}
          <div className="absolute left-0 top-0 bottom-0 w-24 z-10 bg-gradient-to-r from-background to-transparent" />

          {/* 右侧渐变遮罩 */}
          <div className="absolute right-0 top-0 bottom-0 w-24 z-10 bg-gradient-to-l from-background to-transparent" />

          <div className="flex gap-6 animate-marquee hover:[animation-play-state:paused]">
            {testimonials.map((item, i) => (
              <div
                key={i}
                className="flex-shrink-0 w-80 p-6 rounded-xl border border-border bg-card text-card-foreground shadow-sm"
              >
                <p className="text-muted-foreground mb-4">{item.content}</p>
                <div className="flex items-center gap-3">
                  <img
                    src={item.avatar}
                    alt={item.name}
                    className="w-10 h-10 rounded-full object-cover"
                  />
                  <div>
                    <p className="font-medium text-foreground">{item.name}</p>
                    <p className="text-sm text-muted-foreground">{item.role}</p>
                  </div>
                </div>
              </div>
            ))}
          </div>
        </div>
      </div>
    </section>
  );
}
```

注意渐变遮罩用的是 `from-background`，这样在暗色模式下遮罩颜色也会自动切换，不会出现白色遮罩盖在暗色背景上的问题。

---

## 总结

搭建完整暗色模式体系的关键步骤：

1. **CSS 变量定义语义化颜色**：`:root` 和 `.dark` 分别定义
2. **Tailwind 映射 CSS 变量**：`tailwind.config.ts` 里配置
3. **内联脚本避免闪烁**：在 `<head>` 里提前设置 `class`
4. **组件用语义化类名**：`bg-background`、`text-foreground`，而不是 `bg-white`、`text-gray-900`
5. **渐变遮罩用语义化颜色**：`from-background` 而不是 `from-white`

---

## 参考资源

- [Tailwind CSS Dark Mode 文档](https://tailwindcss.com/docs/dark-mode)
- [Shadcn UI 主题配置](https://ui.shadcn.com/docs/theming)
- [next-themes 库](https://github.com/pacocoursey/next-themes)

