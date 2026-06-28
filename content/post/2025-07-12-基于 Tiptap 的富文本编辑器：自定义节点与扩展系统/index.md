---
title: 基于 Tiptap 的富文本编辑器：自定义节点与扩展系统
description: ""
slug: 2025-07-12-基于 Tiptap 的富文本编辑器：自定义节点与扩展系统
date: 2025-07-12
image: Tiptap.jpg
categories:
  - 效率&工具
tags:
  - Tiptap
  - 编辑器
  - RuiToolAI
---

> CMS 需要一个好用的编辑器。Tiptap 是 ProseMirror 的 React 封装，扩展性极强。

## 为什么选 Tiptap

Tiptap 是基于 ProseMirror 的 React 富文本编辑器，相比其他方案：

- **无头设计**：自带样式最小化，完全自定义 UI
- **扩展系统**：每个功能都是独立的扩展，按需组合
- **类型安全**：TypeScript 一等公民
- **JSON 输出**：内容存储为 JSON，不是 HTML，便于程序化处理

## 架构设计

RuiToolAI 的 Tiptap 编辑器分为四层：

```
tiptap-ui/          ← 工具栏按钮（heading-button, list-button 等）
tiptap-node/        ← 自定义节点（alert-block, image-upload 等）
tiptap-extension/   ← 自定义扩展（markdown-paste 等）
tiptap-templates/   ← 编辑器模板（simple-editor 等）
tiptap-ui-primitive/ ← 基础 UI 组件（button, toolbar, popover 等）
```

## 自定义节点：Alert Block

Alert Block 是一个带图标和颜色的提示块，支持 info、warning、error、success 四种类型：

```typescript
// src/components/tiptap-node/alert-block/alert-block-extension.ts

export const AlertBlock = Node.create({
  name: ALERT_BLOCK_NODE_NAME,
  group: "block",
  content: "block+",
  defining: true,

  addAttributes() {
    return {
      variant: {
        default: "info",
        parseHTML: (element) => element.getAttribute("data-variant"),
        renderHTML: (attributes) => ({
          "data-variant": attributes.variant,
        }),
      },
    };
  },

  parseHTML() {
    return [{ tag: `div[data-type="${ALERT_BLOCK_NODE_NAME}"]` }];
  },

  renderHTML({ HTMLAttributes }) {
    return [
      "div",
      mergeAttributes(
        { "data-type": ALERT_BLOCK_NODE_NAME },
        HTMLAttributes
      ),
      0,
    ];
  },

  addCommands() {
    return {
      setAlertBlock:
        (variant: AlertVariant) =>
        ({ commands }) => {
          return commands.setNode(this.name, { variant });
        },
    };
  },
});
```

渲染组件：

```typescript
// alert-block.tsx
export function AlertBlockRenderer({ node, children }) {
  const variant = node.attrs.variant as AlertVariant;
  const Icon = variantIcons[variant];

  return (
    <div className={`alert-block alert-block--${variant}`}>
      <Icon className="alert-block__icon" />
      <div className="alert-block__content">{children}</div>
    </div>
  );
}
```

## 图片上传节点

图片上传节点集成了 CMS 媒体库选择器：

```typescript
// src/components/tiptap-node/image-upload-node/image-upload-node.tsx

export function ImageUploadNode({ editor }) {
  const [showPicker, setShowPicker] = useState(false);

  return (
    <>
      <Button onClick={() => setShowPicker(true)}>
        <ImagePlusIcon /> Insert Image
      </Button>

      {showPicker && (
        <MediaLibraryPicker
          onSelect={(image) => {
            editor
              .chain()
              .focus()
              .setImage({ src: image.url, alt: image.altText })
              .run();
            setShowPicker(false);
          }}
          onClose={() => setShowPicker(false)}
        />
      )}
    </>
  );
}
```

## Markdown 粘贴支持

用户从 Notion、Obsidian 等工具粘贴 Markdown 内容时，自动转换为 Tiptap 的 JSON 格式：

```typescript
// src/components/tiptap-extension/markdown-paste-extension.ts

export const MarkdownPaste = Extension.create({
  name: "markdownPaste",

  addProseMirrorPlugins() {
    return [
      new Plugin({
        props: {
          handlePaste: (view, event) => {
            const text = event.clipboardData?.getData("text/plain");
            if (!text) return false;

            // 检测是否为 Markdown
            if (!looksLikeMarkdown(text)) return false;

            // 用 turndown + markdown-it 转换
            const html = md.render(text);
            const json = htmlToJSON(html);

            // 插入转换后的内容
            view.dispatch(
              view.state.tr.replaceSelectionWith(
                view.state.schema.nodeFromJSON(json)
              )
            );

            return true;
          },
        },
      }),
    ];
  },
});
```

## 工具栏组件化

每个工具栏按钮都是独立的组件，有自己的状态和逻辑：

```typescript
// src/components/tiptap-ui/heading-button/heading-button.tsx

export function HeadingButton({ editor, level }: HeadingButtonProps) {
  const { isActive, toggle } = useHeading({ editor, level });

  return (
    <Button
      variant={isActive ? "active" : "default"}
      onClick={toggle}
      tooltip={`Heading ${level}`}
    >
      <HeadingIcon level={level} />
    </Button>
  );
}
```

每个按钮都有对应的 Hook：

```typescript
// use-heading.ts
export function useHeading({ editor, level }: UseHeadingParams) {
  const isActive = editor.isActive("heading", { level });
  const toggle = useCallback(() => {
    editor.chain().focus().toggleHeading({ level }).run();
  }, [editor, level]);

  return { isActive, toggle };
}
```

## 总结

- 四层架构：UI → Node → Extension → Template
- 自定义节点（Alert Block）实现富文本业务组件
- 图片上传节点集成 CMS 媒体库
- Markdown 粘贴扩展让内容迁移更顺畅
- 工具栏按钮组件化，每个按钮独立可复用
