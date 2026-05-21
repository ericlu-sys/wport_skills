# 可重用組件清單（Component Catalog）

建立 Storybook mockup 時，**必須優先使用**以下既有組件，不要重複造輪子。

> **優先順序**：既有組件 → PrimeVue（v4） → 自建 Mockup 組件

---

## common-components

**路徑**：`/Users/Eric/Documents/Github/W101-Web/main-web/packages/common-components/src/components`
**Import alias**：`@common-components/components/...`

| 組件 | Import 路徑 | 用途 | Props / 備註 |
|------|------------|------|-------------|
| **W101Header** | `W101Header/index.vue` | 網站頂部導覽列，含 Logo、NavItem、UserDropdown、MobileMenu | `ShowMobileCompanyBtn`, `mainCompanyLogo`, `unreadMessageCount`, `point`。整包引入即可。 |
| **W101Footer** | `W101Footer/index.vue` | 網站底部 footer，含社群連結與版權資訊 | 無 props，整包引入。 |
| **W101ResumeList** | `W101ResumeList/index.vue` | 履歷列表容器，含 loading skeleton | `data: ResumeItem[]`, `linkConstants`, `type: 'query'\|'actively'\|'collection'\|'visit'`, `empty`, `pending`。內含 ResumeCard。Slots: `prefix`, `job`, `status`, `other`。 |
| **W101List** | `W101List/index.vue` | 通用列表佈局容器（帶邊框圓角） | Slots: `header`, `default`, `footer`。搭配 `W101ListItem` 使用。 |
| **W101ListItem** | `W101List/W101ListItem.vue` | 列表項目（prefix / content / suffix 佈局） | Slots: `prefix`, `default`, `suffix`。需搭配 `W101List`。 |
| **W101Filter** | `W101Filter/index.vue` | 地點 / 職務多層級篩選下拉選單 | `type: 'address'\|'classify'`, `search: boolean`, `selectionMode: 'single'\|'multiple'`, `maxSelections`。Events: `confirm`。 |
| **Switch** | `Switch/index.vue` | 開關切換（含文字標籤滑動） | `checked`, `disabled`, `offLabel`, `onLabel`。Events: `click`。 |
| **VerifyState** | `VerifyState/index.vue` | 驗證狀態標記（已驗證 ✓ / 未驗證） | `state: boolean`。包裹 input slot 使用。 |
| **W101Toast** | `W101Toast/index.vue` | Toast 通知提示 | `show`, `message`, `type: 'success'\|'info'\|'warning'\|'error'`, `closeTimeout`。 |
| **W101MissionToast** | `W101MissionToast/index.vue` | 任務完成 / G 幣獎勵提示（message 或 dialog 模式） | `show`, `message`, `type: 'message'\|'dialog'`, `missionName`, `missionPoint`, `iconType`, `confirmText`。 |
| **MaintenanceHandler** | `MaintenanceHandler.vue` | 維護模式偵測與頁面切換 | 自動檢測維護狀態，顯示 MaintenancePage 或 slot 內容。 |

---

## recruitment/common

**路徑**：`/Users/Eric/Documents/Github/W101-Web/main-web/packages/recruitment/src/components/common`
**Import alias**：`@recruitment/components/common/...`

### 佈局與容器

| 組件 | Import 路徑 | 用途 | Props / 備註 |
|------|------------|------|-------------|
| **Card** | `Card.vue` | 通用卡片容器（白底、圓角、陰影） | `title`, `buttonText`, `buttonIcon`。Slots: `header`, `header-extra`, `default`。 |
| **PageHeader** | `Header/PageHeader/index.vue` | 頁面標題區塊（標題 + 副標題 + 右側操作區） | `title`, `subTitle`, `noBreadcrumb`。Slots: `header`, `prefix`, `header-extra`。 |
| **Breadcrumb** | `Breadcrumb.vue` | 麵包屑導覽 | 依 `route.meta.breadcrumb` 自動產生。 |
| **BackButton** | `BackButton/index.vue` | 返回按鈕（主題色箭頭 + 文字連結） | `title`, `to`。 |

### 分頁與導覽

| 組件 | Import 路徑 | 用途 | Props / 備註 |
|------|------------|------|-------------|
| **Tabs** | `Tabs/index.vue` | 分頁標籤切換容器 | Events: `tab-change`。Slots: `suffix`, `default`。**需搭配 TabPanel**。 |
| **TabPanel** | `Tabs/TabPanel.vue` | 分頁面板內容 | `tab` (顯示文字), `value` (值)。放在 Tabs 內。 |
| **Pagination** | `Pagination.vue` | 分頁器（封裝 n-pagination） | `pagination: UsePaginationReturn`。 |

### 表單元件

| 組件 | Import 路徑 | 用途 | Props / 備註 |
|------|------------|------|-------------|
| **FormFieldWrapper** | `FormFieldWrapper.vue` | 表單欄位包裝器（label + required 星號 + 錯誤訊息） | `name`, `label`, `required`。使用 **PrimeVue** FormField + Message。 |
| **FormSelect** | `Form/FormSelect.vue` | 自建下拉選擇器 | `placeholder`, `options`, `label`, `disabled`。Events: `selected`。 |
| **FormCheckGroup** | `Form/FormCheckGroup.vue` | Checkbox 群組 | `options`。v-model 支援。 |
| **FormRadioGroup** | `Form/FormRatioGroup.vue` | Radio 群組（可反選取消） | `options`。v-model 支援。 |
| **SearchInput** | `SearchInput.vue` | 搜尋輸入框（含放大鏡 icon） | `placeholder`, `showIcon`, `themeOverrides`。v-model 支援。Events: `change`。 |
| **SearchSelect** | `SearchSelect.vue` | 搜尋下拉選擇 | — |
| **SearchTag** | `SearchTag.vue` | 搜尋標籤 | — |
| **SearchSelectTag** | `SearchSelectTag.vue` | 搜尋選擇標籤組合 | — |
| **DatePicker** | `DatePicker/index.vue` | 日期範圍選擇器（封裝 n-date-picker） | `placeholder`, `propsInputThemeOverrides`。v-model 支援。Events: `confirm`。 |

### 按鈕與操作

| 組件 | Import 路徑 | 用途 | Props / 備註 |
|------|------------|------|-------------|
| **EditButton** | `EditButton.vue` | 編輯按鈕（ghost primary + edit icon） | 使用 Naive UI n-button。 |
| **W101Switch** | `Switch/W101Switch.vue` | 開關切換（recruitment 版本，含 on/off 文字滑動） | `checked`, `disabled`, `offLabel`, `onLabel`。Events: `click`。 |

### 對話框與 Modal

| 組件 | Import 路徑 | 用途 | Props / 備註 |
|------|------------|------|-------------|
| **Dialog** | `Dialog/index.vue` | 通用對話框（封裝 n-modal） | expose: `openDialog(options)`, `StopLoading`。 |
| **ConfirmDeleteDialog** | `Dialog/ConfirmDeleteDialog.vue` | 刪除確認對話框（需輸入 DELETE 確認） | expose: `openDialog(options)`, `StopLoading`。 |
| **ModalAction** | `Modal/ModalAction.vue` | Modal 底部動作列佈局容器 | Slot: `default`（放按鈕）。 |
| **commonModal** | `commonModal.vue` | 通用分類選擇 Modal（職務/地區/證照/技能等三層級勾選） | `type: 'jobSkill'\|'address'\|'tool'\|'classify'\|'certificate'\|'education'`, `single`, `dataCode`, `maxLength`, `isShowMainItem`。Events: `confirm`, `closeModal`。 |

### 資料展示

| 組件 | Import 路徑 | 用途 | Props / 備註 |
|------|------------|------|-------------|
| **W101Statistic** | `Counter/W101Statistic.vue` | 統計數字顯示（label: value / total） | `showIcon`, `label`, `value`, `total`, `textColor`。 |
| **ErrorTip** | `ErrorTip.vue` | 錯誤頁面（403 / 404 / 500 插圖 + 返回首頁按鈕） | `type: '403'\|'404'\|'500'`。 |
| **CollectSideBar** | `CollectSideBar.vue` | 收藏資料夾側邊欄（資料夾列表 + CRUD） | 依賴 API，mockup 中不建議直接使用。 |

### 公司相關

| 組件 | Import 路徑 | 用途 | Props / 備註 |
|------|------------|------|-------------|
| **CompanyHeader** | `Company/CompanyHeader.vue` | 公司資訊 header（logo + 公司名 + 產業類別） | `company: CompanyDetail`（需 import type from `@recruitment/model/company`）。使用 PrimeVue Avatar。Slots: `actions`（右側按鈕區）。Events: `favorite`。 |
| **CompanyDetailView** | `Company/CompanyDetailView.vue` | 公司詳情頁面（header + 基本資訊 + 介紹 + 服務項目 + 福利 + 地圖） | `company: CompanyDetail`, `fileDomain: string`, `type: 'full'\|'list'`（list 模式會顯示展開按鈕）, `pending: boolean`（true 顯示 Skeleton 骨架）。內部使用 CompanyHeader。使用 PrimeVue Card + ScrollPanel + Skeleton。Events: `favorite`, `apply`。**Mockup 中可直接 import 搭配 mock CompanyDetail 資料使用。** |
| **JobOpportunities** | `Company/JobOpportunities.vue` | 公司職缺篩選列 + 職缺列表區塊 | 篩選列使用 PrimeVue Dropdown（工作性質 / 職務類別 / 工作地區三欄），職缺列表使用 `<wb-job-list>` Web Component。**⚠️ Storybook 限制**：`<wb-job-list>` 在 Storybook 中無法渲染，但其**篩選列佈局（3 欄 PrimeVue Select + 灰底背景）應作為參考模式直接複製使用**，職缺卡片清單則用 mock 資料自行呈現。 |

### 履歷相關（整包引入）

| 組件 | Import 路徑 | 用途 | Props / 備註 |
|------|------------|------|-------------|
| **ResumeDetail** | `ResumeDetail/index.vue` | 履歷詳情完整頁面 | 整包引入。內含：PersonalBackground, WorkExperience, Award, Certificates, Languages, JobPreference, Autobiography, PortfolioLinks, Attach, ResumeButtons。 |

### 搜尋列（整包引入）

| 組件 | Import 路徑 | 用途 | Props / 備註 |
|------|------------|------|-------------|
| **SearchBar** | `SearchBar/index.vue` | 複合搜尋列 | 整包引入。內含：SearchKey, SearchArea, SearchEducation, SearchExperience, SearchTool, SearchCertificate, SearchLanguageProficiency, SearchDriverLicense, SearchEmploymentStatus, SearchDesiredConditions 等子組件。 |

---

## 注意事項

1. **部分 recruitment/common 組件內部使用 Naive UI**（如 Dialog, Pagination, DatePicker, EditButton），在 mockup 中直接引用即可，不需要擔心底層實作。
2. **部分組件已使用 PrimeVue**（如 FormFieldWrapper, CompanyHeader, W101ResumeList/ResumeCard），與正式開發一致。
3. **依賴 API 的組件**（如 CollectSideBar, commonModal）在 mockup 中可能無法直接使用，需自建 mockup 版本或用 PrimeVue 替代。
4. **整包引入的組件**（W101Header, W101Footer, ResumeDetail, SearchBar）不需要單獨 import 子組件。
