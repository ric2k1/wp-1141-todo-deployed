# 快速部署指南 - Vercel

這是 `todo-deployed` 專案的快速部署檢查清單。

## 🚀 快速步驟

### 1. 準備資料庫

選擇一個 PostgreSQL 資料庫服務：

- **Vercel Postgres**（推薦，整合最佳）
- **Supabase** - https://supabase.com
- **Neon** - https://neon.tech
- **Railway** - https://railway.app

記下您的資料庫連接字串，格式如下：

```
postgresql://username:password@host:port/database?schema=public
```

### 2. 產生 NEXTAUTH_SECRET

在終端機執行：

```bash
openssl rand -base64 32
```

或使用線上工具：https://generate-secret.vercel.app/32

### 3. 部署到 Vercel

#### 方法 A：透過 Vercel Dashboard（最簡單）

1. 前往 https://vercel.com 並登入
2. 點擊 **Add New...** → **Project**
3. 選擇您的 Git 提供者（GitHub/GitLab/Bitbucket）
4. 搜尋並選擇 `wp-1141-todo-deployed` 倉庫
   - ⚠️ **提示**：如果您的程式碼已經 push 到 GitHub，可以直接在列表中選擇該倉庫
5. **設定專案**：
   - **Root Directory**: 留空（專案已在倉庫根目錄）
   - 其他設定會自動從 `vercel.json` 讀取
6. **設定環境變數**（重要！在部署前設定）：
   - `DATABASE_URL` - 您的 PostgreSQL 連接字串
   - `NEXTAUTH_SECRET` - 步驟 2 產生的密鑰
   - `NEXTAUTH_URL` - 先設定為 `https://your-project.vercel.app`（部署後可更新）
7. 點擊 **Deploy**

#### 方法 B：使用 Vercel CLI

```bash
# 安裝 CLI
npm i -g vercel

# 登入
vercel login

# 連結專案（在專案目錄下）
cd todo-deployed  # 或您存放專案的位置
vercel link        # 連結到現有的 Vercel 專案，或選擇建立新專案

# 設定環境變數（在連結專案後）
vercel env add DATABASE_URL
vercel env add NEXTAUTH_SECRET
vercel env add NEXTAUTH_URL

# 部署到預覽環境
vercel

# 生產環境部署
vercel --prod
```

### 4. 執行資料庫遷移 ⚠️ 必做！

部署完成後，**必須**執行資料庫遷移來建立資料表：

```bash
# 方法 1：使用 Vercel CLI（推薦）
# 確保已安裝 Vercel CLI: npm i -g vercel
cd todo-deployed  # 或您存放專案的位置
vercel link        # 連結到您的 Vercel 專案（如果還沒連結）
vercel env pull .env.production
DATABASE_URL=$(grep DATABASE_URL .env.production | cut -d '=' -f2-) npx prisma migrate deploy

# 方法 2：直接執行（如果有 DATABASE_URL）
DATABASE_URL="your-database-url" npx prisma migrate deploy

# 方法 3：在 package.json 中已配置自動遷移（可選）
# 如果使用 vercel-build 腳本，遷移會自動執行
```

### 5. 驗證部署

1. 訪問您的部署 URL（例如：`https://your-project.vercel.app`）
2. 測試登入功能
3. 測試建立、編輯、刪除 Todo
4. 檢查是否有錯誤（在 Vercel Dashboard → Logs）

## ⚙️ 自動執行遷移（可選）

如果您希望在每次部署時自動執行遷移，`package.json` 已包含 `vercel-build` 腳本。

更新 `vercel.json` 使用該腳本：

```json
{
  "buildCommand": "yarn vercel-build",
  "installCommand": "yarn install",
  "framework": "nextjs",
  "regions": ["hkg1"]
}
```

**注意**：這會在每次建置時執行遷移，可能增加建置時間。如果遷移很少變更，建議手動執行。

## 🔧 環境變數清單

### 必要變數

| 變數名稱          | 說明                      | 範例                                                |
| ----------------- | ------------------------- | --------------------------------------------------- |
| `DATABASE_URL`    | PostgreSQL 連接字串       | `postgresql://user:pass@host:5432/db?schema=public` |
| `NEXTAUTH_SECRET` | NextAuth 密鑰（隨機字串） | 使用 `openssl rand -base64 32` 產生                 |
| `NEXTAUTH_URL`    | 應用程式的完整 URL        | `https://your-project.vercel.app`                   |

### OAuth 變數（如果使用 OAuth）

| 變數名稱                 | 說明                       |
| ------------------------ | -------------------------- |
| `GOOGLE_CLIENT_ID`       | Google OAuth Client ID     |
| `GOOGLE_CLIENT_SECRET`   | Google OAuth Client Secret |
| `GITHUB_CLIENT_ID`       | GitHub OAuth Client ID     |
| `GITHUB_CLIENT_SECRET`   | GitHub OAuth Client Secret |
| `FACEBOOK_CLIENT_ID`     | Facebook OAuth App ID      |
| `FACEBOOK_CLIENT_SECRET` | Facebook OAuth App Secret  |

## ❌ 常見問題

### 建置失敗：Prisma Client 未生成

- 確認 `vercel.json` 中的 `buildCommand` 包含 `prisma generate`
- 或使用 `vercel-build` 腳本（已包含 generate）

### 運行時錯誤：資料表不存在

- **最重要**：執行資料庫遷移（步驟 4）
- 確認 `DATABASE_URL` 正確設定

### 認證失敗

- 確認 `NEXTAUTH_SECRET` 已設定
- 確認 `NEXTAUTH_URL` 與實際 URL 一致（包含 `https://`，無尾隨斜線）

### 資料庫連接錯誤

- 確認資料庫允許來自 Vercel IP 的連接
- 如果使用 Vercel Postgres，確保資料庫已連結到專案

## 📝 更新部署

每次推送到 Git 主分支會自動觸發新部署。

手動部署：

```bash
vercel --prod
```

## 📚 詳細文件

查看 [VERCEL_DEPLOY.md](./VERCEL_DEPLOY.md) 獲取完整的部署說明和故障排除指南。
