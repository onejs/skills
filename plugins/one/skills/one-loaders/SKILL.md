---
name: one-loaders
description: Server-side data loading in One framework. Use when fetching data for pages, implementing caching, redirects, response headers, or file-driven content with loaders.
version: 1.0.0
license: MIT
---

# Loaders

Loaders are server-side data fetching functions that run before rendering. They are tree-shaken from the client bundle — database queries, API keys, and server logic never reach the browser.

## When to Use

- Fetching data for a page (database, external API, filesystem)
- Server-side authentication checks
- Redirecting unauthenticated users
- Setting response headers (caching, cookies)

## When NOT to Use

- Client-side interactions (button clicks, form submissions) — use API routes
- Static data that never changes — hardcode it
- Real-time data — use client-side fetching or websockets

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
  // params  — dynamic route segments ({ slug: 'hello' } for /blog/[slug])
  // path    — full pathname ('/blog/hello')
  // request — Web API Request object (SSR only, undefined in SSG)
}
```

## Return Values

Return any JSON-serializable value:

```tsx
// object (most common)
return { user, posts }

// array
return posts

// Response object (for custom status/headers)
return Response.json({ data }, { status: 200 })
```

## useLoader vs useLoaderState

**`useLoader(loader)`** — returns the data directly:

```tsx
const data = useLoader(loader)
```

**`useLoaderState(loader)`** — returns data with loading state and refetch:

```tsx
const { data, isLoading, error, refetch } = useLoaderState(loader)

// refetch on an interval
useEffect(() => {
  const interval = setInterval(() => refetch(), 30000)
  return () => clearInterval(interval)
}, [])
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

## Response Headers

Set caching, cookies, or custom headers:

```tsx
import { setResponseHeaders } from 'one'

export async function loader() {
  setResponseHeaders(headers => {
    headers.set('Cache-Control', 'public, max-age=3600, s-maxage=86400')
    headers.set('Set-Cookie', 'theme=dark; Path=/')
  })

  return fetchData()
}
```

## Loader Cache

Deduplicate concurrent SSR calls for the same data:

```tsx
export function loaderCache(params, request) {
  return {
    key: `post-${params.slug}`,
    ttl: 60, // seconds
  }
}

export async function loader({ params }) {
  // this only runs once per cache key within the TTL
  return db.posts.find(params.slug)
}
```

## File Watching

Trigger HMR when a file changes (useful for file-driven content):

```tsx
import { watchFile } from 'one'

export async function loader() {
  watchFile('./content/posts.json')
  const posts = JSON.parse(await readFile('./content/posts.json', 'utf-8'))
  return { posts }
}
```

## Route Validation

Validate params before loading:

```tsx
// schema validation
export function validateParams(params) {
  return { slug: z.string().parse(params.slug) }
}

// async validation (e.g., check record exists)
export async function validateRoute({ params }) {
  const exists = await db.posts.exists({ slug: params.slug })
  if (!exists) return false // renders +not-found
  return true
}
```

## Layout Loaders

Layouts can have their own loaders. Child pages access layout data via `useMatches()`:

```tsx
// app/_layout.tsx
export async function loader() {
  const user = await getUser()
  return { user }
}

export default function Layout() {
  const { user } = useLoader(loader)
  return (
    <UserProvider value={user}>
      <Stack />
    </UserProvider>
  )
}
```

```tsx
// app/index.tsx — child page accessing layout data
import { useMatches } from 'one'

export default function Home() {
  const matches = useMatches()
  const layoutData = matches[0]?.data // parent layout's loader data
}
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

## Common Mistakes

**Wrong:** Importing server-only code at the top level of a page component.

```tsx
// BAD — this import leaks to the client
import { db } from './database'

export default function Page() {
  // using db here doesn't work client-side
}
```

**Right:** Keep server code inside the loader — it's tree-shaken automatically.

```tsx
// GOOD — db import is only in the loader, tree-shaken from client
export async function loader() {
  const { db } = await import('./database')
  return db.query()
}
```

**Wrong:** Returning non-serializable values.

```tsx
// BAD — functions, Dates, Maps can't serialize
return { onClick: () => {}, createdAt: new Date() }
```

**Right:** Return JSON-serializable data.

```tsx
// GOOD
return { createdAt: new Date().toISOString() }
```
