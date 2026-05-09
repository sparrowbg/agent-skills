---
name: react19
description: >
  Write modern React 19+ code using the latest hooks, APIs, and patterns. ALWAYS load
  this skill when the user mentions "react", "react 19", "hooks", "components", "jsx",
  "tsx", "useEffect", "useState", "context", "suspense", "forms", "optimistic UI",
  "transitions", or any client-side React work. Also trigger for: component design,
  state management, async UI patterns, form handling, ref usage, memoization, and React
  performance. TypeScript is always used unless the user explicitly says otherwise.
  This skill covers client-side React patterns only — for Next.js App Router, Server
  Components, or Server Actions, the nextjs skill applies instead.
---

# React 19+ Guidelines

React 19.2, TypeScript, client-side patterns. No overlap with the Next.js skill.

## Reference Files

Load these when the topic comes up — don't load all of them upfront:

- `references/routing.md` — React Router v7 and TanStack Router
- `references/state-management.md` — state decision table, Zustand, TanStack Query, RHF
- `references/architecture.md` — project structure, component design, performance

---

## New Hooks at a Glance

| Hook | Replaces / Solves |
|---|---|
| `useActionState` | Manual `useState` + `useEffect` for async operations |
| `useOptimistic` | Manual optimistic state rollback logic |
| `useFormStatus` | Prop-drilling `isPending` into submit buttons |
| `use(promise)` | `useEffect` + `useState` for data fetching |
| `use(Context)` | `useContext` (but works conditionally) |
| `useEffectEvent` | Stale closure bugs in `useEffect` dependencies |

---

## useActionState

Manages async action state — pending, result, error — in one hook:

```tsx
import { useActionState } from 'react'

async function updateUsername(prevState: State, formData: FormData) {
  const username = formData.get('username') as string
  const res = await fetch('/api/user', {
    method: 'POST',
    body: JSON.stringify({ username }),
  })
  if (!res.ok) return { error: 'Failed to update' }
  return { success: true, username }
}

type State = { error?: string; success?: boolean; username?: string } | null

function UsernameForm() {
  const [state, action, isPending] = useActionState(updateUsername, null)

  return (
    <form action={action}>
      <input name="username" />
      <button disabled={isPending}>{isPending ? 'Saving...' : 'Save'}</button>
      {state?.error && <p>{state.error}</p>}
      {state?.success && <p>Updated to {state.username}</p>}
    </form>
  )
}
```

---

## useOptimistic

Show the expected result immediately, roll back automatically if the action fails:

```tsx
import { useOptimistic, useActionState } from 'react'

type Message = { id: string; text: string; pending?: boolean }

function MessageThread({ messages }: { messages: Message[] }) {
  const [optimisticMessages, addOptimistic] = useOptimistic(
    messages,
    (state, newText: string) => [
      ...state,
      { id: crypto.randomUUID(), text: newText, pending: true },
    ]
  )

  const [, sendAction, isPending] = useActionState(
    async (_: null, formData: FormData) => {
      const text = formData.get('text') as string
      addOptimistic(text)
      await sendMessage(text)
      return null
    },
    null
  )

  return (
    <>
      <ul>
        {optimisticMessages.map(m => (
          <li key={m.id} style={{ opacity: m.pending ? 0.5 : 1 }}>{m.text}</li>
        ))}
      </ul>
      <form action={sendAction}>
        <input name="text" />
        <button disabled={isPending}>Send</button>
      </form>
    </>
  )
}
```

---

## useFormStatus

Access the parent form's pending state from inside a child — no prop drilling:

```tsx
import { useFormStatus } from 'react-dom'

// Must be rendered INSIDE a <form>
function SubmitButton({ label }: { label: string }) {
  const { pending } = useFormStatus()
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Saving...' : label}
    </button>
  )
}
```

---

## use()

Read promises and context in render — works conditionally, unlike hooks:

```tsx
import { use, Suspense } from 'react'

// Read a promise — component suspends until resolved
function UserCard({ userPromise }: { userPromise: Promise<User> }) {
  const user = use(userPromise)
  return <div>{user.name}</div>
}

// Wrap in Suspense — required
function Page() {
  const userPromise = fetchUser(userId)  // create promise OUTSIDE render
  return (
    <Suspense fallback={<Skeleton />}>
      <UserCard userPromise={userPromise} />
    </Suspense>
  )
}

// Read context conditionally (unlike useContext)
function ThemedButton({ show }: { show: boolean }) {
  if (!show) return null
  const theme = use(ThemeContext)  // after early return — fine
  return <button style={{ background: theme.primary }}>Click</button>
}
```

Never create promises inside render — creates a new promise on every render:
```tsx
// Never do this
function Bad() { const data = use(fetch('/api').then(r => r.json())) }

// Create outside, pass as prop or via cache()
const promise = fetch('/api').then(r => r.json())
function Good() { const data = use(promise) }
```

---

## useEffectEvent

Extract non-reactive logic from Effects without stale closure bugs:

```tsx
import { useEffect, useEffectEvent } from 'react'

function ChatRoom({ roomId, theme }: { roomId: string; theme: string }) {
  const onConnected = useEffectEvent(() => {
    showNotification('Connected!', theme)  // always reads latest theme
  })

  useEffect(() => {
    const connection = connect(roomId)
    connection.on('connected', onConnected)  // NOT in dependency array
    return () => connection.disconnect()
  }, [roomId])  // only reconnects when roomId changes
}
```

- Never add `useEffectEvent` functions to the dependency array
- Only call them from inside Effects

---

## Refs

Refs are a regular prop in React 19 — no more `forwardRef`:

```tsx
// ref as a prop directly
function Input({ ref, ...props }: React.ComponentProps<'input'>) {
  return <input ref={ref} {...props} />
}

// Cleanup functions in refs
<input
  ref={(node) => {
    doSomething(node)
    return () => cleanup(node)  // called on unmount
  }}
/>
```

---

## Document Metadata

Render `<title>`, `<meta>`, `<link>` directly in components — no `react-helmet`:

```tsx
function ProductPage({ product }: { product: Product }) {
  return (
    <>
      <title>{product.name} | My Store</title>
      <meta name="description" content={product.description} />
      <main><h1>{product.name}</h1></main>
    </>
  )
}
```

React hoists to `<head>` automatically, deduplicates, and handles SSR correctly.

---

## useTransition

Wrap non-urgent state updates to keep the UI responsive:

```tsx
const [isPending, startTransition] = useTransition()

function handleChange(value: string) {
  setQuery(value)                        // urgent — updates immediately
  startTransition(async () => {
    const data = await searchAPI(value)  // non-urgent — can be interrupted
    setResults(data)
  })
}
```

---

## React Compiler

Automatically memoizes components — `useMemo`, `useCallback`, `memo` largely unnecessary:

```tsx
// Before compiler — manual memoization
const value = useMemo(() => compute(a, b), [a, b])
const cb = useCallback(() => doThing(id), [id])

// With compiler — write normal code, compiler handles it
const value = compute(a, b)
function handleClick() { doThing(id) }
```

Don't remove existing `useMemo`/`useCallback` proactively — only when compiler is confirmed enabled.

---

## Context

```tsx
// React 19 — no .Provider needed
<ThemeContext value="dark">
  <Page />
</ThemeContext>

// use() works conditionally, useContext doesn't
function Component({ show }: { show: boolean }) {
  if (!show) return null
  const theme = use(ThemeContext)  // after early return — fine
  return <div data-theme={theme} />
}
```

---

## Suspense & Error Boundaries

```tsx
import { Suspense } from 'react'
import { ErrorBoundary } from 'react-error-boundary'

function Dashboard() {
  return (
    <ErrorBoundary fallback={<ErrorMessage />}>
      <Suspense fallback={<DashboardSkeleton />}>
        <DashboardContent />
      </Suspense>
    </ErrorBoundary>
  )
}
```

---

## Anti-Patterns

Never:
- Use `forwardRef` — refs are regular props in React 19
- Use `react-helmet` — native metadata support built in
- Create promises inside component render
- Disable lint rules to fix stale closures — use `useEffectEvent`
- Manually manage loading/error state for forms — use `useActionState`
- Add `useEffectEvent` functions to dependency arrays
- Reach for a library before trying the native React solution
- Add global state for state that only 1-2 components need

Always:
- TypeScript throughout
- Wrap async reads with `<Suspense>` when using `use(promise)`
- Use `useOptimistic` for mutations that should feel instant
- Let the React Compiler handle memoization
- Default to native React — add a dependency only when you hit a real limitation
