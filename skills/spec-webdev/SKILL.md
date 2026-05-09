---
name: spec-webdev
description: >
  Generate a SPEC.md file for a web development project. ALWAYS load this skill when
  the user asks to "create a spec", "generate a spec", "write a SPEC.md", "set up project
  specs", "define project requirements", or any variation of documenting technical
  decisions for a web project. Covers Node.js backends and Next.js full-stack apps.
  Triggers on: "spec", "SPEC.md", "project spec", "technical spec", "project requirements",
  "define my stack", "document my architecture".
---

# spec-webdev

Generates a `SPEC.md` file by interviewing the user about their project, then outputting
a structured document Claude can reference throughout development.

## Goal

Produce a `SPEC.md` that answers every architectural decision upfront so Claude never
has to guess or ask mid-development.

---

## Interview Process

### Step 1 — Project Type

Ask first:

> "Is this a Node.js backend, a Next.js full-stack app, or both?"

Based on answer, follow the relevant question set below. If both, cover all sections.

---

### Step 2 — Ask Questions

Don't dump all questions at once. Group into rounds of 2–3, wait for answers, then continue.
If user is unsure about something, suggest a sensible default and move on.

**All projects:**

- What does the app do? (1–2 sentences)
- Who are the users? (internal tool, public app, API consumers)
- Expected scale? (prototype, small team, production traffic)

**Node.js backend:**

- Database? (Postgres, MySQL, MongoDB, Redis, none)
  - Default: Postgres + raw SQL or query builder (no ORM unless requested)
- Auth? (JWT, session, API key, none, undecided)
- External services? (email, storage, payments, queues)
- Deployment target? (Docker/container, VPS, serverless, undecided)
- Any forbidden patterns? (e.g. no Express, no global state, specific libs banned)

**Next.js full-stack:**

- Rendering strategy? (SSR, SSG, ISR, mix — or "let Next decide")
- Database? (same as above)
- Auth? (NextAuth, Clerk, custom, none)
- Styling? (Tailwind, CSS Modules, other)
- Deployment? (Vercel, self-hosted, Docker)
- Any forbidden patterns?

**Both:**

- How do they communicate? (Next.js calls Node API, shared monorepo, separate repos)
- Shared types/validation? (Zod schemas shared between frontend and backend)

---

### Step 3 — Confirm & Generate

Summarize decisions back to user in one short paragraph. Ask:

> "Anything to change before I generate the SPEC.md?"

Then generate the file.

---

## SPEC.md Template

Populate all sections. Mark undecided items as `TBD` — never omit a section.

```markdown
# SPEC.md

## Project Overview

[1–2 sentence description. Who uses it, what it does.]

## Project Type

- [ ] Node.js backend only
- [ ] Next.js full-stack only
- [ ] Both (monorepo / separate repos)

## Runtime & Language

- Node.js >= 24
- TypeScript strict
- ESM only (`"type": "module"`)

## Architecture

[Describe the layered structure chosen. Example:]

- `src/routes/`     — routing
- `src/handlers/`   — request handling
- `src/services/`   — business logic
- `src/infra/`      — DB, cache, external APIs

OR for Next.js:

- `src/app/`        — App Router pages and layouts
- `src/features/`   — feature-based components and hooks
- `src/actions/`    — Server Actions
- `src/lib/`        — shared utilities

## Database

- Engine: [Postgres / MySQL / MongoDB / Redis / None]
- Access: [Raw SQL / Query builder (e.g. kysely) / ORM (e.g. Prisma) / None]
- Migrations: [TBD / tool name]

## Authentication

- Strategy: [JWT / Session / API key / NextAuth / Clerk / None]
- Token validation: always server-side
- Notes: [any specific constraints]

## API Contract (Node.js / Route Handlers)

All responses follow this shape:

```json
{
  "success": boolean,
  "data": unknown,
  "error": string
}
```

HTTP status codes:
- 400 bad input, 401 unauthenticated, 403 forbidden
- 404 not found, 409 conflict, 422 business rule, 500 server error

## Validation

- Library: Zod
- Validate at: API boundary, before DB operations, env vars at startup

## Logging

Structured logs only:

```json
{ "level": "info|warn|error", "message": "string", "meta": {} }
```

- Per-request logging: [yes / no]
- External logging service: [TBD / none / service name]

## External Services

- [List services: email, storage, queues, payments, etc. or "None"]

## Styling (Next.js only)

- [Tailwind / CSS Modules / other / N/A]

## Deployment

- Target: [Docker / Vercel / VPS / Serverless / TBD]
- Stateless: [yes / no]
- Config: environment variables only — no hardcoded secrets
- Filesystem: [stateless — no local fs dependency / exceptions noted]

## Forbidden Patterns

- [List anything explicitly not allowed, e.g.:]
- No Express unless explicitly requested
- No ORMs unless explicitly requested
- No global mutable state
- No auto-magical routing
- No stack traces exposed to clients

## Open Questions

- [List anything not yet decided — review before development starts]

## Related Skills

- nodejs24 — Node.js 24+ native APIs and patterns
- nextjs — Next.js 16 App Router patterns
```

---

## Output Instructions

- Save as `SPEC.md` in the project root
- Keep it concise — this is a reference doc, not an essay
- Every section must have a value or `TBD` — no empty sections
- After generating, tell the user: "Reference this file at the start of each session
  so Claude applies your project decisions without asking again."
