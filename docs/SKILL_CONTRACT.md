# Skill Contract

This document defines the TypeScript interfaces for skills. These types live in `packages/core/src/skills/contract.ts`.

## Core Types

### UserMessage

The normalized message from any platform.

```ts
export interface UserMessage {
  /** Which platform this came from */
  channel: "telegram" | "discord" | "whatsapp" | "cli";

  /** Unique user identifier (platform-specific) */
  userId: string;

  /** The message text */
  text: string;

  /** User's locale, e.g. "en", "es" */
  locale?: string;

  /** User's timezone, e.g. "Europe/Madrid" */
  timezone?: string;

  /** Platform message ID (for idempotency) */
  messageId?: string;

  /** When the message was sent */
  timestampIso?: string;

  /** Attached image (for receipt scanning) */
  image?: {
    url?: string;
    base64?: string;
    mimeType: string;
  };
}
```

### BotResponse

What the bot sends back to the user.

```ts
export interface BotResponse {
  /** Main response text */
  text: string;

  /** Rich blocks for platforms that support them */
  blocks?: Array<
    | { type: "markdown"; markdown: string }
    | { type: "list"; items: string[] }
    | { type: "confirm"; prompt: string; confirmText?: string; cancelText?: string }
  >;

  /** Suggested follow-up actions */
  followUps?: string[];
}
```

### SkillResult

What a skill returns after execution.

```ts
export type SkillStatus = "ok" | "needs_input" | "rejected" | "error";

export interface SkillResult {
  /** Execution status */
  status: SkillStatus;

  /** Response to send to user */
  response: BotResponse;

  /** If needs_input, what we're waiting for */
  required?: Array<{ key: string; prompt: string }>;

  /** Structured data for chaining/logging (not user-facing) */
  data?: Record<string, unknown>;
}
```

## Capabilities

Skills receive capabilities via context instead of importing modules directly.

### StorageCap

```ts
export interface StorageCap {
  // Expenses
  saveExpense(expense: {
    amount: number;
    currency: string;
    category: string;
    description?: string;
    merchant?: string;
    date: string; // ISO date
  }): Promise<{ id: string }>;

  getExpenses(filters: {
    from?: string;
    to?: string;
    category?: string;
    limit?: number;
  }): Promise<Expense[]>;

  // Subscriptions
  saveSubscription(sub: {
    name: string;
    amount: number;
    currency: string;
    frequency: "weekly" | "monthly" | "yearly";
    nextChargeDate: string;
    reminderDays?: number;
  }): Promise<{ id: string }>;

  getSubscriptions(filters?: {
    active?: boolean;
  }): Promise<Subscription[]>;

  // Generic key-value (for skill state)
  get<T>(key: string): Promise<T | null>;
  set<T>(key: string, value: T): Promise<void>;
}
```

### AICap

```ts
export interface AICap {
  /** General completion */
  complete(prompt: string, options?: {
    system?: string;
    temperature?: number;
    maxTokens?: number;
  }): Promise<string>;

  /** Structured output */
  parseJson<T>(prompt: string, schema: string): Promise<T>;

  /** Image analysis (for receipts) */
  analyzeImage(image: { base64: string; mimeType: string }, prompt: string): Promise<string>;
}
```

### LoggerCap

```ts
export interface LoggerCap {
  info(msg: string, meta?: Record<string, unknown>): void;
  warn(msg: string, meta?: Record<string, unknown>): void;
  error(msg: string, meta?: Record<string, unknown>): void;
  debug(msg: string, meta?: Record<string, unknown>): void;
}
```

### SkillContext

The context passed to every skill.

```ts
export interface SkillContext {
  /** The incoming message */
  message: UserMessage;

  /** Storage capability */
  storage: StorageCap;

  /** AI capability */
  ai: AICap;

  /** Logger capability */
  logger: LoggerCap;

  /** For idempotency checks */
  idempotencyKey?: string;
}
```

## Skill Interface

```ts
export type SkillId = string;

export interface SkillManifest {
  /** Unique skill identifier */
  id: SkillId;

  /** Semver version */
  version: string;

  /** Human-readable title */
  title: string;

  /** What this skill does */
  description: string;

  /** Example interactions (for documentation) */
  examples: Array<{ user: string; assistant: string }>;

  /** Which capabilities this skill needs */
  requiredCapabilities: Array<"storage" | "ai">;
}

export interface Skill {
  manifest: SkillManifest;

  /** Execute the skill */
  run(ctx: SkillContext): Promise<SkillResult>;
}
```

## Skill Authoring Rules

1. **No direct imports** — Use `ctx.storage`, `ctx.ai`, `ctx.logger` only
2. **Idempotency** — Check `ctx.idempotencyKey` before creating records
3. **Use `needs_input`** — Don't guess missing data, ask for it
4. **Return structured `data`** — For logging and potential chaining
5. **Keep skills focused** — One skill, one job

## Example Skill

```ts
import type { Skill, SkillContext, SkillResult } from "@finbot/core";

export const AddExpenseSkill: Skill = {
  manifest: {
    id: "add-expense",
    version: "0.1.0",
    title: "Add Expense",
    description: "Log an expense transaction",
    examples: [
      { user: "coffee 3.50", assistant: "Logged €3.50 · Coffee" },
      { user: "45 groceries mercadona", assistant: "Logged €45 · Groceries · Mercadona" },
    ],
    requiredCapabilities: ["storage", "ai"],
  },

  async run(ctx: SkillContext): Promise<SkillResult> {
    // 1. Parse the expense using AI
    const parsed = await ctx.ai.parseJson<{
      amount?: number;
      category?: string;
      description?: string;
      merchant?: string;
    }>(
      `Parse this expense: "${ctx.message.text}"`,
      '{ amount: number, category: string, description?: string, merchant?: string }'
    );

    // 2. Check for required fields
    if (!parsed.amount) {
      return {
        status: "needs_input",
        response: { text: "How much was it?" },
        required: [{ key: "amount", prompt: "Amount (e.g. 12.50)" }],
      };
    }

    // 3. Save to storage
    const { id } = await ctx.storage.saveExpense({
      amount: parsed.amount,
      currency: "EUR",
      category: parsed.category || "Other",
      description: parsed.description,
      merchant: parsed.merchant,
      date: new Date().toISOString().split("T")[0],
    });

    // 4. Return success
    const parts = [`Logged €${parsed.amount}`, parsed.category || "Other"];
    if (parsed.merchant) parts.push(parsed.merchant);

    return {
      status: "ok",
      response: { text: parts.join(" · ") },
      data: { expenseId: id, ...parsed },
    };
  },
};
```

## Status Meanings

| Status | Meaning | User Experience |
|--------|---------|-----------------|
| `ok` | Success | Show response |
| `needs_input` | Missing data | Ask follow-up question |
| `rejected` | Can't/won't do it | Explain why |
| `error` | Something broke | Apologize, log error |
