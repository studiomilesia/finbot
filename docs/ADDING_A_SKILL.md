# Adding a Skill

This guide walks through creating a new skill for finbot.

## 1. Create the Skill Folder

```bash
mkdir -p packages/skills/your-skill
cd packages/skills/your-skill
```

## 2. Create package.json

```json
{
  "name": "@finbot/skill-your-skill",
  "version": "0.1.0",
  "private": true,
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "build": "tsc",
    "test": "vitest"
  },
  "dependencies": {
    "@finbot/core": "workspace:*"
  },
  "devDependencies": {
    "typescript": "^5.7.0",
    "vitest": "^2.0.0"
  }
}
```

## 3. Create tsconfig.json

```json
{
  "extends": "../../../tsconfig.json",
  "compilerOptions": {
    "outDir": "dist",
    "rootDir": "src",
    "noEmit": false
  },
  "include": ["src"]
}
```

## 4. Create the Skill Manifest

Create `skill.json`:

```json
{
  "id": "your-skill",
  "version": "0.1.0",
  "title": "Your Skill",
  "description": "What this skill does",
  "examples": [
    { "user": "example input", "assistant": "example response" }
  ],
  "requiredCapabilities": ["storage", "ai"]
}
```

## 5. Implement the Skill

Create `src/index.ts`:

```ts
import type { Skill, SkillContext, SkillResult } from "@finbot/core";
import manifest from "../skill.json";

export const YourSkill: Skill = {
  manifest,

  async run(ctx: SkillContext): Promise<SkillResult> {
    // Your logic here

    return {
      status: "ok",
      response: { text: "Done!" },
    };
  },
};

export default YourSkill;
```

## 6. Add Few-Shot Examples (Optional)

Create `prompts.md` with examples for the LLM:

```markdown
# Your Skill Prompts

## Parsing Examples

User: "example input 1"
Parsed: { "field": "value" }

User: "example input 2"
Parsed: { "field": "other value" }
```

## 7. Write Tests

Create `src/your-skill.test.ts`:

```ts
import { describe, it, expect, vi } from "vitest";
import { YourSkill } from "./index";

describe("YourSkill", () => {
  it("should handle basic input", async () => {
    const ctx = {
      message: { channel: "telegram", userId: "123", text: "test input" },
      storage: { /* mock */ },
      ai: { complete: vi.fn().mockResolvedValue("parsed") },
      logger: { info: vi.fn(), warn: vi.fn(), error: vi.fn(), debug: vi.fn() },
    };

    const result = await YourSkill.run(ctx);

    expect(result.status).toBe("ok");
  });
});
```

## 8. Register the Skill

Add your skill to the router's skill map in `packages/core/src/router.ts`:

```ts
import { YourSkill } from "@finbot/skill-your-skill";

const SKILL_MAP = {
  // ... existing skills
  "your-skill": YourSkill,
};
```

## 9. Add Intent to Router

Update the intent classification prompt to include your skill:

```ts
const INTENTS = [
  // ... existing intents
  { id: "your-skill", description: "When the user wants to..." },
];
```

## Checklist

- [ ] Skill folder created in `packages/skills/`
- [ ] `package.json` with `@finbot/core` dependency
- [ ] `skill.json` manifest with examples
- [ ] `src/index.ts` implementing `Skill` interface
- [ ] Uses only `ctx.storage`, `ctx.ai`, `ctx.logger` (no direct imports)
- [ ] Returns `needs_input` instead of guessing missing data
- [ ] Tests written and passing
- [ ] Registered in router's skill map
- [ ] Intent added to classification prompt
