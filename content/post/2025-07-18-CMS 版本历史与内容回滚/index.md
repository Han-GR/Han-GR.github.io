---
title: CMS 版本历史与内容回滚
description: ""
slug: 2025-07-18-CMS 版本历史与内容回滚
date: 2025-07-18
image: cms.webp
categories:
  - 架构设计
tags:
  - CMS
  - RuiToolAI
---

> 发布后发现改错了？版本历史让你随时回退到任意历史版本。


## 为什么需要版本历史

CMS 内容管理的两个痛点：
1. 多人协作时，不知道谁改了什么
2. 改错了内容，想回退但找不到之前的版本

版本历史解决这两个问题。

## 数据模型

```typescript
// src/db/schema.ts

export const cmsEntryVersionTable = sqliteTable("cms_entry_versions", {
  id: text("id").primaryKey(),
  entryId: text("entry_id")
    .notNull()
    .references(() => cmsEntryTable.id),
  versionNumber: integer("version_number").notNull(),
  title: text("title").notNull(),
  content: text("content").notNull(), // JSON 格式的 Tiptap 内容
  slug: text("slug"),
  excerpt: text("excerpt"),
  seoTitle: text("seo_title"),
  seoDescription: text("seo_description"),
  status: text("status", { enum: ["draft", "published"] }).notNull(),
  createdBy: text("created_by").notNull(),
  changeDescription: text("change_description"),
  createdAt: integer("created_at", { mode: "timestamp" })
    .notNull()
    .$defaultFn(() => new Date()),
});
```

每个版本是 Entry 的一个完整快照，包含标题、内容、SEO 信息等所有字段。

## 创建版本快照

在保存 Entry 时自动创建版本：

```typescript
// src/lib/cms/cms-repository.ts

export async function updateCmsEntry(
  entryId: string,
  data: UpdateCmsEntryInput,
  userId: string
) {
  const db = getDB();

  // 获取当前版本以确定新版本号
  const currentEntry = await db.query.cmsEntryTable.findFirst({
    where: eq(cmsEntryTable.id, entryId),
  });

  const newVersionNumber = (currentEntry?.version || 0) + 1;

  // 创建版本快照
  await db.insert(cmsEntryVersionTable).values({
    entryId,
    versionNumber: newVersionNumber,
    title: data.title ?? currentEntry?.title ?? "",
    content: data.content ?? currentEntry?.content ?? "",
    slug: data.slug ?? currentEntry?.slug,
    excerpt: data.excerpt ?? currentEntry?.excerpt,
    seoTitle: data.seoTitle ?? currentEntry?.seoTitle,
    seoDescription: data.seoDescription ?? currentEntry?.seoDescription,
    status: data.status ?? currentEntry?.status ?? "draft",
    createdBy: userId,
    changeDescription: data.changeDescription,
  });

  // 更新 Entry
  await db
    .update(cmsEntryTable)
    .set({
      ...data,
      version: newVersionNumber,
      updatedAt: new Date(),
    })
    .where(eq(cmsEntryTable.id, entryId));
}
```

## 版本 Diff 对比

用 `diff` 库做文本对比，展示两个版本之间的差异：

```typescript
// src/app/(admin)/admin/cms/[collection]/_components/version-history.tsx

import { diffLines } from "diff/lib/diff/line.js";

function createContentDiff({
  currentContent,
  selectedContent,
}: {
  currentContent: JSONContent;
  selectedContent: JSONContent;
}) {
  return createUnifiedDiff({
    currentValue: contentToMarkdown(currentContent),
    selectedValue: contentToMarkdown(selectedContent),
  });
}

function createUnifiedDiff({
  currentValue,
  selectedValue,
}: {
  currentValue: string;
  selectedValue: string;
}): LocalDiff {
  const changes = diffLines(currentValue, selectedValue);

  const lines: DiffLine[] = changes.map((change) => {
    if (change.added) {
      return { id: createId(), segments: [{ type: "changed", value: change.value }], type: "added" };
    } else if (change.removed) {
      return { id: createId(), segments: [{ type: "changed", value: change.value }], type: "removed" };
    } else {
      return { id: createId(), segments: [{ type: "unchanged", value: change.value }], type: "context" };
    }
  });

  return {
    hasChanges: changes.some((c) => c.added || c.removed),
    lines,
  };
}
```

Diff 结果用颜色区分：
- 绿色：新增内容
- 红色：删除内容
- 白色/灰色：未变化内容

## 回滚到历史版本

```typescript
// src/app/(admin)/admin/cms/_actions/version-actions.ts

export const revertCmsEntryVersionAction = actionClient
  .inputSchema(v.object({
    entryId: v.string(),
    versionId: v.string(),
  }))
  .action(async ({ parsedInput: input }) => {
    await requireAdmin();

    const updatedEntry = await revertCmsEntryToVersion({
      entryId: input.entryId,
      versionId: input.versionId,
    });

    return updatedEntry;
  });
```

回滚操作：
1. 获取目标版本的内容
2. 更新 Entry 的当前内容为目标版本的内容
3. 创建一个新版本记录（标记为"回滚自 version X"）

## 总结

- 每次保存自动创建版本快照
- 用 `diff` 库做版本对比，颜色区分增删
- 回滚也是创建新版本，不会丢失历史
- 版本记录包含创建者和变更描述，便于追溯
