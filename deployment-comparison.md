# Shopify Remix 应用部署比较：Vercel vs Render

本文档比较了在 Vercel 和 Render 上部署 Shopify Remix 应用的不同配置和注意事项，帮助您选择最适合的部署平台。

## Vercel 部署配置

### 优势

- 与 Remix 框架深度集成
- 自动边缘网络分发
- 简单的 CI/CD 流程
- 内置的预览部署功能

### 配置要求

1. **安装 Vercel Remix 预设包**

```bash
npm install @vercel/remix --save-dev
```

2. **修改 vite.config.ts**

```typescript
import { vitePlugin as remix } from "@remix-run/dev";
import { defineConfig, type UserConfig } from "vite";
import tsconfigPaths from "vite-tsconfig-paths";
import { vercelPreset } from '@vercel/remix/vite';

installGlobals({ nativeFetch: true });

// 其他配置...

export default defineConfig({
  // 其他配置...
  plugins: [
    remix({
      ignoredRouteFiles: ["**/.*"],
      presets: [vercelPreset()],
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
  // 其他配置...
}) satisfies UserConfig;
```

3. **创建 vercel.json 配置文件**

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

4. **导入修改**

将代码中的 `@remix-run/node` 导入替换为 `@vercel/remix`，特别是对于流式响应和服务器端功能。

### 注意事项

- Vercel 默认使用 Edge Functions，但 Shopify 应用通常需要完整的 Node.js 环境
- 需要明确指定 Node.js 运行时版本
- 可能需要调整 Prisma 配置以适应 Vercel 的无服务器环境

## Render 部署配置

### 优势

- 提供完整的 Node.js 环境
- 简单的数据库集成
- 更适合需要长时间运行的进程
- 无需特殊的代码修改

### 配置要求

1. **创建 render.yaml 配置文件**

```yaml
services:
  - type: web
    name: shopify-app-template-remix
    env: node
    plan: free
    buildCommand: npm install && npx prisma generate && npm run build
    startCommand: npm run start
    healthCheckPath: /
    envVars:
      - key: NODE_ENV
        value: production
      - key: PORT
        value: 8080
      - key: SHOPIFY_API_KEY
        sync: false
      - key: SHOPIFY_API_SECRET
        sync: false
      - key: SHOPIFY_APP_URL
        sync: false
      - key: SCOPES
        sync: false
      - key: DATABASE_URL
        sync: false
    autoDeploy: true
```

2. **无需修改 vite.config.ts**

Render 部署不需要对 vite.config.ts 进行特殊修改，可以使用原始配置。

### 注意事项

- 需要在 Render 上创建 PostgreSQL 数据库
- 确保设置正确的环境变量
- 可能需要调整 Prisma 配置以适应 Render 的环境

## 选择建议

### 选择 Vercel 的情况

- 如果您需要更快的冷启动时间
- 如果您的应用不依赖于完整的 Node.js API
- 如果您需要更强大的预览部署功能
- 如果您的应用流量模式适合无服务器架构

### 选择 Render 的情况

- 如果您的应用需要完整的 Node.js 环境
- 如果您使用 Prisma 或其他需要长连接的数据库工具
- 如果您的应用有长时间运行的进程
- 如果您希望更简单的部署配置，无需代码修改

## 结论

两个平台都可以成功部署 Shopify Remix 应用，选择取决于您的具体需求：

- **Vercel**: 更适合轻量级、无状态的应用，提供更好的边缘网络性能
- **Render**: 更适合需要完整 Node.js 环境的应用，提供更简单的配置和更好的长连接支持

无论选择哪个平台，都确保正确配置环境变量，特别是 Shopify API 密钥和回调 URL。
