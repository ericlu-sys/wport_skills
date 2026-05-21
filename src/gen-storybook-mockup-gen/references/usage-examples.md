# Feature Requirement Generator 使用範例

本文件提供 `feature-requirement-generator` 技能的使用範例，展示不同情境下的使用方式。

## 範例 1：基本使用（無專案位置）

### 使用者輸入

```
我要新增一個「API 金鑰管理」功能，讓使用者可以建立、查看、刪除 API 金鑰。
```

### 技能執行流程

1. **Stage 0: 專案結構探索**

   - 詢問專案位置（使用者未提供）
   - 跳過專案掃描，標記為「基於需求推斷」

2. **Stage 1: 需求收集**

   - 收集基本需求：API 金鑰管理、CRUD 操作

3. **Stage 2: 需求增強與影響分析**

   - 擴展功能：加入金鑰啟用/停用、過期管理、使用統計等
   - 專案影響分析：基於需求推斷可能需要的頁面和 API

4. **Stage 3-6: 風險識別、確認、增強、文件生成**
   - 生成 User Guide 和 PRD
   - PRD 中包含「專案影響分析」章節，標記為「基於需求推斷」

### 輸出

- `api-keys-management-user-guide.md`
- `api-keys-management-prd.md`（包含推斷的專案影響分析）

---

## 範例 2：使用本地路徑

### 使用者輸入

```
我要新增「API 金鑰管理」功能。

專案位置：
- 前端：/Users/username/projects/wport-admin-frontend
- 後端：/Users/username/projects/wport-admin-backend
```

### 技能執行流程

1. **Stage 0: 專案結構探索**

   - 掃描前端專案：識別為 Nuxt.js，掃描 `pages/` 目錄
   - 掃描後端專案：識別為 Express.js，掃描 `routes/` 目錄
   - 建立專案知識庫：
     - 前端：現有頁面列表（如 `/users`, `/settings` 等）
     - 後端：現有 API 列表（如 `GET /api/users`, `POST /api/users` 等）

2. **Stage 1: 需求收集**

   - 收集基本需求

3. **Stage 2: 需求增強與影響分析**

   - 擴展功能
   - **專案影響分析**：
     - 前端：比對現有頁面，發現 `/settings` 頁面可擴展，需要新增 `/settings/api-keys` 子頁面
     - 後端：需要新增 `GET /api/api-keys`, `POST /api/api-keys`, `DELETE /api/api-keys/:id` 等 API
     - 資料庫：需要新增 `api_keys` 資料表

4. **Stage 3-6: 後續流程**
   - 生成包含完整專案影響分析的 PRD

### 輸出

- `api-keys-management-user-guide.md`
- `api-keys-management-prd.md`（包含完整的專案影響分析，標記為「完整掃描」）

### PRD 中的專案影響分析章節範例

```markdown
## 專案影響分析

### 分析基礎

- **專案資訊來源**：完整掃描
- **前端專案位置**：/Users/username/projects/wport-admin-frontend
- **後端專案位置**：/Users/username/projects/wport-admin-backend

### 前端影響分析

#### 頁面分析

| 功能需求     | 現有頁面           | 狀態      | 說明                               |
| ------------ | ------------------ | --------- | ---------------------------------- |
| API 金鑰列表 | /settings          | 🔄 需修改 | 需要在設定頁面新增「API 金鑰」分頁 |
| API 金鑰建立 | /settings/api-keys | ➕ 需新增 | 需要新增 API 金鑰管理子頁面        |

#### 路由分析

**需要新增的路由**：

- `/settings/api-keys` - API 金鑰管理頁面

**需要修改的路由**：

- `/settings` - 新增「API 金鑰」分頁標籤

### 後端影響分析

#### API 分析

| 功能需求          | 現有 API | 狀態      | 說明                       |
| ----------------- | -------- | --------- | -------------------------- |
| 取得 API 金鑰列表 | -        | ➕ 需新增 | `GET /api/api-keys`        |
| 建立 API 金鑰     | -        | ➕ 需新增 | `POST /api/api-keys`       |
| 刪除 API 金鑰     | -        | ➕ 需新增 | `DELETE /api/api-keys/:id` |
```

---

## 範例 3：使用 GitHub URL

### 使用者輸入

```
我要新增「通知中心」功能，讓使用者可以查看和管理系統通知。

專案位置：
- 前端：https://github.com/company/wport-admin-frontend
- 後端：https://github.com/company/wport-admin-backend
```

### 技能執行流程

1. **Stage 0: 專案結構探索**

   - **解析 GitHub URL**：
     - 前端：`github.com/company/wport-admin-frontend`
     - 後端：`github.com/company/wport-admin-backend`
   - **使用 GitHub API 讀取專案結構**：
     - 讀取 `pages/` 目錄（前端）
     - 讀取 `routes/` 目錄（後端）
     - 讀取 `architecture.md`（如果存在）
   - 建立專案知識庫

2. **Stage 1-2: 需求收集與分析**

   - 收集需求
   - 進行專案影響分析

3. **Stage 3-6: 後續流程**
   - 生成文件

### 輸出

- `notification-center-user-guide.md`
- `notification-center-prd.md`（包含基於 GitHub 掃描的專案影響分析）

---

## 範例 4：混合格式（本地 + GitHub）

### 使用者輸入

```
我要新增「資料匯出」功能。

專案位置：
- 前端：/Users/username/projects/wport-admin-frontend（本地）
- 後端：github.com/company/wport-admin-backend（GitHub）
```

### 技能執行流程

1. **Stage 0: 專案結構探索**

   - 掃描本地前端專案
   - 使用 GitHub API 讀取後端專案
   - 整合兩邊的專案資訊

2. **後續流程**
   - 進行跨專案影響分析

---

## 範例 5：部分專案資訊

### 使用者輸入

```
我要新增「報表管理」功能。

專案位置：
- 前端：/Users/username/projects/wport-admin-frontend
（未提供後端位置）
```

### 技能執行流程

1. **Stage 0: 專案結構探索**

   - 掃描前端專案（完整）
   - 後端專案：標記為「未提供」

2. **Stage 2: 專案影響分析**

   - 前端：完整分析
   - 後端：基於需求推斷

3. **輸出**
   - PRD 中標記後端分析為「基於需求推斷」
   - 在「不確定性與建議」中列出需要確認的後端事項

---

## 範例 6：完整對話流程

### 對話範例

**使用者**：

```
我要新增「使用者權限管理」功能，讓管理員可以設定不同角色的權限。
```

**Claude（使用技能）**：

```
我了解您想要新增「使用者權限管理」功能。為了提供更準確的專案影響分析，請問您是否願意提供專案位置？

我可以分析：
- 前端專案位置（本地路徑或 GitHub URL）
- 後端專案位置（本地路徑或 GitHub URL）

如果提供專案位置，我可以：
- 比對現有頁面和功能
- 識別需要新增或修改的部分
- 提供更精確的實作建議

如果暫時無法提供，我也可以基於需求描述進行分析。
```

**使用者**：

```
前端：github.com/company/wport-admin-frontend
後端：github.com/company/wport-admin-backend
```

**Claude（使用技能）**：

```
好的，我將從 GitHub 讀取專案結構進行分析。

[執行 Stage 0: 掃描專案結構]
- 前端：Nuxt.js 專案，已掃描 15 個頁面
- 後端：Express.js 專案，已掃描 23 個 API 端點

[執行 Stage 1-2: 需求分析與影響分析]

根據分析，我發現：

**前端影響**：
- ✅ 現有 `/admin/users` 頁面可以擴展
- ➕ 需要新增 `/admin/users/:id/permissions` 頁面
- 🔄 需要修改 `/admin/roles` 頁面，加入權限設定功能

**後端影響**：
- ➕ 需要新增 `GET /api/permissions` API
- ➕ 需要新增 `PUT /api/users/:id/permissions` API
- 🔄 需要修改 `GET /api/roles` API，加入權限資訊

**資料庫影響**：
- ➕ 需要新增 `permissions` 資料表
- ➕ 需要新增 `role_permissions` 關聯表

[繼續執行 Stage 3-6: 風險識別、文件生成]

已生成以下文件：
- `user-permission-management-user-guide.md`
- `user-permission-management-prd.md`（包含完整的專案影響分析）
```

---

## 範例 7：處理錯誤情況

### 情境：無法存取 GitHub 倉庫

**使用者輸入**：

```
專案位置：
- 前端：github.com/private-org/private-repo
```

**Claude（使用技能）**：

```
[嘗試使用 GitHub API 讀取專案]

⚠️ 無法存取 GitHub 倉庫：github.com/private-org/private-repo

**可能原因**：
1. 倉庫是私有的，需要 GitHub token 認證
2. 網路連線問題
3. 權限不足

**建議解決方案**：
1. 提供 GitHub Personal Access Token（環境變數 `GITHUB_TOKEN`）
2. 或提供本地 clone 的路徑
3. 或暫時跳過專案掃描，基於需求推斷進行分析

**目前處理方式**：
我將基於需求描述進行分析，並在 PRD 中標記為「基於需求推斷」。當您提供認證資訊或本地路徑後，可以重新執行分析以獲得更準確的結果。
```

---

## 範例 8：專案結構無法識別

### 情境：非標準專案結構

**使用者輸入**：

```
專案位置：
- 前端：/Users/username/projects/custom-frontend
```

**Claude（使用技能）**：

```
[掃描專案結構]

⚠️ 無法明確識別前端框架類型

**發現的結構**：
- 存在 `src/` 目錄
- 存在 `package.json`，但無法確定框架類型
- 未找到標準的路由配置檔案

**處理方式**：
1. 嘗試通用掃描：讀取 `src/` 目錄結構
2. 尋找可能的頁面檔案（`.vue`, `.jsx`, `.tsx` 等）
3. 標記為「結構不明確」，基於可讀取的檔案進行部分分析

**分析結果**：
- 識別到可能的頁面檔案：`src/pages/Home.jsx`, `src/pages/About.jsx`
- 推斷路由結構：`/` (Home), `/about` (About)
- 專案影響分析標記為「部分掃描」
```

---

## 使用技巧

### 1. 何時提供專案位置

**建議提供**：

- 需要精確的影響分析時
- 需要避免重複開發時
- 需要了解現有架構時

**可以不提供**：

- 只是初步需求探索
- 專案位置暫時無法取得
- 想要先看推斷的分析結果

### 2. 專案位置格式

**本地路徑**：

```
/path/to/project
./relative/path
../parent/project
```

**GitHub URL**：

```
https://github.com/username/repo
github.com/username/repo
https://github.com/username/repo/tree/main
https://github.com/username/repo/tree/main/packages/frontend
```

### 3. 取得更準確的分析

- 提供完整的專案位置（前端 + 後端）
- 確保專案結構清晰（標準框架結構）
- 提供架構文件（`architecture.md`）可獲得更好的分析
- 提供 API 文件（Swagger/OpenAPI）可獲得更準確的 API 分析

### 4. 處理私有倉庫

- 設定 `GITHUB_TOKEN` 環境變數
- 或使用本地 clone 的路徑
- 或暫時跳過專案掃描

---

## 輸出文件說明

### User Guide

- 使用者導向的文件
- 包含操作流程、使用情境
- 適合業務人員、設計師、QA

### PRD

- 技術規格文件
- 包含 API 設計、資料結構、技術架構
- **新增**：專案影響分析章節
- 適合開發人員、技術架構師

### 專案影響分析報告（可選）

- 獨立的影響分析報告
- 更詳細的實作建議和優先順序
- 適合專案經理、技術主管

---

## 常見問題

### Q: 如果我只想分析專案影響，不想生成完整的需求文件？

**A**: 可以明確說明：

```
我只想要專案影響分析，不需要完整的 PRD 和 User Guide。
```

技能會生成獨立的專案影響分析報告。

### Q: 可以只提供前端或只提供後端位置嗎？

**A**: 可以。技能會：

- 對提供的專案進行完整分析
- 對未提供的專案進行需求推斷
- 在報告中明確標記分析來源

### Q: GitHub URL 支援私有倉庫嗎？

**A**: 支援，但需要：

- 設定 `GITHUB_TOKEN` 環境變數
- 或使用本地 clone 的路徑

### Q: 分析結果會儲存嗎？

**A**: 不會自動儲存。每次執行都是獨立分析。如果需要重複使用，可以：

- 儲存生成的 PRD 文件
- 或提供相同的專案位置重新執行

---

## 進階使用

### 批次分析多個功能

```
我要分析以下功能對專案的影響：
1. API 金鑰管理
2. 通知中心
3. 報表管理

專案位置：
- 前端：github.com/company/wport-admin-frontend
- 後端：github.com/company/wport-admin-backend
```

技能可以：

- 一次掃描專案結構
- 分別分析每個功能的影響
- 生成整合的影響分析報告

### 比較不同實作方案

```
我要新增「使用者認證」功能，有兩種方案：
方案 A：使用 JWT
方案 B：使用 Session

請分析兩種方案對現有專案的影響。
```

技能可以：

- 分析兩種方案的專案影響
- 比較實作複雜度
- 提供建議
