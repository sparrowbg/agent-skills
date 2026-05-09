# Routing (Standalone React)

For standalone React apps (Vite, etc.) use **React Router v7** or **TanStack Router**.

## React Router v7 — familiar, widely used

```tsx
// main.tsx
import { BrowserRouter, Routes, Route } from 'react-router-dom'

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="/users/:id" element={<UserPage />} />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  )
}

// Navigate programmatically
import { useNavigate, useParams } from 'react-router-dom'

function UserPage() {
  const { id } = useParams<{ id: string }>()
  const navigate = useNavigate()
  return <button onClick={() => navigate('/')}>Home</button>
}

// Protected routes
function ProtectedRoute({ children }: { children: React.ReactNode }) {
  const { user } = useAuthStore()
  if (!user) return <Navigate to="/login" replace />
  return <>{children}</>
}
```

## TanStack Router — fully type-safe, prefer for new projects

```tsx
import { createRouter, createRoute, createRootRoute } from '@tanstack/react-router'

const rootRoute = createRootRoute({ component: RootLayout })

const userRoute = createRoute({
  getParentRoute: () => rootRoute,
  path: '/users/$id',
  component: UserPage,
})

// $id is fully typed — no casting needed
function UserPage() {
  const { id } = userRoute.useParams()  // id: string ✅
}
```

## When to use which

- **React Router v7** — existing projects, team familiarity, simpler needs
- **TanStack Router** — new projects, full type safety on params and search params
