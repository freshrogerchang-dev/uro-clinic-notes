# S1 現有工具分析與擴充架構

分析日期：2026-10-12。基準：GitHub main `3e91445`，本機既有 clone 已更新。此文件為功能分析及實作規格，不是完成的臨床計算器。

## 現有功能與程式證據
Repository 目前只有 `index.html`，無 package.json、測試或 Firebase rules。已有 Google 登入、英語 SOAP、ICD 選擇、IPSS/OABSS/IIEF-5、檢查快捷輸入、處置、回診資料帶入、PSA/體積/Qmax/PVR 趨勢、Firestore 病歷與 CSV/SQL 匯出、localStorage 草稿。

| 功能 | 證據 | 擴充方向 |
|---|---|---|
| PSA density | computeUroMetrics、uroMetricsLines、updateUroMeasureTag | 已有 PSA/volume；移到抽屜或由抽屜共用，增加日期、量測方式、三位小數 |
| 腫瘤參考 | cancerGuidelines、populateGuidelineStages、updateGuidelineOutput | 現為手選分期後查固定文字；新增真正根據輸入判定的風險工具 |
| 回診與趨勢 | carryForwardFromRecord、renderTrends | 加上病理摘要與癌症風險歷次版本；不自動沿用舊風險 |
| 儲存 | collectFormSnapshot、saveToFirebase、lastStructured | 未來保存輸入、模型版本與結果；病人切換需清空 |

## 核心功能建議
| 工具 | 輸入規格草案 | 結果與範圍 |
|---|---|---|
| PSAD | PSA ng/mL、體積 mL、日期、MRI/TRUS；可另選長寬高與單位 | PSA ÷ 體積，ng/mL/cc；不是分期或確診。0、負值、無限值、空白體積不可計算；判定用未四捨五入值 |
| 攝護腺癌 EAU | PSA、ISUP GG、DRE cT、cN、cM；MRI 分期另存 | 局限性疾病風險、局部晚期或轉移性狀態；目前官方表已區分 favorable/unfavorable intermediate，不能直接照舊 D’Amico 表實作 |
| 膀胱癌 EAU NMIBC | Ta/T1/CIS、grade 與 WHO 系統、年齡、最大徑、多發、初發/復發、CIS、LVI、亞型、攝護腺尿道侵犯、BCG 狀態 | low/intermediate/high/very high；BCG 病例與特殊病理需另走適用性分支 |
| 膀胱癌 AUA NMIBC | 共用上述資料，再確認復發間隔與治療史 | low/intermediate/high；與 EAU 分開顯示，不共用一套分類；T2+、轉移不套 NMIBC |
| 腎癌術後 ccRCC | pT、pN、最大徑、grade、壞死、組織型、M 狀態 | Leibovich 2003 術後復發風險；非 ccRCC 另評估 UISS，未開刀不能用術後模型 |
| 轉移性 RCC（第二批） | KPS、診斷至治療時間、Hb/校正鈣/嗜中性球/血小板及 ULN/LLN | IMDC 預後分組，與術後復發風險分開；需另外核對原始模型 |
| UTUC EAU | URS biopsy grade、cytology、影像侵犯、亞型、大小、多發、水腎、N/M | 術前非轉移風險分組；強因子與弱因子分開，不能任一水腎或多發即無条件高風險；缺資料不自動低風險 |

來源對上述工具方向的支持：
- [EAU 攝護腺癌分類 Table 4.3](https://uroweb.org/guidelines/prostate-cancer/chapter/classification-and-staging-systems)：風險分組依 PSA、GG、臨床分期；MRI 與 DRE cT 不混用。
- [EAU PSA density 與 MRI 診斷章節](https://uroweb.org/guidelines/prostate-cancer/chapter/diagnostic-evaluation)：PSAD 解讀需情境；不以單一 0.15 取代整體評估。
- [EAU NMIBC Table 6.2](https://uroweb.org/guidelines/non-muscle-invasive-bladder-cancer/chapter/predicting-disease-recurrence-and-progression)：四組分類，年齡、大小、多發等因素需一起評估。
- [AUA/SUO 2024 NMIBC Table 4](https://www.auanet.org/documents/Guidelines/PDF/2024%20Guidelines/NMIBC%20Amendment%2001-09-24%20Final.pdf)：三組風險，復發與特殊病理影响分類；此來源為 2024 amendment，不宣稱已完成 2026 最新版本排查。
- [EAU RCC 預後模型](https://uroweb.org/guidelines/renal-cell-carcinoma/chapter/prognostic-factors)、[術後追蹤](https://uroweb.org/guidelines/renal-cell-carcinoma/chapter/followup-in-rcc)：模型依組織型及疾病情境選擇。
- [EAU UTUC 風險](https://uroweb.org/guidelines/upper-urinary-tract-urothelial-cell-carcinoma/chapter/risk-stratification)：grade 重要；大小、多發及水腎在低 grade 情境不宜單獨當成侵襲疾病強證據。

已核對官方章節支持方向；尚未整理可執行完整決策表，年份與表格版本需 S3 鎖定，不據此直接發布分類器。

## 還值得增加的功能與優先序
1. **P0 病歷品質**：紅旗三態（有／無／未詢問），只有明確回答無才能寫 Denied；診斷建議須點選確認。現有 syncProstateVolIcd 依體積勾 N40.1，值得一起修正。
2. **P1 核心工具**：共用抽屜 + PSAD 改良 + 攝護腺癌、NMIBC EAU/AUA；主畫面結果可以修改、清除及「加入 Assessment」。
3. **P2 腫瘤延伸**：UTUC、術後 RCC，病理欄位（stage/grade/CIS/LVI/margins）、按模型整理追蹤提醒；提醒不等於自動醫囑。
4. **P3 診療效率**：PSA doubling time（日期與可用點數確認）、結石尺寸/位置/CT HU 與病史摘要、排尿日誌摘要。現有趨勢與問卷不重複建造。
5. **維護基礎**：拆出臨床純函式、版本化規則與測試；核對 Firestore rules、草稿跨帳號/跨病人隔離。repo 沒有 rules 並不代表線上沒有，只代表此次無法驗證。

## 點選右側抽屜規格
- 初始畫面僅顯示工具按鈕及已完成結果；輸入選單完全收起。
- 點按「PSAD／攝護腺癌／膀胱癌／腎癌／UTUC」後抽屜從右側出現；桌機建議寬 420–480px，手機用全寬。
- 輸入題目採單選卡片、按鈕或數值欄位；每個是／否都有「未知」；不得預設陰性。
- 單一題目選擇後只收起該題選項或進到下一題。多欄位工具需完成全部必填才收回整個抽屜；數值欄位以 Enter/完成確認，避免打字途中消失。
- 完成後留下結果卡：模型名稱/版本、風險組、觸發因子、重要輸入、來源、計算時間及修改按鈕。EAU/AUA 可同卡分兩行。
- 缺必要資料→列缺少欄位並留在抽屜；不適用→顯示原因；有模型外特殊條件→需醫師複核，不輸出假精確百分比。
- 取消或 Esc 不產生新結果；重開保留本次草稿；改變輸入就標記舊結果待重新計算；換病人/清除表單時清空。
- dialog accessible name、焦點限制及返回按鈕、遮罩、Esc、aria-live、減少動畫；結果不只以顏色區分。
- 結果預設留在工具區，醫師點「加入 Assessment」才寫入；加入後不可只因 drawer 選項改動就悄悄改病歷。

## Build / Buy / Fork
| 類型/參考 | 選擇 | 原因/限制 |
|---|---|---|
| 既有網站 | 增量自建 | 不需重新登入或新 DB；既有 Firebase 維持 |
| EAU/AUA 規則與官方來源 | 自建版本化函式，附原文 | 文獻不是開源程式授權；大段表格/治療內容重製須確認條款 |
| [MSKCC nomograms](https://www.mskcc.org/nomograms/prostate) | 提供外連參考 | 機率模型有適用族群，不直接照抄結果或假設可嵌入/API |
| 第三方 API/SaaS | 第一批不需要 | 決定性公式在瀏覽器計算，不必把病人資料送到外部 |
| 未知授權開源 cancer calculator | 暫不 fork | 未找到並核對適合此 repo 的完整規則、授權與驗證證據 |

本次為聚焦既有專案擴充，未完成 S1 全市場 3–5 商用／3–5 開源比較；不捏造 stars、授權或最後更新。

## 架構與資料流
```mermaid
flowchart LR
  Button[工具按鈕] --> Drawer[共用右側抽屜]
  Drawer --> Validation[單位與適用性檢查]
  Validation --> Rules[版本化純函式規則]
  Rules --> Card[結果卡與判定依據]
  Card --> Confirm[醫師確認加入 Assessment]
  Confirm --> Note[既有 SOAP]
  Note --> Save[既有 Firestore 儲存]
```

資料：CancerRiskInput（工具、情境、數值、單位、unknown、測量日期）；CancerRiskResult（status: complete/incomplete/not-applicable/review-required、riskGroup、score、missingFields、reasons、modelId、modelVersion、sourceUrl、inputSnapshot、computedAt）。不要把群體進展機率當個人機率。

## MVP 與驗證交接
第一批：抽屜、PSAD、攝護腺癌 EAU、NMIBC EAU/AUA、結果摘要。第二批：UTUC、RCC 術後，第三批 IMDC/PSADT。順序為建議，不減少使用者要求的最終癌別範圍。

S2：確認靜態 ES modules/檔案拆分、Firebase SDK 載入失敗與 demo 行為；不強制新增套件、MCP 或 DB。

S3：每個模型鎖年份/表號與適用族群；WHO 1973 與 2004/2016 grade 系統不自行換算；核對 BCG 狀態、UTUC 圖強弱因子與 RCC grade 模型要求。

S4/S5：測試閾值前後（PSA 10/20、PSAD 常用切點、3cm、70歲等），PSAD 比較用原值；缺值、矛盾、M1、T2+ NMIBC、不適用組織型；EAU/AUA 分歧案例；換病人清空與加入病歷一致性；手機/鍵盤/抽屜完整流程。焦點與真瀏覽器測試尚未執行。

## 本次交付界線
原網站 `index.html` 僅更新至遠端 main，沒有加入未驗證的癌症規則。`docs/prototypes/risk-drawer.html` 是獨立互動原型：PSAD 可算算術結果，癌症工具只示範輸入與「待規則驗證」結果，不作臨床分組。沒有 commit、push 或部署。
