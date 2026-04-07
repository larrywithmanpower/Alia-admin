# Alia & Bra — 後台管理系統

**線上網址：** https://larrywithmanpower.github.io/Alia-admin/

Alia & Bra 品牌後台，供管理者進行商品上架、編輯與分類設定。

---

## 功能

### 商品管理（/）
- 商品列表：顯示所有商品，含縮圖、名稱、分類、價格、標籤、首頁精選狀態、購買連結
- 搜尋：即時依名稱關鍵字篩選商品
- 分類篩選：點擊 tab 快速過濾對應分類商品
- 新增商品：填寫名稱、分類、價格、標籤、購買連結、圖片、是否顯示於首頁精選
- 編輯商品：修改所有欄位，儲存後立即同步前台
- 刪除商品：刪除前需二次確認，刪除後自動移除 Supabase Storage 上的對應圖片
- 圖片上傳：
  - 支援貼上圖片網址
  - 支援直接上傳圖片檔（點擊選擇或拖曳）
  - 上傳前自動壓縮（最大寬度 1200px，JPEG 82% 品質）
  - 更換圖片時自動刪除舊圖（避免 Storage 堆積）

### 分類設定（/categories）
- 顯示所有分類與各分類下的商品數量
- 新增分類：輸入名稱後新增，名稱不可重複
- 重新命名：行內編輯，儲存後自動將所有該分類商品同步更新
- 刪除分類：有商品的分類無法刪除，需先將商品移至其他分類

### 其他
- 登入驗證：Supabase Auth，未登入自動導向登入頁
- 深色／淺色主題切換：手動切換，偏好設定儲存於 localStorage
- 自動部署：push 到 `main` 觸發 GitHub Actions 部署

---

## 技術

- Nuxt 3（SSG，github-pages preset）
- Supabase（PostgreSQL + Auth + Storage + RLS）
- 部署：GitHub Pages

---

## 本地開發

```bash
npm install
npm run dev
```

開啟 http://localhost:3001/Alia-admin/

---

## 環境變數

建立 `.env`：

```
NUXT_PUBLIC_SUPABASE_URL=你的 Supabase URL
NUXT_PUBLIC_SUPABASE_KEY=你的 Publishable Key
```

GitHub Actions 需在 repo Settings → Secrets 設定相同兩個變數。

---

## 部署

push 到 `main` 分支會自動觸發 GitHub Actions 部署到 GitHub Pages。
