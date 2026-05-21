# Skill Index（人類快查表）

> 給開發者用的速覽手冊。**AI 不靠這份**：Claude Code / Cursor 啟動時會自動讀每個 `SKILL.md` 的 YAML frontmatter `description:` 來決定何時 invoke skill。
> 此 INDEX 只是讓你（人）能 5 秒內找到「我要叫哪個 skill」。

---

## 🛠️ Executors（5 個）— 對外執行的「手腳」

### `exec-google-cli`
- **觸發詞**：`gcloud`、Google Sheets、ADC、scope/403 errors
- **核心動作**：認證帳號切換、ADC scope 設定、讀寫 Google Sheets
- **代表指令**：
  ```bash
  gcloud auth list
  gcloud auth application-default login --scopes=<SCOPES>
  curl -H "Authorization: Bearer $(gcloud auth print-access-token)" \
       "https://sheets.googleapis.com/v4/spreadsheets/<ID>/values/<RANGE>"
  ```
- **常見排錯**：`ACCESS_TOKEN_SCOPE_INSUFFICIENT` → 重跑 ADC login 加正確 scope

---

### `exec-github-cli`
- **觸發詞**：`gh`、PR、issue、workflow run、release
- **核心動作**：PR 全生命週期、issue 管理、Actions 監控、release 發布
- **代表指令**：
  ```bash
  gh pr create --base <BRANCH> --title "<TITLE>" --body "<BODY>"
  gh pr view <NUM> --comments
  gh run view <RUN_ID> --log
  gh release create <TAG> --notes "<NOTES>"
  ```
- **常見排錯**：401 / scope 不足 → `gh auth refresh -s workflow,repo`

---

### `exec-cloudflare-cli`
- **觸發詞**：Wrangler、Cloudflare Workers、Cloudflare Pages、deploy 錯誤
- **核心動作**：判斷 Workers vs Pages、執行對應 deploy 指令、CI token 設定
- **代表指令**：
  ```bash
  npx wrangler whoami
  npx wrangler pages deploy <DIST_DIR> --project-name <NAME> --branch <BRANCH>
  npx wrangler deploy    # 給 Worker 用，不是 Pages
  ```
- **關鍵判讀**：`wrangler.toml` 看到 `[assets]` 或 Worker 服務 → `wrangler deploy`；Pages 專案 → `wrangler pages deploy`

---

### `exec-cloudinary-cli`
- **觸發詞**：Cloudinary、`cld`、image upload、background removal
- **核心動作**：認證設定、上傳圖片並 AI 去背、下載 transformed 圖
- **代表指令**：
  ```bash
  cld config
  cld uploader upload <IMG> background_removal=cloudinary_ai folder=<F> public_id=<ID>
  curl -o out.png "https://res.cloudinary.com/<CLOUD>/image/upload/e_background_removal/<ID>.png"
  ```
- **前置需求**：帳號需開通 AI Background Removal 功能

---

### `exec-obsidian-cli`
- **觸發詞**：Obsidian、`obsidian://` URI、開啟 vault、scripting note
- **核心動作**：開 vault / note、搜尋、新增 / append note、觸發 workspace 指令
- **代表指令**：
  ```bash
  obsidian open --vault <VAULT> --path "<NOTE>"
  obsidian new --vault <VAULT> --path "<PATH>" --content "<MD>"
  open "obsidian://search?vault=<V>&query=<Q_ENCODED>"
  ```
- **硬需求**：Obsidian 桌面 app 必須**已啟動**；CLI 是 URI 派發器，不能 headless

---

## 🧠 Generators（6 個）— 思考輸出的「大腦」

### `gen-prd`
- **觸發詞**：建立 PRD、新功能規格、Eric 點子收斂
- **Persona**：Senior PM（務實）／ Research-Driven Technical PM（預設）
- **產出**：依 `references/prd_template.md` 的完整 PRD（zh-TW）
- **流程要點**：9-stage 互動式 Q&A，內建商業規則對齊 + Edge case 檢查
- **配對**：完成後送 `gen-prd-checker` 做對抗審查

---

### `gen-prd-checker`
- **觸發詞**：PRD 審查、外部 PRD audit、Stage 8 final gate
- **Persona**：Backend RD ／ Security Engineer ／ SRE（三鏡頭獨立）
- **產出**：分級 issue list（P0 / P1 / P2）+ 整體分數（0–10）+ Verdict（✅ / ⚠️ / ❌）
- **關鍵設計**：三 persona pass 互不污染，避免 self-review blind spot
- **不適用**：Stage 1–4 work-in-progress、UX / 純產品決策

---

### `gen-cto-advisor`
- **觸發詞**：feasibility、architecture、business impact、team capacity、技術決策
- **產出**：CTO 級回覆模板（Recommendation → Technical feasibility → Business impact → Capacity → Risks → Cost → Phased Timeline → Next actions）
- **適用情境**：跨領域權衡（feature 是否做、技術選型、人力配置）

---

### `gen-storybook-mockup-gen`
- **觸發詞**：依 PRD 產出 Storybook mockup、設計稿轉 story
- **產出**：Storybook stories（嚴格模式：existing-page increment 或 new-page skeleton）
- **硬規則**：**Never connect to real APIs**；isolation index 規則參考 `references/isolation-index.md`

---

### `gen-trello-project-setup`
- **觸發詞**：建立 Trello 卡片、專案 setup、PM board 同步
- **產出**：1 個 goal card + 2 個 delivery cards（FE + BE）含 PRD 連結 + Storybook URL
- **Storybook base**：`https://storybook.wport.me/`（**不是** local port 6006）

---

### `gen-wport-code-assistant`
- **觸發詞**：WPORT 後端/前端邏輯、欄位語意、unread/read 行為、cross-repo ownership
- **產出**：經過實際 codebase 驗證的 PRD-ready 結論（不靠記憶）
- **方法**：跨 repo 查證 + 明列 assumptions / gaps

---

## 配對關係（reviewGate）

| 生成 | 必過 reviewer |
|------|--------------|
| `gen-prd` | `gen-prd-checker` |

詳見專案內 `wport.config.json`。

---

## 找不到合適的 skill？

依分類決策樹判斷該不該新增（詳 [`../docs/architecture.md`](../docs/architecture.md)）：

```text
Q1：要對外部系統發指令？  → exec-*
Q2：要產出文件/規格/決策？  → gen-*
Q3：純查詢/純展示？        → 不需要寫成 skill
```
