# Affirmation Mirror — Implementation Spec

## Overview

Daily personalized affirmation feature. One affirmation per user per day, generated from their journal history using SoulTalk's coaching voice. Resets at midnight in the user's timezone.

---

## User Flow

1. User taps the Affirmation Mirror card on the Home screen
2. Screen loads with mirror visual and a "Click to Reveal" button
3. User taps the button — affirmation text fades in
4. If the affirmation was AI-generated from journal data, a "Personalized for you" badge appears
5. Returning later the same day shows the same affirmation directly (no button)
6. At midnight (user's timezone), the cycle resets — new affirmation available behind the reveal button

---

## Backend

### Database

**Table: `daily_affirmations`** (migration 021)

| Column | Type | Notes |
|--------|------|-------|
| id | UUID | PK |
| user_id | UUID | FK → users, CASCADE |
| date_key | Date | User's local date |
| timezone | String(50) | IANA timezone used |
| affirmation_text | Text | The affirmation |
| source | String(20) | `ai` or `fallback` |
| model_version | String(50) | Nullable, e.g. `claude-haiku-4-5-20251001` |
| generated_at | DateTime(tz) | When generated |

Unique constraint: `(user_id, date_key)` — enforces one per user per day and handles concurrent requests.

**Column added: `user_ai_profiles.timezone`** (migration 022)
- String(50), default "UTC"
- Used to resolve the user's local date for daily reset

### API

**`GET /api/affirmation-mirror/today`** (authenticated)

Response:
```json
{
  "date_key": "2026-03-20",
  "affirmation_text": "I do not need to prove my worth through pressure...",
  "is_revealed_allowed": true,
  "source": "ai",
  "next_reset_time": "2026-03-21T00:00:00+05:30"
}
```

Logic:
1. Resolve user's current date from their timezone setting
2. If affirmation exists for today → return it
3. If not → generate (or fallback) → store → return

### Generation

- **Model**: Haiku (cheapest, fast for 1-3 sentence output)
- **Input**: Recent journal context from `retrieval_service.build_recent_context()` (last 7 entries' emotions, topics, NS state, coping patterns)
- **Output**: First-person "I" statement, 1-3 sentences, grounded SoulTalk voice
- **Fallback**: 20 curated affirmations in SoulTalk style, used when:
  - User has no journal entries
  - LLM call fails
- Fallbacks are stored as the daily record so the experience is consistent

### Prompt Design

Three-part prompt (system/developer/user):
- First-person "I" statements only
- Grounded inner mirror tone — not clichés, not hype
- Reflects user's emotional tone, themes, nervous system state
- Safety: no self-harm, no medical advice, no diagnosis
- Good examples from spec baked into system prompt

### Concurrency

Unique constraint on `(user_id, date_key)`. If a race condition causes a duplicate insert, the service catches the constraint violation, rolls back, and returns the existing row.

---

## Mobile

### Files

| File | Change |
|------|--------|
| `src/screens/AffirmationMirrorScreen.tsx` | New screen |
| `src/services/JournalService.ts` | Added `getTodayAffirmation()` |
| `App.tsx` | Registered screen in AppStack |
| `src/screens/HomeScreen.tsx` | Removed "Coming Soon" overlay, added navigation |

### Reveal Persistence

- On reveal: saves `date_key` to AsyncStorage (`@soultalk_affirmation_revealed_date`)
- On screen load: if stored date matches today's `date_key`, skips the button and shows affirmation directly
- At midnight: new `date_key` from API won't match stored value → button reappears

### Option A (v1)

Frontend calls `GET /affirmation-mirror/today` on page load. Server returns `affirmation_text` but the frontend hides it behind the reveal button until tapped. No server-side reveal tracking.

---

## Known Gaps & Improvements

### High Priority

**1. Timezone is never set**
The `timezone` field exists on `user_ai_profiles` but there is no mobile UI or auto-detection to populate it. Every user defaults to UTC, which means the midnight reset is wrong for non-UTC users. Fix: auto-detect timezone from the device (`Intl.DateTimeFormat` or `expo-localization`) and sync to the backend on app launch or profile update.

**2. Prompt doesn't use the user's profile**
The generation prompt only receives `recent_context` (aggregated journal patterns). It ignores `main_focus` and `tone_preference` from `user_ai_profiles`. These fields are already in the database — passing them to the prompt would make affirmations noticeably more relevant (e.g., someone focused on relationships gets a different affirmation than someone focused on career).

**3. Fallback repetition**
The 20 hardcoded fallbacks are selected via `random.choice()` with no awareness of what the user received on previous days. A user with no journal entries could see the same affirmation twice in a week. Fix: query the last N `daily_affirmations` for the user and exclude those texts before picking.

### Medium Priority

**4. SoulBar points not wired**
Product bible specifies +0.5 SoulBar points per affirmation draw. The reveal action currently awards nothing. Needs: call `soul_bar_service` on first reveal of the day (can key off the same `(user_id, date_key)` to prevent double-awarding).

**5. Context is statistical, not personal**
`build_recent_context()` returns aggregated patterns ("Dominant emotions: anxiety, sadness"). Including a one-line excerpt from the most recent journal entry would make the affirmation feel like it's responding to something the user actually said, not just a trend summary.

**6. Minimum entry threshold**
Personalized affirmations start after the first journal entry, but one entry gives very thin context. Consider requiring 3+ entries before switching from fallback to AI generation, or adjusting the prompt to acknowledge limited context.

### Low Priority

**7. Offline/cache handling**
If the user opens the screen with no internet, they see an error message. Could cache the last fetched affirmation locally and show it as a fallback — still better than an empty screen.

**8. Cloud disperse animation**
The spec mentions a "cloud disperse animation" designed by Maul. Current implementation uses a fade-in + scale reveal. Depends on receiving cloud/scene assets from design.

**9. Reveal state is client-side only**
Revealed state is stored in AsyncStorage. If a user switches devices mid-day, they see the reveal button again on the new device (though tapping it returns the same affirmation). Not a broken experience, but inconsistent. Could track reveal server-side in a future version (Option B from spec).

---

## Out of Scope (v1)

- Browsing old affirmations / history feed
- Manual refresh or multiple reveals per day
- User theme controls
- A/B testing
- SoulBar points (+0.5 per affirmation draw — listed as gap above)
- Heart/favorite functionality (future)
