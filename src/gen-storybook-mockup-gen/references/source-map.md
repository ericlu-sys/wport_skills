# Source Map Index (Existing Increment)

這份檔案是「既有頁面增量」的母版索引。  
目的：避免 agent 只抓 section，不抓整頁。

---

## Usage

每次 existing increment，請先填以下區塊再開始實作：

```md
page_type: existing_increment
source_page_path: <absolute path>
source_components:
  - <absolute path>
in_scope_sections:
  - <section id / block name>
out_of_scope_sections:
  - <section id / block name>
```

---

## Suggested Entries

### Recruitment - Edit Job

- source_page_path: `/Users/Eric/Documents/Github/w101-web/main-web/packages/recruitment/src/views/jobs/edit-job/index.vue`
- source_components:
  - `/Users/Eric/Documents/Github/w101-web/main-web/packages/recruitment/src/components/JobManager/JobDescriptionCard.vue`
  - `/Users/Eric/Documents/Github/w101-web/main-web/packages/recruitment/src/components/JobManager/RecruitmentConditionsCard.vue`
  - `/Users/Eric/Documents/Github/w101-web/main-web/packages/recruitment/src/components/JobManager/ApplicationMethodCard.vue`
- preserve_blocks_minimum:
  - `PageHeader`
  - `header (更新時間 / 提示區)`
  - `body card order`
  - `action buttons`

### Main Site - Job List

- source_page_path: `/Users/Eric/Documents/Github/w101-web/main-site/pages/jobs/index.vue`
- source_components:
  - `/Users/Eric/Documents/Github/w101-web/main-site/components/job/JobCard.vue`
  - `/Users/Eric/Documents/Github/w101-web/main-site/components/SearchFilterCard/index.vue`

### Main Site - Job Detail

- source_page_path: `/Users/Eric/Documents/Github/w101-web/main-site/pages/job/[id].vue`
- source_components:
  - `/Users/Eric/Documents/Github/w101-web/main-site/components/job/JobDetailView.vue`

### User - Resume Job Preference

- source_page_path: `/Users/Eric/Documents/Github/w101-web/main-web/packages/user/src/components/ResumeSections/JobPreference/index.vue`
- source_components:
  - `/Users/Eric/Documents/Github/w101-web/main-web/packages/user/src/components/ResumeSections/JobPreference/preview-mode.vue`

---

## Verification Template

交付前請附：

```md
## Clone Verification
- Referenced files:
- Preserved blocks:
- Delta-only changes:
- Missing blocks (with reason):
```
