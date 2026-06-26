---
title: GitHub Actions 全自动 CI/CD 流水线
description: ""
slug: 2025-06-18-GitHub Actions 全自动 CI/CD 流水线
date: 2025-06-18
image: github-actions.jpg
categories:
  - 运维&云原生
  - 效率&工具
tags:
  - GitHubActions
  - CI/CD
  - cloudflare
  - RuiToolAI
---

> 每次 push 自动跑测试、自动部署到 Cloudflare Workers。这篇记录 RuiToolAI 的 CI/CD 搭建过程。

## 流水线设计

RuiToolAI 的 CI/CD 分两个阶段：

```
Push to main
    ↓
[CI] Lint + TypeCheck + Unit Tests
    ↓ 通过
[CD] Build + e2e Tests + Deploy to Cloudflare Workers
    ↓ 通过
生产环境更新
```

PR 只跑 CI，不部署。合并到 main 才触发部署。

---

## 基础 CI 配置

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint-and-typecheck:
    name: Lint & TypeCheck
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup pnpm
        uses: pnpm/action-setup@v4
        with:
          version: 9

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "pnpm"

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Lint
        run: pnpm run lint

      - name: TypeCheck
        run: pnpm run typecheck

  unit-tests:
    name: Unit Tests
    runs-on: ubuntu-latest
    needs: lint-and-typecheck

    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with:
          version: 9
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "pnpm"
      - run: pnpm install --frozen-lockfile
      - run: pnpm run test
```

---

## Cloudflare Workers 自动部署

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    name: Deploy to Cloudflare Workers
    runs-on: ubuntu-latest
    # 只在 CI 通过后部署
    needs: [lint-and-typecheck, unit-tests]

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup pnpm
        uses: pnpm/action-setup@v4
        with:
          version: 9

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "pnpm"

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Build
        run: pnpm run build
        env:
          # 构建时需要的环境变量
          NODE_ENV: production

      - name: Deploy to Cloudflare Workers
        run: pnpm run deploy
        env:
          CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          CLOUDFLARE_ACCOUNT_ID: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
```

---

## Secrets 管理

GitHub Actions 里的 Secrets 分两种：

- **Repository Secrets**：只有这个仓库能用
- **Environment Secrets**：绑定到特定环境（production/staging），可以设置审批流程

在 GitHub 仓库 → Settings → Secrets and variables → Actions 里配置。

RuiToolAI 需要的 Secrets：

```
CLOUDFLARE_API_TOKEN      # Cloudflare API Token（Workers 部署权限）
CLOUDFLARE_ACCOUNT_ID     # Cloudflare 账户 ID
TEST_USER_EMAIL           # e2e 测试用的测试账号
TEST_USER_PASSWORD        # e2e 测试用的测试密码
```

### 生成 Cloudflare API Token

1. 登录 Cloudflare Dashboard
2. My Profile → API Tokens → Create Token
3. 选择 "Edit Cloudflare Workers" 模板
4. 复制 Token，粘贴到 GitHub Secrets

**注意**：Token 只显示一次，要立即保存。

---

## e2e 测试集成

e2e 测试需要启动完整的应用，比较耗时，放在单独的 job 里：

```yaml
# .github/workflows/e2e.yml
name: E2E Tests

on:
  push:
    branches: [main]

jobs:
  e2e:
    name: End-to-End Tests
    runs-on: ubuntu-latest
    timeout-minutes: 30

    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
        with:
          version: 9

      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "pnpm"

      - run: pnpm install --frozen-lockfile

      # 安装 Playwright 浏览器
      - name: Install Playwright Browsers
        run: pnpm exec playwright install --with-deps chromium

      # 初始化本地测试数据库
      - name: Setup test database
        run: pnpm db:migrate:local

      # 运行 e2e 测试
      - name: Run e2e tests
        run: pnpm test:e2e
        env:
          TEST_USER_EMAIL: ${{ secrets.TEST_USER_EMAIL }}
          TEST_USER_PASSWORD: ${{ secrets.TEST_USER_PASSWORD }}

      # 失败时上传报告
      - name: Upload Playwright Report
        uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 7
```

---

## 数据库迁移自动化

部署前需要先跑数据库迁移，确保 schema 是最新的：

```yaml
- name: Run database migrations
  run: pnpm db:migrate:remote
  env:
    CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
    CLOUDFLARE_ACCOUNT_ID: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}

- name: Deploy to Cloudflare Workers
  run: pnpm run deploy
  env:
    CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
    CLOUDFLARE_ACCOUNT_ID: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
```

`package.json` 里的迁移命令：

```json
{
  "scripts": {
    "db:migrate:local": "wrangler d1 migrations apply DB --local",
    "db:migrate:remote": "wrangler d1 migrations apply DB --remote"
  }
}
```

---

## 部署通知

部署成功或失败时发通知，方便及时知道状态：

```yaml
- name: Notify on success
  if: success()
  run: |
    curl -X POST ${{ secrets.SLACK_WEBHOOK_URL }} \
      -H 'Content-type: application/json' \
      -d '{"text": "✅ RuiToolAI 部署成功！"}'

- name: Notify on failure
  if: failure()
  run: |
    curl -X POST ${{ secrets.SLACK_WEBHOOK_URL }} \
      -H 'Content-type: application/json' \
      -d '{"text": "❌ RuiToolAI 部署失败，请检查 GitHub Actions 日志。"}'
```

---

## 完整流水线示例

把所有 job 整合到一个文件里，用 `needs` 控制依赖关系：

```yaml
# .github/workflows/main.yml
name: CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  # 1. 代码质量检查（所有 push 和 PR 都跑）
  quality:
    name: Lint & TypeCheck
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with: { version: 9 }
      - uses: actions/setup-node@v4
        with: { node-version: "20", cache: "pnpm" }
      - run: pnpm install --frozen-lockfile
      - run: pnpm run lint
      - run: pnpm run typecheck

  # 2. 单元测试
  test:
    name: Unit Tests
    runs-on: ubuntu-latest
    needs: quality
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with: { version: 9 }
      - uses: actions/setup-node@v4
        with: { node-version: "20", cache: "pnpm" }
      - run: pnpm install --frozen-lockfile
      - run: pnpm run test

  # 3. e2e 测试（只在 main 分支跑）
  e2e:
    name: E2E Tests
    runs-on: ubuntu-latest
    needs: test
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with: { version: 9 }
      - uses: actions/setup-node@v4
        with: { node-version: "20", cache: "pnpm" }
      - run: pnpm install --frozen-lockfile
      - run: pnpm exec playwright install --with-deps chromium
      - run: pnpm db:migrate:local
      - run: pnpm test:e2e
        env:
          TEST_USER_EMAIL: ${{ secrets.TEST_USER_EMAIL }}
          TEST_USER_PASSWORD: ${{ secrets.TEST_USER_PASSWORD }}
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 7

  # 4. 部署（只在 main 分支，所有测试通过后）
  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    needs: [test, e2e]
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with: { version: 9 }
      - uses: actions/setup-node@v4
        with: { node-version: "20", cache: "pnpm" }
      - run: pnpm install --frozen-lockfile
      - run: pnpm run build
      - name: Run DB migrations
        run: pnpm db:migrate:remote
        env:
          CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          CLOUDFLARE_ACCOUNT_ID: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
      - name: Deploy to Workers
        run: pnpm run deploy
        env:
          CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          CLOUDFLARE_ACCOUNT_ID: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
```

---

## 常见问题排查

### 问题 1：pnpm install 很慢

加缓存：

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: "20"
    cache: "pnpm"  # 这行很重要
```

### 问题 2：Playwright 浏览器每次都重新下载

```yaml
- name: Cache Playwright browsers
  uses: actions/cache@v4
  with:
    path: ~/.cache/ms-playwright
    key: playwright-${{ runner.os }}-${{ hashFiles('**/package.json') }}

- name: Install Playwright Browsers
  run: pnpm exec playwright install --with-deps chromium
```

### 问题 3：部署失败但没有详细错误信息

```yaml
- name: Deploy
  run: pnpm run deploy
  env:
    CLOUDFLARE_API_TOKEN: ${{ secrets.CLOUDFLARE_API_TOKEN }}
  # 加上 --verbose 看详细日志
  # 或者检查 wrangler 版本是否兼容
```

### 问题 4：e2e 测试超时

```yaml
jobs:
  e2e:
    timeout-minutes: 30  # 增加超时时间
```

---

## 总结

CI/CD 流水线的核心原则：

1. **快速反馈**：Lint 和 TypeCheck 最快，先跑；e2e 最慢，最后跑
2. **依赖关系**：用 `needs` 控制 job 顺序，测试通过才部署
3. **分支策略**：PR 只跑 CI，main 分支才部署
4. **Secrets 安全**：敏感信息放 GitHub Secrets，不要硬编码
5. **失败保留证据**：上传测试报告，方便排查

---

## 参考资源

- [GitHub Actions 文档](https://docs.github.com/en/actions)
- [Wrangler GitHub Action](https://github.com/cloudflare/wrangler-action)
- [pnpm/action-setup](https://github.com/pnpm/action-setup)
- [Playwright CI 配置](https://playwright.dev/docs/ci-intro)

