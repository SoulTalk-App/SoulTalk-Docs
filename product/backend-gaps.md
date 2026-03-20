# Backend Gaps & Issues

Comprehensive audit of the SoulTalk backend. Items are grouped by priority and feature area.

---

## P0 — Critical

### 1. Embedding Dimension Mismatch
`VOYAGE_EMBEDDING_DIMENSIONS=512` in config, but Voyage 3-lite returns 1024-dim vectors. Migration 019 attempted a fix but config/code may still be inconsistent. Causes vector insert failures if dimensions don't match the pgvector column.

**Files:** `app/core/config.py`, `app/services/ai/embedding_service.py`, `alembic/versions/019_*`

### 2. Rate Limiting Not Enforced
`RATE_LIMIT_PER_MINUTE: 60` exists in config but no middleware applies it. Auth endpoints (register, login, reset-password) are wide open to brute force and DOS.

**Fix:** Add SlowAPI or custom middleware. Priority on `/api/auth/login`, `/api/auth/register`, `/api/auth/reset-password`.

### 3. Admin Auth is a Plaintext Passcode
`ADMIN_PASSCODE: "soultalk-admin"` checked via `X-Admin-Token` header. No hashing, no rotation, no per-admin identity, no audit trail.

**Fix:** At minimum hash with bcrypt. Better: API key auth or OAuth with admin roles.

### 4. Timezone Bug Across Features
Daily moods, streaks, journal one-per-day checks, and affirmation date_key all use `date.today()` (UTC). The `timezone` field on `UserAIProfile` exists but is never consumed by these features. Every user effectively operates on UTC midnight resets.

**Affected:** `journal_service.has_entry_today()`, `mood_service`, `streak_service`, `affirmation_service`

---

## P1 — High

### 5. No Data Deletion / GDPR Compliance
No `DELETE /api/auth/me` endpoint. Journal entries use hard deletes with no audit trail. No data export endpoint. No retention policy.

### 6. Offset-Based Pagination Only
`GET /api/journal/` uses page/per_page (offset-based). O(n) scan for page 100+. Needs keyset cursor pagination using `(created_at, id)`.

### 7. No Error Alerting / Monitoring
Pipeline exceptions silently stored in `ai_processing_error`. No Sentry, Datadog, or CloudWatch integration. Failures only visible via admin dashboard or direct DB query.

### 8. Unbounded AI Retry Count
`ai_retry_count` (SmallInteger) has no max cap. A persistently failing entry retries on every GET request with no backoff or limit. Should cap at 3 retries.

**File:** `app/api/journal.py` (GET /{id} retry logic)

### 9. ai_processing_error Leaks Internals
Raw Python exception text stored in `ai_processing_error`. Could expose API keys, internal paths, or model names to anyone with DB read access.

**Fix:** Sanitize to generic error categories before storage.

---

## P2 — Medium

### 10. No Timeout on LLM Calls
Anthropic client calls in tagging_service, response_service, and affirmation_service have no explicit timeout. Network issues could block the pipeline indefinitely.

**Fix:** Set `timeout=30` on Anthropic client initialization.

### 11. Pipeline Steps Are Sequential
5 steps run in sequence (~3-5s total). Steps 2 (embedding) and 3 (mode selection) are independent and could run in parallel via `asyncio.gather()`.

**File:** `app/services/ai/pipeline.py`

### 12. SoulBar Points as Float
`points` column is Float. Accumulating 0.5 increments can cause floating-point precision errors (0.5 + 0.5 might not exactly equal 1.0). Reset threshold (6.0) is hardcoded.

**Fix:** Use Numeric(precision=4, scale=1) or integer cents (multiply by 10).

### 13. tone_preference Only Partially Implemented
Mode selector checks for "direct" but "softer" has no special handling. Prompt doesn't adjust language style based on preference.

**File:** `app/services/ai/mode_selector.py`

### 14. soulpal_name Not Used in Prompts
Users can set a custom SoulPal name but it's never injected into response or affirmation prompts. The AI always speaks as "SoulTalk".

### 15. main_focus No Length Limit
`main_focus` is unbounded Text. A user could set a 50KB string that gets injected into every response prompt, inflating token costs.

### 16. spiritual_metadata No Schema Validation
Accepts arbitrary JSON. No validation on structure, size, or content. Could store malicious payloads.

### 17. Social Auth profile_data Stored Unvalidated
Raw JSON from Google/Facebook stored in `social_accounts.profile_data`. No sanitization or size limit.

### 18. Password Reset Token in URL
Deep link `soultalk://reset-password/{token}` exposes token in URL. Could be logged in device history or analytics.

### 19. Username No Format Validation
No regex pattern for valid usernames. Only length check (if any). Could allow special characters, spaces, or Unicode exploits.

### 20. WebSocket Token as Plaintext First Message
JWT sent as raw text in first WS message. Should use WSS (TLS) in production. 5-second auth timeout may be too strict on slow connections.

### 21. Raw SQL in Retrieval Service
`text()` raw SQL for PostgreSQL array overlap queries. Tags are parameterized (safe), but pattern is not idiomatic SQLAlchemy and harder to maintain.

**File:** `app/services/ai/retrieval_service.py`

---

## P3 — Low

### 22. No Soft Deletes
Journal entries, moods, and user data use hard deletes. No audit trail for what was deleted or when.

### 23. Connection Pool Sizing
`pool_size=5, max_overflow=10` = 15 max connections. May bottleneck at >15 concurrent requests. Should benchmark and increase.

### 24. No Pagination Cursor for Any List Endpoint
All list endpoints (journal, admin) use offset pagination. Consider keyset for any endpoint that could return 100+ items.

### 25. Email Verification Expiry
24-hour window for email verification. Could be shortened to 1 hour for security.

### 26. No Account Lockout
No failed login attempt tracking. No temporary lockout after N failed attempts.

### 27. PII in Logs
Journal entry text may appear in debug logs (pipeline logging, error messages). No log sanitization.

### 28. No Database Partition Strategy
`journal_entries` and `entry_tags` tables grow unbounded. No time-based partitioning for archival.

---

## Test Coverage Gaps

Current test suite: 2 files (pipeline integration + Kaggle validation). No:
- Unit tests for any service
- API endpoint tests
- Auth flow tests
- Database relationship/cascade tests
- Error/edge case tests
- Concurrency tests (race conditions)
- Security tests (injection, token expiry)

See test plan in the test suite for coverage targets.

---

## Feature Completeness

| Feature | Status | Gap |
|---------|--------|-----|
| Auth (email/password) | Complete | Rate limiting, lockout |
| Auth (Google OAuth) | Complete | — |
| Auth (Facebook) | Complete | — |
| Journal CRUD | Complete | Timezone, pagination |
| AI Pipeline (5 steps) | Complete | Timeout, parallelism |
| Mode Selector | Complete | tone_preference partial |
| Crisis Detection | Complete | Localization |
| Mood Tracking | Complete | Timezone |
| Streaks | Complete | Timezone |
| SoulBar | Complete | Float precision |
| AI Profile | Complete | Field validation |
| Affirmation Mirror | Complete | Timezone, dedup, profile |
| WebSocket | Complete | Security, reconnection |
| Admin Dashboard | Complete | Auth, audit |
| Transcription | Stub (501) | Not implemented |
| Soulsight Reports | Model exists | No API endpoint |
| Daily Aggregates | Model exists | Not populated |
| Data Export | Missing | GDPR requirement |
| Search (full-text) | Missing | Only basic filters |
