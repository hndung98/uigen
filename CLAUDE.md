# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run setup          # Install deps, generate Prisma client, run migrations
npm run dev            # Next.js dev server with Turbopack (http://localhost:3000)
npm run build          # Production build
npm run lint           # ESLint (next/core-web-vitals)
npm test               # Vitest (jsdom environment)
npm run db:reset       # Reset database migrations (destructive)
```

Run a single test file: `npx vitest run src/path/to/file.test.tsx`

## Architecture

UIGen is an AI-powered React component generator with live preview. Users describe components in natural language, Claude generates JSX code, and it renders immediately in a sandboxed iframe.

### Stack
- **Next.js 15** (App Router, Turbopack) with **React 19** and **TypeScript**
- **Tailwind CSS v4** with shadcn/ui components (Radix UI primitives)
- **SQLite** with **Prisma ORM** (schema in `/prisma/schema.prisma`, db at `/prisma/dev.db`)
- **Vercel AI SDK** (`ai` + `@ai-sdk/anthropic`) for Claude streaming
- **Monaco Editor** for code editing, **Babel standalone** for client-side JSX transpilation

### Key Directories
- `src/app/api/chat/` — Streaming chat endpoint using `streamText`
- `src/lib/contexts/` — Two main React contexts:
  - **FileSystemContext** — In-memory virtual file system (create/update/delete/rename files)
  - **ChatContext** — Conversation state via Vercel AI SDK's `useAIChat`
- `src/lib/tools/` — AI tool definitions (Zod schemas):
  - `str-replace.ts` — View/create/edit/insert into files
  - `file-manager.ts` — Rename/delete files
- `src/lib/prompts/generation.tsx` — System prompt for Claude
- `src/lib/file-system.ts` — VirtualFileSystem implementation (tree structure, Map-based)
- `src/lib/transform/jsx-transformer.ts` — Babel JSX-to-JS pipeline with esm.sh import maps
- `src/lib/provider.ts` — Language model provider; falls back to MockLanguageModel when `ANTHROPIC_API_KEY` is unset

### Data Flow
1. User types in chat → `ChatContext.handleSubmit()` → POST `/api/chat`
2. Claude streams response with tool calls (`str_replace_editor`, `file_manager`)
3. `FileSystemContext.handleToolCall()` updates the VirtualFileSystem
4. PreviewFrame re-renders: Babel transforms JSX → sandboxed iframe executes
5. `onFinish` callback persists project to Prisma (authenticated users only)

### Database
The database schema is defined in `/prisma/schema.prisma`. Reference it anytime you need to understand the structure of data stored in the database.

### Auth
JWT-based sessions with httpOnly cookies (7-day expiry). Bcrypt password hashing. Anonymous users get sessionStorage-based tracking (`anon-work-tracker.ts`); signing in saves their work.

### Routing
- `/` — Home/dashboard; authenticated users redirect to latest project
- `/[projectId]` — Main workspace with resizable chat/preview/code panels

### Path Alias
`@/*` maps to `./src/*` in tsconfig.

### Testing
Vitest + React Testing Library + jsdom. Tests cover components (chat, editor, preview), contexts, and the JSX transformer.

## Code Style

- Use comments sparingly. Only comment complex, non-obvious code.
