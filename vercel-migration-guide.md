# Shopify Remix 应用迁移到 Vercel 指南

本文档提供了将 Shopify Remix 应用迁移到 Vercel 平台的详细步骤和代码修改指南。

## 迁移步骤

### 1. 安装 Vercel Remix 包

```bash
npm install @vercel/remix --save-dev
```

### 2. 更新 vite.config.ts

将 Vercel 预设添加到 Remix 配置中：

```typescript
import { vitePlugin as remix } from "@remix-run/dev";
import { defineConfig, type UserConfig } from "vite";
import tsconfigPaths from "vite-tsconfig-paths";
import { vercelPreset } from '@vercel/remix/vite';

// ...

export default defineConfig({
  // ...
  plugins: [
    remix({
      ignoredRouteFiles: ["**/.*"],
      presets: [vercelPreset()], // 添加 Vercel 预设
      future: {
        // 保持现有的 future 配置
        v3_fetcherPersist: true,
        v3_relativeSplatPath: true,
        v3_throwAbortReason: true,
        v3_lazyRouteDiscovery: true,
        v3_singleFetch: false,
        v3_routeConfig: true,
      },
    }),
    tsconfigPaths(),
  ],
  // ...
}) satisfies UserConfig;
```

### 3. 创建 vercel.json 配置文件

在项目根目录创建 `vercel.json` 文件：

```json
{
  "buildCommand": "npx prisma generate && npm run build",
  "devCommand": "npm run dev",
  "installCommand": "npm install",
  "framework": "remix",
  "outputDirectory": "build/client",
  "regions": ["hkg1"],
  "functions": {
    "app/routes/**/*.tsx": {
      "runtime": "node@18"
    },
    "app/routes/**/*.ts": {
      "runtime": "node@18"
    }
  },
  "env": {
    "NODE_ENV": "production"
  }
}
```

### 4. 导入替换

将代码中的 `@remix-run/node` 导入替换为 `@vercel/remix`，特别是对于以下功能：

#### 流式响应

```typescript
// 替换前
import { defer, json } from "@remix-run/node";

// 替换后
import { defer, json } from "@vercel/remix";
```

#### 请求/响应处理

```typescript
// 替换前
import { type ActionFunctionArgs, type LoaderFunctionArgs } from "@remix-run/node";

// 替换后
import { type ActionFunctionArgs, type LoaderFunctionArgs } from "@vercel/remix";
```

#### Session 管理

```typescript
// 替换前
import { createCookieSessionStorage } from "@remix-run/node";

// 替换后
import { createCookieSessionStorage } from "@vercel/remix";
```

### 5. 特殊处理

#### 处理 installGlobals

如果你的代码中有 `installGlobals` 的导入和使用，需要更新：

```typescript
// 替换前
import { installGlobals } from "@remix-run/node";

// 替换后
import { installGlobals } from "@vercel/remix";
```

#### 处理 Prisma

对于使用 Prisma 的应用，确保在 `vercel.json` 的 `buildCommand` 中包含 Prisma 生成步骤：

```json
"buildCommand": "npx prisma generate && npm run build"
```

## 验证迁移

完成上述步骤后，执行以下操作验证迁移是否成功：

1. 本地测试：

```bash
vercel dev
```

2. 部署到 Vercel：

```bash
vercel
```

3. 检查部署日志，确保没有错误。

4. 测试应用功能，特别是 Shopify OAuth 流程和数据库操作。

## 常见问题解决

### 1. 运行时错误

如果遇到 "Function Runtimes must have a valid version" 错误，检查 `vercel.json` 文件中的 `runtime` 配置是否正确。确保使用 `node@18` 格式而不是 `nodejs18.x`。

### 2. 导入错误

如果遇到与 `@remix-run/node` 相关的导入错误，确保已将所有相关导入替换为 `@vercel/remix`。

### 3. 数据库连接问题

在 Vercel 的无服务器环境中，长时间运行的数据库连接可能会遇到问题。考虑使用以下策略：

- 使用 Prisma Data Proxy
- 实现连接池管理
- 在每个请求开始时创建连接，结束时关闭连接

### 4. 环境变量

确保在 Vercel 控制台中设置了所有必要的环境变量，特别是：

- `SHOPIFY_API_KEY`
- `SHOPIFY_API_SECRET`
- `SHOPIFY_APP_URL`
- `SCOPES`
- `DATABASE_URL`

## 结论

通过以上步骤，你的 Shopify Remix 应用应该已经成功迁移到 Vercel 平台，并能够利用 Vercel 的特性和优势。如果遇到任何问题，请参考 [Vercel Remix 文档](https://vercel.com/docs/frameworks/remix) 或 [Shopify 应用开发文档](https://shopify.dev/apps/getting-started/create)。
