# Decision Checklist

Use this checklist to close all decision gaps before implementation. Each item should be resolved in the docs (Product/Architecture/Development) or in a dedicated decision note. Once all boxes are checked, delete this file.

**Legend:**
- `[FOUNDATIONAL]` = Affects multiple other decisions; resolve early
- `[BLOCKING]` = Blocks implementation of a specific feature
- `[POLISH]` = Can be deferred to post-MVP if needed

---

## 1) Timezone + Locale (FOUNDATIONAL)

These decisions cascade into parsing, queries, and scheduling. Resolve first.

- [ ] Define timezone source of truth: message tz, user config tz, or server tz.
- [ ] Define how timezone is determined on first use (prompt user? infer from locale?).
- [ ] Define default locale values (currency, language) and when to prompt vs infer.
- [ ] Define DST transition handling for scheduled jobs.

---

## 2) Conversation State + Multi-Turn Interactions (FOUNDATIONAL)

When a skill returns `needs_input`, the next message must route correctly.

- [ ] Define whether conversation state is stateful (session) or stateless (re-parse with context).
- [ ] If stateful: define session timeout and storage mechanism.
- [ ] If stateless: define how follow-up messages include prior context (reply threading? message history?).
- [ ] Define behavior when user ignores clarification and sends unrelated message.
- [ ] Define maximum clarification rounds before giving up.

---

## 3) AI Behavior + Failure Strategy

- [ ] Define exact intent classification prompt format and required JSON schema.
- [ ] Define expense parsing prompt format and required JSON schema.
- [ ] Specify validation rules for AI outputs (types, ranges, required fields).
- [ ] Define retry policy: when to retry, how many times, and with what prompt adjustments.
- [ ] Define fallback behavior if AI fails after retries (`needs_input` vs rule-based parsing vs reject).
- [ ] Define user-facing error messages for AI failures (concise, non-technical).
- [ ] Define logging/metrics for AI failures (what to log, redaction rules for prompts).
- [ ] Define behavior when API key is missing, invalid, or quota exhausted.
- [ ] Define behavior when provider is rate-limited or unavailable.
- [ ] Define timeout threshold for AI calls and behavior after timeout.

---

## 4) Input Parsing Rules

Currency and amounts:
- [ ] Define currency symbol parsing (`$`, `€`, `£`) and mapping to ISO codes.
- [ ] Define ISO code parsing (`USD`, `EUR`) case sensitivity.
- [ ] Define default currency when none specified (from config).
- [ ] Define decimal separator handling (`,` vs `.`) and locale detection strategy.
- [ ] Define thousand separator handling (reject? strip? infer from locale?).
- [ ] Define amount precision (2 decimal places? round or truncate?).

Dates and times:
- [ ] Define relative date parsing (`yesterday`, `last week`, `3 days ago`).
- [ ] Define explicit date formats accepted (`2024-01-15`, `Jan 15`, `15/01`).
- [ ] Define default date when none specified (message timestamp? today in user tz?).
- [ ] Define "future date" handling (allow? warn? reject?).

Ambiguity:
- [ ] Define exact criteria for returning `needs_input` (missing category? missing amount? both?).
- [ ] Define multi-item input handling: in scope or out of scope for v1 (`coffee 3€ and lunch 12€`).

---

## 5) Category Management

- [ ] Define whether users can add custom categories (v1 or later).
- [ ] Define whether users can rename categories.
- [ ] Define whether users can delete/hide categories.
- [ ] Define behavior for expenses in a deleted/hidden category.
- [ ] Define category matching strategy (exact match? fuzzy? AI-assisted?).
- [ ] Define behavior when AI suggests a category not in the user's list.

---

## 6) Multi-Currency Handling

- [ ] Define whether multi-currency tracking is in scope for v1.
- [ ] If yes: define how summaries handle mixed currencies (group by currency? convert?).
- [ ] If converting: define exchange rate source and caching strategy.
- [ ] Define default display currency for summaries.

---

## 7) Edit + Delete Expense UX

- [ ] Define how users reference an expense to edit/delete (`delete last`, `delete the coffee one`, by ID?).
- [ ] Define confirmation flow for destructive actions.
- [ ] Define time window for edits (last 24h? forever?).
- [ ] Define what fields can be edited (amount, category, date, all?).
- [ ] Define audit trail behavior (soft delete? hard delete? keep history?).

---

## 8) Idempotency + Deduplication

- [ ] Define idempotency key per adapter (Telegram message ID, Discord message ID, etc.).
- [ ] Define dedupe window (forever? 24h? configurable?).
- [ ] Define behavior on duplicate detection (silently ignore? acknowledge?).
- [ ] Define behavior for edited messages (treat as new? update existing? ignore edit?).
- [ ] Define behavior for message resend/retry events from platform.
- [ ] Define storage constraints (unique index on message_id? composite key?).

---

## 9) Query Semantics

Date ranges:
- [ ] Define "today" boundaries (midnight to midnight in user tz).
- [ ] Define "this week" start day (Monday? Sunday? configurable?).
- [ ] Define "last week" as the 7 days prior or the previous calendar week.
- [ ] Define "this month" / "last month" boundaries.
- [ ] Define "this year" / "last year" boundaries.

Response formatting:
- [ ] Define default query scope when user gives only a category (`coffee` = this month? all time?).
- [ ] Define grouping rules (by day? by category? configurable per query?).
- [ ] Define ordering rules (newest first? oldest first? by amount?).
- [ ] Define amount formatting (always 2 decimals? currency symbol position?).
- [ ] Define response length limits (truncate at N items? paginate?).

---

## 10) Subscription Scheduling

Frequency rules:
- [ ] Define supported frequencies: weekly, monthly, yearly (others?).
- [ ] Define weekly anchor (same day of week as creation? configurable?).
- [ ] Define monthly anchor and handling for months without that day (31st → 28th?).
- [ ] Define yearly anchor and leap year handling (Feb 29 → Feb 28?).

Reminders:
- [ ] Define reminder scheduling (N days before charge? configurable per subscription?).
- [ ] Define reminder delivery time (morning in user tz? immediate?).
- [ ] Define behavior if reminder time is missed (send late? skip?).
- [ ] Define behavior if bot was offline during scheduled reminder.

Lifecycle:
- [ ] Define pause behavior (stop reminders, keep next_charge_date frozen?).
- [ ] Define cancel/delete behavior (soft delete? remove entirely?).
- [ ] Define reactivate behavior (recalculate next_charge_date from today?).

---

## 11) Concurrency + Race Conditions

- [ ] Define behavior when same user sends multiple messages simultaneously.
- [ ] Define locking strategy for idempotency checks (DB transaction? mutex?).
- [ ] Define behavior if two messages try to create the same subscription.

---

## 12) Storage + Migrations

- [ ] Define migration tool/approach (custom scripts, knex, drizzle, better-sqlite3-migrations).
- [ ] Define schema versioning strategy (version table? filename convention?).
- [ ] Define migration run trigger (on startup? CLI command?).
- [ ] Define rollback strategy (if any).
- [ ] `[POLISH]` Define data retention rules (if any).

---

## 13) Security + Access Control

- [ ] Define allowed users model (allowlist in config? first user claims bot?).
- [ ] Define behavior for unauthorized users (ignore silently? respond with rejection?).
- [ ] Define logging redaction policy (no tokens, no full messages? hash user IDs?).
- [ ] Define config file permissions guidance (600? warn if world-readable?).
- [ ] `[POLISH]` Define rate limiting for authorized users (if any).
- [ ] `[POLISH]` Define abuse protection for public-facing deployments (if relevant).

---

## 14) Data Portability

- [ ] Define CSV export format (columns, date format, encoding).
- [ ] Define whether JSON export is in scope for v1.
- [ ] `[POLISH]` Define "export all my data" flow (GDPR-style full dump).
- [ ] `[POLISH]` Define import functionality (in scope? format?).

---

## 15) CLI Wizard + Config

- [ ] Define wizard questions and order (Telegram token → AI key → locale?).
- [ ] Define validation for each wizard input (test Telegram token? test AI key?).
- [ ] Define where config is stored (`~/.finbot/config.json`? XDG spec?).
- [ ] Define config file creation (wizard creates? template + manual edit?).
- [ ] Define behavior when config is missing on `finbot start`.
- [ ] Define config migration strategy when schema changes between versions.

---

## 16) Message Formatting + Platform Constraints

- [ ] Define maximum response length (Telegram limit is 4096 chars).
- [ ] Define truncation strategy when response exceeds limit.
- [ ] Define markdown/formatting subset to use (bold, italic, code?).
- [ ] Define emoji usage policy (always? configurable? per-platform?).

---

## 17) Error Recovery + Resilience

- [ ] Define behavior when SQLite file is corrupted (detect? recover? warn user?).
- [ ] Define behavior when SQLite file is locked by another process.
- [ ] Define graceful shutdown behavior (finish in-flight requests? timeout?).
- [ ] Define startup behavior after crash (replay missed? continue fresh?).

---

## 18) Testing + Quality Gates

- [ ] Define minimum test coverage target per package.
- [ ] Define golden tests for intent classification (input → expected intent).
- [ ] Define golden tests for expense parsing (input → expected fields).
- [ ] Define test doubles for AI provider (mock responses).
- [ ] Define test doubles for storage (in-memory SQLite?).
- [ ] Define CI checks (lint, typecheck, test, build).

---

## 19) Operational Behavior

- [ ] Define logging format (JSON? plain text?) and levels (debug/info/warn/error).
- [ ] Define where logs go (stdout? file? configurable?).
- [ ] Define scheduler implementation (node-cron? custom loop? system cron?).
- [ ] Define scheduler persistence (survive restart? re-scan DB on startup?).
- [ ] Define error reporting for background jobs (log only? notify user?).
- [ ] `[POLISH]` Define health check or status command output.

---

## Decision Dependencies Graph

```
Timezone/Locale (§1)
    ├── Input Parsing: Dates (§4)
    ├── Query Semantics: Date Ranges (§9)
    └── Subscription Scheduling (§10)

Conversation State (§2)
    └── AI Behavior: needs_input flow (§3)

Idempotency (§8)
    └── Concurrency (§11)

Category Management (§5)
    └── AI Behavior: category suggestions (§3)
```

---

## Suggested Resolution Order

1. **§1 Timezone + Locale** — Foundational, affects 3+ other sections
2. **§2 Conversation State** — Architectural decision, affects routing design
3. **§3 AI Behavior** — Core functionality, many implementation details
4. **§4 Input Parsing** — Depends on §1
5. **§8 Idempotency** — Critical for correctness
6. **§5 Category Management** — Affects AI prompts
7. **§9 Query Semantics** — Depends on §1
8. **§10 Subscription Scheduling** — Depends on §1
9. **§7 Edit/Delete UX** — User-facing decisions
10. **§6 Multi-Currency** — Scope decision (can defer)
11. **Everything else** — Lower priority, can resolve as encountered
