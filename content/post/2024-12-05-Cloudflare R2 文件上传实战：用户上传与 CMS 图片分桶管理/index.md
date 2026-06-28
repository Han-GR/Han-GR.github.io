---
title: Cloudflare R2 文件上传实战：用户上传与 CMS 图片分桶管理
description: ""
slug: 2024-12-05-Cloudflare R2 文件上传实战：用户上传与 CMS 图片分桶管理
date: 2024-12-05
image: R2.jpg
categories:
  - 数据库&缓存
tags:
  - cloudflare
  - R2
  - 对象存储
  - RuiToolAI
---

> Workers 没有文件系统，所有文件都得存 R2。聊聊用户上传和 CMS 图片管理的完整方案。

## 为什么用 R2

Cloudflare Workers 没有文件系统，不能像 Node.js 那样 `fs.writeFile()`。R2 是 Cloudflare 的对象存储，兼容 S3 API：

- 全球分布式，边缘节点就近访问
- 无出口流量费（对比 S3 的 0.09/GB）
- 与 Workers 天然集成，零配置

## 分桶策略

RuiToolAI 用两个 R2 Bucket 隔离不同用途的文件：

| Bucket | 用途 | 访问权限 |
|--------|------|----------|
| `IMAGES_R2` | CMS 图片（运营上传） | 公开读取 |
| `USER_UPLOADS_BUCKET` | 用户上传的文件 | 需认证 |

分桶的好处：
- 权限隔离：CMS 图片公开访问，用户文件需要鉴权
- 成本隔离：可以分别设置生命周期策略
- 数据隔离：不会因为用户上传影响 CMS 内容

## CMS 图片上传

CMS 图片上传流程：

```typescript
// src/actions/upload-image.action.ts

export const uploadImageAction = actionClient
  .inputSchema(uploadImageSchema)
  .action(async ({ parsedInput: input }) => {
    return withRateLimit(async () => {
      // 1. 权限检查：必须是管理员
      const session = await requireAdmin({ doNotThrowError: true });
      if (!session?.user?.id) {
        throw new ActionError("FORBIDDEN", "You must be logged in");
      }

      const { file, collection } = input;

      // 2. 文件大小校验
      if (file.size > CMS_IMAGE_MAX_FILE_SIZE) {
        throw new ActionError("INPUT_PARSE_ERROR",
          `File size exceeds maximum of ${CMS_IMAGE_MAX_FILE_SIZE / 1024 / 1024}MB`);
      }

      // 3. 文件类型校验（检查 magic bytes，不是 MIME type）
      const arrayBuffer = await file.arrayBuffer();
      const detectedType = await fileTypeFromBuffer(arrayBuffer);

      if (!detectedType || !CMS_ALLOWED_IMAGE_TYPES.includes(detectedType.mime)) {
        throw new ActionError("INPUT_PARSE_ERROR", "Invalid file type");
      }

      // 4. 生成唯一文件名
      const uniqueFilename = generateUniqueFilename({
        originalFilename: file.name,
        extension: detectedType.ext,
      });

      // 5. 上传到 R2
      const r2Key = getCmsImageR2Key({ collection, filename: uniqueFilename });
      const { env } = await getCloudflareContext();
      await env.IMAGES_R2.put(r2Key, arrayBuffer, {
        httpMetadata: { contentType: detectedType.mime },
      });

      // 6. 记录到 CMS 媒体库
      const publicUrl = getCmsImagePublicUrl(r2Key);
      await db.insert(cmsMediaTable).values({
        filename: uniqueFilename,
        originalFilename: file.name,
        r2Key,
        publicUrl,
        mimeType: detectedType.mime,
        fileSize: file.size,
        uploadedBy: session.user.id,
        collection,
      });

      return { url: publicUrl };
    }, RATE_LIMITS.SETTINGS);
  });
```

关键设计：

- **magic bytes 校验**：用 `file-type` 库检查文件的实际内容，而不是信任 `Content-Type` 头
- **唯一文件名**：`{cuid2}-{sanitized-filename}.ext`，避免冲突
- **文件名清理**：`sanitizeFilename` 去掉路径分隔符和特殊字符
- **记录到数据库**：媒体库需要可搜索、可管理

## 用户文件上传

用户上传的文件存到 `USER_UPLOADS_BUCKET`，按用户 ID 隔离：

```typescript
// src/actions/upload-file.action.ts

function getUserUploadR2Key({
  userId,
  fileId,
  filename,
}: {
  userId: string;
  fileId: string;
  filename: string;
}): string {
  return `users/${userId}/${fileId}-${sanitizeFilename(filename)}`;
}

export const uploadFileAction = actionClient
  .inputSchema(uploadFileSchema)
  .action(async ({ parsedInput: input }) => {
    return withRateLimit(async () => {
      // 需要已验证邮箱
      const session = await requireVerifiedEmail();
      if (!session?.user?.id) {
        throw new ActionError("NOT_AUTHORIZED", "Unauthorized");
      }

      const { file } = input;
      const fileId = createId();
      const r2Key = getUserUploadR2Key({
        userId: session.user.id,
        fileId,
        filename: file.name,
      });

      const arrayBuffer = await file.arrayBuffer();
      const { env } = await getCloudflareContext();
      await env.USER_UPLOADS_BUCKET.put(r2Key, arrayBuffer, {
        httpMetadata: { contentType: file.type },
      });

      return { r2Key, filename: file.name };
    }, RATE_LIMITS.SETTINGS);
  });
```

路径格式：`users/{userId}/{fileId}-{filename}`，每个用户的文件在独立前缀下。

## 文件类型校验

```typescript
// 允许的图片类型
const CMS_ALLOWED_IMAGE_TYPES = [
  "image/jpeg",
  "image/png",
  "image/gif",
  "image/webp",
  "image/svg+xml",
] as const;

// 用 file-type 库检测 magic bytes
const detectedType = await fileTypeFromBuffer(arrayBuffer);

if (!detectedType || !CMS_ALLOWED_IMAGE_TYPES.includes(detectedType.mime)) {
  throw new ActionError("INPUT_PARSE_ERROR", "Invalid file type");
}
```

不要信任 `file.type`（前端可以伪造），必须用服务端 magic bytes 检测。

## 通过 API 访问文件

R2 的文件不直接暴露 URL，而是通过 API 路由代理：

```typescript
// src/app/api/cms-images/[...path]/route.ts
export async function GET(request: Request) {
  const { env } = await getCloudflareContext();
  const path = /* 从 URL 中提取 */;

  const object = await env.IMAGES_R2.get(path);
  if (!object) {
    return new Response("Not found", { status: 404 });
  }

  return new Response(object.body, {
    headers: {
      "Content-Type": object.httpMetadata?.contentType || "application/octet-stream",
      "Cache-Control": "public, max-age=31536000, immutable",
    },
  });
}
```

用户文件同理，但需要先验证 Session。

## 总结

- 两个 Bucket 隔离 CMS 和用户文件
- 用 `file-type` 做 magic bytes 校验，不信任前端 MIME type
- 文件名用 `cuid2` 保证唯一性，`sanitizeFilename` 防止路径穿越
- 文件通过 API 路由代理访问，可以做权限控制和缓存策略