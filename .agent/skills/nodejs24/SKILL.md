---
name: nodejs24
description: >
  Write modern Node.js 24+ code using native built-in APIs. ALWAYS load this skill
  when the user mentions "node", "nodejs", "node.js", "npm", or any server-side
  JavaScript/TypeScript task — even casually (e.g. "write me a node script", "make an
  express API", "build a backend in JS"). Also trigger for: HTTP servers, REST APIs,
  WebSockets, file I/O, environment variables, async code, backend scripts, CLI tools,
  and any request to write or review JavaScript that will run on the server. Trigger when
  the user mentions packages like express, ws, axios, node-fetch, dotenv, nodemon, uuid,
  jest, or mocha — these are all Node.js contexts. Do not wait for the user to say
  "Node 24" or "LTS" — load this skill for any Node.js work regardless of version
  specificity.
---

# Node.js 24+ Coding Guidelines

Node.js 24 (LTS from October 2025, supported until April 2028) brings many APIs to
native stability. Always prefer native Node.js APIs over third-party packages wherever
they exist. This guide is the authoritative reference — apply it for any Node.js code.

---

## Module System

**Always use ESM** (ES Modules) for new code unless the user explicitly needs CommonJS:

```js
// ✅ ESM (preferred)
import { readFile } from 'node:fs/promises';

// ❌ CommonJS (only if required)
const fs = require('fs');
```

- Always use the `node:` prefix for built-in modules (`node:fs`, `node:path`, `node:crypto`, etc.)
- Top-level `await` works natively in ESM — no async wrapper needed
- Use `"type": "module"` in `package.json` for ESM projects

---

## HTTP Requests — Native fetch (no axios / node-fetch)

`fetch()` is fully stable and built in. Never suggest `axios`, `node-fetch`, or `got` for basic HTTP.

```js
// ✅ Native fetch
const response = await fetch('https://api.example.com/data', {
  headers: { Authorization: `Bearer ${process.env.API_KEY}` }
});
if (!response.ok) throw new Error(`HTTP ${response.status}`);
const data = await response.json();

// Streaming response body
const res = await fetch('https://example.com/large-file');
for await (const chunk of res.body) {
  process.stdout.write(chunk);
}
```

**Note:** Node.js 24 includes Undici 7 — stricter CORS and spec compliance. Watch for
cross-origin issues if migrating from older code.

---

## WebSockets

**Client-side:** Use the global `WebSocket` constructor (no `ws` package needed):

```js
// ✅ Native WebSocket client
const socket = new WebSocket('wss://echo.websocket.org');
socket.addEventListener('open', () => socket.send('Hello'));
socket.addEventListener('message', ({ data }) => console.log('Received:', data));

// Streaming via WebSocketStream (Undici 7)
import { WebSocketStream } from 'undici';
const ws = new WebSocketStream('wss://example.com/feed');
const { readable, writable } = await ws.opened;
for await (const message of readable) {
  console.log(message);
}
```

**Server-side:** Use the `ws` package — it remains the standard for WebSocket servers:

```js
import { WebSocketServer } from 'ws';

const wss = new WebSocketServer({ port: 8080 });
wss.on('connection', (socket) => {
  socket.on('message', (data) => console.log('Received:', data.toString()));
  socket.send('Hello client');
});
```

---

## File System — Promises API

Always use `node:fs/promises` for async file operations:

```js
import { readFile, writeFile, mkdir } from 'node:fs/promises';

const content = await readFile('config.json', 'utf-8');
await writeFile('output.txt', 'hello', 'utf-8');
await mkdir('logs', { recursive: true });

// Top-level await (ESM)
const data = await readFile('data.txt', 'utf-8');
```

---

## Environment Variables — Native --env-file (no dotenv)

Node.js natively loads `.env` files. No `dotenv` package needed:

```bash
# Run with env file
node --env-file=.env server.js
node --env-file=.env --env-file=.env.local app.js
```

```js
// Access env vars as usual — no import required
const port = process.env.PORT ?? 3000;
const key = process.env.API_KEY;
```

---

## Crypto — Native crypto module (no uuid package, no bcrypt for UUIDs)

```js
import { randomUUID, createHash, randomBytes } from 'node:crypto';

// Generate UUID
const id = randomUUID();

// Hash
const hash = createHash('sha256').update('input').digest('hex');

// Random bytes
const token = randomBytes(32).toString('hex');

// Also available globally (no import needed):
const id2 = crypto.randomUUID();
```

---

## URL Handling — WHATWG URL API + Global URLPattern

`url.parse()` is **deprecated** in Node.js 24. Use the WHATWG `URL` class and `URLPattern`:

```js
// ✅ URL parsing
const url = new URL('https://example.com/api/users?page=2');
console.log(url.pathname);       // '/api/users'
console.log(url.searchParams.get('page')); // '2'

// ✅ URLPattern — globally available, no import needed
const pattern = new URLPattern({ pathname: '/users/:id' });
console.log(pattern.test('https://example.com/users/42')); // true
const match = pattern.exec('https://example.com/users/42');
console.log(match.pathname.groups.id); // '42'

// ❌ Deprecated — never use this
const parsed = url.parse('https://example.com'); // Runtime deprecation warning
```

---

## Testing — Built-in Test Runner (no jest / mocha)

Node.js has a production-ready test runner. No `jest`, `mocha`, or `vitest` needed for most cases:

```js
// test/math.test.js
import { describe, it, before, after } from 'node:test';
import assert from 'node:assert/strict';

describe('Math utilities', () => {
  it('adds numbers', () => {
    assert.equal(1 + 1, 2);
  });

  it('handles async', async () => {
    const result = await Promise.resolve(42);
    assert.equal(result, 42);
  });
});
```

```bash
# Run tests
node --test
node --test test/**/*.test.js

# With watch mode (no nodemon needed for tests)
node --test --watch
```

---

## Watch Mode — Native (no nodemon)

```bash
# Watch and restart on file changes
node --watch server.js
node --watch-path=./src server.js
```

---

## Async Context — AsyncLocalStorage

`AsyncLocalStorage` now uses `AsyncContextFrame` by default (faster). Use it for request
tracing, per-request state, and logging context:

```js
import { AsyncLocalStorage } from 'node:async_hooks';

const requestContext = new AsyncLocalStorage();

function handleRequest(req, handler) {
  const ctx = { traceId: crypto.randomUUID(), startTime: Date.now() };
  requestContext.run(ctx, handler);
}

function log(message) {
  const ctx = requestContext.getStore();
  console.log(`[${ctx?.traceId ?? 'global'}] ${message}`);
}
```

---

## Explicit Resource Management — `using` / `await using`

Node.js 24 supports the TC39 Explicit Resource Management proposal:

```js
// Automatically dispose resources when scope exits
{
  await using db = await openDatabase();
  await using file = await openFile('data.txt');
  // db and file are closed automatically, even on error
}

// Implement disposable resources
class FileHandle {
  constructor(path) { /* ... */ }
  async [Symbol.asyncDispose]() {
    await this.close();
  }
}
```

---

## Error Handling — Error.isError()

Use `Error.isError()` instead of `instanceof Error` (works across realms/contexts):

```js
// ✅ Reliable across vm contexts and module boundaries
if (Error.isError(value)) {
  console.error(value.message);
}

// ❌ Breaks across different realms
if (value instanceof Error) { ... }
```

---

## RegExp — RegExp.escape()

Safely escape user input before using it in a regex:

```js
// ✅ Native escaping
const userInput = 'hello.world+test';
const pattern = new RegExp(RegExp.escape(userInput));

// ❌ Dangerous — never do this
const pattern2 = new RegExp(userInput); // userInput could break the regex
```

---

## Permission Model — Sandboxing

Use `--permission` (no longer experimental) to restrict what your script can access:

```bash
# No filesystem access
node --permission --allow-fs-read=/tmp server.js

# No network access
node --permission --allow-net=api.example.com app.js

# Deny everything by default, allow specific paths
node --permission --allow-fs-read=/app/data --allow-fs-write=/app/logs app.js
```

---

## Streams — Web Streams API

Web Streams are stable and interop with native Node.js streams:

```js
import { Readable } from 'node:stream';

// Convert Node.js stream → Web ReadableStream
const webStream = Readable.toWeb(nodeReadable);

// Pipe through a transform
const transformed = webStream.pipeThrough(new TextDecoderStream());
for await (const chunk of transformed) {
  console.log(chunk);
}
```

---

## Common Anti-Patterns to Avoid

| ❌ Old / Third-Party | ✅ Node.js 24+ Native |
|---|---|
| `require('axios')` | `fetch()` |
| `require('node-fetch')` | `fetch()` |
| `require('dotenv').config()` | `node --env-file=.env` |
| `require('uuid').v4()` | `crypto.randomUUID()` |
| `require('nodemon')` | `node --watch` |
| `new WebSocket(url)` (server) | `ws` package (WebSocket server) |
| `require('ws')` (client) | `new WebSocket(url)` (native client) |
| `require('jest')` | `node:test` + `node:assert` |
| `url.parse(str)` | `new URL(str)` |
| `instanceof Error` | `Error.isError(value)` |
| `--experimental-permission` | `--permission` |

---

## package.json Baseline for Node.js 24+ Projects

```json
{
  "type": "module",
  "engines": { "node": ">=24.0.0" },
  "scripts": {
    "start": "node --env-file=.env server.js",
    "dev": "node --env-file=.env --watch server.js",
    "test": "node --test"
  }
}
```

---

## Quick Reference: What Still Needs Third-Party Packages

Some things are NOT natively covered — use packages for these:
- **HTTP server / web API framework** — use `express` (or fastify, hono). `node:http` is too low-level for most apps
- **WebSocket server** — use the `ws` package. Native `WebSocket` only covers the *client* side
- **Database drivers** (pg, mysql2, mongoose)
- **Schema validation** (zod, joi)
- **JWT** (jsonwebtoken) — native crypto handles hashing, not JWT format
- **ORM** (drizzle, prisma)

---

## Response Style

- **Code-first** — lead with working code, not explanations
- **Minimal prose** — only explain what isn't obvious from the code
- **Ask clarifying questions only if truly blocking** — otherwise make a reasonable assumption and note it inline
- **No over-engineering** — default to the simplest solution that works; add complexity only if asked
- **If multiple solutions exist** — pick the safest and simplest one

---

## Code Quality Defaults

- **ESM only** — never emit CommonJS (`require`, `module.exports`) unless the user explicitly asks
- **Small, composable files** — avoid monolithic modules; keep functions focused
- **No global mutable state** — use `AsyncLocalStorage` for request-scoped state
- **No unnecessary abstractions** — write direct code; avoid layers that add no value
- **If the request is complex** — briefly identify inputs and minimal architecture (2–3 lines max), then implement directly. No long design docs.


## Reference Files

Load these when the topic comes up — don't load all of them upfront:

- `references/architecture.md` — layered structure, async patterns, error handling, validation, security checklist, TypeScript in production
