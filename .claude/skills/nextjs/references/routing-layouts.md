# Routing, Layouts & Streaming

## Routing Patterns

```
app/page.tsx              → /
app/about/page.tsx        → /about
app/blog/[slug]/page.tsx  → /blog/:slug
app/[...slug]/page.tsx    → catch-all
app/(marketing)/page.tsx  → / (route group, no URL segment)
```

Dynamic params in Next.js 16 are async:

```tsx
// params is a Promise
export default async function Page({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params
  // ...
}
```

---

## Layouts & Special Files

```
layout.tsx     — persistent shell; doesn't remount on navigation
template.tsx   — remounts on every navigation (use when you don't want state persistence)
loading.tsx    — Suspense fallback for the route segment
error.tsx      — error boundary for the route segment
not-found.tsx  — rendered when notFound() is called
```

```tsx
// app/layout.tsx — root layout, always required
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  )
}
```

---

## Streaming with Suspense

Use `loading.tsx` for route-level loading, `<Suspense>` for component-level streaming:

```tsx
// app/dashboard/loading.tsx — shown while page data loads
export default function Loading() {
  return <DashboardSkeleton />
}
```

```tsx
// Stream slow parts independently
import { Suspense } from 'react'

export default function Page() {
  return (
    <div>
      <StaticHeader />           {/* renders immediately */}
      <Suspense fallback={<Skeleton />}>
        <SlowDataComponent />    {/* streams in when ready */}
      </Suspense>
    </div>
  )
}
```
