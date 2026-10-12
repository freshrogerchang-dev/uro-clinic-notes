# S4 Build Log
2026-10-12

## 啟動
`python -m http.server 8000 --bind 127.0.0.1`；以 `/?demo=1` 試用。沿用 Firebase，沒有新增 secrets、DB 或套件。

## 切片
- [x] 共用右側抽屜、PSAD、癌症工具與五個主訴入口。
- [x] PSADT、病理/結石/排尿日誌摘要，結果来源與追蹤原文。
- [x] SOAP opt-in、保存输入/模型版本、歷次結果、草稿 UID 隔離、換病人 reset、保存前 stale check。
- [x] 紅旗三態、移除體積自动勾 N40.1。
- [x] 核心邏輯與 offline browser smoke；桌面與手機截图。
- [ ] 正式 Firebase 權限與真實帳號保存验证、醫師案例驗收、Safari/iPhone 實機。

## 修改
index.html 接上 assets，保留既有風格和流程。clinical-core.js 為純函式与可執行模型。clinical-tools.js 為版本化狀態与抽屜 UI；已修正關閉時遲來 change 事件讓結果誤標 stale，並限制 auto-close 只觸發於最後 select 操作，文字/數值以完成按鈕。

## 驗證
24 項 Node test 通過；Playwright 用 headless Edge、Firebase stub 走十工具、unknown/stale/reset、手機無橫向溢出与焦點/escape。測試過程錯誤已修復後重跑，結果見 05-audit-report.md；不將 mock 結果稱正式雲端验证。
