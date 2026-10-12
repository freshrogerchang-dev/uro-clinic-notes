# uro-clinic-notes — Dev-Flow State
最後更新：2026-10-12（S4/S5 擴充整合）

## 一句話目標
泌尿科門診工具新增癌症主訴與右側抽屜、癌症風險分组、PSAD/PSADT 和臨床摘要，結果保留版本与來源。

## Phase 狀態
- S1：聚焦既有專案的功能/架構已完成；未作全面市場調查。
- S2：軟體相容性精簡版已完成，維持靜態網站/Firebase。
- S3：本次模型規則已核對與文件化；醫師獨立臨床審查未完成。
- S4：本次要求的擴充已整合正式 index.html，本次已授權提交/推送，發布狀態以 GitHub 執行結果為準。
- S5：本次純函式與模擬 Firebase 的瀏覽器驗證完成；真實 Firebase、Safari/裝置與臨床驗收未驗證。

## 關鍵決策
| 日期 | Session | 決策 | 理由 |
|---|---|---|---|
| 2026-10-12 | 既有/S4 | 保留 HTML/CSS/JS + Firebase，新增 assets | 增量整合，不大改架構 |
| 2026-10-12 | S3 | 指引版本与核閱日期分開 | EAU online 2026-10-12；AUA NMIBC 明列2024 |
| 2026-10-12 | S4 | 點選開抽屜；最後選單完成收回，文字/數值按完成 | 避免輸入途中關閉 |
| 2026-10-12 | S4 | 未知不當陰性、stale不加入病歷、醫師opt-in | 保留來源与確認 |
| 2026-10-12 | S4 | UID草稿隔離；身分/病人更換清空本次工具 | 防止舊結果帶入 |

## 本次完成
五癌症入口；PSAD、prostate EAU、NMIBC EAU/AUA、術後 RCC（Leibovich/UISS/VENUSS）、IMDC、UTUC、PSADT、病理/結石/日誌。結果/版本/輸入保存、歷次顯示、追蹤參考、紅旗三態、體積不自動勾診斷。Node tests 24/24；headless Edge desktop/390px UI、模擬UID與保存一致性通過。

## 未決 / 限制
- Firestore rules 未取得，正式登入/雲端存取權限未驗證。
- 醫師驗收、iPhone/Safari/VoiceOver未執行。
- NMIBC僅WHO2004/2016；特殊亚型需複核；UTUC仅弱因子不確定分組。
- RCC 原模型grade需臨床確認；PSADT未自動識別治療階段。
- 追蹤提供提示与官方表，不自動排醫囑；舊固定治療摘要未全面年度更新。
- 舊v1草稿未刪除、不自動載入；新草稿v2按UID。

## 下一步
完成本次push與公開Pages部署檢查；後續 /s5-verify 使用醫師指定案例及正式權限環境驗收。
