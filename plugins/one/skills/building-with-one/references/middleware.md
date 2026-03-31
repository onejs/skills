# Middleware

Middlewares intercept requests before they reach routes. Place `_middleware.ts` files in any `app/` directory.

## Basic Usage

```ts
// app/_middleware.ts
import { createMiddleware } from 'one'

export default createMiddleware(async ({ request, next }) => {
  console.log(`${request.method} ${request.url}`)
  return await next()
})
```

## API

```ts
type MiddlewareProps = {
  request: Request              // standard Web API Request
  next: () => Promise<Response> // call next middleware/route
  context: Record<string, any> // shared mutable object for passing data
}
```

**Return values:**
- `Response` — stops the chain, returns immediately
- `await next()` — continues to next middleware/route
- `null` or `undefined` — same as calling `next()`

## Hierarchical Execution

Middlewares run root → leaf:

```
Request to /blog/post/123
  1. app/_middleware.ts         (runs first)
  2. app/blog/_middleware.ts    (runs second)
  3. route handler              (runs last)
  Responses bubble back up
```

## Common Patterns

### Auth Protection

```ts
// app/api/_middleware.ts
import { createMiddleware } from 'one'

export default createMiddleware(async ({ request, next, context }) => {
  const token = request.headers.get('authorization')?.replace('Bearer ', '')
  if (!token) {
    return Response.json({ error: 'Unauthorized' }, { status: 401 })
  }
  context.user = await validateToken(token)
  return await next()
})
```

### CORS

```ts
import { createMiddleware, setResponseHeaders } from 'one'

export default createMiddleware(async ({ request, next }) => {
  setResponseHeaders((headers) => {
    headers.set('Access-Control-Allow-Origin', '*')
    headers.set('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS')
    headers.set('Access-Control-Allow-Headers', 'Content-Type, Authorization')
  })
  if (request.method === 'OPTIONS') return new Response(null, { status: 204 })
  return await next()
})
```

### Redirects

```ts
export default createMiddleware(async ({ request, next }) => {
  const url = new URL(request.url)
  if (url.pathname === '/old') return Response.redirect(new URL('/new', url.origin), 301)
  return await next()
})
```

### Sharing Context

```ts
// app/_middleware.ts — set context
context.requestId = crypto.randomUUID()
context.user = await getUserFromToken(token)

// app/api/_middleware.ts — read context from parent
console.log(`Request ${context.requestId} from ${context.user?.id}`)
```

## Platform Support

Server-only. Works on Node, Vercel, Cloudflare. Does not run on native or during static generation.

## Middleware vs Loader Redirects

| | Middleware | Loader Redirects |
|---|---|---|
| Best for | URL rewrites, CORS, API auth | Protecting page data, SSR auth guards |
| Runs on | Every server request | When loader data is fetched |
| Client-side nav | Not involved | Server intercepts, prevents data leak |

For protecting SSR pages, prefer loader redirects. For request-level concerns (CORS, API auth, URL rewrites), use middleware.
