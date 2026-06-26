---
title: Vitest + Playwright 全栈 e2e 测试实战
description: ""
slug: 2025-04-23-Vitest + Playwright 全栈 e2e 测试实战
date: 2025-04-23
image: vitest.png
categories:
  - 测试
tags:
  - Vitest
  - Playwright
  - SQLite
  - e2e
  - CI/CD
---

> 测试不是可选项。这篇记录 RuiTool AI 的测试体系搭建，以及踩过的 SQLite 并发锁这个大坑。

## 测试策略

RuiTool AI 的测试分两层：

| 层级      | 工具         | 覆盖范围                      |
| ------- | ---------- | ------------------------- |
| 单元/集成测试 | Vitest     | 工具函数、Server Actions、数据库操作 |
| e2e 测试  | Playwright | 完整用户流程（注册、登录、生成图片）        |

原则：**e2e 测试覆盖核心用户旅程，单元测试覆盖复杂业务逻辑。** 不追求 100% 覆盖率，追求关键路径不出错。

---

## Vitest 单元测试配置

```typescript
// vitest.config.ts
import { defineConfig } from "vitest/config";
import path from "path";

export default defineConfig({
  test: {
    environment: "node",
    globals: true,
    setupFiles: ["./src/test/setup.ts"],
  },
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
});
```

```typescript
// src/test/setup.ts
import { beforeAll, afterAll } from "vitest";

// 全局测试前置
beforeAll(async () => {
  // 初始化测试数据库
});

afterAll(async () => {
  // 清理
});
```

---

## Playwright e2e 测试配置

```typescript
// playwright.config.ts
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "./e2e",
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: "html",

  use: {
    baseURL: "http://localhost:3000",
    trace: "on-first-retry",
  },

  projects: [
    {
      name: "chromium",
      use: { ...devices["Desktop Chrome"] },
    },
  ],

  // 测试前启动开发服务器
  webServer: {
    command: "pnpm dev",
    url: "http://localhost:3000",
    reuseExistingServer: !process.env.CI,
    timeout: 120 * 1000,
  },
});
```

---

## 测试数据库隔离

e2e 测试不能用生产数据库，需要独立的测试数据库。

在 Cloudflare Workers 本地开发环境里，Wrangler 会自动创建本地 SQLite 文件：

```
.wrangler/state/v3/d1/
└── <database-id>/
    └── db.sqlite  ← 本地测试用这个
```

测试前需要初始化 schema：

```typescript
// e2e/fixtures/db.ts
import { execSync } from "child_process";

export function setupTestDb() {
  // 应用所有迁移
  execSync("pnpm db:migrate:local", { stdio: "inherit" });
}

export function cleanTestDb() {
  // 清理测试数据
  execSync(
    'wrangler d1 execute DB --local --command "DELETE FROM users WHERE email LIKE \'%test%\'"',
    { stdio: "inherit" }
  );
}
```

---

## SQLite 并发锁问题及解决

这是我踩过最深的坑。

### 问题现象

CI 里 e2e 测试随机失败，错误信息：

```
SqliteError: database is locked
```

### 原因分析

Vitest 默认并行运行多个测试文件，每个文件在独立的 Worker 线程里执行。多个线程同时写同一个 SQLite 文件，就会出现锁冲突。

SQLite 的写锁是文件级别的，同一时刻只允许一个写操作。

### 解决方案

```typescript
// vitest.e2e.config.ts
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    // 关键：禁用文件级并行，改为串行执行
    fileParallelism: false,
    // 只用一个 Worker
    maxWorkers: 1,
    // 测试文件内部的测试用例仍然可以并行
    // （但写数据库的测试用例最好也串行）
  },
});
```

代价是测试速度变慢，但在 CI 里稳定性更重要。

### 为什么之前没问题？

本地开发时，测试文件少，并发冲突概率低。CI 里测试文件多，并发冲突必然发生。

---

## 常用测试模式

### 测试 Server Action

```typescript
// src/app/(auth)/sign-up/sign-up.actions.test.ts
import { describe, it, expect, beforeEach } from "vitest";
import { signUpAction } from "./sign-up.actions";

describe("signUpAction", () => {
  it("应该成功注册新用户", async () => {
    const result = await signUpAction({
      email: "test@example.com",
      password: "Password123!",
    });

    expect(result?.data?.success).toBe(true);
  });

  it("应该拒绝已存在的邮箱", async () => {
    // 先注册一次
    await signUpAction({
      email: "existing@example.com",
      password: "Password123!",
    });

    // 再注册同一个邮箱
    const result = await signUpAction({
      email: "existing@example.com",
      password: "Password123!",
    });

    expect(result?.serverError).toBeDefined();
  });
});
```

### Playwright 测试登录流程

```typescript
// e2e/auth.spec.ts
import { test, expect } from "@playwright/test";

test.describe("认证流程", () => {
  test("用户可以注册并登录", async ({ page }) => {
    // 访问注册页
    await page.goto("/sign-up");

    // 填写表单
    await page.fill('[name="email"]', "e2e-test@example.com");
    await page.fill('[name="password"]', "TestPassword123!");

    // 提交
    await page.click('[type="submit"]');

    // 验证跳转到 dashboard
    await expect(page).toHaveURL("/dashboard");
    await expect(page.locator("h1")).toContainText("欢迎");
  });

  test("错误密码应该显示错误提示", async ({ page }) => {
    await page.goto("/sign-in");
    await page.fill('[name="email"]', "user@example.com");
    await page.fill('[name="password"]', "wrongpassword");
    await page.click('[type="submit"]');

    await expect(page.locator('[role="alert"]')).toBeVisible();
  });
});
```

### 测试需要认证的页面

```typescript
// e2e/fixtures/auth.ts
import { test as base, expect } from "@playwright/test";

// 扩展 test，自动登录
export const test = base.extend({
  page: async ({ page }, use) => {
    // 登录
    await page.goto("/sign-in");
    await page.fill('[name="email"]', process.env.TEST_USER_EMAIL!);
    await page.fill('[name="password"]', process.env.TEST_USER_PASSWORD!);
    await page.click('[type="submit"]');
    await page.waitForURL("/dashboard");

    await use(page);
  },
});

export { expect };
```

```typescript
// e2e/history.spec.ts
import { test, expect } from "./fixtures/auth";

test("历史页面应该显示生成记录", async ({ page }) => {
  await page.goto("/history");
  await expect(page.locator("h1")).toContainText("生成历史");
});
```

### 使用 Page Object Model

对于复杂页面，用 Page Object Model 封装操作：

```typescript
// e2e/pages/image-gen.page.ts
import { Page } from "@playwright/test";

export class ImageGenPage {
  constructor(private page: Page) {}

  async goto() {
    await this.page.goto("/image-gen");
  }

  async generateImage(prompt: string) {
    await this.page.fill("textarea", prompt);
    await this.page.click('[data-testid="generate-button"]');
  }

  async waitForGeneration() {
    // 等待生成完成（最多 60 秒）
    await this.page.waitForSelector('[data-testid="generated-image"]', {
      timeout: 60000,
    });
  }

  async getGeneratedImageUrl() {
    return this.page
      .locator('[data-testid="generated-image"]')
      .getAttribute("src");
  }
}
```

```typescript
// e2e/image-gen.spec.ts
import { test, expect } from "./fixtures/auth";
import { ImageGenPage } from "./pages/image-gen.page";

test("用户可以生成图片", async ({ page }) => {
  const imageGenPage = new ImageGenPage(page);
  await imageGenPage.goto();
  await imageGenPage.generateImage("a beautiful sunset over the ocean");
  await imageGenPage.waitForGeneration();

  const imageUrl = await imageGenPage.getGeneratedImageUrl();
  expect(imageUrl).toBeTruthy();
});
```

---

## CI 里运行测试

```yaml
# .github/workflows/ci.yml
- name: Run e2e tests
  run: pnpm test:e2e
  env:
    TEST_USER_EMAIL: ${{ secrets.TEST_USER_EMAIL }}
    TEST_USER_PASSWORD: ${{ secrets.TEST_USER_PASSWORD }}

- name: Upload test results
  uses: actions/upload-artifact@v4
  if: failure()
  with:
    name: playwright-report
    path: playwright-report/
    retention-days: 7
```

测试失败时上传报告，方便排查问题。

---

## 总结

全栈测试体系的关键点：

1. **分层测试**：单元测试覆盖逻辑，e2e 测试覆盖用户旅程
2. **数据库隔离**：测试用独立的本地 SQLite，不污染生产数据
3. **串行执行**：SQLite 不支持并发写，`fileParallelism: false` 是必须的
4. **Page Object Model**：复杂页面封装操作，测试代码更易维护
5. **CI 上传报告**：失败时保留截图和 trace，方便排查

---

## 参考资源

- [Vitest 文档](https://vitest.dev/)
- [Playwright 文档](https://playwright.dev/)
- [Playwright Page Object Model](https://playwright.dev/docs/pom)
- [SQLite WAL 模式](https://www.sqlite.org/wal.html)

**标签**：`Vitest` `Playwright` `e2e测试` `SQLite` `CI/CD` `测试策略`
