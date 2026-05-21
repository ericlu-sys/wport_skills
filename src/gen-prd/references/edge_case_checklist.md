# PRD Edge Case Checklist (Mockup/Spec Preflight)

> Purpose: mandatory checks in `gen-prd` Stage 1 (discovery), Stage 2 (scenario expansion), and Stage 7 (final gate).  
> Goal: prevent happy-path-only PRDs and ensure edge behavior is implementable, testable, and recoverable.

---

## How To Use

- Apply all five categories at least once for each major flow.
- For every selected edge case, document in PRD:
  - **System behavior** (what the system does)
  - **UI feedback** (what the user sees)
  - **State transition** (retry/recoverability)
  - **Data consistency** (duplicate write/loss prevention)
- If a case is not applicable, write `N/A + reason`. Never leave blank.

---

## 1) Network & Performance

### 1.1 Timeout
- If a request exceeds N seconds, does the system stop automatically?
- Does the UI show a clear error (for example: request timed out, please retry)?
- Is retry available without infinite loading?

### 1.2 Race Condition (Repeated Trigger)
- If users double-click submit, is only one request accepted?
- Does backend have idempotency key or deduplication strategy?
- Is submit action disabled while request is in progress?

### 1.3 Disconnection
- During upload/AI parsing, if network drops, is a disconnected state shown?
- After disconnect, can user resume/retry, or must restart?
- Is UI protected from crash/freeze?

### 1.4 High Latency
- Are progress bar, skeleton screen, or staged loading messages provided?
- Is there persistent feedback that processing is still ongoing?

---

## 2) Data & Input

### 2.1 Format Mismatch
- Are file extension and MIME type both validated (for example, PDF only)?
- Are unsupported formats blocked early with clear explanation?

### 2.2 Size Limit
- Are limits defined for file size, text length, and field max length?
- Does error copy clearly include the allowed limit?

### 2.3 Null/Empty State
- If parsing result is empty or whitespace-only, how does UI render?
- Is there a recovery path (re-upload, manual input)?

### 2.4 Special Characters
- Are emoji, variant characters, and multilingual text stored/rendered correctly?
- Is XSS handled (escaping/sanitization/allowlist)?

---

## 3) User Interruption

### 3.1 Navigation
- While AI is running, does leaving the page require confirmation?
- On leave, is task behavior consistent (continue/pause/cancel)?

### 3.2 Close Window
- Does background task continue, and is server-side job tracking needed?
- When user returns, can progress be restored?

### 3.3 Refresh
- After refresh, can loading state be restored?
- If not restorable, is a clear retry/resume entry shown?

---

## 4) Auth & Lifecycle

### 4.1 Session Timeout
- If session expires mid-flow, does system redirect to login and preserve return path?
- After re-login, can user return to interrupted step?

### 4.2 Concurrency
- If same record is edited in multiple tabs/users, how is conflict resolved?
- Last-write-wins, version lock, or merge strategy? Is conflict surfaced in UI?

### 4.3 Trust Boundary (External Identity / Provider Data)

For any flow that consumes data from a third-party provider, external API, OAuth callback, or other user-controllable external source:

- **Verified vs user-controllable classification**: For each field read from the external source, label whether it is provider-verified (cryptographically signed, e.g., Apple `sub`, Google `email_verified=true`) or user-controllable (provider allows user to fill freely, e.g., LINE self-filled email, LinkedIn display name).
- **Auto-link / auto-merge gate**: If the system uses an external-provided email/identifier to link, merge, or grant access to an existing account, does the logic require `verified=true`? If not, document the threat model explicitly (account takeover via spoofed unverified email is a real attack).
- **Step-up auth for sensitive actions**: For unlink credential, bind-new-provider, change-primary-email, account-delete — does an active session alone suffice, or is re-authentication (re-OAuth, password re-entry) required? Default expectation: sensitive actions need step-up.
- **Cross-session impact**: When credentials change (unlink, password change, primary email change), are sessions on other devices invalidated?
- **UID stability assumptions**: For each provider, is the UID stable across re-authentications? (Apple `sub` stable; Apple `email` only returned on first auth — don't assume it's always available.)
- **Cross-provider collision**: Could two different providers return the same UID format and collide? (Defense: namespace UIDs by provider.)

### 4.4 Rate Limiting & Abuse

For any endpoint that can be triggered without a logged-in session, or that touches authentication / account lifecycle:

- Rate limit dimensions named (per IP, per email, per account, or composite)?
- Lockout / backoff policy after N failed attempts?
- Captcha / proof-of-work threshold for suspicious patterns?

---

## 5) Logical Inconsistency

### 5.1 Partial Success
- If AI parses only some fields, how is overall status defined?
- Does UI clearly separate successful vs failed fields and allow correction?

### 5.2 Step Skipping
- If user jumps directly to step 2 without step 1, is it blocked?
- For deep links, are prerequisites validated and redirected correctly?

---

## PRD Snippet (Copy/Paste)

```md
### Edge Case Coverage

| Category | Scenario | System Behavior | UI Feedback | Retry | Data Consistency Strategy | Notes |
|---|---|---|---|---|---|---|
| Network & Performance | Timeout > 30s | Cancel request and return timeout code | Show "Request timed out. Please retry." | Yes | No write on timeout | |
| Data & Input | Non-PDF upload | Reject at FE and validate again at BE | Show allowed file formats | N/A | N/A | |
```

---

## Definition of Done

- Each major flow covers at least one edge case from all five categories.
- Every high-risk action (submit/upload/parse/navigation) includes a failure branch.
- Timeout, retry, idempotency, and session-expiry handling are never `TBD`.
