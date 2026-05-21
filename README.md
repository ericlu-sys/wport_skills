# Shared Cursor Skills

Shared [Cursor Agent Skills](https://cursor.com/docs) for CLI and platform workflows. Teammates clone this repo and open it (or add it as a project) so agents can load project skills from `.cursor/skills/`.

## Setup

1. Clone this repository.
2. Open the repo root in Cursor (or symlink/copy `.cursor/skills` into another project if your team prefers a monorepo layout).
3. In chat, reference a skill by name (e.g. `google-cli`) or describe the task; Cursor discovers skills from `.cursor/skills/<name>/SKILL.md`.

## Available skills

| Skill | Use when |
|-------|----------|
| `google-cli` | `gcloud`, Google Sheets API, ADC / scopes |
| `github-cli` | `gh` for PRs, issues, runs, releases |
| `cloudflare-wrangler` | Wrangler, Workers vs Pages deploy |
| `cloudinary-cli` | `cld`, upload, background removal |
| `obsidian-cli` | Obsidian URI / `obsidian-cli`, open vault/notes, create notes (app must be running) |

## 為什麼要做這份 Skill Repo？（Design Philosophy）

既然 AI 已經知道很多 CLI 用法，為什麼不直接把官方幾千頁的 `gcloud` / `gh` / `wrangler` 文件全部塞給它，或讓它每次去爬官網？因為在實務上會踩到三個現實成本。

### 1) Context Window 是有成本的

LLM 的脈絡長度雖然很大，但**吞下去的字越多，代價越高**：

- **Token 成本爆炸**：input tokens 直接計費。把整本 CLI 手冊塞進每次 prompt，跑一個「讀試算表」都要付出不成比例的費用，產品研發預算扛不住。
- **延遲變長**：脈絡越厚，AI 推理越慢。本來 2 秒能完成的指令，可能拖成 30 秒，DX（開發者體驗）會被摧毀。

### 2) 資訊過載會誘發幻覺（Needle in a Haystack）

當雜訊太多，AI 反而抓不到關鍵：

- 一個操作可能有 5 種進階做法（其中多數是給大型機房或跨國場景），AI 會開始糾結、組出不符合我們系統架構的複雜指令，產生 hallucination。
- 這份 repo 的每個 `SKILL.md` 本質上都是**過濾器（Filter）**：把 95% 用不到的進階指令濾掉，只留 5% 最安全、最高頻、最符合團隊規格的精華路徑。

### 3) 為什麼不靠即時 Live Web Search？

「要用再去爬官網」聽起來完美，但在 CLI 自動化上會出兩種事：

- **時間差**：工程師在終端機下指令期待秒級回應；AI 每步都去 Google + 解析 SPA + 回頭寫程式碼，工具會因為太慢而被團隊拋棄。
- **網頁結構 / Anti-bot**：官方文件常是 SPA，且有反爬機制（包含 Cloudflare challenge）。爬蟲被擋，自動化流程就卡死在半空中。

### 三層金字塔架構（Three-Layer Pyramid）

我們不選擇「全塞」也不選擇「全爬」，而是把 AI 的知識來源切成三層：

| 層級 | 來源 | 負責 |
|------|------|------|
| 底層 | AI 內建知識 | 通用 CLI 語法與基本概念（免費、最快） |
| 中層 | 這份 repo 的 `SKILL.md`（方向盤） | 團隊規範、資安守則、核心高頻指令、操作 SOP |
| 頂層 | 本地 `--help`（如 `gh pr --help`、`gcloud sheets --help`、`wrangler --help`） | 冷門指令與版本差異——直接讀本地輸出，比爬網路精準、快 10 倍，且永遠符合當前安裝版本 |

**操作原則**：Skill 文件只寫高頻流程與排錯路徑；遇到冷門子指令，請 AI 在本地跑 `<tool> <subcommand> --help` 取得當前版本的真實輸出，而不是憑記憶或爬網路。

## Contributing

- **Do not commit** secrets, API keys, tokens, private key paths, public keys, credential JSON filenames, internal IPs, dashboard URLs, or machine-specific absolute paths.
- Use placeholders: `<ACCOUNT_EMAIL>`, `<HOST>`, `<PROJECT_NAME>`, `<SPREADSHEET_ID>`, etc.
- Keep each `SKILL.md` focused on workflows and troubleshooting; put long reference material in sibling files only if needed.

## Security

If you accidentally commit sensitive data, rotate the affected credentials and rewrite git history before sharing the branch.
