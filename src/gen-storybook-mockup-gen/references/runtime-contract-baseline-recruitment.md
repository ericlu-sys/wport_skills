# Recruitment Runtime Contract Baseline

用於 `existing_increment`、且母版來源為 recruitment package 時的起始基線。  
此檔為 baseline，不代表所有頁面通用；每次任務仍需產出「當次 Runtime Contract 附錄」。

## Alias Baseline

- `@recruitment` -> `W101-Web/main-web/packages/recruitment/src`
- `@user` -> `W101-Web/main-web/packages/user/src`
- `@common-components` -> `W101-Web/main-web/packages/common-components/src`
- `@w101/assets` -> `W101-Web/main-web/packages/assets`
- `@main-site` -> `W101-Web/main-site`
- `@/stores` -> Storybook shim（例：`storybook/src/shims/recruitment-stores.ts`）
- `@/stores/app` -> Storybook shim（例：`storybook/src/shims/recruitment-app-store.ts`）
- `@/api` / `@/api/job` / `@/api/talent` -> recruitment api entry 或 shim
- `^@/` catch-all -> recruitment src

## Providers Baseline

- Pinia
- i18n（至少覆蓋該頁用到的 key）
- Naive UI provider（含 theme overrides）
- PrimeVue provider（若母版元件依賴）

## Auto-import / Macro Parity Baseline

若母版專案依賴 `unplugin-auto-import`，Storybook 需等價補齊，至少覆蓋：

- `vue`: `ref`, `computed`, `watch`, `onMounted`, `h`
- `pinia`: `storeToRefs`

## Data / Shim Baseline

- shim 僅限 data layer（store/api adapter）
- 不可 shim UI 組件本身
- mock data 欄位需對齊真元件常見欄位（例：`disabled`, `totalCount`）

## Isolation Baseline (edit-job family)

- `JobDescriptionCard`
- `RecruitmentConditionsCard`
- `ApplicationMethodCard`
- `JobImportantTipsModal`

## Required QA Baseline

1. `npm run build-storybook`
2. `npm run qa:storybook -- --story <story-id>`
3. 確認 `storybook/.qa/last-report.json` 為 pass
