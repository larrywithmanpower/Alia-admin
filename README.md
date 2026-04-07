# Alia & Bra — 後台管理系統

Alia & Bra 品牌後台，用於管理商品上架與分類設定。

## 功能

- 商品管理：新增、編輯、刪除商品
- 分類管理：自訂商品分類
- 深色／淺色主題切換
- Supabase Auth 登入驗證

## 技術

- Nuxt 3
- Supabase（PostgreSQL + Auth）
- 部署：GitHub Pages

## 本地開發

```bash
npm install
npm run dev
```

開啟 http://localhost:3001

## 環境變數

建立 `.env`：

```
NUXT_PUBLIC_SUPABASE_URL=你的 Supabase URL
NUXT_PUBLIC_SUPABASE_KEY=你的 Publishable Key
```

## 部署

push 到 `main` 分支會自動觸發 GitHub Actions 部署到 GitHub Pages。
