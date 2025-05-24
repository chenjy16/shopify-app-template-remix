# Shopify Remix 应用模板

这是一个基于 Remix 框架的 Shopify 应用模板，可以部署在 Vercel 或 Render 上。

## 部署指南

### Vercel 部署

1. **准备工作**

   确保你已经安装了 Vercel CLI：

   ```bash
   npm install -g vercel
   ```

2. **安装依赖**

   ```bash
   npm install @vercel/remix --save-dev
   ```

3. **配置文件**

   项目已包含必要的配置文件：
   - `vite.config.ts` - 已配置 Vercel 预设
   - `vercel.json` - 已配置 Node.js 运行时

4. **环境变量设置**

   在 Vercel 控制台中设置以下环境变量：

   - `SHOPIFY_API_KEY` - 你的 Shopify API 密钥
   - `SHOPIFY_API_SECRET` - 你的 Shopify API 密钥
   - `SHOPIFY_APP_URL` - 你的应用 URL (例如 https://your-app.vercel.app)
   - `SCOPES` - 应用所需的权限范围
   - `DATABASE_URL` - 数据库连接字符串

5. **部署**

   ```bash
   vercel
   ```

   或者通过 GitHub 集成自动部署。

### 重要注意事项

1. **Node.js 运行时**

   Vercel 默认使用 Edge Functions，但 Shopify 应用通常需要完整的 Node.js 环境。`vercel.json` 文件已配置为使用 Node.js 运行时。

2. **导入修改**

   在使用流式响应或其他服务器端功能时，将导入从 `@remix-run/node` 改为 `@vercel/remix`。

3. **数据库连接**

   如果使用 Prisma，确保你的数据库连接配置适用于无服务器环境。考虑使用连接池或 Prisma 的 Data Proxy。

4. **环境变量**

   确保在 Vercel 控制台中设置了所有必要的环境变量。

### Render 部署选项

如果你希望在 Render 上部署，请参考 `deployment-comparison.md` 文件了解详细信息。

## 本地开发

1. **安装依赖**

   ```bash
   npm install
   ```

2. **设置环境变量**

   创建 `.env` 文件并设置必要的环境变量。

3. **启动开发服务器**

   ```bash
   npm run dev
   ```

## 故障排除

### Vercel 部署问题

1. **运行时错误**

   如果遇到 "Function Runtimes must have a valid version" 错误，检查 `vercel.json` 文件中的 `runtime` 配置是否正确。

2. **Prisma 生成错误**

   确保在构建命令中包含 `npx prisma generate`。

3. **环境变量问题**

   检查是否正确设置了所有必要的环境变量。

### 更多信息

有关在 Vercel 上托管 Remix 应用的更多信息，请参阅 [Vercel Remix 文档](https://vercel.com/docs/frameworks/remix)。
