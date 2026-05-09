# State Management

**Default to native React** — add a library only when the native approach is genuinely insufficient.

## Decision Table

| State Type | Native first | Reach for a library when... |
|---|---|---|
| Local UI state | `useState` | never |
| Complex local state | `useReducer` | never |
| Shared state nearby | lift state + props | prop drilling goes 3+ levels deep → Context |
| Global app state | Context + `useReducer` | store is large / updated frequently → Zustand |
| Async server data | `fetch` + `useState` + `useEffect` | caching, deduplication, refetch needed → TanStack Query |
| Form state | controlled inputs + `useActionState` | many fields, complex validation → React Hook Form + Zod |

## Native First

```tsx
// ✅ Start here — often enough for simple data fetching
function useUser(id: string) {
  const [user, setUser] = useState<User | null>(null)
  const [isPending, setIsPending] = useState(true)

  useEffect(() => {
    fetchUser(id).then(setUser).finally(() => setIsPending(false))
  }, [id])

  return { user, isPending }
}

// ✅ Context + useReducer before reaching for Zustand
const AuthContext = createContext<AuthState | null>(null)

function AuthProvider({ children }: { children: React.ReactNode }) {
  const [state, dispatch] = useReducer(authReducer, initialState)
  return <AuthContext value={{ state, dispatch }}>{children}</AuthContext>
}
```

## When to Upgrade

```tsx
// ✅ TanStack Query — when you need caching, background refetch, deduplication
const { data: user, isPending } = useQuery({
  queryKey: ['user', userId],
  queryFn: () => fetchUser(userId),
})

// Invalidate after mutation
const queryClient = useQueryClient()
await mutateUser(data)
queryClient.invalidateQueries({ queryKey: ['user', userId] })

// ✅ Zustand — when Context re-renders become a problem or store is complex
import { create } from 'zustand'

const useAuthStore = create<AuthStore>((set) => ({
  user: null,
  setUser: (user) => set({ user }),
  logout: () => set({ user: null }),
}))

// ✅ React Hook Form + Zod — many fields, complex validation, no re-render per keystroke
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'

const schema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
})

function LoginForm() {
  const { register, handleSubmit, formState: { errors } } = useForm({
    resolver: zodResolver(schema),
  })
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email')} />
      {errors.email && <p>{errors.email.message}</p>}
    </form>
  )
}
```
