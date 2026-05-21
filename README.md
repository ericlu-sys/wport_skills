# wport-skills (Meta Repository)

> **狀態變更**：所有 `SKILL.md` 已於 2026-05 遷移至 [`prd/.claude/`](../prd/.claude)。
> 此 repo 現在只保留**設計哲學**與**架構規範**文件，不再放置可執行 skill。

---

## 為什麼留這個 repo？

Skill 檔案搬到了實際使用它們的專案（`prd/`），但「**為什麼要這樣寫 skill**」這件事，仍然需要一個跨專案的中央文件。這個 repo 就是負責回答這類問題：

- 哪些東西該寫成 skill？哪些不該？
- Executor 與 Generator 的分界線在哪？
- Token 成本、幻覺風險、live web search 的取捨原則？

如果你只是要**使用** skill，去 [`prd/.claude/`](../prd/.claude)；如果你要**寫新的** skill，先讀這裡。

---

## Skill 兩大分類

| 分類 | 命名前綴 | 核心目的 | YAML 重心 | 內文重心 |
|------|---------|---------|----------|---------|
| 🛠️ **Executors** | `exec-*` | 教 AI 正確下指令、不踩資安紅線 | `gcloud`, `gh`, `deploy`, `run command` | 程式碼區塊、Troubleshooting 對照表 |
| 🧠 **Generators** | `gen-*` | 教 AI 像大師一樣思考、不吐垃圾 | `PRD`, `Business Logic`, `Edge Case` | 思維鏈引導、標準模板、Checklist |

完整撰寫規範見 [`docs/architecture.md`](docs/architecture.md)。

---

## 當前 Skill 索引（位於 `prd/.claude/`）

### 🛠️ Executors

| Skill | 用途 |
|-------|------|
| `exec-google-cli` | `gcloud`、Google Sheets API、ADC / scopes |
| `exec-github-cli` | `gh` for PRs、issues、runs、releases |
| `exec-cloudflare-cli` | Wrangler、Workers vs Pages deploy |
| `exec-cloudinary-cli` | `cld`、upload、background removal |
| `exec-obsidian-cli` | Obsidian URI / `obsidian-cli` |

### 🧠 Generators

| Skill | 用途 |
|-------|------|
| `gen-prd` | 互動式 PRD 生成（多 Persona、多 stage 流程） |
| `gen-prd-checker` | PRD 對抗性審查（Backend RD / Security / SRE 三鏡頭） |
| `gen-cto-advisor` | CTO 級決策建議 |
| `gen-storybook-mockup-gen` | Storybook mockup 生成 |
| `gen-trello-project-setup` | Trello 專案初始化 |
| `gen-wport-code-assistant` | wport 編碼助手 |

---

## 設計哲學

詳細哲學論述見 [`docs/three-layer-pyramid.md`](docs/three-layer-pyramid.md)：

- 為什麼不把整本 CLI 手冊塞給 AI？（Token 成本 + 延遲）
- 為什麼不靠 live web search？（時間差 + Anti-bot）
- **三層金字塔**：AI 內建知識（底）／SKILL.md（中）／`--help`（頂）

---

## Contributing

寫新 skill 之前：

1. 先讀 [`docs/architecture.md`](docs/architecture.md) 判斷 executor / generator 分類
2. 命名套用前綴規範（`exec-` 或 `gen-`）
3. 安全紅線：
   - **Do not commit** secrets、API keys、tokens、credential JSON 檔名、內網 IP、機器絕對路徑
   - 用 placeholder：`<ACCOUNT_EMAIL>`、`<HOST>`、`<PROJECT_NAME>`、`<SPREADSHEET_ID>`

---

## Security

如果意外提交了敏感資料：

1. 立刻 rotate 受影響的 credential
2. 用 `git filter-repo` 或 `git rebase -i` 重寫歷史
3. 在分享 branch 前確認 secret 已從 git history 完全清除
