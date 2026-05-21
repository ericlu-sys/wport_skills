# PRD Checker — Per-Persona Detailed Checklists

> Companion to `SKILL.md`. Load at Stage 0 and reference during Stage 1 per-persona passes.
> Each list is non-exhaustive — use judgment to surface domain-specific gaps not listed here.
> All findings must follow `SKILL.md` Rule 3 (cite file_path:line_number) and Rule 4 (concrete fix wording).

---

## 1) Backend RD Reviewer Checklist

**Lens**: "I'm the engineer writing the OpenAPI Spec from this PRD next sprint. What stops me, what misleads me, what bites me at integration time?"

### 1.1 Technical Premise Verification

For every named vendor, SDK, framework, or service in the PRD:

- [ ] Does the vendor natively support the stated capability? Cite docs.
- [ ] Common traps:
  - **Firebase Auth**: does NOT natively support LINE, LinkedIn, WeChat, Kakao, Naver. Those need custom token flow (verify ID token yourself + Firebase Admin SDK `createCustomToken`).
  - **Stripe Connect**: regional restrictions (e.g., Taiwan account limitations).
  - **Auth0 / Cognito**: feature parity differs by tier (e.g., enterprise SSO requires paid plan).
  - **SendGrid / Mailgun**: deliverability differs by region; sandbox mode behaves differently.
- [ ] For "use X to handle Y" statements: cite docs, or downgrade to "RD-selectable direction" per `gen-prd` Rule 31.

### 1.2 Data Semantics Completeness

For every entity declared in PRD §5:

- [ ] Lookup key explicit (what field finds this row)?
- [ ] Uniqueness constraints stated (semantically, not as DB index)?
- [ ] Retention policy stated (forever / N days / event-driven)?
- [ ] Cross-entity relationships described (N:1, N:N, optional)?
- [ ] Audit / event log retention duration named?
- [ ] Soft-delete vs hard-delete behavior declared?
- [ ] Sensitive data classified (PII, secret, public)?

Boundary check (per `gen-prd` Rule 30): PRD describes **WHAT** to track, NOT field names or DB schema. If PRD prescribes field names, that's actually a *PRD discipline* violation, not a missing-data finding — but flag it as P2.

### 1.3 Migration / Backwards Compatibility (per `gen-prd` Rule 22)

**Trigger this section if**: feature touches existing users' data model, login methods, stored state, or persisted business identifiers — even for additive features.

- [ ] Migration plan explicit (in-flight dual-write, cutover, rollback)?
- [ ] Backfill default for new required fields stated?
- [ ] Dry-run capability mentioned?
- [ ] Rollback path described (especially for irreversible schema changes)?
- [ ] Cross-deploy compatibility (FE deployed without BE, vice versa)?
- [ ] Existing data lookup behavior during migration (do old rows still resolve)?

### 1.4 Idempotency & Concurrency

- [ ] Idempotency key construction explicit? Candidate keys:
  - `(resource_id, action, version)`
  - `(user_id, action, request_id)`
  - `(provider, provider_uid)` for natural keys
- [ ] Idempotency cache TTL stated?
- [ ] Concurrent same-resource modifications strategy (last-write-wins / version lock / CAS)?
- [ ] Race conditions between two devices / two tabs covered?
- [ ] Webhook re-delivery idempotency (if external service involved)?

### 1.5 Dependency Mapping

- [ ] External services this feature depends on listed?
- [ ] For each dependency: failure mode + system behavior when down?
- [ ] Circular dependency check (this service depends on X, X depends on this service)?
- [ ] Cold-start dependencies (what must be up at boot)?
- [ ] Cascading failure prevention (circuit breaker, timeout, fallback)?

### 1.6 Error Contract Completeness

- [ ] Error codes follow existing wport conventions OR define new namespace explicitly?
- [ ] 4xx vs 5xx categorization correct for each documented error?
- [ ] Error code human-readable copy provided for FE?
- [ ] Retryable vs non-retryable errors distinguished?

### 1.7 Performance Budgets

- [ ] p50 / p99 latency targets stated (or "sync with existing X" reference)?
- [ ] Throughput expectations stated for high-volume flows?
- [ ] Synchronous vs async work explicit (don't make user wait for 30s job)?

### 1.8 API Contract Completeness (per `gen-prd` Rule 21 / 26)

These must NOT be TBD:
- [ ] Token TTL
- [ ] Token revoke policy
- [ ] Expiration behavior
- [ ] Idempotency
- [ ] Real-time auth re-check requirement for sensitive actions

---

## 2) Security Engineer Reviewer Checklist

**Lens**: "Where can an attacker break this? Where does this PRD assume external input is trustworthy when it isn't?"

### 2.1 Trust Boundary (per `edge_case_checklist.md` §4.3)

For every external source (provider OAuth, file upload, user input form, webhook):

- [ ] Each field classified as **provider-verified** (cryptographically signed) or **user-controllable** (provider lets user fill freely)?
- [ ] Specific known traps:
  - **LINE Login**: `email` is user-controllable. LINE does NOT verify email by default.
  - **LinkedIn**: `email` is verified at LinkedIn account creation, but display name and headline are user-controllable.
  - **Apple Sign-In**: `sub` is verified and stable. `email` only returned on FIRST authentication; later auths don't include it.
  - **Google**: `email_verified` field present; check it.
  - **Custom file upload**: filename, MIME type are user-controllable; magic bytes are the truth.
- [ ] For auto-link / auto-merge logic using external email as key: requires `verified=true`?
- [ ] Cross-account lookups via external email: lowercased + normalized form for comparison?

### 2.2 Authentication & Step-Up Auth

For sensitive actions, an active session alone is usually insufficient:

- [ ] **Unlink credential / provider** — step-up auth required? (Default expectation: YES.)
- [ ] **Bind new provider** — naturally a step-up via fresh OAuth. Verify it's not bypassed.
- [ ] **Change primary email** — step-up required?
- [ ] **Change password** — old password required?
- [ ] **Delete account** — step-up required?
- [ ] **Add payment method** — step-up required?
- [ ] **Privilege transfer** (e.g., ownership transfer) — step-up required for both parties?

### 2.3 Session Lifecycle

- [ ] On credential change (password / unlink / primary email): are sessions on other devices invalidated?
- [ ] Session token storage strategy (httpOnly cookie vs localStorage)?
- [ ] Session timeout / sliding expiration policy?
- [ ] Concurrent session limit (if any)?

### 2.4 Account Lifecycle Threats

Threat model walkthrough:

- [ ] **Account takeover via spoofed email + auto-merge** (the PM_39 hole — unverified email used as merge key).
- [ ] **Provider UID collision** (different providers returning same identifier; namespace by provider).
- [ ] **Race condition during account creation** (two concurrent sign-ups with same email).
- [ ] **Email change without re-verification** (attacker who gains session can redirect future password resets).
- [ ] **Account squatting** (registering accounts with future-intended emails to extort later).
- [ ] **Account recovery flow** (forgot password, lost provider access) — separate threat model needed.
- [ ] **Provider account compromise propagation** (user's LINE got hacked → can attacker pivot to wport?).

### 2.5 Authorization

- [ ] For multi-tenant data: tenant boundary enforced server-side, not client-side?
- [ ] For role-based actions: role check explicit at API layer?
- [ ] Privilege escalation paths considered (user → admin, viewer → editor)?
- [ ] IDOR (Insecure Direct Object Reference) defenses (e.g., `/users/{id}` checks current user can access `{id}`)?

### 2.6 Rate Limiting & Abuse (per `edge_case_checklist.md` §4.4)

- [ ] Rate limit on auth-adjacent endpoints (login, OAuth callback, password reset, signup)?
- [ ] Rate limit dimension stated (per IP, per email, per account, or composite)?
- [ ] Captcha / proof-of-work for suspicious patterns?
- [ ] Account lockout policy after N failed attempts?
- [ ] Backoff progression (linear vs exponential)?

### 2.7 Audit & Forensic Logging

Sensitive events to log:

- [ ] Login, logout, failed login (with reason)
- [ ] Credential add / remove / change
- [ ] Email change
- [ ] Account merge / link
- [ ] Privilege escalation / role change
- [ ] Sensitive data access (export, view PII)

For each:
- [ ] Sufficient context for forensic investigation (who, what, when, from where)?
- [ ] Log retention duration stated?
- [ ] Tamper-evidence (append-only, signed)?
- [ ] Logs accessible to security team / SRE on-call?

### 2.8 PII & Compliance

- [ ] PII fields minimized (collect only what's used)?
- [ ] PII storage encrypted at rest?
- [ ] Right-to-delete (GDPR / Taiwan PIPA / etc.) flow explicit?
- [ ] Data export flow (user data portability) explicit?
- [ ] Provider-specific compliance:
  - **Apple App Store 4.8**: Sign in with Apple required if other third-party login offered
  - **LINE Login terms**: data usage restrictions
  - **LinkedIn Developer Agreement**: rate / use case restrictions

### 2.9 Cryptographic Hygiene

- [ ] Tokens / secrets storage strategy stated (not in localStorage if XSS-prone)?
- [ ] Token rotation strategy?
- [ ] Sensitive fields hashing / encryption named (semantically, not algorithm — per `gen-prd` Rule 30)?
- [ ] Cryptographic operations delegated to platform (don't roll your own)?

---

## 3) SRE / Operator Reviewer Checklist

**Lens**: "It's 3am. This feature is broken. What do I see in the dashboard, and what's my runbook?"

### 3.1 Failure Mode Coverage

For every external dependency (provider, Firebase, SendGrid, DB, cache, queue):

- [ ] What happens when it's down? (degrade? fail-fast? queue and retry?)
- [ ] Timeout settings stated?
- [ ] Retry policy stated (attempts, backoff)?
- [ ] Circuit breaker / bulkhead pattern?
- [ ] Partial failure handling (some providers up, others down)?
- [ ] Stale data tolerance (can we serve cached when origin is down)?

### 3.2 Observability

- [ ] Metrics defined:
  - Success rate (per provider, per scenario)
  - Latency (p50, p95, p99)
  - Error rate (by error category)
  - Throughput / RPS
  - Queue depth (if async)
- [ ] Logging strategy (structured, with correlation ID)?
- [ ] Tracing across service boundaries (especially for provider-mediated flows)?
- [ ] Health check endpoint for new services?
- [ ] User-impact dashboard (separate from infra dashboard)?

### 3.3 Alerting Thresholds

- [ ] Error rate threshold for paging (e.g., > 5% error rate sustained 5 min)?
- [ ] Latency threshold for paging (e.g., p99 > 2s sustained 5 min)?
- [ ] Capacity warning thresholds (e.g., 80% of provisioned)?
- [ ] Dependency-down detection (synthetic monitor)?
- [ ] Alert noise budget (don't page on every blip)?

### 3.4 Capacity & Backpressure

- [ ] Expected load (RPS, concurrent users)?
- [ ] Peak vs steady-state estimates?
- [ ] Auto-scaling strategy or fixed capacity?
- [ ] Backpressure mechanism (queue, throttling, circuit breaker)?
- [ ] Cache strategy (TTL, invalidation triggers)?
- [ ] Cold-start time tolerance?

### 3.5 Rollback & Feature Flag Strategy

- [ ] Feature flag for safe rollout?
- [ ] Rollout strategy (percentage, geo, user cohort)?
- [ ] Migration rollback plan (especially for data migrations)?
- [ ] Schema rollback compatibility (additive only? destructive requires extra plan)?
- [ ] Cross-deploy compatibility (FE deployed without BE)?
- [ ] Kill switch for runaway behavior?

### 3.6 Runbook Readiness

- [ ] Common failure scenarios documented (or scheduled to be)?
- [ ] On-call escalation path for new service named?
- [ ] SLA / SLO target stated?
- [ ] Customer-facing status page strategy?

### 3.7 Cost & Resource

- [ ] Vendor cost estimation (per-call Firebase, per-SMS Twilio, per-MB egress)?
- [ ] Storage growth projection?
- [ ] Egress / bandwidth cost projection?
- [ ] Free-tier / quota exhaustion behavior?

### 3.8 Operational Surface Area

- [ ] New configuration surface (env vars, feature flags) documented?
- [ ] New runtime dependency (process / sidecar / cron) declared?
- [ ] New external network egress (firewall rules, allow-lists)?
- [ ] New cron / scheduled job (retry policy, monitoring)?

---

## Notes on Persona Independence (Critical)

- Run each persona pass without seeing the others' findings — this is what gives `gen-prd-checker` its adversarial value over `gen-prd` self-check.
- After all 3 passes finish, the Stage 2 synthesis de-duplicates and merges.
- When two personas raise the same issue from different angles (e.g., RD sees "no idempotency key", Security sees "replay attack possible"), keep both lenses in the synthesis but score once at the higher severity.
- Don't editorialize within a persona pass ("RD says X but Security would say Y"). Each pass is the persona's voice only.

---

## Scoring Calibration Reference

Use this to anchor scores honestly:

| Score | Description |
|-------|-------------|
| 10/10 | Industry-leading. Could ship to a top SaaS team and pass review. |
| 9/10  | Excellent. Minor polish needed. |
| 8/10  | Strong PRD with 1-2 specific gaps. (PM_39 baseline) |
| 7/10  | Solid but missing important detail in 1+ dimension. |
| 6/10  | Workable but RD will ask many clarifying questions. |
| 5/10  | Has the right shape but lots of gaps. Below "ready to hand off". |
| 4/10  | Significant rework needed before RD can start. |
| ≤3/10 | Return to `gen-prd` Stage 2/3. |

When in doubt, score lower. Inflated scores defeat the calibration purpose.
