# S2 軟體相容性精簡版
2026-10-12

- 保留既有單頁 HTML + Firebase compat SDK 10.13.0，不新增主要 framework、API、MCP 或 DB。
- 新增 classic scripts `assets/clinical-core.js`（瀏覽器 global / Node CommonJS）及 `clinical-tools.js`、CSS；相對路徑適用 GitHub Pages repository 子目錄。
- 使用原生 dialog.showModal、動態表單、aria-live 與 prefers-reduced-motion；已測 Microsoft Edge 桌面/390px viewport，Safari/iPhone 未實機測試。
- Node built-in test runner 驗證純函式；Playwright 使用開發環境 bundled module 與系統 Edge。無 package.json/build，不另裝套件。
- 共用既有病歷儲存流程，新增 clinicalTools snapshot；數據改動後須重新 generate，避免保存舊 noteText 配上新工具輸入。
- 本機 draft key 由 UID 決定；demo 不保存，登入身分改變先清表單再恢復該 UID 的草稿。舊 v1 草稿不自動恢復。
- Firestore 正式 rules 未取得，Firebase 登入/權限/寫入尚未實測；瀏覽器測試用 stub 確認資料物件、帳號切換与 stale-save guard。
- 資產帶版本 query；部署時需包含 assets/。
