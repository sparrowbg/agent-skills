# Architecture & Best Practices

## Architecture

Scale structure to project size — always ask "will this grow?" before adding layers:

```
Small script / prototype  →  single file is fine
Growing API               →  layered structure:

Request
  └── Route/Controller    (HTTP specifics, input validation at boundary)
        └── Service       (business logic, framework-agnostic)
              └── Repository  (data access, DB queries only)
```

- Business logic belongs in the **service layer**, never in controllers
- Keep repository layer ignorant of HTTP — it should work in any context
- For small projects, controller + service in one file is acceptable

---

## Async Patterns

| Pattern | Use When |
|---|---|
| `async/await` | Sequential async operations |
| `Promise.all` | Parallel independent operations |
| `Promise.allSettled` | Parallel where some can fail |
| `Promise.race` | Timeout or first-response-wins |

**Event loop awareness:**
- I/O-bound work (DB, HTTP, files) → async handles it fine
- CPU-bound work (image processing, crypto, complex calculations) → offload to **worker threads**, async won't help
- Never use sync methods in production (`fs.readFileSync`, `crypto.pbkdf2Sync`, etc.)
- Use streaming for large data — never buffer everything into memory

---

## Error Handling

**Centralized pattern:** throw custom errors from any layer, catch and format at the top (middleware/handler).

What the **client** receives:
- Appropriate HTTP status code
- Error code for programmatic handling
- User-friendly message
- **No internal details** (no stack traces, no DB errors)

What **logs** capture:
- Full stack trace
- Request context (method, path, params)
- User ID if applicable
- Timestamp

**HTTP status code guide:**

| Situation | Status |
|---|---|
| Bad input | 400 |
| Missing/invalid credentials | 401 |
| Valid auth, not permitted | 403 |
| Resource doesn't exist | 404 |
| Duplicate or state conflict | 409 |
| Business rule violation | 422 |
| Our fault | 500 |

---

## Validation

Validate at every **boundary** — don't trust data that crosses a layer:
- API entry point (request body, params, query)
- Before database operations
- External API responses and file uploads
- Environment variables at startup (fail fast if missing)

Use **Zod** for TypeScript-first schema validation with full type inference.

---

## Security Checklist

Before shipping any Node.js service:

- [ ] All inputs validated (body, params, headers, cookies)
- [ ] Parameterized queries — no string concatenation for SQL
- [ ] Passwords hashed with `bcrypt` or `argon2`
- [ ] JWT signature and expiry always verified
- [ ] Rate limiting in place
- [ ] Security headers (`helmet` or equivalent)
- [ ] CORS properly configured
- [ ] Secrets in environment variables only — never hardcoded
- [ ] Dependencies audited (`npm audit`)

**Trust nothing:** validate query params, body, headers, cookies, file uploads, and external API responses.

---

## TypeScript in Production

Write `.ts` files, compile with `tsc`, run the compiled JS with Node 24. No tsx, no ts-node, no experimental flags.

```bash
npm install -D typescript
npx tsc
node dist/server.js
```

Minimal `tsconfig.json` for Node 24:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "outDir": "dist",
    "strict": true,
    "skipLibCheck": true
  },
  "include": ["src"]
}
```

```json
{
  "scripts": {
    "build": "tsc",
    "start": "node --env-file=.env dist/server.js",
    "dev": "node --env-file=.env --watch --experimental-strip-types src/server.ts"
  }
}
```

- `dev` uses experimental strip-types for fast iteration — acceptable in development
- `start` always runs compiled JS — no experimental flags in production
