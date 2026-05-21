---
name: gen-trello-project-setup
description: Automated Trello card creation for project setup with standardized naming conventions and labels. Creates one goal card and two delivery cards (FE + BE) on the PM board with prd repo link, PRD path, and Storybook base URL. Storybook is hosted at https://storybook.wport.me/ (no local port 6006). When mockups are generated, the specific story link must be attached to these cards and to the PRD.
---

# Trello Project Setup

## Overview

This skill creates **three cards** on the PM board per PRD:
- **One goal card**: `[level:goal] {project_name_chinese}` with `level:goal` label only
- **Two delivery cards**: `[level:delivery] {project_name_chinese}-畫面` with `level:delivery` + `role:fe`, and `[level:delivery] {project_name_chinese}-api` with `level:delivery` + `role:be`

CRITICAL (hard constraint):
For each PRD file, this skill MUST create EXACTLY 3 cards total (1 goal + 2 delivery).
It MUST NOT create any additional cards (no epics, no QA cards, no FE/BE sub-cards, no section-splitting into more cards).
If you need more granularity, put it into checklist items ONLY within the two delivery cards.

All cards get the same description, using EXACTLY this format:

GitHub Repository: prd > https://github.com/w101-admin/prd
PRD: {prd_relative_path}  (e.g. `doc/feature/pm_31/siteadmin-dashboard-mockup.md`)
Storybook: {storybook_url}

Where:
- `{prd_relative_path}` = relative path from repo root
- `{storybook_url}` = extract the full URL from the PRD markdown field `Storybook 連結` (must be the full `https://storybook.wport.me/?path=...` link)
- `{pm_number}` = parse from the PRD folder `doc/feature/pm_XX/...` (used only for detection; do not include it in the description unless the PRD explicitly contains it)

**PRD paths in cards:** Use **relative path** (複製相對路徑) from repo root (e.g. `doc/feature/pm_25/xxx.md`) so teammates can open the same file regardless of where they cloned the repo.

## Board Mapping

This skill only creates cards on the **PM** board. FE/BE board IDs are for reference when you move or create cards yourself.

- **PM** (Project Management): `STdWiajz` — used by this skill
- **FE** (Frontend): `IYi0tcMo` — reference only
- **BE** (Backend): `bbEZeABT` — reference only

**Reference:** See [board-mapping.md](references/board-mapping.md) for detailed board information.

## Workflow

Creating a project setup involves these steps:

0. **Detect PRD File Path**: Extract PRD file path from user's input and **convert to relative path** (relative to repo root, e.g. `doc/feature/pm_25/add-non-registered-member-mockup.md`). Use relative path so cards are shareable with teammates (everyone has different absolute paths). The PRD file is typically referenced when triggering card creation.
0.1 **Project name (Chinese)**: Derive from PRD filename, e.g. `recruitment-campaign-mockup.md` → 招募活動. Prefer translating the kebab-case part (e.g. recruitment-campaign → 招募活動), not the full doc title.
1. **Create Main Goal Card on PM Board**:
   - Set PM board as active (STdWiajz)
   - Create card with name `[level:goal] {project_name_chinese}` (project name in Chinese)
   - Add only `level:goal` label to the main card
   - Add prd repository link, **PRD** path (relative), and **Storybook** path to description
2. **Create Two Delivery Cards on PM Board**:
   - Card A: name `[level:delivery] {project_name_chinese}-畫面`, labels: `level:delivery` + `role:fe` (frontend)
   - Card B: name `[level:delivery] {project_name_chinese}-api`, labels: `level:delivery` + `role:be` (backend)
   - Same description as the goal card (prd repo, PRD path, Storybook path)
   - Both go in the same list as the goal card (e.g. first list).

HARD CONSTRAINT:
Do not split PRD sections into multiple cards. Only create 3 cards per PRD (goal + FE畫面 + BE-api).

## Step-by-Step Implementation

### Step 0: Detect PRD File Path (Use Relative Path)

**Before starting any Trello operations**, extract the PRD file path from the user's input and **convert to relative path** (relative to repository root):

- Look for file references in the user's message (e.g., `@/path/to/prd/file.md` or explicit file paths; paths may be absolute on the user's machine)
- **Convert to relative path**: strip the repo root so the path is relative (e.g. `doc/feature/pm_25/add-non-registered-member-mockup.md`). This allows teammates to open the same file regardless of where they cloned the repo (複製相對路徑).
- Store the **relative path** to use in card description

**Example:**
```
User input: "Create Trello cards using @/Users/Eric/Documents/Github/prd/doc/feature/pm_25/add-non-registered-member-mockup.md"
Absolute path (user's machine): /Users/Eric/Documents/Github/prd/doc/feature/pm_25/add-non-registered-member-mockup.md
PRD path for card (relative):   doc/feature/pm_25/add-non-registered-member-mockup.md
```

**Important:**
- Extract the PRD file path from user's input automatically
- **Always use relative path in the card description** (e.g. `doc/feature/pm_25/xxx.md`) so teammates can use the same link
- If the path contains the repo name (e.g. `.../prd/doc/feature/...`), the relative path is `doc/feature/...` from the repo root
- If no PRD file is referenced, you may proceed without it (but typically users provide it)

### Step 1: Create Main Goal Card on PM Board

Set the PM board as active:

```typescript
mcp_trello_set_active_board({ boardId: "STdWiajz" })
```

Get the list ID, then create the main goal card with Chinese project name:

```typescript
// Get lists first
const lists = mcp_trello_get_lists({ boardId: "STdWiajz" })
const listId = lists[0].id  // Use first list

// Create main card
mcp_trello_add_card_to_list({
  boardId: "STdWiajz",
  listId: listId,
  name: "[level:goal] {project_name_chinese}"
})
```

Add the `level:goal` label and description (prd repo, PRD path, Storybook path):

```typescript
mcp_trello_update_card_details({
  boardId: "STdWiajz",
  cardId: "{card_id}",
  labels: ["{level_goal_label_id}"],
  description: `GitHub Repository: prd > https://github.com/w101-admin/prd
PRD: {prd_relative_path_from_step_0}
Storybook: {storybook_url}`
})
```

### Step 2: Create Two Delivery Cards on PM Board

Create two cards with the same name and description as the goal card, but with different labels:

- **Delivery (FE)**: `[level:delivery] {project_name_chinese}-畫面`, labels: `level:delivery` + `role:fe`
- **Delivery (BE)**: `[level:delivery] {project_name_chinese}-api`, labels: `level:delivery` + `role:be`

Use the same listId as the goal card. Get label IDs from `get_board_labels` on the PM board.

HARD CONSTRAINT:
Create only these two delivery cards. Never create epics/tasks/QA/sub-cards based on PRD sections.

### Step 3: 回填 Coda（必做）

建立完三張卡後，**必須**立即回填 Coda，避免遺漏追蹤資料。

在 repo 的 `coda` 目錄執行：

```bash
npm run sync:upsert-log -- "{goal_card_id}" "{goal_card_id}" "[level:goal] {project_name_chinese}" "{owner_name}" "{start_at_iso}" "" "active" "trello_skill:create_goal" "GOAL" "high" "true"
```

If this step fails, report the error and retry. Do not create Trello cards without Coda sync.

## Labels (must be applied)
- Goal card: `level:goal` ONLY
- FE delivery card (畫面): `level:delivery` + `role:fe`
- BE delivery card (api): `level:delivery` + `role:be`

