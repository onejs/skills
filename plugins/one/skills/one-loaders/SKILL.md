---
name: one-loaders
description: Server-side data loading in One framework. Use when fetching data for pages, implementing caching, redirects, response headers, ISR, refetching, or file-driven content with loaders.
version: 1.0.0
license: MIT
---

# Loaders

Official docs: [Loaders](https://onestack.dev/docs/routing-loader)

Loaders are server-side data fetching functions that run before rendering. They are tree-shaken from the client bundle — database queries, API keys, and server logic never reach the browser.

## When to Use

- Fetching data for a page (database, external API, filesystem)
- Server-side authentication checks and redirects
- Setting response headers (caching, cookies)
- File-driven content (MDX, JSON) with hot reload

## When NOT to Use

- Client-side interactions (button clicks, form submissions) — use API routes
- Real-time data — use client-side fetching or websockets after initial load

## Basic Usage

```tsx
import { useLoader } from 'one'

export async function loader({ params }) {
  const post = await db.posts.findUnique({ where: { slug: params.slug } })
  return { post }
}

export default function PostPage() {
  const { post } = useLoader(loader)
  return <Text>{post.title}</Text>
}
```

## Loader Arguments

```tsx
export async function loader({ params, path, request }) {
  // params  — dynamic route segments ({ slug: 'hello' })
  // path    — full pathname ('/blog/hello')
  // request — Web API Request (SSR only, undefined in SSG)
}
```

## Return Values

Return any JSON-serializable value:

```tsx
return { user, posts }             // object (most common)
return posts                       // array
return Response.json({ data })     // Response object
```

## Automatic Refetching

Loaders automatically refetch when:
- Pathname changes (`/home` → `/about`)
- Dynamic params change (`/user/1` → `/user/2`)
- Search params change (`?q=hello` → `?q=world`)

## useLoader vs useLoaderState

| Feature | `useLoader` | `useLoaderState` |
|---------|-------------|------------------|
| Returns data | `data` | `{ data, refetch, state }` |
| Manual refetch | No | Yes |
| Loading state | No | `'idle'` or `'loading'` |
| Works without loader arg | No | Yes (access from anywhere) |

### useLoaderState

```tsx
import { useLoaderState } from 'one'

// with loader — replaces useLoader
export default function Page() {
  const { data, refetch, state } = useLoaderState(loader)

  return (
    <>
      {state === 'loading' && <Spinner />}
      <Content data={data} />
      <Button onPress={refetch}>Refresh</Button>
    </>
  )
}
```

### Refetch from Anywhere

All `useLoaderState` hooks on the same route share state. Call `refetch()` from a child component — all subscribers update:

```tsx
// in a header button, sidebar, or any child component
function RefreshButton() {
  const { refetch, state } = useLoaderState()
  return (
    <Button onPress={refetch} disabled={state === 'loading'}>
      Refresh
    </Button>
  )
}
```

### Common Patterns

**Pull-to-refresh:**
```tsx
const { refetch, state } = useLoaderState()

<PullToRefresh onRefresh={refetch} refreshing={state === 'loading'}>
  {children}
</PullToRefresh>
```

**Polling:**
```tsx
const { refetch, state } = useLoaderState(loader)

useEffect(() => {
  const interval = setInterval(() => {
    if (state === 'idle') refetch()
  }, 5000)
  return () => clearInterval(interval)
}, [refetch, state])
```

**Form revalidation:**
```tsx
const { refetch } = useLoaderState()

const handleSubmit = async (data) => {
  await submitForm(data)
  refetch()  // reload page data after mutation
}
```

## Redirects

Throw a redirect to prevent unauthorized content from ever reaching the client:

```tsx
import { redirect } from 'one'

export async function loader({ request }) {
  const user = await getUser(request)
  if (!user) {
    throw redirect('/login')
  }
  return { user }
}
```

Both `throw redirect()` and `return redirect()` work. `throw` stops execution immediately.

**How it works during client-side navigation:** The server detects the redirect, transforms it into a JS module with redirect metadata (not a raw 302), and the client intercepts it before rendering — no sensitive data reaches the client.

## Response Headers and Caching

Set caching, cookies, or custom headers:

```tsx
import { setResponseHeaders } from 'one'

export async function loader() {
  await setResponseHeaders((headers) => {
    headers.set('Cache-Control', 'public, s-maxage=3600, stale-while-revalidate=86400')
  })
  return fetchData()
}
```

Common cache patterns:
- `s-maxage=3600` — CDN caches for 1 hour
- `stale-while-revalidate=86400` — serve stale while revalidating (up to 1 day)
- `max-age=0, must-revalidate` — always revalidate with origin
- `private, no-store` — never cache (user-specific data)

Note: `stale-while-revalidate` requires CDN support (Vercel, CloudFront, Fastly). Cloudflare does not currently support it.

### Cookies

```tsx
await setResponseHeaders((headers) => {
  headers.append('Set-Cookie', `session=${token}; HttpOnly; Secure; SameSite=Strict; Path=/`)
})
```

## Loader Cache

Deduplicate concurrent SSR calls:

```tsx
export function loaderCache(params, request) {
  return {
    key: `post-${params.slug}`,
    ttl: 60,  // seconds
  }
}
```

## File Watching (Hot Reload)

Register a file dependency — when it changes, the loader re-runs and data refreshes without full page reload:

```tsx
import { watchFile } from 'one'
import { readFile } from 'fs/promises'

export async function loader({ params }) {
  const filePath = `./content/${params.slug}.mdx`
  watchFile(filePath)
  return { content: await readFile(filePath, 'utf-8') }
}
```

No-op in production and on the client.

## Route Validation

Validate params before loading:

```tsx
// schema validation
export function validateParams(params) {
  return { slug: z.string().parse(params.slug) }
}

// async validation (check record exists)
export async function validateRoute({ params }) {
  const exists = await db.posts.exists({ slug: params.slug })
  if (!exists) return false  // renders +not-found
  return true
}
```

## Layout Loaders

Layouts can export loaders. Access layout data from pages via `useMatches()`:

```tsx
// app/_layout.tsx
export async function loader() {
  return { user: await getUser() }
}

// app/index.tsx — child page
import { useMatches } from 'one'

const matches = useMatches()
const layoutData = matches[0]?.loaderData  // parent layout data
```

## Static Generation with Loaders

For SSG pages with dynamic routes, export `generateStaticParams`:

```tsx
// app/blog/[slug]+ssg.tsx
export async function generateStaticParams() {
  const posts = await db.posts.findMany()
  return posts.map(post => ({ slug: post.slug }))
}

export async function loader({ params }) {
  return db.posts.find(params.slug)
}
```

## Rendering Mode Behavior

- **SSG**: Loader runs at build time, data baked into HTML
- **SSR**: Loader runs every request, has access to `request` object
- **SPA**: No server-side loader; `useLoaderState` refetch works client-side

## Common Mistakes

**Wrong:** Importing server code at the top level of a component file.
```tsx
// BAD — leaks to client
import { db } from './database'
```

**Right:** Keep server code inside the loader:
```tsx
export async function loader() {
  const { db } = await import('./database')
  return db.query()
}
```

**Wrong:** Returning non-serializable values:
```tsx
return { onClick: () => {}, createdAt: new Date() }
```

**Right:** Return JSON-serializable data:
```tsx
return { createdAt: new Date().toISOString() }
```
