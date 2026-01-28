# Development

This document covers setting up a development environment for finbot.

## Prerequisites

- Node.js 18+
- pnpm 9+

## Setup

```bash
git clone https://github.com/youruser/finbot.git
cd finbot
pnpm install
pnpm build
```

## Commands

| Command | Description |
|---------|-------------|
| `pnpm build` | Build all packages |
| `pnpm dev` | Run in development mode |
| `pnpm test` | Run tests |
| `pnpm lint` | Lint all packages |
| `pnpm typecheck` | Type check all packages |

## MCP Servers

This project includes [MCP (Model Context Protocol)](https://modelcontextprotocol.io/) servers configured in `.mcp.json`. These help AI assistants (Claude Code, Cursor, etc.) work more effectively on the codebase.

### Configured Servers

#### filesystem
**Package:** `@modelcontextprotocol/server-filesystem`

Secure file operations. Configured to access the project root.

#### context7
**Package:** `@upstash/context7-mcp`

Up-to-date documentation for npm packages. When working with AI assistants, you can ask for current docs:

- "use context7 to look up grammY bot setup"
- "use context7 for better-sqlite3 API"
- "use context7 for Zod validation"

#### sequential-thinking
**Package:** `@modelcontextprotocol/server-sequential-thinking`

Helps with complex problem-solving by breaking down tasks into thought sequences.

#### memory
**Package:** `@modelcontextprotocol/server-memory`

Persistent memory across sessions. Useful for recalling project decisions and context.

### Adding MCP Servers

Browse available servers:
- [MCP Registry](https://registry.modelcontextprotocol.io/)
- [Awesome MCP Servers](https://github.com/wong2/awesome-mcp-servers)

Add to `.mcp.json`:

```json
{
  "mcpServers": {
    "new-server": {
      "command": "npx",
      "args": ["-y", "@scope/package-name@version"]
    }
  }
}
```

## Project Structure

See [ARCHITECTURE.md](./ARCHITECTURE.md) for the full folder structure and design decisions.

## Creating a New Package

1. Create folder in the appropriate location:
   - Skills: `packages/skills/your-skill/`
   - Adapters: `packages/adapters/your-adapter/`
   - Storage: `packages/storage/your-backend/`

2. Add `package.json`:
```json
{
  "name": "@finbot/your-package",
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
  }
}
```

3. Add `tsconfig.json`:
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

4. Create `src/index.ts` with your implementation.

## Testing

We use [Vitest](https://vitest.dev/) for testing.

```bash
# Run all tests
pnpm test

# Run tests for a specific package
pnpm --filter @finbot/skill-add-expense test

# Watch mode
pnpm test -- --watch
```

## Code Style

- TypeScript strict mode
- No direct DB/HTTP imports in skills (use capabilities)
- Prefer `needs_input` over guessing missing data
- Keep skills focused on one task
