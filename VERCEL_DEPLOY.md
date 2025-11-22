# Vercel 部署指南

本指南將幫助您將 todo-deployed 應用程式部署到 Vercel。

## 前置需求

1. **Vercel 帳號**: 如果還沒有的話，請到 [vercel.com](https://vercel.com) 註冊
2. **PostgreSQL 資料庫**: 需要一個 PostgreSQL 資料庫實例（可以使用 Vercel Postgres、Supabase、Neon 等）
3. **GitHub/GitLab/Bitbucket 帳號**: 用於連接程式碼倉庫

## 步驟 1: 準備資料庫

### 選項 A: 使用 Vercel Postgres（推薦）

1. 在 Vercel Dashboard 中建立新專案後，前往 **Storage** 標籤
2. 選擇 **Create Database** → **Postgres**
3. 建立資料庫並記下連接資訊

### 選項 B: 使用其他 PostgreSQL 服務

- **Supabase**: https://supabase.com
- **Neon**: https://neon.tech
- **Railway**: https://railway.app

## 步驟 2: 設定環境變數

在 Vercel Dashboard 中，前往 **Settings** → **Environment Variables**，添加以下變數：

### 必要變數

```env
DATABASE_URL="postgresql://username:password@host:port/database?schema=public"
NEXTAUTH_SECRET="your-secret-key-here"
NEXTAUTH_URL="https://your-project.vercel.app"
```

**如何產生 NEXTAUTH_SECRET：**

```bash
openssl rand -base64 32
```

或使用線上工具：https://generate-secret.vercel.app/32

### OAuth 變數（如果使用 OAuth 認證）

```env
GOOGLE_CLIENT_ID="your-google-client-id"
GOOGLE_CLIENT_SECRET="your-google-client-secret"
GITHUB_CLIENT_ID="your-github-client-id"
GITHUB_CLIENT_SECRET="your-github-client-secret"
FACEBOOK_CLIENT_ID="your-facebook-client-id"
FACEBOOK_CLIENT_SECRET="your-facebook-client-secret"
```

**重要提醒：**

- `NEXTAUTH_URL` 在第一次部署時可以使用 `https://your-project.vercel.app`，之後可以在部署完成後更新為實際的域名
- 確保 `NEXTAUTH_URL` 與您的實際部署 URL 一致（包括 `https://` 前綴，沒有尾隨斜線）

## 步驟 3: 部署到 Vercel

### 方法 A: 透過 Vercel Dashboard（推薦）

1. **連接到 Git 倉庫：**

   - 登入 Vercel Dashboard
   - 點擊 **Add New...** → **Project**
   - 選擇您的 Git 提供者（GitHub、GitLab 或 Bitbucket）
   - 選擇 `todo-deployed` 倉庫

2. **設定專案：**

   - **Framework Preset**: Next.js（應該會自動偵測）
   - **Root Directory**: 如果專案在子目錄，設定為 `todo-deployed`
   - **Build Command**: `prisma generate && next build`（應該會從 vercel.json 讀取）
   - **Output Directory**: `.next`（預設）
   - **Install Command**: `yarn install`（應該會從 vercel.json 讀取）

3. **環境變數：**

   - 在部署前添加所有必要的環境變數（參考步驟 2）

4. **部署：**
   - 點擊 **Deploy**
   - 等待建置完成

### 方法 B: 使用 Vercel CLI

1. **安裝 Vercel CLI：**

```bash
npm i -g vercel
```

2. **登入：**

```bash
vercel login
```

3. **部署：**

```bash
cd todo-deployed
vercel
```

按照提示操作：

- 連結現有專案或建立新專案
- 確認設定（vercel.json 會自動套用）

4. **設定環境變數：**

```bash
vercel env add DATABASE_URL
vercel env add NEXTAUTH_SECRET
vercel env add NEXTAUTH_URL
```

5. **生產環境部署：**

```bash
vercel --prod
```

## 步驟 4: 執行資料庫遷移

部署完成後，需要執行資料庫遷移來建立資料表結構。

### 方法 A: 使用 Vercel CLI（推薦）

```bash
# 連接到生產環境
vercel env pull .env.production

# 執行遷移
DATABASE_URL=$(grep DATABASE_URL .env.production | cut -d '=' -f2-) npx prisma migrate deploy

# 或使用 Prisma 的 migrate deploy（適用於生產環境）
npx prisma migrate deploy --schema=./prisma/schema.prisma
```

### 方法 B: 使用 Vercel Postgres CLI

如果使用 Vercel Postgres：

```bash
# 安裝 Vercel Postgres CLI
npm i -g @vercel/postgres

# 執行 SQL 遷移
vercel postgres migrate --yes
```

### 方法 C: 手動執行遷移 SQL

1. 在 Vercel Dashboard 中，找到您的資料庫連接資訊
2. 使用資料庫管理工具（如 pgAdmin、TablePlus 等）連接
3. 執行 `prisma/migrations/*/migration.sql` 中的所有 SQL 檔案

### 方法 D: 使用 Prisma Migrate Deploy Script（推薦用於 CI/CD）

您可以在 `package.json` 中添加一個腳本：

```json
{
  "scripts": {
    "postbuild": "prisma migrate deploy"
  }
}
```

但這會讓每次建置都執行遷移。更好的方式是使用 Vercel 的 `postinstall` 腳本或在建置後手動執行。

## 步驟 5: 驗證部署

1. **檢查部署狀態：**

   - 在 Vercel Dashboard 中查看部署日誌
   - 確認建置成功且沒有錯誤

2. **測試應用程式：**

   - 訪問您的部署 URL（例如：`https://your-project.vercel.app`）
   - 測試登入功能
   - 測試建立、編輯、刪除 Todo

3. **檢查環境變數：**
   - 確認所有環境變數都已正確設定
   - 檢查 `NEXTAUTH_URL` 是否與實際 URL 一致

## 故障排除

### 建置失敗

**問題：Prisma Client 未生成**

解決方案：確保 `vercel.json` 中的 `buildCommand` 包含 `prisma generate`

```json
{
  "buildCommand": "prisma generate && next build"
}
```

**問題：資料庫連接錯誤**

解決方案：

- 確認 `DATABASE_URL` 環境變數正確設定
- 檢查資料庫是否允許來自 Vercel IP 的連接
- 如果使用 Vercel Postgres，確保資料庫已正確連結到專案

### 運行時錯誤

**問題：認證失敗**

解決方案：

- 確認 `NEXTAUTH_SECRET` 已設定
- 確認 `NEXTAUTH_URL` 與實際部署 URL 一致
- 檢查 OAuth 設定（如果使用）

**問題：資料表不存在**

解決方案：

- 執行資料庫遷移（參考步驟 4）
- 確認遷移檔案已正確執行

**問題：Migration 執行失敗**

解決方案：

- 確保 `prisma/migrations` 目錄已包含在 Git 倉庫中
- 檢查遷移檔案語法是否正確
- 確認資料庫用戶有執行 DDL 的權限

### 環境變數問題

**問題：環境變數未更新**

解決方案：

- 在 Vercel Dashboard 中更新環境變數後，需要重新部署
- 使用 CLI：`vercel env pull` 然後 `vercel --prod`

## 更新部署

每次推送程式碼到 Git 倉庫的主分支時，Vercel 會自動觸發新的部署。

如果要手動部署：

```bash
vercel --prod
```

## 自動執行資料庫遷移（進階）

如果您希望在每次部署時自動執行遷移，可以在 `package.json` 中添加：

```json
{
  "scripts": {
    "postinstall": "prisma generate",
    "vercel-build": "prisma migrate deploy && next build"
  }
}
```

並更新 `vercel.json`：

```json
{
  "buildCommand": "yarn vercel-build",
  "installCommand": "yarn install",
  "framework": "nextjs",
  "regions": ["hkg1"]
}
```

**注意**：這會在每次建置時執行遷移，可能會增加建置時間。如果資料庫遷移很少變更，建議手動執行。

## 自訂域名

1. 在 Vercel Dashboard 中，前往 **Settings** → **Domains**
2. 添加您的自訂域名
3. 按照指示設定 DNS 記錄
4. 更新 `NEXTAUTH_URL` 環境變數為新的域名

## 監控和日誌

- **部署日誌**: 在 Vercel Dashboard 中查看每次部署的日誌
- **運行時日誌**: 在 **Logs** 標籤中查看即時日誌
- **分析**: 在 **Analytics** 標籤中查看效能指標

## 參考資源

- [Vercel 文件](https://vercel.com/docs)
- [Next.js 部署指南](https://nextjs.org/docs/deployment)
- [Prisma 部署指南](https://www.prisma.io/docs/guides/deployment)
- [NextAuth.js 文件](https://next-auth.js.org/)
