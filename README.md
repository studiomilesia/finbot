# finbot

**AI-powered expense tracking through chat.**

Message a bot on Telegram to log spending, ask about your habits, and get weekly summaries. Self-hosted — your data stays on your machine.

## Features

- **Natural language logging** — "45 groceries", "coffee 3.50", "uber 12"
- **Smart categorization** — Automatically detects categories from your message
- **Conversational queries** — "This month", "Restaurants last week", "How bad is it?"
- **Subscription tracking** — Add recurring expenses, get reminders before charges
- **Receipt scanning** — Send a photo, get it parsed (coming soon)
- **Weekly digest** — Configurable summary with pattern detection
- **CSV export** — Your data, your format
- **Multi-platform** — Telegram now, Discord and WhatsApp coming

## Quick Start

```bash
npx create-finbot@latest
```

The setup wizard will guide you through:
1. Creating a Telegram bot
2. Adding your Claude API key
3. Configuring your preferences

Then just message your bot and start tracking.

## How It Works

```
You: "18€ groceries"
Bot: "Logged €18 · Groceries"

You: "this week"
Bot: "€247 this week
     Groceries €89 · Restaurants €62 · Coffee €24 · Other €72"

You: "vibe check"
Bot: "Pretty normal. Restaurants slightly up from last month.
     You're not spiraling."
```

## Self-Hosted

finbot runs entirely on your machine:

- **SQLite database** — Your expenses stay local
- **Your API keys** — You control access to Claude
- **No account needed** — No signup, no cloud dependency
- **Open source** — MIT licensed, audit the code yourself

## Requirements

- Node.js 18+
- A Telegram account (to create a bot)
- A Claude API key from [Anthropic](https://console.anthropic.com)

## Configuration

After setup, your config lives in `~/.finbot/config.json`. You can customize:

- Categories and keywords
- Weekly summary timing
- Subscription reminder preferences
- Notification settings

## Roadmap

- [ ] Telegram integration
- [ ] Expense logging and queries
- [ ] Subscription tracking
- [ ] Weekly digest
- [ ] CSV export
- [ ] Receipt OCR
- [ ] Discord integration
- [ ] WhatsApp integration
- [ ] Firefly III sync
- [ ] Hosted version (optional, for those who don't want to self-host)

## License

MIT