---
name: one-api-routes
description: Create API routes in One framework. Use when building HTTP endpoints, webhooks, REST APIs, or server-side logic with +api.ts files.
version: 1.0.0
license: MIT
---

# API Routes

Official docs: [Routing](https://onestack.dev/docs/routing)

API routes are server-only HTTP endpoints. Name files with the `+api.ts` suffix.

## When to Use

- REST API endpoints
- Webhook handlers
- Form submissions
- Server-side operations (database writes, external API calls)
- Authentication endpoints

## When NOT to Use

- Fetching data for a page — use loaders instead
- Client-side logic — use regular components
- Static data — use SSG with loaders

## File Structure

```
app/
  api/
    route+api.ts              GET /api
    users/
      route+api.ts            GET/POST /api/users
      [id]+api.ts             GET/PUT/DELETE /api/users/:id
    webhooks/
      stripe+api.ts           POST /api/webhooks/stripe
```

## Basic Usage

Export named functions for HTTP methods:

```ts
// app/api/health+api.ts
export function GET() {
  return Response.json({ status: 'ok' })
}
```

## HTTP Methods

```ts
// app/api/users/route+api.ts

export async function GET(request: Request) {
  const users = await db.users.findMany()
  return Response.json(users)
}

export async function POST(request: Request) {
  const body = await request.json()
  const user = await db.users.create(body)
  return Response.json(user, { status: 201 })
}
```

```ts
// app/api/users/[id]+api.ts

export async function GET(request: Request, { params }: { params: { id: string } }) {
  const user = await db.users.find(params.id)
  if (!user) {
    return Response.json({ error: 'Not found' }, { status: 404 })
  }
  return Response.json(user)
}

export async function PUT(request: Request, { params }: { params: { id: string } }) {
  const body = await request.json()
  const user = await db.users.update(params.id, body)
  return Response.json(user)
}

export async function DELETE(request: Request, { params }: { params: { id: string } }) {
  await db.users.delete(params.id)
  return new Response(null, { status: 204 })
}
```

## Default Handler

A default export catches all HTTP methods:

```ts
// app/api/proxy+api.ts
export default async function handler(request: Request) {
  // handles GET, POST, PUT, DELETE, etc.
  return fetch(`https://upstream.api${new URL(request.url).pathname}`, {
    method: request.method,
    headers: request.headers,
    body: request.body,
  })
}
```

## Request

Standard Web API `Request` object:

```ts
export async function POST(request: Request) {
  // JSON body
  const json = await request.json()

  // Form data
  const form = await request.formData()

  // Raw text
  const text = await request.text()

  // Query params
  const url = new URL(request.url)
  const q = url.searchParams.get('q')

  // Headers
  const auth = request.headers.get('Authorization')
}
```

## Response

Standard Web API `Response` object:

```ts
// JSON
return Response.json({ data: 'value' })

// with status
return Response.json({ error: 'bad request' }, { status: 400 })

// with headers
return new Response(JSON.stringify(data), {
  status: 200,
  headers: {
    'Content-Type': 'application/json',
    'Cache-Control': 'public, max-age=3600',
  },
})

// redirect
return Response.redirect('/login', 302)

// no content
return new Response(null, { status: 204 })
```

## Typed Routes

Use `Endpoint` type for typed params:

```ts
import type { Endpoint } from 'one'

export type MyEndpoint = Endpoint<'/api/users/[id]'>

export async function GET(
  request: Request,
  { params }: { params: MyEndpoint['params'] }
) {
  // params.id is typed
}
```

Or use `createAPIRoute`:

```ts
import { createAPIRoute } from 'one'

export const GET = createAPIRoute<'/api/users/[id]'>(
  async (request, { params }) => {
    return Response.json({ id: params.id })
  }
)
```

## Common Patterns

### Authentication middleware

```ts
async function requireAuth(request: Request) {
  const token = request.headers.get('Authorization')?.replace('Bearer ', '')
  if (!token) {
    throw Response.json({ error: 'Unauthorized' }, { status: 401 })
  }
  return verifyToken(token)
}

export async function GET(request: Request) {
  const user = await requireAuth(request)
  return Response.json({ user })
}
```

### Error handling

```ts
export async function POST(request: Request) {
  try {
    const body = await request.json()
    const result = await processData(body)
    return Response.json(result)
  } catch (error) {
    return Response.json(
      { error: error instanceof Error ? error.message : 'Internal error' },
      { status: 500 }
    )
  }
}
```

### Streaming response

```ts
export async function GET() {
  const stream = new ReadableStream({
    async start(controller) {
      for await (const chunk of generateChunks()) {
        controller.enqueue(new TextEncoder().encode(chunk))
      }
      controller.close()
    },
  })

  return new Response(stream, {
    headers: { 'Content-Type': 'text/event-stream' },
  })
}
```

## Rules

1. API route files must end with `+api.ts` (or `.js`)
2. API routes are server-only — they never ship to the client bundle
3. Use standard Web API Request/Response — no framework-specific wrappers
4. Dynamic route params use the same `[param]` syntax as page routes
5. Do not export React components from API route files
