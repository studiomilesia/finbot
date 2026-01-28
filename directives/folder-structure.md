# Folder Structure

This document defines the project folder structure for finbot.

```
finbot/
├── cli/
│   ├── index.ts              # npx entry point
│   ├── wizard.ts             # Setup wizard flow
│   ├── commands/             # CLI commands (start, stop, export, etc.)
│   └── validators.ts         # Token/key validation helpers
│
├── src/
│   ├── bot/
│   │   ├── index.ts          # Main bot orchestrator
│   │   ├── platforms/        # Platform adapters
│   │   │   ├── telegram.ts
│   │   │   ├── discord.ts
│   │   │   └── whatsapp.ts
│   │   └── handlers/         # Message handlers by intent
│   │       ├── expense.ts    # Logging expenses
│   │       ├── query.ts      # Querying data
│   │       ├── subscription.ts
│   │       └── help.ts
│   │
│   ├── ai/
│   │   ├── client.ts         # Claude API client
│   │   └── prompts/          # System prompts
│   │       ├── parse-expense.ts
│   │       ├── categorize.ts
│   │       └── query.ts
│   │
│   ├── storage/
│   │   ├── interface.ts      # Abstract storage interface
│   │   ├── sqlite.ts         # SQLite implementation
│   │   ├── migrations/       # Database migrations
│   │   └── queries.ts        # Common query helpers
│   │
│   └── utils/
│       ├── logger.ts
│       ├── scheduler.ts      # Weekly summaries, reminders
│       └── export.ts         # CSV export
│
├── config/
│   ├── default.ts            # Default configuration values
│   └── schema.ts             # Config validation (zod or similar)
│
├── directives/               # Project decisions and guidelines
│
├── package.json
├── tsconfig.json
├── Dockerfile
├── docker-compose.yml
├── README.md
└── LICENSE
```

## Key Decisions

### Why `cli/` is separate from `src/`
The CLI is the user-facing installer and management tool. It imports from `src/` but has its own entry point for `npx create-finbot@latest`.

### Why `platforms/` uses adapters
Each messaging platform (Telegram, Discord, WhatsApp) has different APIs. The adapter pattern lets us normalize messages into a common format that handlers can process without knowing the source.

### Why `storage/` has an interface
We start with SQLite for self-hosted. The abstract interface allows adding an API backend later for a hosted version without changing handler code.

### Why prompts are in separate files
Each AI task (parsing expenses, categorizing, handling queries) has specific prompt engineering. Separate files make them easier to iterate on and test.

## User Data Location

User configuration and data live outside the project:

```
~/.finbot/
├── config.json      # User configuration
└── data/
    └── finbot.db    # SQLite database
```
