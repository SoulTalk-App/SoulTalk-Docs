# SoulTalk
## Task Breakdown & Time Estimates — Sprint 2

---

### Backend — AI Pipeline v2

| # | Task | Description | Hrs |
|---|------|-------------|-----|
| 1 | Anthropic migration | Replace OpenAI with Anthropic across all AI services. Remove Whisper transcription dependency. Update config, environment variables, and requirements. | 1.5 |
| 2 | AI pipeline database layer | Design and implement 6 new SQLAlchemy models (EntryTags, AIResponse, SoulSight, ScenarioPlaybook, DailyAggregate, UserAIProfile). Set up pgvector extension. Write 8 Alembic migrations (008–015). Refactor journal_entry model for new relationships. | 4.0 |
| 3 | Tags schema & config | Build TagsV1 Pydantic schema with all 12 tag sections and safety enforcement. Author 20 scenario playbook definitions in JSON. Write seed migration 016 for playbook data. | 2.0 |
| 4 | AI pipeline core services | Implement 8 service modules from scratch: tagging service (Haiku extraction with JSON retry), embedding service (Voyage AI integration), deterministic mode selector (7 modes, 4 hint types), retrieval service (similar entries via pgvector, scenario matching, recent context), response service (Sonnet coaching generation with crisis bypass), safety module (tag validation, crisis detection), pipeline orchestrator, and prompt templates for tagging, response, and soulsight. | 8.0 |
| 5 | API integration | Wire journal endpoints to AI pipeline with selectinload joins for tags and AI response. Add WebSocket push on pipeline completion. Implement auto re-trigger for failed/stuck entries (>5 min). Build new AI profile endpoints (GET/PUT). Update journal schemas with nested TagsSummary, AIResponseSummary, and ai_processing_status enum. | 3.0 |
| 6 | Pipeline calibration | Tighten mode selector thresholds based on test results. Fix retrieval service SQL queries. Clean up and improve tag normalizer logic. Iterate on prompt templates for accuracy. | 2.0 |

| | | **AI Pipeline Subtotal** | **20.5** |
|---|---|---|---|

### Backend — Admin Dashboard

| # | Task | Description | Hrs |
|---|------|-------------|-----|
| 7 | Admin dashboard UI | Build full HTML/JS admin panel (~1,358 lines) with pipeline debugger (run tagging/embedding/response steps individually), config inspector, tag viewer, and passcode-protected access. | 4.0 |
| 8 | Config service | Implement runtime prompt management with versioning (~400 lines). Add DB-backed AIConfig and PromptVersion models. Write migration 017. Support hot-swapping prompts without redeployment. | 3.0 |
| 9 | API usage tracking | Build per-API-call cost estimation and logging system. Add ApiUsageLog model and migration 018. Integrate token counting into tagging and response services. Add usage analytics tab to admin UI. | 2.0 |

| | | **Admin Dashboard Subtotal** | **9.0** |
|---|---|---|---|

### Backend — Testing

| # | Task | Description | Hrs |
|---|------|-------------|-----|
| 10 | Pipeline test infrastructure | Write 514-line pipeline test suite covering all pipeline stages. Build 253-line Kaggle emotion validation test for tagging accuracy. Create 495-line mock persona dataset with diverse journal scenarios. Implement emotion category mapping. | 3.0 |

| | | **Testing Subtotal** | **3.0** |
|---|---|---|---|

### Backend — Production Fixes

| # | Task | Description | Hrs |
|---|------|-------------|-----|
| 11 | Fix 500 on journal create | Diagnose MissingGreenlet error caused by async lazy-loading of unloaded relationships. Refactor create endpoint to return response directly without accessing entry_tags/ai_response. | 0.5 |
| 12 | Fix 500 on journal update | Same MissingGreenlet pattern on PUT endpoint. Refactor update endpoint to construct response directly with tags=None, ai_response=None. | 0.5 |
| 13 | Fix pipeline re-trigger | Diagnose UniqueViolationError when re-processing failed entries. Add stale EntryTags/AIResponse deletion before re-insert. Test idempotent re-trigger flow. | 1.0 |
| 14 | Fix embedding dimensions | Trace pipeline failure to dimension mismatch — voyage-3-lite outputs 512-dim vectors but DB column was Vector(1024). Update config, model, and create migration 019 to alter column. | 1.0 |
| 15 | Fix tagging validation failures | Diagnose pipeline crash caused by LLM returning coping mechanism values outside the strict Literal enum (e.g. "reflection"). Add comprehensive normalization with alias mapping and fallbacks for coping mechanisms and topics. | 1.0 |
| 16 | Comprehensive enum normalization | Extend normalization to all 18 Literal enum fields in TagsV1 schema. Add 300+ alias mappings across emotions, nervous system state, somatic cues, topics, coping mechanisms, coping function, self-talk style, cognitive distortions, behavioral loops, orientation, risk levels, and confidence scores. Ensure no LLM output can crash Pydantic validation. Add all alias defaults to config service for runtime editability. | 2.0 |
| 17 | Admin panel — normalization aliases | Expand admin dashboard Aliases tab from single emotion map to 10 collapsible alias categories (emotions, nervous system, somatic cues, topics, coping mechanisms, coping functions, self-talk, cognitive distortions, behavioral loops, risk levels). Each with mapping table, count badge, source indicator, editable JSON, save/reset controls. | 1.0 |
| 18 | AI retry counter & credit protection | Add ai_retry_count column to journal_entries (migration 020). Cap auto-retrigger at 3 attempts in GET endpoint. Increment counter in pipeline orchestrator. Prevent infinite credit burn on persistently failing entries. | 1.0 |

| | | **Production Fixes Subtotal** | **8.0** |
|---|---|---|---|

### Backend — Infrastructure & DevOps

| # | Task | Description | Hrs |
|---|------|-------------|-----|
| 19 | Production deployment | SSH to EC2, Docker image rebuild (resolve restart vs build issue), disk space cleanup (docker system prune), multiple deploy-test-fix cycles across several days. | 3.0 |
| 20 | Branch management | Cherry-pick fixes from feat/testing-dataset to main. Resolve merge conflicts for files not on main (admin.py, admin.html). Manual file checkout strategy. Git pull --rebase to sync remote. | 1.0 |
| 21 | Production verification | Verify end-to-end pipeline with real user journal entries. Analyse server logs across multiple debug iterations. Confirm AI response quality and delivery via WebSocket. | 1.5 |

| | | **Infrastructure Subtotal** | **5.5** |
|---|---|---|---|

### Backend — Audit

| # | Task | Description | Hrs |
|---|------|-------------|-----|
| 22 | Full backend security audit | Comprehensive 4-area audit covering API security (auth, input validation, rate limiting), database layer (models, migrations, query patterns), AI pipeline (prompt injection, crisis handling, cost controls), and scalability (concurrency, caching, resource limits). Detailed findings and recommendations report. | 2.0 |

| | | **Audit Subtotal** | **2.0** |
|---|---|---|---|

---

### Mobile

| # | Task | Description | Hrs |
|---|------|-------------|-----|
| 23 | JournalService interface alignment | Rewrite JournalEntry TypeScript interface from flat fields to nested structure (TagsSummary, AIResponseSummary). Migrate is_ai_processed boolean to ai_processing_status enum. Add getAIPreferences() and updateAIPreferences() service methods. | 1.5 |
| 24 | JournalEntryScreen rewrite | Update screen for nested response data (entry.ai_response?.text, entry.tags?.topics, entry.tags?.coping_mechanisms). Add full entry detail fetch on screen open (list endpoint omits nested data). Implement polling fallback for AI completion. Add failed state UI. | 2.0 |
| 25 | JournalScreen filter update | Update filter parameters and status indicators from is_ai_processed to ai_processing_status === 'complete'. | 0.5 |
| 26 | WebSocket handler alignment | Update JournalContext to transform flat WebSocket payload (response_text, mode, tags_summary) into nested JournalEntry structure for state consistency. | 1.0 |
| 27 | SoulPal name backend sync | Add backend sync call to updateAIPreferences() after saving SoulPal name to AsyncStorage. Best-effort with silent failure. | 0.5 |
| 28 | TestFlight deployments (×3) | Three full iOS deploy cycles: expo prebuild → xcodebuild archive → export IPA → xcrun altool upload to TestFlight. Troubleshoot build issues between cycles. | 1.5 |
| 29 | CEO feedback — UI changes | Implement 7 CEO feedback items: make inspiration dropdown non-clickable with guidance text, add live voice transcription to journal text box, update daily limit message, change post-submit routing to analysis screen, reconfigure home screen (rename SoulSight, remove goals card, remove mood bar), increase journal font size, fix splash screen fallback timer. | 3.0 |
| 30 | Returning user onboarding fix | Fix bug where returning users on new devices were shown username/SoulPal setup screens again. Auto-detect existing username from server and skip onboarding + setup flows. | 0.5 |

| | | **Mobile Subtotal** | **11.5** |
|---|---|---|---|

---

### Consulting

| # | Task | Description | Hrs |
|---|------|-------------|-----|
| 31 | AI cost analysis | Per-entry cost breakdown across Haiku tagging (~$0.009), Voyage embedding (~$0.00001), and Sonnet response (~$0.011). Projection to 1,000 DAU with monthly and annual estimates. | 0.5 |
| 32 | Privacy policy review | Technical audit of privacy policy against actual implementation. Identified 6 inaccuracies (raw text retention claim, HIPAA language, missing account deletion endpoint, undisclosed AI processors). Recommendations for compliance. | 0.5 |

| | | **Consulting Subtotal** | **1.0** |
|---|---|---|---|

---

### Summary

| Category | Hours |
|----------|-------|
| Backend — AI Pipeline v2 | 20.5 |
| Backend — Admin Dashboard | 9.0 |
| Backend — Testing | 3.0 |
| Backend — Production Fixes | 8.0 |
| Backend — Infrastructure & DevOps | 5.5 |
| Backend — Audit | 2.0 |
| Mobile | 11.5 |
| Consulting | 1.0 |
| **Total Estimated Hours** | **~61** |

---

### Backend — CEO Feedback & Safety (Mar 20)

| # | Task | Description | Hrs |
|---|------|-------------|-----|
| 33 | Crisis response overhaul | Replace hardcoded 3-line safety redirect with LLM-generated brief reflection + specific crisis resources (988 Lifeline, Crisis Text Line 741741, IASP). Update response_service to run crisis-aware prompt instead of full skip. | 2.0 |
| 34 | Anti-AI-ism prompt rules | Add rules to developer message: no em dashes, no "it's not X — it's Y" construction, no asterisks/markdown, short direct sentences. Update config_service defaults to match. | 1.0 |

| | | **CEO Feedback Subtotal** | **3.0** |
|---|---|---|---|

### Backend — Unit Testing (Mar 20–23)

| # | Task | Description | Hrs |
|---|------|-------------|-----|
| 35 | Unit test suite — round 1 | 181 unit tests covering mode selector, safety module, JWT auth, password validation, tags schema, and config service. | 3.0 |
| 36 | Unit test suite — round 2 | Expand to 480 total tests reaching 81% code coverage. Cover notification service, API endpoints, pipeline orchestrator, tagging service, response service, embedding service, retrieval service. | 4.0 |

| | | **Testing Subtotal** | **7.0** |
|---|---|---|---|

### Backend — Push Notifications (Mar 24)

| # | Task | Description | Hrs |
|---|------|-------------|-----|
| 37 | Push notification service | PushToken model, Alembic migration, NotificationService with Expo Push API integration. Token register/unregister/deactivate endpoints. DeviceNotRegistered auto-deactivation. | 3.0 |

| | | **Push Notifications Subtotal** | **3.0** |
|---|---|---|---|

### Backend — Affirmation Pipeline Upgrade (Mar 26)

| # | Task | Description | Hrs |
|---|------|-------------|-----|
| 46 | Affirmation model upgrade | Upgrade from Haiku to Sonnet for higher quality affirmations. Add anti-AI-ism rules to prompts (no em dashes, no markdown, no rhetorical devices). Fix fallback affirmations to match style rules. | 1.0 |
| 47 | Remove fallback affirmations | Comment out 20 static fallbacks. Return error on failure (503 with message) instead of generic affirmation. Distinct messages for no-journal-entries vs LLM-failure. | 0.5 |
| 48 | Admin panel affirmation support | Add runtime config for model, temperature, max_tokens via config service. Add GET /api/affirmations endpoint to view all generated affirmations. Add POST /api/pipeline/affirmation debugger endpoint. | 1.0 |

| | | **Affirmation Pipeline Subtotal** | **2.5** |
|---|---|---|---|

---

### Mobile — CEO Feedback (Mar 18–20)

| # | Task | Description | Hrs |
|---|------|-------------|-----|
| 38 | CEO feedback v1 PR | UI copy changes, daily limit message update, home screen adjustments. | 1.0 |
| 39 | Rename SoulPal's Reflection | Change "SoulPal's Reflection" → "SoulTalk Reflection", loading text → "Preparing your reflection...", strip markdown asterisks from AI text, "Save" → "Submit" button. | 1.0 |

| | | **CEO Feedback Subtotal** | **2.0** |
|---|---|---|---|

### Mobile — Affirmation Mirror Redesign (Mar 20–26)

| # | Task | Description | Hrs |
|---|------|-------------|-----|
| 40 | Affirmation Mirror screen | Full redesign: two looping video backgrounds (idle + revealed) with expo-video, three cloud PNG overlay layers, reveal animation sequence (button fade, clouds fade, video crossfade, text fade-in). Glass-morphism "Click to Reveal" button with LinearGradient. Back button as inline SVG matching Figma pink color. | 4.0 |
| 41 | Custom screen transition | Disable default React Navigation transition, implement custom entry animation (video slides from right via Reanimated translateX, clouds appear instantly). | 0.5 |
| 42 | Homepage card update | Replace three-image composite card with single Figma design asset. Update border radius to 20px. Remove unused old mirror assets. | 0.5 |
| 43 | Asset compression | Compress video assets from 16MB to 871KB each (1200x1200, 30fps, CRF 23). Compress card image from 921KB to 49KB. Total assets reduced from 37MB to 5.3MB. | 0.5 |
| 44 | Color alignment | Update accent.pink from #EA3678 to #E93678 to match Figma exactly. | 0.25 |

| | | **Affirmation Mirror Subtotal** | **5.75** |
|---|---|---|---|

### Mobile — Push Notifications (Mar 24)

| # | Task | Description | Hrs |
|---|------|-------------|-----|
| 45 | Expo Notifications integration | expo-notifications setup, permission request flow, token registration on login, deregistration on logout, notification listeners for foreground/background. | 2.0 |

| | | **Push Notifications Subtotal** | **2.0** |
|---|---|---|---|

---

### Updated Summary

| Category | Hours |
|----------|-------|
| Backend — AI Pipeline v2 | 20.5 |
| Backend — Admin Dashboard | 9.0 |
| Backend — Testing (pipeline + unit) | 10.0 |
| Backend — Production Fixes | 8.0 |
| Backend — Infrastructure & DevOps | 5.5 |
| Backend — Audit | 2.0 |
| Backend — CEO Feedback & Safety | 3.0 |
| Backend — Push Notifications | 3.0 |
| Backend — Affirmation Pipeline Upgrade | 2.5 |
| Mobile (Sprint 2 original) | 11.5 |
| Mobile — CEO Feedback | 2.0 |
| Mobile — Affirmation Mirror Redesign | 5.75 |
| Mobile — Push Notifications | 2.0 |
| Consulting | 1.0 |
| **Total Estimated Hours** | **~85.5** |

---

*Note: Estimates assume a senior developer with AI-assisted tooling. Sprint covered Mar 6–26, 2026. Codebase delta: ~10,000+ lines added across 60+ files with 14 Alembic migrations.*

---

## Appendix: AI Pipeline Comparison — Journal vs Affirmation

### Similarities

| Aspect | Detail |
|--------|--------|
| Provider | Anthropic Claude API |
| Context source | `build_recent_context()` — aggregates last 7 journal entry tags |
| Prompt structure | System + Developer + User (3-part) |
| Usage tracking | `record_usage()` after each LLM call |
| Config service | Runtime-overridable prompts and models via admin panel |
| Safety constraints | No diagnosis, no medical advice, no clinical language |
| Anti-AI-ism rules | No em dashes, no markdown, no "not X, but Y" constructions, plain text only |
| Admin panel | Model, temperature, max tokens, prompts all runtime-configurable |

### Differences

| Aspect | Journal Pipeline | Affirmation Pipeline |
|--------|-----------------|---------------------|
| Steps | 6 (tag → embed → mode → retrieve → respond → complete) | 1 (context → generate) |
| Model | Haiku (tagging) + Sonnet (response) | Sonnet only |
| Temperature | 0.0 (tagging) / 0.7 (response) | 0.9 (creative) |
| Max tokens | 1500 (tagging) / 500 (response) | 300 |
| Embedding | Voyage AI vector stored per entry | None |
| Mode selection | 7 deterministic modes (crisis, soft landing, clean mirror, etc.) | None |
| Scenario matching | pgvector similarity + playbook matching | None |
| Tagging | Full 12-section TagsV1 extraction | None (reads existing tags) |
| Crisis handling | Full safety module, crisis override mode, safety redirect | None (relies on journal having handled it) |
| Trigger | Background task after journal save | On-demand GET request |
| Caching | One response per journal entry | One per user per calendar day (timezone-aware) |
| Output | 120-220 word coaching reflection | 1-3 sentence first-person affirmation |
| Cost per call | ~$0.02 (Haiku + Sonnet + Voyage) | ~$0.01 (Sonnet only) |
| Fallback | Safety redirect with crisis resources | 503 error (no static fallbacks) |

### Architecture

The journal pipeline is the heavy workhorse: 6 sequential steps, 2 models, vector search, mode selection, and scenario matching. It produces a personalized coaching reflection.

The affirmation pipeline is lightweight and parasitic on journal data. It reuses `build_recent_context()` which reads existing `entry_tags` (created by the journal pipeline). One Sonnet call turns aggregated patterns into a first-person affirmation. No tagging, no embedding, no mode selection.

The retrieval service is the bridge: journal entries get tagged and embedded, and the affirmation service reads those tags to build context without additional AI calls.
