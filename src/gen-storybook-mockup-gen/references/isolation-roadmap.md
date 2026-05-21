# Isolation 長期規劃表（main-site / main-web job）

> 目的：建立「哪些母版頁應優先搬進 `storybook/src/stories/isolation/`」的長期路線圖，讓 mockup 開發先做 runtime 風險拆解，再做 full-page existing increment。

---

## 為什麼要做這份表

- 目前 `isolation` 只覆蓋部分 user/recruitment 元件，對 `main-site` 與 `jobs` 全頁流程仍有空洞。
- 每次進新 PRD 才補 alias/shim，容易重複踩坑（`@/hooks/*`, `@/stores/*`, `@/api/*`, icon/public asset）。
- 把高頻母版頁先拆成 isolation story，可以把錯誤提前到「元件層」就發現。

---

## 規劃範圍（你提到的 main-site / main-job）

- **main-site**：`/Users/Eric/Documents/Github/W101-Web/main-site/**`
- **main-job（對應 recruitment jobs 模組）**：`/Users/Eric/Documents/Github/W101-Web/main-web/packages/recruitment/src/views/jobs/**`

---

## 分期規劃（Phase）

| Phase | 目標 | 來源系統 | 交付重點 | 完成定義（DoD） |
|---|---|---|---|---|
| P0 | 先補最常被 mockup 引用的招聘核心 | recruitment jobs | `jobs/index`、`jobs/edit-job` 相關首層元件 isolation 全綠 | build + qa story pass |
| P1 | 補 main-site 求職主路徑 | main-site | 職缺列表、職缺詳情、搜尋篩選、卡片元件 | 可支援 SEO/首頁/活動相關 mockup |
| P2 | 補跨頁共用區塊 | main-site + recruitment | Header/Footer、Filter、Pagination、JobCard 互動狀態 | 減少重複實作 mockup 元件 |
| P3 | 補邊界流程 | recruitment | add-job、preview-job、group-setting | 讓職缺管理 PRD 可全鏈路演示 |

---

## 建議優先搬入 Isolation 的模組清單

| 優先級 | 來源頁 / 元件 | Source Path | 建議 isolation story | 理由 |
|---|---|---|---|---|
| High | 管理職缺頁（列表） | `/Users/Eric/Documents/Github/W101-Web/main-web/packages/recruitment/src/views/jobs/index.vue` | `isolation/RecruitmentJobsIndex.stories.ts` | PM36/職缺類需求高頻入口 |
| High | 編輯職缺頁 | `/Users/Eric/Documents/Github/W101-Web/main-web/packages/recruitment/src/views/jobs/edit-job/index.vue` | `isolation/RecruitmentEditJobPage.stories.ts` | existing_increment 母版核心 |
| High | 新增職缺頁 | `/Users/Eric/Documents/Github/W101-Web/main-web/packages/recruitment/src/views/jobs/add-job/index.vue` | `isolation/RecruitmentAddJobPage.stories.ts` | AI 匯入/手動新增共用 |
| High | 職缺卡（列表） | `/Users/Eric/Documents/Github/W101-Web/main-site/components/job/JobCard.vue` | `isolation/MainSiteJobCard.stories.ts` | main-site 多數頁共用 |
| High | 職缺搜尋篩選 | `/Users/Eric/Documents/Github/W101-Web/main-site/components/SearchFilterCard/index.vue` | `isolation/MainSiteSearchFilterCard.stories.ts` | SEO/列表頁都會碰到 |
| High | 職缺列表頁 | `/Users/Eric/Documents/Github/W101-Web/main-site/pages/jobs/index.vue` | `isolation/MainSiteJobsIndexPage.stories.ts` | main-site 核心母版頁 |
| Medium | 職缺詳情頁 | `/Users/Eric/Documents/Github/W101-Web/main-site/pages/job/[id].vue` | `isolation/MainSiteJobDetailPage.stories.ts` | 多 PRD 會要求詳情增量 |
| Medium | 預覽職缺頁 | `/Users/Eric/Documents/Github/W101-Web/main-web/packages/recruitment/src/views/jobs/preview-job/index.vue` | `isolation/RecruitmentPreviewJobPage.stories.ts` | 發布前檢查流程 |
| Medium | 檢視職缺頁 | `/Users/Eric/Documents/Github/W101-Web/main-web/packages/recruitment/src/views/jobs/view-job/index.vue` | `isolation/RecruitmentViewJobPage.stories.ts` | 與 main-site detail 對照 |
| Low | 職缺分組管理 | `/Users/Eric/Documents/Github/W101-Web/main-web/packages/recruitment/src/views/jobs/group-setting/index.vue` | `isolation/RecruitmentJobGroupSettingPage.stories.ts` | 進階後台流程 |

---

## 建議每個 isolation story 的固定欄位（避免規格飄移）

1. `source_page_path`
2. `source_components`
3. `runtime_contract`（alias/provider/shim）
4. `mock_data_contract`（最小必要欄位）
5. `known_gaps`（若無法 1:1 的原因）

---

## 執行節奏建議（下班前可交接）

- 週節奏：每週固定清 2-3 個 high priority isolation story。
- 新 PRD 規則：若目標母版頁不在 isolation index，先補 isolation 再做 mockup。
- QA gate：每次至少跑 `build-storybook` + 目標 story 的 `qa:storybook -- --story <id>`。

---

## 我的建議：需要 index 表嗎？

**需要，且建議做兩層 index：**

1. **Roadmap index（本文件）**：看優先順序與分期。
2. **Execution index（下一份文件）**：讓 `gen-storybook-mockup-gen` 可直接查「這個來源頁該用哪個 isolation story + 現在狀態」。

這樣才同時滿足「長期規劃」和「生成器可直接 reference」。
