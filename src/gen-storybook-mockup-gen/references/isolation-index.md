# Isolation Index（給 gen-storybook-mockup-gen 直接查表）

> 目的：提供「來源頁 -> isolation story -> 狀態」的單一入口。  
> 使用方式：mockup-gen 在 `existing_increment` 開工前，先查這張表；若 `status != ready`，先補 isolation。

---

## Status 定義

- `ready`：可直接引用
- `planned`：已規劃未落地
- `in_progress`：正在建置
- `blocked`：有依賴阻塞（需先補 alias/shim/provider）

---

## Index Table

| source_system | source_page_or_component | source_path | isolation_story_id | isolation_story_path | status | notes |
|---|---|---|---|---|---|---|
| recruitment | jobs/index page | `/Users/Eric/Documents/Github/W101-Web/main-web/packages/recruitment/src/views/jobs/index.vue` | `isolation-recruitment-jobs-index` | `storybook/src/stories/isolation/RecruitmentJobsIndex.stories.ts` | planned | PM36 高優先 |
| recruitment | jobs/edit-job page | `/Users/Eric/Documents/Github/W101-Web/main-web/packages/recruitment/src/views/jobs/edit-job/index.vue` | `isolation-recruitment-edit-job-page` | `storybook/src/stories/isolation/RecruitmentEditJobPage.stories.ts` | planned | 與 `JobAiImport` 強關聯 |
| recruitment | jobs/add-job page | `/Users/Eric/Documents/Github/W101-Web/main-web/packages/recruitment/src/views/jobs/add-job/index.vue` | `isolation-recruitment-add-job-page` | `storybook/src/stories/isolation/RecruitmentAddJobPage.stories.ts` | planned | 新增流程母版 |
| recruitment | JobDescriptionCard | `/Users/Eric/Documents/Github/W101-Web/main-web/packages/recruitment/src/components/JobManager/JobDescriptionCard.vue` | `isolation-job-description-card` | `storybook/src/stories/isolation/JobDescriptionCard.stories.ts` | ready | 已存在 |
| recruitment | RecruitmentConditionsCard | `/Users/Eric/Documents/Github/W101-Web/main-web/packages/recruitment/src/components/JobManager/RecruitmentConditionsCard.vue` | `isolation-recruitment-conditions-card` | `storybook/src/stories/isolation/RecruitmentConditionsCard.stories.ts` | ready | 已存在 |
| main-site | jobs index page | `/Users/Eric/Documents/Github/W101-Web/main-site/pages/jobs/index.vue` | `isolation-main-site-jobs-index` | `storybook/src/stories/isolation/MainSiteJobsIndexPage.stories.ts` | planned | main-site 核心頁 |
| main-site | job detail page | `/Users/Eric/Documents/Github/W101-Web/main-site/pages/job/[id].vue` | `isolation-main-site-job-detail` | `storybook/src/stories/isolation/MainSiteJobDetailPage.stories.ts` | planned | SEO/內容頁高關聯 |
| main-site | SearchFilterCard | `/Users/Eric/Documents/Github/W101-Web/main-site/components/SearchFilterCard/index.vue` | `isolation-main-site-search-filter-card` | `storybook/src/stories/isolation/MainSiteSearchFilterCard.stories.ts` | planned | 列表與搜尋共用 |
| main-site | JobCard | `/Users/Eric/Documents/Github/W101-Web/main-site/components/job/JobCard.vue` | `isolation-main-site-job-card` | `storybook/src/stories/isolation/MainSiteJobCard.stories.ts` | planned | 高複用展示元件 |
| user | my-resume index page | `/Users/Eric/Documents/Github/W101-Web/main-web/packages/user/src/views/my-resume/Index.vue` | `isolation-my-resume-index` | `storybook/src/stories/isolation/MyResumeIndex.stories.ts` | ready | 已存在 |
| user | add-resume-modal | `/Users/Eric/Documents/Github/W101-Web/main-web/packages/user/src/views/my-resume/component/add-resume-modal.vue` | `isolation-add-resume-modal` | `storybook/src/stories/isolation/AddResumeModal.stories.ts` | ready | 已存在 |
| user | edit-resume sections | `/Users/Eric/Documents/Github/W101-Web/main-web/packages/user/src/components/ResumeSections/index.vue` | `isolation-edit-resume-sections` | `storybook/src/stories/isolation/EditResumeSections.stories.ts` | ready | 已存在 |

---

## `gen-storybook-mockup-gen` 引用規則（建議）

1. 讀 `source-map.md` 找到 `source_page_path`。
2. 用該 path 比對本 index：
   - `status=ready` -> 可直接引用 isolation + 進入 full-page baseline。
   - `status=planned/in_progress/blocked` -> 先建立 isolation story，不可直接宣告 existing_increment 完成。
3. 交付回報時附上：
   - `isolation_story_id`
   - `status` 變更（例如 planned -> ready）
   - `runtime_contract` 是否新增 alias/shim

---

## 維護規則

- 新增任何 existing_increment 母版前，先加一列到本表。
- 每完成一個 isolation story，必更新 `status` 與 `notes`。
- 若 story id / path 改名，必同步更新本表，避免 mockup-gen 指向舊路徑。
