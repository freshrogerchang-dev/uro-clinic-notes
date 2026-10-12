# 泌尿科門診病歷輔助工具

靜態 HTML/CSS/JavaScript 網站，使用既有 Firebase Auth/Firestore 保存病歷。

公開網站：[uro-clinic-notes](https://freshrogerchang-dev.github.io/uro-clinic-notes/)。網址加 `?demo=1` 可試用，不登入、不儲存病人資料。正式環境須核對 Firestore rules，前端授權名單不是資料庫存取控制。

## 本機啟動

在 repository 執行 `python -m http.server 8000 --bind 127.0.0.1`，開啟 `http://127.0.0.1:8000/?demo=1`。沒有 npm build；部署需包含 `index.html` 和 `assets/`。

## 功能

- 既有 SOAP、ICD、問卷、回診帶入、數據趨勢與處置紀錄。
- 主訴入口新增攝護腺癌、膀胱癌、腎癌、UTUC、睪丸癌；選主訴不等於確诊，診斷需另行勾選。
- 點選右側抽屜：PSAD、攝護腺癌 EAU、膀胱癌 EAU/AUA、術後 RCC、IMDC、UTUC、PSA doubling time、病理、結石、排尿日誌。
- 全部必填完成後最後點選選單會收回；文字/數值完成後按「完成並收回」。未知資料保留待填；模型不適用或需複核時不作確定分類。
- 結果卡可修改/清除、查看輸入/版本/來源；只有醫師選「加入 Assessment」才寫入病歷。編輯後舊結果不沿用，保存前必須重新產生病歷。
- 癌別追蹤提醒與官方追蹤表外連；沒有自動開立醫囑。歷次結果保存版本，不自動當成本次結果。
- 紅旗改有/無/未詢問；只有明確「無」寫 Denied。攝護腺體積不再自動勾診斷。
- 本機草稿依 UID 隔離（v2）；舊 v1 草稿不自動帶入其他帳號，也未自動刪除。

## 規則與限制

分期、風險分組、機率為不同概念。詳見 [研究與規則來源](docs/dev-flow/03-research.md)。EAU 採 2026-10-12 核閱之網頁內容，AUA 採 2024 amendment；未宣稱所有指引都是 2026 年版。

NMIBC 僅支援 WHO 2004/2016 LG/HG/PUNLMP；不自行換算 WHO 1973。特殊亞型列需複核。RCC 使用原模型的 grade，須核對分級相容性。UTUC 僅弱因子列需個別評估。PSADT 此工具要求至少三筆 PSA >0.1、跨至少 90 天，尚未用於判定治療反應。

## 驗證

`node --test tests/clinical-core.test.cjs` 執行決定性規則測試，無額外套件。

`node tests/browser-smoke.cjs` 需要 Node、Playwright 與 Microsoft Edge。可用 `PLAYWRIGHT_MODULE` 指定 Playwright 模組路徑；測試內 fallback 使用此開發環境的 bundled runtime。瀏覽器測試模擬 Firebase、不写正式雲端，結果與截圖見 `docs/verification/`。

本次自動化測試不代表醫師臨床驗收或正式 Firebase 權限審核。
