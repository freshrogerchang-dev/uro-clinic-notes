# S5 本次擴充驗證報告
2026-10-12

## 已驗證
- `node --test tests/clinical-core.test.cjs`：24/24 通過，涵蓋 PSAD 缺值/精度、PSA 10/20、年齡70/大小3cm、EAU/AUA差異、CIS/LVI、RCC分數2/3/5/6、IMDC上下限、UTUC強弱/未知、PSADT無效日期與下降、摘要驗證。
- HTML inline 與新 JS 語法檢查；git diff --check。
- headless Microsoft Edge：十工具結果、五癌症入口、完成/auto-close、unknown、opt-in、修改使舊結果失效、reset、Esc/焦點返回、390px無橫向溢出、demo localStorage 不寫入、無 pageerror。Firebase 為 stub。
- 模擬帳號測試確認 UID 草稿隔離、變更後保存阻擋、保存物件包含模型版本（雲端資料庫未實測）。
- `docs/verification/clinical-tools-mobile.png`、desktop.png 截圖為合成輸入，无真實病人。

## 檢閱結果與界線
| 類別 | 狀態 |
|---|---|
| 原本未勾紅旗寫 Denied | 已修：明確選無才寫 |
| 體積自動勾 N40.1 | 已修：僅提示，保留醫師勾診斷 |
| 填完後收回、旧结果被遲來事件改 stale | 已修并重跑 |
| 不同登入帳號本機草稿共用 | 新草稿 UID 隔離；旧 v1 未刪除、不自動恢復 |
| 保存 noteText 與更新輸入不一致 | 保存前比較生成時 snapshot，變更须重產 |
| 真實 Firestore rules / 登入保存 | 未验证，沒有權限設定证據 |
| 臨床獨立審查、全模型人工驗收 | 未執行；自動測試不是臨床認證 |
| Safari/iPhone/VoiceOver | 未實機驗证 |
| 既有固定治療摘要年度更新 | 本次保留，未全面重驗 |

無 npm 依賴新增，無 npm build/lint/typecheck 配置；未執行 repo 全面的第三方 security/code-review 插件掃描。此為本次變更範圍檢查，不是全站安全認證。
