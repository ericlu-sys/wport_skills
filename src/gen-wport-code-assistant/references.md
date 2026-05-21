# WPORT Repo 查找對照

## 常見問題 -> 優先查找

- **訊息已讀/未讀**
  - 關鍵字：`is_read`, `read_at`, `last_read_at`, `unread_count`, `conversation`
  - 優先 repo：`W101-AMS`（資料與 API）→ `W101-Web`（顯示與互動）

- **通知與寄信**
  - 關鍵字：`notification`, `mail`, `email`, `template_key`, `event`, `queue`, `retry`
  - 優先 repo：`W101-AMS`

- **新應徵/人才回覆觸發點**
  - 關鍵字：`job_application_created`, `talent_reply_created`, `message_created`
  - 優先 repo：`W101-AMS`，再對照 `W101-Web` 的前端觸發 API

- **偏好設定頁顯示條件**
  - 關鍵字：`hasCompanyMembership`, `preferences`, `toggle`, `account settings`
  - 優先 repo：`W101-Web`

## 建議查找順序

1. 在後端找「資料定義 + 事件來源」
2. 在前端找「顯示邏輯 + 互動觸發」
3. 若有跨服務，再回頭補齊中間層（queue/consumer）

## 回答時最低證據標準

- 至少兩個證據點（例如：schema + service，或 handler + UI condition）
- 若找不到就明確標示「未找到」
- 不把推測寫成既有事實
