# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# First-time setup
npm run setup        # Install deps + Prisma generate + migrate

# Development
npm run dev          # Next.js dev server with Turbopack

# Testing
npm test             # Run all Vitest tests
npx vitest run src/lib/__tests__/file-system.test.ts  # Run a single test file

# Linting
npm run lint         # ESLint

# Database
npm run db:reset     # Reset and re-migrate the SQLite database
npx prisma studio    # Browse the database in a web UI
```

## Environment

Set `ANTHROPIC_API_KEY` in `.env` to enable real Claude-powered generation. Without it, the app falls back to a mock provider that returns static code.

## Architecture

**UIGen** is a Next.js 15 (App Router) web app where users chat with Claude to generate React components, which are immediately previewed in the browser.

### Request flow

```
User message → ChatInterface (client)
  → POST /api/chat (server)
    → Vercel AI SDK streamText() with Anthropic model
    → Claude responds using tools:
        str_replace_editor  – create/edit files
        file_manager        – move/delete/list files
  → VirtualFileSystem updated in React context
  → PreviewFrame re-renders component (Babel in iframe)
  → Project auto-saved to Prisma DB (authenticated users only)
```

### Key directories

- `src/app/api/chat/route.ts` — streaming chat endpoint; tool definitions and AI call live here
- `src/lib/prompts/generation.ts` — system prompt sent to Claude
- `src/lib/tools/` — tool schemas passed to the model
- `src/lib/file-system.ts` — in-memory virtual file system (VirtualFileSystem class); source of truth for generated code
- `src/lib/contexts/` — `FileSystemContext` and `ChatContext` that wire AI responses to UI state
- `src/components/chat/` — chat UI (ChatInterface, MessageList, MessageInput, MarkdownRenderer)
- `src/components/editor/` — Monaco-based code editor + file tree
- `src/components/preview/PreviewFrame.tsx` — renders generated components via Babel in an iframe
- `src/actions/` — Next.js Server Actions for auth and project CRUD
- `src/lib/auth.ts` — JWT session management (jose + bcrypt, 7-day expiry)
- `prisma/schema.prisma` — two models: `User` and `Project` (messages + file system state stored as JSON strings)

## Database Schema

When working with database data, always read `prisma/schema.prisma` first to understand the data models and field formats.

### Tech stack

| Layer | Choice |
|---|---|
| Framework | Next.js 15, React 19 |
| Language | TypeScript (strict) |
| Styling | Tailwind CSS v4, shadcn/ui (new-york style) |
| AI | Anthropic Claude via `@ai-sdk/anthropic` + Vercel AI SDK |
| ORM | Prisma + SQLite (`prisma/dev.db`) |
| Editor | Monaco Editor |
| Testing | Vitest + jsdom |

### Path aliases

`@/*` maps to `src/*` — use this in all imports.

### Node compatibility

`node-compat.cjs` is required at startup (`NODE_OPTIONS='--require ./node-compat.cjs'`) to patch out broken Web Storage globals on Node 25+. The dev and build scripts already include this.
