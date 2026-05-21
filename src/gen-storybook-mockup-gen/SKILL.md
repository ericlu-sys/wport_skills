---
name: gen-storybook-mockup-gen
description: Generate Storybook mockups from a PRD. Use strict mode selection: existing-page increment (full-page clone) or new-page (skeleton allowed). Never connect to real APIs.
---

# Storybook Mockup Generator

## Goal

從 PRD 產出 Storybook mockup，且依頁面型態採用正確策略：
- **既有頁面增量**：整頁母版 clone（full-page）。
- **全新頁面**：可做骨架版（skeleton）。

---

## Core Rules (Always Apply)

1. **No real API**：只能用 local mock data（inline 或 `storybook/src/mock/`）。
2. **Storybook-only changes**：只允許改 `storybook/`，不可動 production code。
3. **Component priority**：先讀 `references/component-catalog.md`，順序為  
   既有組件 -> PrimeVue(v4) -> 自建 `Mockup*` 元件。
4. **Language**：所有 UI 文案用繁體中文。
5. **Theme**：主題色使用 `#5BBBC5`，不得回退舊色 `#56C7BB`。
6. **State control**：情境切換用 Storybook Controls，不在畫面內做測試控制台。
7. **Template safety**：根模板禁用裸 `<template>` 作容器。
8. **Business check**：生成前讀 `business-rules.md`，避免衝突。
9. **Master page first (hard gate)**：只要有母版（existing_increment），必須先 import 母版真元件達成 1:1 基線，未達成前不得進入 delta 實作。
10. **先母版後 PRD 細節（新增，強制）**：`existing_increment` 一律先完成「母版可渲染 + 視覺一致」再做 PRD 增量；不得邊 clone 邊發散實作需求。
11. **先確認頁面清單再開工（新增，強制）**：收到 PRD 後，先用 1 個區塊列出「本次要做哪些頁面（baseline/delta）」給使用者確認；未確認前不得開始寫 story。

---

## Mode Selection (Mandatory First Step)

開始前必須先判定：
- `page_type: existing_increment | new_page`

若未明確，先問使用者，禁止直接實作。

---

## Mockup Minimal Question Mode（新增）

當使用者目標是「先做可展示的 mockup」時，必須啟用最小提問模式，避免被後端/資料庫細節阻塞。

> **優先級說明（強制）**：此模式只影響「要問幾個問題」，**不會放寬** `existing_increment` 的 1:1 母版要求。  
> 只要 `page_type=existing_increment`，仍必須遵守 Mode A 的 full-page clone + delta-only。

### 原則

1. **只問會改變畫面的問題**：優先確認 UI 結構與互動分支。
2. **非畫面必要資訊採預設**：先做 mockup，再補工程細節。
3. **提問上限**：最多 4 題；若 PRD 已明確，則 0-2 題即可開工。
4. **禁止 over-questioning**：不得詢問 API path、資料表、token 成本、監控口徑等非 mockup 必要資訊。
5. **先問頁面範圍，不先問工程細節（新增）**：第一輪只確認「要做哪些頁面、每頁是 baseline 還是 delta」。

### 第一輪固定輸出（新增，必做）

在任何實作前，先輸出並等待使用者確認：

```md
## Page Scope Confirmation
- page_1: <page name / route> | type: existing_increment|new_page | deliverables: baseline + delta|skeleton
- page_2: ...
```

只有在使用者確認此清單後，才可進下一步（Source of Truth 與實作）。

### 必問 4 題（僅在 PRD 缺漏時提問）

1. **入口位置**：按鈕放在哪個區塊（含文案）。
2. **流程容器**：使用 modal / drawer / 獨立頁。
3. **必要狀態**：本次一定要展示哪些狀態（例如 parsing/failed/success/expired）。
4. **成功導流**：成功後停留預覽、跳轉編輯頁，或返回列表。

### 可直接使用預設（不需先問）

- API endpoint 與 method 細節
- 後端 schema validator 實作細節
- 資料表設計與排程策略
- LLM provider 成本最佳化策略
- KPI / dashboard 指標口徑

### 產出前檢查（Mockup Gate）

在開始寫 story 前，先輸出：

```md
## Mockup Readiness
- UI-required answers: <已確認項目>
- Using defaults: <本次採用的預設>
- Deferred engineering decisions: <延後決策清單>
```

若 `UI-required answers` 已完整，必須直接開始實作，不得繼續追問工程細節。

---

## Mode A: Existing Page Increment (Full-page Clone Required)

### Required Input Contract

當 `page_type=existing_increment`，必須先取得以下資料再開始：

```md
## Source of Truth
- source_page_path: <absolute path>
- source_components:
  - <absolute path>
  - <absolute path>
- in_scope_sections:
  - <section id / block name>
- out_of_scope_sections:
  - <section id / block name>
```

### Hard Requirements

1. **Refer strategy first (hard gate)**：既有頁面一律先判定 `refer` 策略，禁止先手刻 clone 再回頭改 refer。
2. **Runtime contract before page render**：先補齊 Storybook runtime（alias map、providers、mock data contract）才可開始渲染頁面。
3. **Isolation-first verification**：先讓單元件可渲染，再進整頁組裝；禁止直接一次掛整頁救火。
4. **Single source page first**：先鎖定單一母版 page（single source of truth）。
5. **Full-page clone**：必須保留整頁主要區塊順序與容器層級；不可只做 section。
6. **Delta-only changes**：僅實作需求變更點（例如新增幣別欄位）。
7. **No cross-source composition**：同一 story/page 禁止混用多產品來源拼接。
8. **Reference reading required**：先讀母版頁 + 關鍵子元件原始碼，才可開始寫。
9. **Composition-only at full page stage**：整頁組裝階段只能做 composition，不改真元件；delta 只能落在 wrapper/slot。
10. **No internal PRD wording in UI**：畫面文案禁止出現內部字樣（例如 `PM_34`）。
11. **No fallback overlay UI before baseline**：在母版 1:1 基線完成前，禁止用「外掛整層 UI / 自訂面板 / 同構重畫」當替代交付。
12. **Baseline before delta (hard order)**：流程必須是 `先 1:1 母版 -> 再依 PRD 做 delta`，不可顛倒。

### Runtime Prerequisites Checklist (Must Pass Before Rendering)

- **Alias alignment**：Storybook alias 必須對齊來源專案（含 `@`/store/composable/constants 等實際 import 路徑）。
- **Providers ready**：至少確認 `pinia`、`i18n`、Naive/PrimeVue provider/theme 已在 `.storybook/preview.*` 正確掛載。
- **Auto-import / macro parity**：母版若依賴 `unplugin-auto-import` 等（例如未寫 import 的 `ref`、`storeToRefs`、`computed`），Storybook 必須等價補齊，否則真元件會 runtime 炸。
- **i18n key coverage**：直接 import 母版的完整 locale JSON（例 `@w101/i18n-locales/zh-TW.json`），不手刻子集；缺 key 或 `$t` 非函式視為 contract 未過。
- **Data contract ready**：mock data shape 要符合真元件預期欄位與型別（含 `disabled`、`totalCount` 等常見欄位）。store menu 選項（如 `jobFeatures`、`degrees`）必須有至少一筆 `{ code, name }` mock 資料，空陣列不算通過。
- **Shim boundary**：shim 只允許隔離 data layer（store/API adapter），禁止 shim UI 組件本身。
- **CSS runtime profile alignment（新增）**：必須先判定母版屬於哪個產品 CSS profile（至少 `user` / `recruitment`），並載入對應的全域樣式入口；禁止混用或沿用錯誤 profile 造成版型漂移。
- **Tailwind content path parity（新增）**：若母版大量使用 utility class，必須檢查 Storybook `tailwind.config.*` 的 `content` 路徑是否實際指向來源專案（例如 `../../W101-Web/main-web/packages/...`）；路徑錯誤會導致 utility 未生成，版型雖可渲染但非 1:1。
- **Icon asset parity（新增）**：若母版依賴 HTML head 注入的 icon 資源（如 Font Awesome CDN），Storybook 必須等價注入（preview head link 或等價 import），否則「預覽/複製/刪除/燈泡/編輯」等 icon 會遺失。
- **SVG sprite parity（新增）**：若母版使用 `SvgIcon` + `vite-plugin-svg-icons`（`virtual:svg-icons-register` + `symbolId`），Storybook 也必須在 `.storybook/main.ts` 註冊同一 plugin，且在 `preview.ts` import `virtual:svg-icons-register`；只補元件不補 sprite 註冊會導致「27歲/電話/定位」等 icon 缺失。
- **Mock visual data parity（新增）**：Baseline mock 不可用空白關鍵視覺欄位（例如 `photo_url: ''`、富文字內容 `null`）；需提供可顯示的 avatar URL 與中文樣本文字，避免誤判為「亂碼/資產缺失」。
- **Rich text runtime parity（新增）**：若母版含 PrimeVue Editor / Quill（例如「在校經歷」），Storybook 必須顯式載入 `quill/dist/quill.snow.css`，並檢查 toolbar icon（粗體/斜體/列表/連結）與 editor 容器邊框是否存在；缺少此樣式會出現 icon 亂碼或工具列錯位。
- **Form border parity（新增）**：`existing_increment` 驗收必查表單邊框（Input/Select/Editor）是否與母版一致；若被全域 reset 或主題層覆寫，需在 story 層補回對應 border token（通常 `#D7D7D7`）後才可宣稱 1:1。

### Dual CSS Profile Contract（新增，強制）

專案同時存在 `user` 與 `recruitment` 兩套樣式系統時，必須遵守：

1. **Profile declare first**：在 `Runtime Contract 附錄` 明確寫出本次使用 `css_profile: user | recruitment`。
2. **Profile-specific imports**：Story 必須載入對應來源套件的 CSS 入口（例如 user 的 `main.css` / `styles/index.scss`）；`preview.ts` 禁止預設載入單一產品線 CSS，避免跨 profile 汙染。
3. **No cross-profile fallback**：若母版是 user，不得以 recruitment CSS 當 baseline；反之亦然。
4. **Visual sanity checklist required**：至少核對按鈕尺寸、卡片間距、主要字級、關鍵操作區塊（如「新增履歷 / 預覽 / 複製 / 刪除」）是否與母版一致。
5. **Mismatch handling**：若 profile 未對齊導致視覺差異，狀態必須回報 `BLOCKED`，不得宣稱 1:1 已完成。

### Source System Mapping（新增，強制）

判定母版來源時，必須先辨識來源系統並套用對應 runtime：

- `W101-Web/main-web/packages/user/**` -> `css_profile: user`
- `W101-Web/main-web/packages/recruitment/**` -> `css_profile: recruitment`
- `W101-Web/main-site/**` -> `css_profile: main-site`（獨立處理，不可沿用 main-web 預設）

若來源是 `main-site`，不得沿用 `main-web` 的 alias/provider/css 設定；必須獨立列出 contract 與 isolation 範圍。

### Shared Infrastructure Layer（每個來源套件一次性建置）

每當首次 import 一個來源套件（如 `recruitment`）的真元件進 Storybook，必須先建好該套件的共用基礎，後續同套件的新頁面可直接沿用。基礎包含：

| 層 | 做法 | 檔案位置 |
|---|------|---------|
| **Alias** | 在 `.storybook/main.ts` 的 `viteFinal` 中 `resolve.alias` 設定：storybook local → store shim → hooks shim → API shim → model → assets → catch-all `@/` | `.storybook/main.ts` |
| **Store shim** | 為來源套件所有 store（`useJobStore`、`useGlobalStore` 等）建純 mock 版，內含 menu 選項 mock 資料、getter 回空陣列。**不 import 真實 store**（避免拉進 API 依賴鏈）| `src/shims/recruitment-stores.ts` |
| **API shim** | 匯出 default Proxy 物件，所有方法回 `Promise.resolve({ data: { data: [] } })`；阻斷真實網路呼叫 | `src/shims/recruitment-api.ts` |
| **Hooks shim** | 複製母版 hooks 的 signature 但去掉外部依賴（如 `useBoolean` 直接用 `ref` 實作）| `src/shims/recruitment-hooks.ts` |
| **Auto-import** | `unplugin-auto-import/vite` 設 `imports: ['vue', 'pinia']`、`include` regex 涵蓋外部 `.vue`/`.ts` | `.storybook/main.ts` plugins |
| **i18n** | 直接 import 母版完整 locale JSON，不手刻子集 | `.storybook/preview.ts` |
| **全域元件** | 母版透過 `unplugin-vue-components` 自動註冊的元件（如 `Card`、`PageHeader`、`JobImportantTipsModal`），在 `preview.ts` 的 `setup()` 手動 `app.component(...)` | `.storybook/preview.ts` |
| **pug / scss** | 母版元件若用 `<template lang="pug">` 或 `<style lang="scss">`，需安裝對應 devDependency（`pug`、`sass-embedded`）| `package.json` |

**重要**：不同來源套件（如 recruitment vs user）各自需要獨立的 shim 層；做過 recruitment 不代表 user 也能跑。

### Skill 寫什麼 vs 每次沿用母版要產什麼

- **寫在 skill 裡（固定、可重複）**：流程順序、紅線、驗證方式、shim 邊界、isolation-first、禁止捏造母版沒有的 UI 文案或區塊語意。
- **不要寫死在 skill 裡（會過期）**：某一頁的完整 alias 表、全站 i18n key 清單、某包專屬 store 欄位表。
- **每次 `existing_increment` 必須另產一份「Runtime Contract 附錄」**（可貼在 PR 描述或 story 的 `parameters.docs`）：  
  本次實際用到的 **alias 增補**、**providers**、**css profile 與樣式入口**、**i18n 新增 key**、**shim / mock data**、**isolation story 清單**。  
  下一張母版頁若多引了新的 `@/...`，附錄就要更新，**不可假設做過一次 recruitment 就全站免疫**。

### 母版一致：要對齊的是哪一層

1. **流程與區塊順序**：與母版同一資訊架構與主要區塊順序（可先做低保真占位，但不可永久當成已對齊母版）。
2. **文案與合規**：入口字、標題、注意事項等須與產品 i18n 或母版一致；禁止自創「看起來像」的合規文案。
3. **真元件 + 行為**：掛母版子元件時，DOM、驗證、互動與母版一致；達成前須滿足本節 Runtime checklist 與 isolation。

### Existing Page Execution Protocol (Mandatory Order)

1. **Shared Infrastructure ready**：確認該來源套件的基礎層（上表）已建好。若是首次引用，先完成基礎層再繼續。
2. **Import baseline page first (新增必做)**：直接 import 母版頁真元件，建立 `Baseline` story（無 delta）。若 Baseline 無法 render，停止並修 runtime，不得跳過。
3. **Source component inventory first（新增必做）**：讀完母版頁後，先列出「母版直接 import + 首層區塊元件」清單（至少包含容器、關鍵互動、關鍵表單/對話框）。未產生清單不得進入 isolation。  
4. **Isolation stories first（強化）**：對上一步清單中的每個元件，在 `stories/isolation/` 建立對應 `.stories.ts`（例：`stories/isolation/JobDescriptionCard.stories.ts`）。每個 story 至少一個 Default variant 可渲染、console 零 error。  
   - Isolation stories 是技術煙霧測試，不是產品交付物；永久保留以偵測 regression。
   - 日後新增母版頁時，共用子元件若已有 isolation story 且仍綠燈，可跳過。
5. **Full-page compose second**：所有關鍵元件 isolation 綠燈後，才可進整頁 composition。
6. **Apply delta last**：僅在 Baseline 可用且視覺對齊後，才可加 PRD delta。
7. **Visual parity check last**：最後才做整頁視覺對齊與微調，不可顛倒流程。

### Regression Postmortem Rules（新增，避免重犯）

若曾出現「看起來接近但顏色/按鈕狀態錯誤」的案例，後續任務必須先檢查以下高風險點：

1. **Provider parity first**：`user` 母版若透過 `n-config-provider` 套 theme overrides，Storybook `cssProfile=user` 必須等價套用（不可只設 CSS 變數）。
2. **Theme source parity**：優先直接 import 母版 theme source（例如 `@user/stores/app/theme.json`），避免手刻覆寫造成 hover/pressed/outline 偏差。
3. **UI shim 禁止替代真元件**：`ConfirmDialog` 這類關鍵互動元件不得用 mock 替代母版實作；shim 僅限 store/API/hook data layer。
4. **Icon load parity**：icon 必須使用穩定且可重現的載入方式（優先本地 import），避免 CDN 漏載造成「圖示缺失 + 版型誤判」。
5. **Class-to-class check before done**：至少針對「新增履歷」與「刪除確認」做母版 class/狀態對照（default/hover/pressed/disabled）後，才可回報完成。
6. **Position + size parity must be measured（新增）**：對「同列新增按鈕」類需求，禁止只憑肉眼或單一 viewport 宣稱完成；必須量測 bounding box（x/y/width/height）確認 `dy=0`、`height diff=0`，並驗證不超出可視區。
7. **Responsive placement fallback（新增）**：若按鈕插在母版右側，必須檢查窄視窗是否 overflow；空間不足時須同列 fallback 到左側或下一列，且不得破壞母版主按鈕位置。
8. **No self-invented visual language（新增）**：任何 modal/card/button 的視覺不得「自行設計一套」；必須先引用母版對應元件（例如 `add-resume-modal.vue`）作為 style source，至少對齊 `radius/border/shadow/padding/button tokens`，未完成對齊不得回報完成。

### Isolation Coverage Contract（新增，強制）

當 `page_type=existing_increment` 時，交付內容必須包含：

1. `## Source Component Inventory`（母版相關元件清單）
2. `## Isolation Story List`（實際建立或沿用的 isolation story id / path）
3. `## Isolation Gaps`（若有未覆蓋元件，必填原因與風險）

若缺少以上任一區塊，視為未完成 isolation coverage，不可宣稱完成 existing_increment。

#### Edit Resume 最低 isolation 清單（新增，強制）

若母版頁包含 `my-resume/edit-resume`，至少必須有下列 isolation stories（可同檔多個 export）：

1. `Header`
2. `PersonalInformation`
3. `Education`
4. `WorkExperience`
5. `JobPreference`
6. `ProfessionalSkill`
7. `Editor`（富文字編輯器，必須獨立驗證 Quill toolbar icon/邊框/字級）

未覆蓋時不得宣稱「已把該放 isolation 的元件補齊」。

### Package Bootstrap Gate（新增，必做）

當母版來源 package 與既有可運行 package 不同（例如：已打通 recruitment，但本次是 user）時：

1. 必須先新增該 package 專屬 shim（stores / api / hooks / 必要元件替身）。
2. 必須補齊該 package 專屬 alias 與 provider 依賴。
3. 在未通過上述 gate 前，**禁止**使用手刻同構頁面作為交付替代。

### Red Flags (Stop and Fix Isolation First)

若出現以下任一錯誤，**禁止繼續堆整頁**，必須退回 isolation 修復：

- `defineStore is not defined` → auto-import 未覆蓋外部 .vue/.ts
- `ref is not defined` / `storeToRefs is not defined` → 同上，`unplugin-auto-import` 需加 `include` regex
- `Cannot read properties of undefined (reading 'disabled')` → store menu 選項為空陣列或欄位結構不符
- `Cannot read properties of undefined (reading 'totalCount')` → 同上
- `Failed to fetch dynamically imported module` → alias 遺漏或路徑不正確
- `$t is not a function` → i18n 未正確掛載或 globalProperties 缺 `$t`
- `Failed to resolve import "@/..."` → `.storybook/main.ts` alias 遺漏；常見於首次引入新來源套件
- QA script 報 404 / `net::ERR_ABORTED` → 確認 QA 是否自帶 static server（不可依賴外部 dev server）

### Definition of Done (Must Output)

交付前必須輸出：

```md
## Clone Verification
- Referenced files:
- Baseline story id:
- Preserved blocks:
- Delta-only changes:
- Missing blocks (with reason):
```

若無法提供此區塊，不可宣稱「已對齊母版」。

且必須一併輸出：

```md
## Source Component Inventory
- <component path / symbol>

## Isolation Story List
- <story id / file path>

## Isolation Gaps
- <component>: <reason / risk / follow-up>
```

### Existing Increment Fail-Fast Contract（新增，強制）

若任一條件未達成，最終回覆必須標記為 `BLOCKED`，不得以「可展示 mockup」或「暫時版本」宣稱完成：

1. 無法提供 `Baseline` story，且 Baseline 未直接 import 母版真元件。
2. 無法提供 Baseline + Delta 各自的 QA 成功證據。
3. 無法列出本次 runtime contract 附錄（alias/provider/shim/isolation）。

### Forbidden Fallbacks（新增，強制）

在 `existing_increment` 中，以下做法一律視為違規交付：

- inline demo / 手刻同構頁取代母版真元件
- 以 overlay 或外掛面板假裝 baseline 已完成
- 僅更新 docs 宣稱 1:1，但實際未 import 母版

### QA Gate（強制）

完成 `existing_increment` 必須同時通過：

1. `build-storybook` 成功
2. `qa:storybook -- --story <baseline-story-id>` 成功
3. `qa:storybook -- --story <delta-story-id>` 成功

任一失敗即視為未完成，不得回報「已完成」。

### Required Verification Commands

每次調整都必跑：

1. `npm run build-storybook`（或專案約定的 storybook build 指令）
2. `npm run qa:storybook -- --story <story-id>`（建議使用專案 QA script 統一檢查）
3. **Runtime 驗證**：開啟對應 story（建議 `iframe.html?...&viewMode=story`），確認 **console error / pageerror / Storybook 錯誤 overlay 皆無**；僅 build 通過不算完成。

### QA Automation (Recommended)

- **QA script 必須自帶 server**：`scripts/qa-storybook.mjs` 在 build 完成後用 `node:http` 起一個隨機 port 的靜態 server serve `storybook-static/`，Playwright headless 檢查後自動關閉。**絕不依賴外部 dev server**（dev server 可能是舊 session、alias 過期）。
- 建議啟用 project hook：`.cursor/hooks.json` -> `stop` event -> `.cursor/hooks/storybook-qa-gate.sh`。
- Hook 失敗時應回傳 follow-up，要求先修 runtime/build error，再允許結束回合。
- 建議將預設檢查 story id 透過環境變數 `STORYBOOK_QA_STORY_ID` 覆蓋。
- QA 檢查點：`console.error`、`pageerror`、`requestfailed`、Storybook error overlay text。

---

## Mode B: New Page (Skeleton Allowed)

### Allowed Behavior

1. 可做 30% 骨架版（版面 + 主要流程 + mock data）。
2. 仍需遵守 Core Rules，不可接 API。
3. 若 PRD 明確要求高還原度（95%），則提升到高保真實作。

### Output Expectation

- 以可展示流程為主，不強制 full-page clone。
- 需明確標註「骨架版」或「高還原版」。

---

## File Organization

所有檔案只能在 `storybook/`：

```text
storybook/src/mock/                    # 共用 mock data
storybook/src/mockupcomponents/        # Mockup 專用 UI 元件
storybook/src/mockups/                 # 頁面級 mockup demo 元件
storybook/src/shims/                   # Store / API / hooks shim（per source package）
storybook/src/stories/mockups/         # 產品交付的 story 檔
storybook/src/stories/isolation/       # 子元件煙霧測試 story（技術用，非交付）
storybook/scripts/                     # QA / automation 腳本
storybook/.storybook/main.ts           # Vite alias + auto-import + plugins
storybook/.storybook/preview.ts        # Providers + i18n + 全域元件註冊
```

---

## Update Mode

修改既有 mockup 時，預設走增量更新：
- 只改必要檔案，避免整包重建。
- 保持穩定 story id / title / export 名稱（除非需求要求）。
- 回報「修改檔案清單 + 變更點 + 是否影響既有 URL」。

---

## References

- `references/component-catalog.md`（必讀）
- `references/source-map.md`（既有頁母版索引）
- `references/isolation-roadmap.md`（isolation 長期規劃）
- `references/isolation-index.md`（來源頁 -> isolation story 查表）
- `references/runtime-contract-baseline-recruitment.md`（recruitment baseline）
- `/Users/Eric/Documents/Github/prd/design/README.md`
- `/Users/Eric/Documents/Github/prd/business-rules.md`
- `https://storybook.wport.me/`
