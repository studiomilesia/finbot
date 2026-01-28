# Decision Checklist

Use this checklist to close all decision gaps before implementation. Each item should be resolved in the docs (Product/Architecture/Development) or in a dedicated decision note. Once all boxes are checked, delete this file.

---

## 1) AI Behavior + Failure Strategy
- [ ] Define exact intent classification prompt format and required JSON schema.
- [ ] Define expense parsing prompt format and required JSON schema.
- [ ] Specify validation rules for AI outputs (types, ranges, required fields).
- [ ] Define retry policy: when to retry, how many times, and with what prompt adjustments.
- [ ] Define fallback behavior if AI fails (needs_input vs rule-based parsing vs reject).
- [ ] Define user-facing responses for AI errors (concise, non-technical).
- [ ] Define logging/metrics for AI failures (what to log, redaction rules).
- [ ] Define behavior when API key is missing, invalid, or out of credits.
- [ ] Define behavior when provider is rate-limited or unavailable.
- [ ] Define timeout thresholds for AI calls and what happens after timeout.

## 2) Input Parsing Rules (Non-AI + Edge Cases)
- [ ] Currency parsing rules (symbols, ISO codes, defaults).
- [ ] Decimal separators ("," vs ".") and locale handling.
- [ ] Date parsing rules ("yesterday", "last week", explicit dates).
- [ ] Timezone source of truth (message tz, config tz, server tz).
- [ ] Ambiguous messages: exact criteria for `needs_input`.
- [ ] Multi-item inputs (explicitly in or out of scope for v1).

## 3) Idempotency + Dedupe
- [ ] Define idempotency keys per adapter (message ID vs hash).
- [ ] Define dedupe window and behavior on duplicate detection.
- [ ] Define behavior for edited messages or resend events.
- [ ] Define storage constraints (unique indexes, failure handling).

## 4) Query Semantics
- [ ] Define "this week" / "last week" range boundaries.
- [ ] Define "this month" / "last month" range boundaries.
- [ ] Define "this year" / "last year" range boundaries.
- [ ] Define default query scope when user gives only a category.
- [ ] Define grouping and ordering rules in responses.
- [ ] Define rounding/formatting rules for amounts.

## 5) Subscription Scheduling
- [ ] Define supported frequencies (weekly/monthly/yearly) and exact rules.
- [ ] Define "monthly on 31st" handling for short months.
- [ ] Define timezone for next_charge_date calculations.
- [ ] Define reminder scheduling rules (e.g., reminder_days = 1).
- [ ] Define behavior if reminder time is missed (catch-up vs skip).
- [ ] Define pause/cancel/reactivate logic for subscriptions.

## 6) Storage + Migrations
- [ ] Define migration tool/approach (custom, knex, drizzle, etc.).
- [ ] Define schema versioning strategy.
- [ ] Define backup/restore strategy (if any).
- [ ] Define data retention rules (if any).

## 7) Security + Access Control
- [ ] Define allowed users model and behavior for unauthorized users.
- [ ] Define logging redaction policy (tokens, PII).
- [ ] Define rate limiting or abuse protection (if any).
- [ ] Define config file permissions guidance.

## 8) CLI Wizard + Config
- [ ] Define wizard questions and validation.
- [ ] Define default config values (locale, currency, timezone).
- [ ] Define config migration/update strategy.
- [ ] Define where config is stored and created.

## 9) Testing + Quality Gates
- [ ] Define minimum test coverage for each package.
- [ ] Define golden tests for parsing and intent classification.
- [ ] Define test doubles for AI and storage.
- [ ] Define CI checks (lint/typecheck/test).

## 10) Operational Behavior
- [ ] Define logging format/levels and where logs go.
- [ ] Define health checks or status command behavior.
- [ ] Define scheduler lifecycle (start/stop, persistence).
- [ ] Define error reporting for background jobs.

