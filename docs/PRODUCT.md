# Product Specification

This document defines **what** finbot does. See [ARCHITECTURE.md](./ARCHITECTURE.md) for **how** it's built.

## Overview

finbot is a personal finance assistant that tracks expenses through chat. Users message a bot on Telegram (or other platforms) to log spending, ask questions, and get insights.

**Core philosophy:** Awareness without anxiety. No budgets, no guilt, just patterns.

## MVP Scope

### In Scope (v1.0)

- [x] Project setup (monorepo, config, docs)
- [ ] Text expense logging ("18€ groceries")
- [ ] Smart categorization via AI
- [ ] Conversational queries ("this week", "coffee this month")
- [ ] Subscription tracking (add, list, reminders)
- [ ] Weekly digest ("vibe check")
- [ ] CSV export
- [ ] Telegram integration
- [ ] CLI setup wizard

### Out of Scope (v1.1+)

- Receipt OCR (photo parsing)
- Discord integration
- WhatsApp integration
- Firefly III sync
- Multi-currency support
- Batch logging ("coffee 3.5, lunch 12, beer 4")
- Merchant memory ("same place")
- Pattern alerts ("cigarettes 2x usual")
- Hosted/SaaS version

## Default Categories

| Category | Emoji | Keywords |
|----------|-------|----------|
| Groceries | 🛒 | supermarket, groceries |
| Restaurants | 🍽️ | restaurant, dinner, lunch, delivery, takeout |
| Coffee | ☕ | coffee, café, starbucks, costa |
| Transport | 🚗 | uber, taxi, gas, petrol, metro, bus, train |
| Subscriptions | 📄 | netflix, spotify, subscription |
| Rent | 🏠 | rent, mortgage, housing |
| Shopping | 🛍️ | amazon, clothes, shopping |
| Health | ⚕️ | pharmacy, doctor, gym, medicine |
| Travel | ✈️ | flight, hotel, airbnb, booking |
| Other | 📦 | (default fallback) |

## Database Schema

### expenses

```sql
CREATE TABLE expenses (
  id TEXT PRIMARY KEY,           -- uuid
  amount REAL NOT NULL,
  currency TEXT DEFAULT 'EUR',
  category TEXT NOT NULL,
  description TEXT,
  merchant TEXT,
  date TEXT NOT NULL,            -- ISO 8601 date: "2026-01-28"
  message_id TEXT,               -- for idempotency
  created_at TEXT DEFAULT (datetime('now')),
  updated_at TEXT DEFAULT (datetime('now'))
);

CREATE INDEX idx_expenses_date ON expenses(date);
CREATE INDEX idx_expenses_category ON expenses(category);
CREATE INDEX idx_expenses_message_id ON expenses(message_id);
```

### subscriptions

```sql
CREATE TABLE subscriptions (
  id TEXT PRIMARY KEY,           -- uuid
  name TEXT NOT NULL,
  amount REAL NOT NULL,
  currency TEXT DEFAULT 'EUR',
  frequency TEXT NOT NULL,       -- 'weekly', 'monthly', 'yearly'
  next_charge_date TEXT NOT NULL,-- ISO 8601 date
  reminder_days INTEGER DEFAULT 1,
  active INTEGER DEFAULT 1,      -- boolean
  created_at TEXT DEFAULT (datetime('now')),
  updated_at TEXT DEFAULT (datetime('now'))
);

CREATE INDEX idx_subscriptions_next_charge ON subscriptions(next_charge_date, active);
```

### settings

```sql
CREATE TABLE settings (
  key TEXT PRIMARY KEY,
  value TEXT NOT NULL,           -- JSON encoded
  updated_at TEXT DEFAULT (datetime('now'))
);
```

## Config Schema

User config lives at `~/.finbot/config.json`:

```json
{
  "version": "1.0.0",

  "telegram": {
    "botToken": "123456789:ABC...",
    "allowedUsers": [987654321]
  },

  "ai": {
    "provider": "anthropic",
    "apiKey": "sk-ant-api03-...",
    "model": "claude-sonnet-4-20250514"
  },

  "storage": {
    "type": "sqlite",
    "path": "~/.finbot/data/finbot.db"
  },

  "locale": {
    "currency": "EUR",
    "timezone": "Europe/Madrid",
    "language": "en"
  },

  "features": {
    "weeklyDigest": {
      "enabled": true,
      "dayOfWeek": "sunday",
      "time": "20:00"
    },
    "subscriptionReminders": {
      "enabled": true,
      "defaultReminderDays": 1
    }
  },

  "categories": [
    { "name": "Groceries", "emoji": "🛒", "keywords": ["mercadona", "lidl"] },
    { "name": "Coffee", "emoji": "☕", "keywords": ["coffee", "café"] }
  ]
}
```

## Intent Classification

The router classifies user messages into these intents:

| Intent | Description | Examples |
|--------|-------------|----------|
| `add-expense` | Log a new expense | "18€ groceries", "coffee 3.50", "spent 45 on dinner" |
| `query` | Ask about spending | "this week", "how much on coffee?", "restaurants last month" |
| `add-subscription` | Add recurring expense | "add netflix 12.99 monthly", "gym 25 every month" |
| `list-subscriptions` | View subscriptions | "my subscriptions", "what am I subscribed to?" |
| `summary` | Get spending overview | "vibe check", "how am I doing?", "summary" |
| `export` | Export data | "export csv", "download my data" |
| `help` | Get help | "help", "what can you do?", "commands" |
| `unknown` | Can't determine intent | (fallback, ask for clarification) |

## Bot Tone Guidelines

### Principles

1. **Observational, not judgmental** — Notice patterns, don't lecture
2. **Concise** — Short responses, no fluff
3. **Supportive** — Acknowledge without praising or criticizing
4. **Honest** — Show the data as it is

### Examples

**Good (observational):**
```
"Logged €18 · Groceries"
"€247 this week. Restaurants up from last week."
"Noticing more coffee purchases lately."
```

**Bad (judgmental):**
```
"⚠️ Budget exceeded!"
"You spent too much on restaurants"
"Great job saving money! 🎉"
```

### The "Vibe Check" Response

When user asks "vibe check" or "how am I doing?":

```
"This week: €247
Last 4 weeks avg: €198

Restaurants was the difference (+€62).
Everything else normal."
```

If everything is fine:
```
"Pretty normal week. €231 total.
You're not spiraling."
```

### Handling Missing Information

When expense is ambiguous:

```
User: "coffee"
Bot: "How much?"

User: "3.50"
Bot: "Logged €3.50 · Coffee"
```

Don't guess. Ask.

## Example Conversations

### Logging Expenses

```
User: "18€ groceries mercadona"
Bot: "Logged €18 · Groceries · Mercadona"

User: "coffee 3.50"
Bot: "Logged €3.50 · Coffee"

User: "uber 12"
Bot: "Logged €12 · Transport"
```

### Queries

```
User: "this week"
Bot: "€247 this week
     Groceries €89 · Restaurants €62 · Coffee €24 · Other €72"

User: "coffee this month"
Bot: "€42 on Coffee this month (12 purchases)"

User: "how much on restaurants?"
Bot: "€186 on Restaurants in January"
```

### Subscriptions

```
User: "add subscription netflix 12.99 monthly"
Bot: "Added Netflix €12.99/month
     Next charge: Feb 28
     I'll remind you 1 day before."

User: "my subscriptions"
Bot: "3 active subscriptions (€47.97/month):
     · Netflix €12.99 (renews Feb 28)
     · Spotify €9.99 (renews Mar 5)
     · Gym €25 (renews Mar 1)"
```

### Weekly Digest (Automated)

Sent Sunday at configured time:

```
"Weekly summary:

This week: €247
4-week avg: €198

By category:
· Groceries €89
· Restaurants €62
· Coffee €24
· Transport €18
· Other €54

Restaurants was +€40 above your usual."
```

---

## Implementation TODO

Complete these in order. Check off as you go.

### Phase 1: Foundation

- [ ] **1.1** `packages/core` — Types and interfaces (UserMessage, BotResponse, SkillResult, capabilities)
- [ ] **1.2** `packages/config` — Config loading and validation with Zod
- [ ] **1.3** `packages/storage/sqlite` — SQLite implementation with migrations

### Phase 2: AI Layer

- [ ] **2.1** `packages/ai` — AI provider abstraction (start with Anthropic)
- [ ] **2.2** Intent classification prompt
- [ ] **2.3** Expense parsing prompt

### Phase 3: First Skill + Adapter

- [ ] **3.1** `packages/adapters/telegram` — Telegram bot adapter
- [ ] **3.2** `packages/core/router` — Router with intent classification
- [ ] **3.3** `packages/skills/help` — Simple help skill (test the flow)
- [ ] **3.4** `packages/skills/add-expense` — Core expense logging skill

### Phase 4: More Skills

- [ ] **4.1** `packages/skills/query` — Query spending data
- [ ] **4.2** `packages/skills/add-subscription` — Add subscriptions
- [ ] **4.3** `packages/skills/list-subscriptions` — List subscriptions
- [ ] **4.4** `packages/skills/summary` — Vibe check / summary

### Phase 5: CLI + Polish

- [ ] **5.1** `apps/cli` — Setup wizard
- [ ] **5.2** `apps/cli` — Start/stop/status commands
- [ ] **5.3** `packages/skills/export` — CSV export
- [ ] **5.4** Weekly digest scheduler
- [ ] **5.5** Subscription reminder scheduler

### Phase 6: Release

- [ ] **6.1** Dockerfile + docker-compose
- [ ] **6.2** npm publish setup (create-finbot)
- [ ] **6.3** README final polish
- [ ] **6.4** GitHub release v1.0.0
