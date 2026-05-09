# Deployment, Proxy, Validation & TypeScript

## Proxy (formerly Middleware)

`middleware.ts` is **deprecated** in Next.js 16 — renamed to `proxy.ts`. Runs in Node.js runtime (not edge):

```ts
// proxy.ts (root of project) — replaces middleware.ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function proxy(request: NextRequest) {
  const token = request.cookies.get('token')
  if (!token) {
    return NextResponse.redirect(new URL('/login', request.url))
  }
  return NextResponse.next()
}

export const config = {
  matcher: ['/dashboard/:path*', '/api/:path*'],
}
```

- `proxy.ts` runs Node.js runtime — Node APIs available (unlike old edge middleware)
- Keep it lightweight — no heavy DB logic
- `middleware.ts` still works but will be removed in a future version — migrate when starting new projects

---

## Validation

Always validate at the boundary with Zod:

```ts
import { z } from 'zod'

const CreatePostSchema = z.object({
  title: z.string().min(1).max(200),
  body: z.string().min(1),
  published: z.boolean().default(false),
})

type CreatePostInput = z.infer<typeof CreatePostSchema>
```

---

## TypeScript Defaults

Key types to know:

```tsx
// Page props
type PageProps = {
  params: Promise<{ id: string }>        // params is async (Next.js 15+)
  searchParams: Promise<{ q?: string }>  // searchParams is async (Next.js 15+)
}

// Layout props
type LayoutProps = {
  children: React.ReactNode
  params: Promise<{ id: string }>
}
```

---

## Vercel Deployment

```bash
# Install Vercel CLI
npm i -g vercel

# Deploy (auto-detects Next.js)
vercel

# Deploy to production
vercel --prod
```

Or connect the GitHub repo in the Vercel dashboard — every push to `main` deploys automatically.

**Environment variables:**
- Set in Vercel dashboard → Project → Settings → Environment Variables
- Prefix with `NEXT_PUBLIC_` to expose to the browser, otherwise server-only

**`vercel.json` — only needed for custom config:**
```json
{
  "regions": ["fra1"],
  "headers": [
    {
      "source": "/api/(.*)",
      "headers": [{ "key": "Cache-Control", "value": "no-store" }]
    }
  ]
}
```

**Vercel handles automatically:**
- Edge Network CDN for static assets
- Serverless Functions for Route Handlers and Server Actions
- ISR / `'use cache'` works out of the box — no config needed
- Preview deployments for every PR
