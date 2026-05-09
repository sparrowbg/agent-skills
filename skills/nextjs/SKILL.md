---
name: nextjs
description: >
  Build full-stack Next.js applications using the App Router (Next.js 13+). ALWAYS load
  this skill when the user mentions "next.js", "nextjs", "next", "app router", "server
  components", "server actions", "route handlers", or any React full-stack work. Also
  trigger for: pages, layouts, loading states, API routes in Next.js, SSR, SSG, ISR,
  caching, middleware, Vercel deployment, or any mention of building a full-stack React
  app. TypeScript is always used unless the user explicitly says otherwise. Do not wait
  for the user to say "App Router" — assume it for any new Next.js project.
---

# Next.js App Router Guidelines

Next.js 16 (latest: 16.2.6), App Router, TypeScript, full-stack. This is the authoritative
reference — apply it for any Next.js work.

---

## Project Structure

```
src/
├── app/                        # App Router — routes live here
│   ├── layout.tsx              # Root layout (required) — renders <html> and <body>
│   ├── page.tsx                # Home route
│   ├── loading.tsx             # Suspense fallback (streaming)
│   ├── error.tsx               # Error boundary
│   ├── not-found.tsx           # 404 handler
│   ├── (auth)/                 # Route group — no URL impact
│   │   ├── login/page.tsx
│   │   └── register/page.tsx
│   ├── dashboard/
│   │   ├── layout.tsx          # Nested layout — persists across child routes
│   │   ├── page.tsx
│   │   └── [id]/page.tsx       # Dynamic segment
│   └── api/                    # Route Handlers (external API consumers)
│       └── webhook/route.ts
├── components/
│   ├── ui/                     # Presentational, reusable
│   └── features/               # Feature-specific components
├── lib/                        # Shared utilities, db client, helpers
├── actions/                    # Server Actions
└── types/                      # Shared TypeScript types
```

---

## Server vs Client Components

**Default: everything is a Server Component.** Add `'use client'` only when needed.

| Use Server Component for | Use Client Component (`'use client'`) for |
|---|---|
| Data fetching | `onClick`, `onChange`, event handlers |
| DB queries, secrets, API keys | `useState`, `useEffect`, hooks |
| Heavy dependencies (SDKs, markdown) | Browser APIs (`window`, `localStorage`) |
| Static/streamed UI | Animations, focus management |

**Key rules:**
- Keep `'use client'` as far down the tree as possible — it makes the entire subtree client-side
- Server Components can render Client Components as children — not the other way around
- Pass server-fetched data as props into Client Components

```tsx
// ✅ Server Component fetches, passes to small client island
export default async function PostPage({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params
  const post = await db.post.findUnique({ where: { id } })
  return <PostView post={post} /> // PostView is the 'use client' component
}
```

---

## Data Fetching

Fetch directly in Server Components — no `useEffect`, no API ping-pong:

```tsx
// ✅ Fetch in Server Component
export default async function UsersPage() {
  const users = await db.user.findMany()
  return <UserList users={users} />
}
```

**Caching in Next.js 16 — opt-in with `'use cache'`:**

Everything is dynamic by default. Cache explicitly with the `'use cache'` directive:

```tsx
// Cache a whole page
'use cache'
export default async function BlogPage() {
  const posts = await db.post.findMany()
  return <PostList posts={posts} />
}

// Cache a specific component
async function RecentPosts() {
  'use cache'
  const posts = await db.post.findMany({ take: 5 })
  return <ul>{posts.map(p => <li key={p.id}>{p.title}</li>)}</ul>
}

// Cache a data function with lifetime and tags
import { cacheLife, cacheTag } from 'next/cache'

async function getPosts() {
  'use cache'
  cacheLife('hours')       // or 'seconds', 'days', 'weeks', or { revalidate: 3600 }
  cacheTag('posts')        // tag for targeted invalidation
  return db.post.findMany()
}

// Invalidate by tag after mutation
import { revalidateTag } from 'next/cache'
revalidateTag('posts')
```

**Still works for fetch-based caching:**
```tsx
const data = await fetch('https://api.example.com/posts', {
  next: { revalidate: 3600, tags: ['posts'] }
})
```

**Route segment config (still valid):**
```tsx
export const dynamic = 'force-dynamic'   // always dynamic
export const revalidate = 60             // ISR fallback
```

---

## Server Actions

Use for mutations — form submissions, DB writes, state changes. No separate API route needed for your own UI.

```tsx
// actions/post.ts
'use server'

import { revalidateTag } from 'next/cache'
import { z } from 'zod'

const schema = z.object({ title: z.string().min(1), body: z.string().min(1) })

export async function createPost(formData: FormData) {
  const parsed = schema.safeParse({
    title: formData.get('title'),
    body: formData.get('body'),
  })
  if (!parsed.success) return { error: 'Invalid input' }

  await db.post.create({ data: parsed.data })
  revalidateTag('posts')
  return { success: true }
}
```

```tsx
// Use directly in a Server Component form
import { createPost } from '@/actions/post'

export default function NewPostForm() {
  return (
    <form action={createPost}>
      <input name="title" />
      <textarea name="body" />
      <button type="submit">Publish</button>
    </form>
  )
}
```

**Server Actions vs Route Handlers — when to use which:**

| Situation | Use |
|---|---|
| Form submission / mutation from your own UI | Server Action |
| Data fetching for your own UI | Async Server Component |
| External consumers (mobile app, webhooks, third-party) | Route Handler |
| SSE / streaming to non-React clients | Route Handler |
| Auth callbacks, webhooks | Route Handler |

**Never use Server Actions as query endpoints** — they're POST-only and meant for mutations:

```tsx
// ❌ Wrong — don't fetch data via Server Actions
export async function getUsers() {
  'use server'
  return db.user.findMany()
}

// ✅ Right — fetch directly in the Server Component
export default async function UsersPage() {
  const users = await db.user.findMany()
  return <UserList users={users} />
}
```

---

## Route Handlers

For external API consumers, webhooks, SSE — lives in `app/api/*/route.ts`:

```ts
// app/api/webhook/route.ts
import { NextRequest } from 'next/server'

export async function POST(req: NextRequest) {
  const body = await req.json()
  // validate, process
  return Response.json({ received: true })
}

// Note: GET handlers are NOT cached by default in Next.js 16
export async function GET() {
  const data = await db.post.findMany()
  return Response.json(data)
}
```

---

---

## Anti-Patterns

❌ Never:
- Put `'use client'` high in the tree — it disables RSC for the entire subtree
- Use Server Actions to fetch data (they're for mutations only)
- Read `cookies()` or `headers()` in layouts — forces dynamic rendering everywhere
- Fetch in `useEffect` when a Server Component would do
- Create Route Handlers for mutations that only your own UI calls
- Use `any` — always type properly
- Hardcode secrets — env variables only, never exposed to client
- Use `unstable_cache` — replaced by `'use cache'` directive in Next.js 16
- Start new projects with `middleware.ts` — use `proxy.ts` instead

✅ Always:
- Server Component by default, `'use client'` only when needed
- Cache explicitly with `'use cache'` + `cacheTag` — don't rely on implicit caching
- Validate all inputs with Zod in Server Actions and Route Handlers
- Use `revalidateTag` after mutations to invalidate cached data
- Stream slow data with `<Suspense>` boundaries
- Keep `proxy.ts` lightweight — no DB calls, no heavy logic

---

## package.json Baseline

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  }
}
```

Turbopack is the default bundler in Next.js 16 — no `--turbopack` flag needed.

```bash
# Bootstrap
npx create-next-app@latest my-app --typescript --tailwind --app --src-dir
```

---

## Reference Files

Load these when the topic comes up — don't load all of them upfront:

- `references/routing-layouts.md` — routing patterns, layouts, special files, streaming with Suspense
- `references/deployment.md` — proxy.ts, validation, TypeScript defaults, Vercel deployment
