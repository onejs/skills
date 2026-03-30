---
name: one-render-modes
description: Configure render modes (SSG, SSR, SPA, API) in One framework. Use when choosing how pages are rendered, setting up static generation, or mixing render strategies.
version: 1.0.0
license: MIT
---

# Render Modes

One lets you set the render mode globally or per-page. Mix modes freely — a single app can have SSG marketing pages, SSR dashboards, SPA admin panels, and API endpoints.

## Quick Reference

| Mode | Suffix | Renders | Server Needed | SEO |
|------|--------|---------|---------------|-----|
| SSG | `+ssg` | At build time | No (static files) | Yes |
| SSR | `+ssr` | Every request | Yes | Yes |
| SPA | `+spa` | Client only | No | No |
| API | `+api` | On request | Yes | N/A |

## Setting the Mode

### File suffix (per-page)

```
app/
  index+ssg.tsx          Static
  dashboard+spa.tsx      Client-only
  feed+ssr.tsx           Server-rendered
  api/data+api.ts        API endpoint
```

### Global default

```ts
// vite.config.ts
one({
  web: {
    defaultRenderMode: 'ssg', // pages without a suffix use this
  },
})
```

### Folder suffix (all routes in folder)

```
app/
  admin+spa/
    _layout.tsx          SPA
    index.tsx             SPA (inherited)
    users.tsx             SPA (inherited)
```

### Layout render mode

```
app/
  _layout+ssg.tsx        Static shell
  dashboard+spa/
    index.tsx             SPA content
```

A static layout wrapping SPA pages gives you fast initial paint with client-rendered content.

## SSG — Static Site Generation

**Default mode.** Pages are pre-rendered at build time.

Best for: marketing pages, blogs, docs, landing pages.

```tsx
// app/index+ssg.tsx (or just index.tsx if SSG is default)
export async function loader() {
  return { posts: await fetchPosts() }
}

export default function Home() {
  const { posts } = useLoader(loader)
  return <PostList posts={posts} />
}
```

For dynamic SSG routes, export `generateStaticParams`:

```tsx
// app/blog/[slug]+ssg.tsx
export async function generateStaticParams() {
  const posts = await getPosts()
  return posts.map(p => ({ slug: p.slug }))
}

export async function loader({ params }) {
  return getPost(params.slug)
}
```

## SSR — Server-Side Rendering

Pages are rendered on the server for every request.

Best for: personalized content, real-time data, pages that need fresh data.

```tsx
// app/feed+ssr.tsx
import { setResponseHeaders } from 'one'

export async function loader({ request }) {
  const user = await getUser(request)

  setResponseHeaders(headers => {
    headers.set('Cache-Control', 'public, max-age=60')
  })

  return { feed: await getFeed(user.id) }
}
```

**Tip:** Always set `Cache-Control` headers on SSR pages to reduce server load. Use a CDN in front of your server.

## SPA — Single Page Application

Pages are rendered entirely on the client. No server-side rendering.

Best for: authenticated dashboards, admin panels, apps where SEO doesn't matter.

```tsx
// app/admin+spa/index.tsx
export default function Admin() {
  const [data, setData] = useState(null)

  useEffect(() => {
    fetch('/api/admin/stats').then(r => r.json()).then(setData)
  }, [])

  if (!data) return <Loading />
  return <Dashboard data={data} />
}
```

**Pattern:** Combine SPA pages with an SSG layout for a fast static shell:

```
app/
  _layout+ssg.tsx          Static navigation renders instantly
  admin+spa/
    index.tsx              Client-rendered after shell loads
```

## API — HTTP Endpoints

Server-only endpoints that return data, not UI.

```ts
// app/api/users+api.ts
export async function GET() {
  return Response.json(await db.users.findMany())
}

export async function POST(request: Request) {
  const body = await request.json()
  return Response.json(await db.users.create(body), { status: 201 })
}
```

See the `one-api-routes` skill for complete API route patterns.

## Mixing Modes

A real app typically mixes modes:

```
app/
  _layout+ssg.tsx              Static shell (nav, footer)
  index+ssg.tsx                Static home page
  blog/
    index+ssg.tsx              Static blog index
    [slug]+ssg.tsx             Static blog posts
  dashboard+spa/
    _layout.tsx                SPA dashboard
    index.tsx                  Client-rendered
    analytics.tsx              Client-rendered
  settings+ssr.tsx             Server-rendered (needs auth)
  api/
    users+api.ts               API endpoint
    webhooks/stripe+api.ts     Webhook handler
```

## Decision Guide

| Question | Answer → Mode |
|----------|---------------|
| Does the content change per user? | SSR |
| Does it change rarely? | SSG |
| Is SEO irrelevant? | SPA |
| Is it a data endpoint, not a page? | API |
| Need fast initial load + dynamic content? | SSG layout + SPA pages |
| Need fresh data + good SEO? | SSR + Cache-Control headers |
