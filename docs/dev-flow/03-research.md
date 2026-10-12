# S3 臨床規則與來源
核閱日期 2026-10-12。採官方章節和原始文獻，不用模型推測缺少的病理資料。實作與人工臨床驗收分開。

| 模型 | 規則版本與來源 | 實作界線 |
|---|---|---|
| PSAD | [EAU diagnostic evaluation](https://uroweb.org/guidelines/prostate-cancer/chapter/diagnostic-evaluation) | PSA/volume；長寬高採 cm、π/6；保留未四捨五入值，不給單一數值自動切片建議 |
| 攝護腺癌 | [EAU Table 4.3](https://uroweb.org/guidelines/prostate-cancer/chapter/classification-and-staging-systems) | 採核閱當日網頁，DRE cT；含兩種中度風險，N+/T3–4/M1 另列；不稱 AJCC stage group |
| 膀胱癌 EAU | [2021 NMIBC model / Table 6.2](https://uroweb.org/guidelines/non-muscle-invasive-bladder-cancer/chapter/predicting-disease-recurrence-and-progression) | WHO 2004/2016；初發/復發及附加因子；特殊亞型需複核，無個人進展百分比 |
| 膀胱癌 AUA | [AUA/SUO 2024 Table 4](https://www.auanet.org/documents/Guidelines/PDF/2024%20Guidelines/NMIBC%20Amendment%2001-09-24%20Final.pdf) | 獨立三組分類，BCG failure 由醫師確認，不用程式自行推定 BCG-unresponsive |
| RCC | [EAU Tables 6.3–6.4](https://uroweb.org/guidelines/renal-cell-carcinoma/chapter/prognostic-factors) | Leibovich 2003（pT1b=2）、papillary VENUSS、其他 UISS N0M0；IMDC 與術後模型分開，使用實驗室上下限 |
| UTUC | [EAU risk stratification](https://uroweb.org/guidelines/upper-urinary-tract-urothelial-cell-carcinoma/chapter/risk-stratification) | 非轉移術前資料，強因子判高；僅弱因子需個別評估，未知不判低 |
| PSA doubling time | [Arlen et al. 2008, PMID 18423743](https://pubmed.ncbi.nlm.nih.gov/18423743/)、[可讀全文](https://pmc.ncbi.nlm.nih.gov/articles/PMC2667701/) | ln(PSA) 對月 OLS、ln2/slope；本工具至少三筆、90 天、PSA>0.1。未實作所有原文採樣間隔/治療階段条件，解讀須確認 |
| Leibovich 原始研究 | [Leibovich et al. 2003, PMID 12655523](https://pubmed.ncbi.nlm.nih.gov/12655523/) | 術後 clear-cell RCC 預後模型；PubMed 已核對題名/方法，不從摘要重新導出權重；權重取 EAU 表 |

PubMed connector 取得上述两篇 metadata；abstract 限 1500 字元而截短，此次不用截短摘要宣稱完整文獻回顧或新模型效能。

## 實作演算法與評估
規則只產生分組、分數、命中因子、不適用/資料不足/需複核狀態。輸入的 unknown 不視為 no。PSA、年齡、大小的數值比較用原值；顯示才四捨五入。臨床核閱日與模型歷史年份分開。

PSADT 日期需有效、不重複、至少90天；下降/平穩不回傳負 doubling time。日誌按醫師輸入的完整24小時天數摘要；結石和病理純轉錄，不推定病理、成分或處置。

## Data dictionary / provenance
clinicalTools.drafts：尚未完成輸入。results[modelId]：status、summary、score/riskGroup（適用時）、reasons、inputSnapshot、modelVersion、sourceUrls、checkedOn、computedAt、stale。included：醫師是否加入 Assessment。日期/量測方式另存於輸入快照，病理與風險結果不相互偷偷覆寫。

## 追蹤提醒與適用性
结果卡附官方 follow-up 章節，提醒確認手術/治療日期與完整性；不自动排具体醫囑或推估個人存活機率。RCC 原模型 grade 與現行病理 grade 需醫師確認；UISS 此處只作 N0M0 簡化適用範圍。NMIBC WHO1973 不直接換算；UTUC 弱因子不能单獨替代病理侵襲性。

## 模擬、資料與隱私
使用 deterministic fixtures + Playwright UI smoke。没有下載病人 dataset、沒有新病人資料蒐集或將測試內容傳送第三方臨床服務。純函式測試不代表前瞻臨床驗證；正式 Firestore rules 与現有參考治療文字的全面年度更新尚未核對。
