---
name: gen-wport-code-assistant
description: Investigate product and technical questions across WPORT repositories, verify behavior from real code before answering, and produce PRD-ready conclusions with explicit assumptions and gaps. Use when discussing WPORT backend/frontend logic, fields, events, unread/read behavior, notification flow, or cross-repo ownership.
---

# WPORT 程式助手

## Purpose

用於回答「我們系統現在實際怎麼做」這類問題。  
先查 code 再下結論，避免只靠記憶或猜測。

## Repo Map

- `W101-Web`：主要前端（使用者端、招募端 UI 與互動）
- `W101-Admin-Web`：管理後台前端
- `W101-AMS`：後端服務（API、事件、資料處理）
- `W101-TalentSearchHub`：人才搜尋相關服務/模組

## When To Use

遇到以下情境時優先套用：

- 「這個欄位到底有沒有？」（如 `is_read`, `last_read_at`, `unread_count`）
- 「通知/寄信在哪裡觸發？」
- 「某個規則現在是 PRD 寫的，還是 code 已經做了？」
- 「同一需求涉及前後端多 repo，想先確認真實結構」

## Workflow

### 1) Classify the question

先判斷問題類型，決定優先查哪個 repo：

- **資料欄位/事件觸發/API** → 先查 `W101-AMS`
- **畫面顯示/互動/條件分支** → 先查 `W101-Web` 或 `W101-Admin-Web`
- **人才搜尋邏輯** → 查 `W101-TalentSearchHub`

### 2) Verify with code, not assumptions

至少找出以下其中兩種證據再回答：

- 型別/資料模型（interface, schema, migration）
- API handler / service 實作
- 事件 producer / consumer
- UI 條件分支（v-if, computed, state）

### 3) Output format (required)

回覆時固定輸出三段：

1. **已確認事實**（附檔案路徑）
2. **尚未確認/需決策**（明確標示不確定）
3. **PRD 建議回寫內容**（可直接貼進 PRD 的句子）

## Guardrails

- 找不到證據就明說「目前在 repo 中未找到」。
- 不要把推測寫成既有事實。
- 跨 repo 結論必須對齊名詞（同一欄位不要用兩種叫法）。

## Quick Checklist

- [ ] 有先確認問題屬於哪個 repo 範圍
- [ ] 有至少 2 個程式證據
- [ ] 有區分「現況」與「建議」
- [ ] 有給可回寫 PRD 的短句

## Additional Resources

- 關鍵字與目錄對照：`references.md`
