---
title: CSS 无限滚动 Marquee 动画实现
description: ""
slug: 2025-09-04-CSS 无限滚动 Marquee 动画实现
date: 2025-09-04
image: css.png
categories:
  - 前端
tags:
  - CSS
  - Tailwind
  - Marquee
  - RuiToolAI
---

> 不用任何 JS 库，纯 CSS 实现无缝循环滚动。这是 RuiToolAI 客户评价组件的实现记录。

## 效果目标

- 卡片从右向左无限滚动
- 滚动无缝衔接，看不出循环点
- 鼠标悬停时暂停
- 两侧有渐变遮罩，边缘自然淡出
- 支持暗色模式

---

## 核心原理

Marquee 动画的本质是：**用 CSS `translateX` 把一排元素从 0 移动到 -50%，然后瞬间重置到 0，循环播放。**

为什么是 -50%？因为我把内容复制了一份，总宽度是原来的 2 倍。移动 -50% 正好是一份内容的宽度，重置后用户看不出来。

```
原始内容：[A][B][C][D]
复制后：  [A][B][C][D][A][B][C][D]
          ←←←←←←←←←←←←←←←←←←←←
动画：从 translateX(0) 到 translateX(-50%)
重置：瞬间回到 translateX(0)，用户看到的还是 [A][B][C][D]
```

---

## 基础实现

先用纯 CSS 实现：

```css
/* 定义 marquee 动画 */
@keyframes marquee {
  from {
    transform: translateX(0);
  }
  to {
    transform: translateX(-50%);
  }
}

.marquee-track {
  display: flex;
  animation: marquee 30s linear infinite;
}

/* 容器需要 overflow-hidden */
.marquee-container {
  overflow: hidden;
}
```

```html
<div class="marquee-container">
  <div class="marquee-track">
    <!-- 原始内容 -->
    <div class="item">卡片 1</div>
    <div class="item">卡片 2</div>
    <div class="item">卡片 3</div>
    <!-- 复制一份，实现无缝循环 -->
    <div class="item">卡片 1</div>
    <div class="item">卡片 2</div>
    <div class="item">卡片 3</div>
  </div>
</div>
```

---

## 无缝循环的关键：复制元素

在 React 里，用 JS 复制数组：

```typescript
const items = ["卡片1", "卡片2", "卡片3"];
// 复制一份，总共 6 个元素
const doubled = [...items, ...items];
```

```tsx
<div className="flex animate-marquee">
  {doubled.map((item, i) => (
    <div key={i} className="flex-shrink-0 w-80">
      {item}
    </div>
  ))}
</div>
```

`flex-shrink-0` 很重要，防止卡片被压缩。

---

## 加入 Tailwind

在 `tailwind.config.ts` 里注册自定义动画：

```typescript
// tailwind.config.ts
import type { Config } from "tailwindcss";

const config: Config = {
  theme: {
    extend: {
      keyframes: {
        marquee: {
          from: { transform: "translateX(0)" },
          to: { transform: "translateX(-50%)" },
        },
      },
      animation: {
        // 30s 速度，linear 匀速，infinite 无限循环
        marquee: "marquee 30s linear infinite",
      },
    },
  },
};
```

然后直接用 `animate-marquee` 类名：

```tsx
<div className="flex animate-marquee">
  {/* 卡片 */}
</div>
```

---

## hover 暂停

Tailwind 支持 `[animation-play-state:paused]` 这种任意值写法：

```tsx
<div className="flex animate-marquee hover:[animation-play-state:paused]">
  {/* 鼠标悬停时暂停 */}
</div>
```

但有个问题：鼠标悬停在卡片上时，事件冒泡到父元素，动画暂停。但如果鼠标悬停在卡片之间的间隙，动画不会暂停。

更好的做法是把 hover 监听放在外层容器：

```tsx
<div className="group overflow-hidden">
  <div className="flex animate-marquee group-hover:[animation-play-state:paused]">
    {/* 卡片 */}
  </div>
</div>
```

`group` + `group-hover:` 让父元素的 hover 状态控制子元素的动画。

---

## 两侧渐变遮罩

渐变遮罩让边缘自然淡出，视觉上更好看：

```tsx
<div className="relative overflow-hidden max-w-7xl mx-auto">
  {/* 左侧遮罩 */}
  <div
    className="absolute left-0 top-0 bottom-0 w-24 z-10 pointer-events-none
                bg-gradient-to-r from-background to-transparent"
  />

  {/* 右侧遮罩 */}
  <div
    className="absolute right-0 top-0 bottom-0 w-24 z-10 pointer-events-none
                bg-gradient-to-l from-background to-transparent"
  />

  {/* 滚动内容 */}
  <div className="flex gap-6 animate-marquee">
    {/* 卡片 */}
  </div>
</div>
```

**关键细节**：
- `pointer-events-none`：遮罩不拦截鼠标事件，hover 暂停仍然有效
- `from-background`：用语义化颜色，暗色模式自动适配
- `z-10`：遮罩在卡片上层

---

## 响应式和暗色模式

卡片宽度在移动端可以小一点：

```tsx
<div className="flex-shrink-0 w-72 sm:w-80 p-4 sm:p-6 rounded-xl border border-border bg-card">
  <p className="text-sm sm:text-base text-muted-foreground">{content}</p>
</div>
```

暗色模式用语义化颜色，不需要额外的 `dark:` 前缀：

```tsx
// bg-card、text-card-foreground、border-border 都会随主题自动切换
<div className="bg-card text-card-foreground border border-border">
```

---

## 完整组件代码

```tsx
// src/components/landing/testimonials.tsx
"use client";

const testimonials = [
  {
    content: "RuiToolAI 的图片生成速度非常快，质量也很高。",
    name: "Sarah Chen",
    role: "UI 设计师",
    avatar: "https://images.unsplash.com/photo-1494790108377-be9c29b29330?w=100",
  },
  {
    content: "积分制度很合理，按需购买，不浪费。",
    name: "Marcus Johnson",
    role: "内容创作者",
    avatar: "https://images.unsplash.com/photo-1507003211169-0a1dd7228f2d?w=100",
  },
  // ... 更多评价
];

// 复制一份实现无缝循环
const doubled = [...testimonials, ...testimonials];

export function Testimonials() {
  return (
    <section className="py-16 bg-background overflow-hidden">
      <div className="max-w-7xl mx-auto px-4 mb-12">
        <h2 className="text-3xl font-bold text-center text-foreground">
          用户怎么说
        </h2>
        <p className="text-center text-muted-foreground mt-4">
          来自真实用户的反馈
        </p>
      </div>

      {/* 滚动区域，限制最大宽度 */}
      <div className="relative max-w-7xl mx-auto overflow-hidden group">
        {/* 左侧渐变遮罩 */}
        <div className="absolute left-0 top-0 bottom-0 w-24 z-10 pointer-events-none bg-gradient-to-r from-background to-transparent" />

        {/* 右侧渐变遮罩 */}
        <div className="absolute right-0 top-0 bottom-0 w-24 z-10 pointer-events-none bg-gradient-to-l from-background to-transparent" />

        {/* 滚动轨道 */}
        <div className="flex gap-6 animate-marquee group-hover:[animation-play-state:paused]">
          {doubled.map((item, i) => (
            <div
              key={i}
              className="flex-shrink-0 w-80 p-6 rounded-xl border border-border bg-card shadow-sm"
            >
              {/* 星级评分 */}
              <div className="flex gap-1 mb-3">
                {Array.from({ length: 5 }).map((_, j) => (
                  <span key={j} className="text-yellow-400 text-sm">
                    ★
                  </span>
                ))}
              </div>

              {/* 评价内容 */}
              <p className="text-muted-foreground text-sm leading-relaxed mb-4">
                "{item.content}"
              </p>

              {/* 用户信息 */}
              <div className="flex items-center gap-3">
                <img
                  src={item.avatar}
                  alt={item.name}
                  className="w-10 h-10 rounded-full object-cover"
                />
                <div>
                  <p className="font-medium text-foreground text-sm">
                    {item.name}
                  </p>
                  <p className="text-xs text-muted-foreground">{item.role}</p>
                </div>
              </div>
            </div>
          ))}
        </div>
      </div>
    </section>
  );
}
```

---

## 常见问题

### Q：滚动速度怎么调？

修改 `tailwind.config.ts` 里的动画时长：

```typescript
animation: {
  marquee: "marquee 20s linear infinite", // 20s 更快，60s 更慢
},
```

### Q：卡片数量少时循环点很明显怎么办？

复制更多份：

```typescript
// 复制 3 份而不是 2 份
const tripled = [...items, ...items, ...items];
// 动画移动到 -33.33% 而不是 -50%
```

或者增加卡片数量，让一屏显示不完。

### Q：移动端性能差怎么办？

用 `will-change: transform` 开启 GPU 加速：

```css
.marquee-track {
  will-change: transform;
}
```

Tailwind 里：

```tsx
<div className="flex animate-marquee will-change-transform">
```

### Q：能反向滚动吗（从左向右）？

```typescript
// tailwind.config.ts
keyframes: {
  "marquee-reverse": {
    from: { transform: "translateX(-50%)" },
    to: { transform: "translateX(0)" },
  },
},
animation: {
  "marquee-reverse": "marquee-reverse 30s linear infinite",
},
```

---

## 总结

纯 CSS Marquee 的核心就三步：

1. **复制内容**：把数组 `[...items, ...items]`，总宽度翻倍
2. **定义动画**：`translateX(0)` → `translateX(-50%)`，linear，infinite
3. **overflow-hidden**：容器裁剪，只显示一半内容

加上渐变遮罩和 hover 暂停，效果就很完整了。

---

## 参考资源

- [CSS animation MDN 文档](https://developer.mozilla.org/en-US/docs/Web/CSS/animation)
- [Tailwind CSS 自定义动画](https://tailwindcss.com/docs/animation#customizing-your-theme)
- [CSS transform MDN 文档](https://developer.mozilla.org/en-US/docs/Web/CSS/transform)

