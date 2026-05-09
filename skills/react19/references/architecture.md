# Architecture & Project Structure

## Scale Structure to Project Size

Don't over-engineer early. Let structure grow naturally as the project does.

### Small app (<15 components) — flat, by type

```
src/
├── components/       # all components
├── hooks/            # custom hooks
├── lib/              # utilities, helpers
├── types/            # shared TS types
└── App.tsx
```

### Medium/large app — feature-based, by domain

Organize by what code *does*, not what it *is*:

```
src/
├── features/
│   ├── auth/
│   │   ├── components/     # AuthForm, LoginButton
│   │   ├── hooks/          # useAuth, useSession
│   │   ├── api.ts          # auth API calls
│   │   └── types.ts
│   ├── dashboard/
│   │   ├── components/
│   │   ├── hooks/
│   │   └── api.ts
│   └── settings/
├── components/       # shared, reusable UI (Button, Modal, Input)
├── hooks/            # shared hooks (useDebounce, useLocalStorage)
├── lib/              # shared utilities
├── types/            # global types
└── App.tsx
```

**Rules:**
- A feature folder owns everything related to that domain
- If a component is used in 2+ features → move it to top-level `components/`
- Co-locate tests next to the files they test (`Button.test.tsx` beside `Button.tsx`)
- Don't create a folder until you have 3+ files that belong in it

## Component Design

**Single responsibility** — one component, one job. Split when a component does more than
one thing or exceeds ~100 lines.

**Composition over props** — pass children instead of drilling config props:

```tsx
// ❌ Prop-heavy — hard to extend
<Card title="Hello" subtitle="World" footer={<Button />} showBorder />

// ✅ Composition — flexible, readable
<Card>
  <Card.Header>Hello</Card.Header>
  <Card.Body>World</Card.Body>
  <Card.Footer><Button /></Card.Footer>
</Card>
```

**Extract logic into custom hooks** — keep components focused on rendering:

```tsx
// ❌ Logic mixed into component
function UserProfile() {
  const [user, setUser] = useState(null)
  const [loading, setLoading] = useState(true)
  useEffect(() => { fetchUser().then(setUser).finally(...) }, [])
  // ...
}

// ✅ Hook owns the logic, component owns the UI
function useUser(id: string) {
  const [user, setUser] = useState<User | null>(null)
  const [isPending, setIsPending] = useState(true)
  useEffect(() => { fetchUser(id).then(setUser).finally(() => setIsPending(false)) }, [id])
  return { user, isPending }
}

function UserProfile({ id }: { id: string }) {
  const { user, isPending } = useUser(id)
  if (isPending) return <Skeleton />
  return <div>{user?.name}</div>
}
```

**Naming conventions:**
- Components: `PascalCase` (`UserCard`, `AuthForm`)
- Hooks: `useCamelCase` (`useAuth`, `useDebounce`)
- Event handlers: `handleNoun` (`handleSubmit`, `handleClose`)
- Boolean props: `isX` / `hasX` (`isLoading`, `hasError`)

## Performance

**Profile before optimizing** — React DevTools Profiler shows what's actually slow. Don't guess.

With the React Compiler enabled, `useMemo`, `useCallback`, and `memo` are largely unnecessary.
Without it, use them only after profiling confirms a problem.

**Virtualize long lists** — never render 1000+ DOM nodes:

```tsx
import { useVirtualizer } from '@tanstack/react-virtual'

function VirtualList({ items }: { items: Item[] }) {
  const parentRef = useRef<HTMLDivElement>(null)
  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50,
  })

  return (
    <div ref={parentRef} style={{ height: '400px', overflow: 'auto' }}>
      <div style={{ height: virtualizer.getTotalSize() }}>
        {virtualizer.getVirtualItems().map(item => (
          <div key={item.key} style={{ transform: `translateY(${item.start}px)` }}>
            {items[item.index].name}
          </div>
        ))}
      </div>
    </div>
  )
}
```

**Code splitting** — lazy load routes and heavy components:

```tsx
import { lazy, Suspense } from 'react'

const HeavyChart = lazy(() => import('./HeavyChart'))

function Dashboard() {
  return (
    <Suspense fallback={<Skeleton />}>
      <HeavyChart />
    </Suspense>
  )
}
```
