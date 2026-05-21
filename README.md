# wport-skills

> 中央 Skill Repository。所有 Cursor / Claude Code 用的 `SKILL.md` 都住在這裡，
> 各 wport 專案透過 **symlink** 連回來，做到「一處維護、處處生效」。

---

## 結構

```text
wport_skills/
├── src/                                  # 🌟 所有 skill 的 src of truth
│   ├── exec-google-cli/SKILL.md          # 🛠️ Executors
│   ├── exec-github-cli/SKILL.md
│   ├── exec-cloudflare-cli/SKILL.md
│   ├── exec-cloudinary-cli/SKILL.md
│   ├── exec-obsidian-cli/SKILL.md
│   ├── gen-prd/SKILL.md                  # 🧠 Generators
│   ├── gen-prd-checker/SKILL.md
│   ├── gen-cto-advisor/SKILL.md
│   ├── gen-storybook-mockup-gen/SKILL.md
│   ├── gen-trello-project-setup/SKILL.md
│   └── gen-wport-code-assistant/SKILL.md
├── docs/
│   ├── architecture.md                   # executors vs generators 規範
│   └── three-layer-pyramid.md            # 設計哲學
└── README.md
```

各專案的 `.claude/<skill-name>` 是 symlink，指回 `../../wport_skills/src/<skill-name>`。

---

## Skill 兩大分類

| 分類 | 命名前綴 | 核心目的 |
|------|---------|---------|
| 🛠️ **Executors** | `exec-*` | 教 AI 正確下 CLI 指令、不踩資安紅線（手腳） |
| 🧠 **Generators** | `gen-*` | 教 AI 像大師一樣思考、不吐廢話（大腦） |

完整撰寫規範見 [`docs/architecture.md`](docs/architecture.md)。

---

## 當前 Skill 清單

### 🛠️ Executors (5)

| Skill | 用途 |
|-------|------|
| `exec-google-cli` | `gcloud`、Google Sheets API、ADC / scopes |
| `exec-github-cli` | `gh` for PRs、issues、runs、releases |
| `exec-cloudflare-cli` | Wrangler、Workers vs Pages deploy |
| `exec-cloudinary-cli` | `cld`、upload、background removal |
| `exec-obsidian-cli` | Obsidian URI / `obsidian-cli` |

### 🧠 Generators (6)

| Skill | 用途 |
|-------|------|
| `gen-prd` | 互動式 PRD 生成（多 Persona、多 stage 流程） |
| `gen-prd-checker` | PRD 對抗性審查（Backend RD / Security / SRE 三鏡頭） |
| `gen-cto-advisor` | CTO 級決策建議 |
| `gen-storybook-mockup-gen` | Storybook mockup 生成 |
| `gen-trello-project-setup` | Trello 專案初始化 |
| `gen-wport-code-assistant` | wport 編碼助手 |

---

## 如何在你的專案裡使用這些 skill

### 前置：確保兩個 repo 在同一個父目錄
```text
~/Documents/Github/
├── wport_skills/        # 此 repo
└── <your-project>/      # 例如 prd/、frontend/、backend/
```

### 連結 skill 到專案
```bash
cd <your-project>
mkdir -p .claude
cd .claude
# 連結需要的 skill（範例：把全部 skill 都連過來）
for d in ../../wport_skills/src/*/; do
  name=$(basename "$d")
  ln -s "../../wport_skills/src/$name" "$name"
done
```

### 驗證
```bash
ls -la .claude/                                    # 應該看到一堆 lrwxr-xr-x（symlink）
head -3 .claude/exec-google-cli/SKILL.md           # 應該讀得到實際內容
```

Claude Code / Cursor agent 在 `<your-project>` 下啟動時，會自動掃描 `.claude/<skill>/SKILL.md`，透過 symlink 讀到 `wport_skills/src/` 的真實內容。

---

## 為什麼用 symlink，不用 git submodule？

| 方案 | 優點 | 缺點 |
|------|------|------|
| **Symlink（採用）** | 即時更新、零同步成本、不需要任何 git 知識 | 跨機器需確認兩 repo 在相對位置 |
| Git submodule | 跨機器自動 clone | 操作複雜（`--recursive`、`submodule update`），新人易踩雷 |
| 各專案複製一份 | 部署簡單 | drift 風險高，11 個 skill × N 個專案要手動同步 |

---

## 設計哲學

詳見 [`docs/three-layer-pyramid.md`](docs/three-layer-pyramid.md)：

- 為什麼不把整本 CLI 手冊塞給 AI？
- 為什麼不靠 live web search？
- **三層金字塔**：AI 內建知識（底）／SKILL.md（中）／`--help`（頂）

---

## Contributing

新增或修改 skill：

1. 先讀 [`docs/architecture.md`](docs/architecture.md) 判斷 executor / generator 分類
2. 命名套用前綴規範（`exec-` 或 `gen-`）
3. 在 `src/<skill-name>/SKILL.md` 寫實作內容
4. 安全紅線：
   - **Do not commit** secrets、API keys、tokens、credential JSON 檔名、內網 IP、機器絕對路徑
   - 用 placeholder：`<ACCOUNT_EMAIL>`、`<HOST>`、`<PROJECT_NAME>`、`<SPREADSHEET_ID>`
5. Push 後，使用此 skill 的專案因為是 symlink，**自動拿到最新版**（不需要在專案端做任何動作）

---

## Security

如果意外提交了敏感資料：

1. 立刻 rotate 受影響的 credential
2. 用 `git filter-repo` 或 `git rebase -i` 重寫歷史
3. 由於各專案以 symlink 連到此 repo，敏感資料一旦進 history 就會被 N 個專案的 agent 讀到——更要謹慎
