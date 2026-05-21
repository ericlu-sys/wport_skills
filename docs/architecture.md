# Skill Architecture

> 本文件定義 wport 體系下「Skill」的兩大分類、撰寫規範與決策邏輯。
> 所有新 skill 提交前必須對照此文件。

---

## 目標目錄結構

Skill 的 **src of truth** 在本 repo 的 `src/`；各專案（如 `prd/`）的 `.claude/` 用 symlink 連回來。

```text
wport_skills/                              # 此 repo（src of truth）
├── src/
│   ├── exec-google-cli/SKILL.md           # 🛠️ Executor
│   ├── exec-github-cli/SKILL.md
│   ├── exec-cloudflare-cli/SKILL.md
│   ├── exec-cloudinary-cli/SKILL.md
│   ├── exec-obsidian-cli/SKILL.md
│   │
│   ├── gen-prd/SKILL.md                   # 🧠 Generator
│   ├── gen-prd-checker/SKILL.md
│   ├── gen-cto-advisor/SKILL.md
│   ├── gen-storybook-mockup-gen/SKILL.md
│   ├── gen-trello-project-setup/SKILL.md
│   └── gen-wport-code-assistant/SKILL.md
└── docs/

prd/                                       # 各使用此 skill 的專案
├── .claude/
│   ├── exec-google-cli → ../../wport_skills/src/exec-google-cli   (symlink)
│   ├── ...                                                          (symlinks)
│   └── gen-wport-code-assistant → ../../wport_skills/src/gen-wport-code-assistant
└── wport.config.json
```

> 為什麼採「攤平 + 前綴」而不是 `executors/` / `generators/` 子資料夾？
> 因為 Claude Code 與 Cursor 預設只掃描 `.claude/<skill-name>/SKILL.md` 一層深度。前綴方案兼顧分類視覺與工具相容性。

> 為什麼用 symlink 而不是把實體放在 `prd/.claude/`？
> 中央集權：所有專案共用同一個 src of truth，改一次處處生效，不會 drift。詳見 README「為什麼用 symlink」一節。

---

## 1. 🛠️ Executors（任務導向）

### 核心目的

**教 AI 如何正確下指令、不踩資安紅線。**

Executor 處理的是「**手腳**」：對外部世界發出實際命令（CLI、API、deploy）。這類 skill 的失敗成本很高——一條錯指令可能洩漏 token、刪錯資料、把 staging 部署到 production。

### YAML Frontmatter 規範

```yaml
---
name: exec-google-cli
description: gcloud CLI for Google Sheets, ADC, account switching. Use when the user mentions gcloud, Google Sheets, ADC, or scope/403 errors.
category: executor
keywords: [gcloud, sheets, ADC, deploy, oauth]
risk_level: medium    # low / medium / high（牽涉認證/部署都至少 medium）
---
```

### 內文結構（必備區塊）

1. **What you can do** — 高頻使用情境清單
2. **Quick commands** — 程式碼區塊，附最常用 5–10 個指令
3. **Troubleshooting** — 對照表（錯誤訊息 → 修法）
4. **Security guardrails** — 紅線清單（不要 commit / 不要硬編 / 必須驗證...）

### 撰寫禁忌

- 不要把整本 CLI 手冊抄進來（用 `<tool> --help` 即時查冷門指令）
- 不要寫成教學文章（直接給可貼上執行的指令）
- 不要省略安全提醒（即使覺得是常識）

### 範本參考

`src/exec-google-cli/SKILL.md`、`src/exec-github-cli/SKILL.md`。

---

## 2. 🧠 Generators（角色與思考導向）

### 核心目的

**教 AI 如何像大師一樣思考、不吐出敷衍的廢話。**

Generator 處理的是「**大腦**」：把模糊的需求收斂成標準產出（PRD、規格、決策建議、Mockup）。這類 skill 的失敗模式是「看起來合理但實際空洞」——所以重點在 persona 設定、思維鏈引導、與審查 checklist。

### YAML Frontmatter 規範

```yaml
---
name: gen-prd
description: Generates or refines product requirement documents (PRD). Use when the user provides a raw feature idea, asks for a spec document, or wants to structure a feature for wport 2.0.
category: generator
persona: Staff Product Manager
output_format: markdown
review_required: true   # 如為 true，產出必須由配對的 reviewer skill 過一次
review_pairing: gen-prd-checker
---
```

### 內文結構（必備區塊）

1. **Persona & Tone** — 明確定義 AI 扮演的角色（職稱、年資、價值觀、溝通風格）
2. **Core Template Structure** — 產出物的標準模板（區塊、順序、層級）
3. **Agent Workflow** — 步驟化指令（含「Ask before assuming」「Challenge the user」這類元規則）
4. **Output Format** — 嚴格的格式約束（Markdown 層級、語言策略 zh-TW / en）
5. **Review Checklist**（可選）— 自我檢查清單，或指向 reviewer skill

### 撰寫禁忌

- 不要只寫「請生成一份 PRD」這種模糊指令
- 不要省略 persona 設定（沒有角色 = 沒有判斷標準）
- 不要把多種 persona 混在同一個 skill（生成與審查必須拆開——self-review 抓不到 self 的 blind spot）

### 範本參考

`src/gen-prd/SKILL.md`、`src/gen-prd-checker/SKILL.md`。

---

## 分類決策樹

寫新 skill 前，先回答這三題：

```text
Q1：這個 skill 會「對外部系統發出指令」嗎？
   （gcloud、gh、wrangler、curl 某 API、deploy...）
   ├─ Yes → Executor (exec-*)
   └─ No  → 繼續 Q2

Q2：這個 skill 會「產出文件 / 規格 / 決策建議」嗎？
   （PRD、Mockup、Code review report、決策...）
   ├─ Yes → Generator (gen-*)
   └─ No  → 繼續 Q3

Q3：這個 skill 是純查詢 / 純展示？
   ├─ Yes → 重新考慮：可能不需要寫成 skill，
   │        AI 內建知識 + --help 就能處理
   └─ No  → 回到 Q1 重新分析需求
```

---

## 命名規範

| 規則 | 範例 |
|------|------|
| 前綴必須是 `exec-` 或 `gen-` | `exec-google-cli`、`gen-prd` |
| 使用 kebab-case | `gen-storybook-mockup-gen` |
| 對 CLI 工具，名稱用「工具品牌」 | `exec-cloudflare-cli` 而非 `exec-cf-deploy` |
| 避免冗餘後綴 | `gen-prd`（OK）；`gen-prd-gen`（重複） |

---

## 與 `wport.config.json` 的關係

`prd/wport.config.json`（住在使用此 repo 的專案內）是 **runtime 設定**（哪個 skill 用什麼 review pairing、business rules 在哪），而本文件是 **撰寫規範**。兩者互補：

- 想知道**怎麼寫**新 skill → 本文件
- 想知道 skill **怎麼被執行 / 配對** → 專案內的 `wport.config.json`

> 注意：`wport.config.json` 是專案層級設定（每個專案可有自己的版本），不放在 src of truth `wport_skills` 裡。
