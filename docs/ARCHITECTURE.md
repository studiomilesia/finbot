# Architecture

finbot is a self-hosted expense tracking bot that works through messaging platforms. This document explains how the system is structured.

## High-Level Flow

```
User sends message
       ↓
   Adapter (Telegram/Discord/etc.)
       ↓
   Normalizes to UserMessage
       ↓
   Router
       ↓
   LLM classifies intent → "add-expense" | "query" | "subscription" | ...
       ↓
   Dispatches to Skill
       ↓
   Skill executes with Capabilities (storage, ai, logger)
       ↓
   Returns SkillResult
       ↓
   Adapter sends BotResponse to user
```

## Core Concepts

### Adapters

Adapters connect finbot to messaging platforms. Each adapter:
- Receives platform-specific messages (Telegram update, Discord message, etc.)
- Converts them to a `UserMessage`
- Sends `BotResponse` back in platform-specific format

Adding a new platform = adding one adapter. All skills work automatically.

### Router

The router decides which skill handles a message:
1. Receives `UserMessage` from adapter
2. Calls LLM to classify intent
3. Looks up skill in registry
4. Calls `skill.run(ctx)`
5. Returns result to adapter

The router is the only place that decides "what does the user want?"

### Skills

Skills are isolated units of business logic. Each skill:
- Has a manifest (metadata, examples)
- Has a `run(ctx)` function
- Receives capabilities via context (no direct imports)
- Returns a `SkillResult`

Skills don't know which platform they're serving. They don't access databases directly. They use capabilities.

### Capabilities

Instead of skills importing modules directly, they receive capabilities:

| Capability | Purpose |
|------------|---------|
| `ctx.storage` | Read/write data (expenses, subscriptions) |
| `ctx.ai` | Call LLM for parsing, classification |
| `ctx.logger` | Structured logging |

Benefits:
- Easy to mock in tests
- Skills can't do unauthorized things
- Swap implementations without changing skills

### Storage

Storage is abstracted behind `StorageCap`. For self-hosted, we use SQLite. For future SaaS, we'd add PostgreSQL. Skills don't know or care which backend is used.

### AI Providers

Users can configure different LLM providers (Claude, GPT, Gemini). The `ai` package normalizes the interface so skills just call `ctx.ai.complete()`.

## Package Dependency Graph

```
apps/cli
   ├── packages/core
   ├── packages/adapters/telegram
   ├── packages/storage/sqlite
   ├── packages/ai
   └── packages/skills/*

packages/core
   └── (no internal deps, just types)

packages/skills/*
   └── packages/core (for types)

packages/adapters/*
   └── packages/core (for types)

packages/storage/*
   └── packages/core (for types)

packages/ai
   └── packages/core (for types)
```

## Invariants

These rules keep the codebase maintainable:

1. **Skills use capabilities only** — No direct DB/HTTP imports in skill code
2. **Adapters normalize messages** — Skills never see platform-specific data
3. **Router owns intent classification** — Skills don't decide if they should run
4. **Idempotency** — Same `messageId` must not create duplicate transactions
5. **`needs_input` over guessing** — If data is missing, ask; don't assume

## Adding New Things

| Want to add... | Create... |
|----------------|-----------|
| New platform | `packages/adapters/yourplatform/` |
| New skill | `packages/skills/yourskill/` |
| New LLM provider | `packages/ai/src/providers/yourprovider.ts` |
| New storage backend | `packages/storage/yourbackend/` |

See [ADDING_A_SKILL.md](./ADDING_A_SKILL.md) for a step-by-step guide.

## Folder Structure

```
finbot/
├── apps/
│   └── cli/                        # npx create-finbot wizard + runner
│       ├── index.ts
│       ├── wizard.ts
│       ├── commands/
│       └── daemon.ts
│
├── packages/
│   ├── core/                       # Router, skill runtime, types
│   │   └── src/
│   │       ├── router.ts
│   │       ├── skills/contract.ts
│   │       ├── capabilities/
│   │       └── types.ts
│   │
│   ├── skills/                     # First-party skills
│   │   ├── add-expense/
│   │   ├── query/
│   │   ├── subscription/
│   │   ├── summary/
│   │   └── help/
│   │
│   ├── adapters/
│   │   └── telegram/
│   │
│   ├── storage/
│   │   └── sqlite/
│   │
│   ├── ai/                         # LLM provider adapters
│   │   └── src/
│   │       ├── providers/
│   │       │   ├── anthropic.ts
│   │       │   ├── openai.ts
│   │       │   └── google.ts
│   │       └── prompts/
│   │
│   └── config/
│
├── docs/
├── pnpm-workspace.yaml
├── turbo.json
└── package.json
```

## User Data Location

User configuration and data live outside the project:

```
~/.finbot/
├── config.json           # User configuration
└── data/
    └── finbot.db         # SQLite database
```

## Future Additions (Not in MVP)

- `packages/adapters/discord/`
- `packages/adapters/whatsapp/`
- `packages/connectors/firefly/` — Firefly III sync
- `packages/storage/postgres/` — For SaaS version
- `apps/worker/` — Queue worker for async tasks
