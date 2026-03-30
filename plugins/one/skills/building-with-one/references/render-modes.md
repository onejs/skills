# Render Modes

One supports four render modes, configurable globally or per-page.

## Modes

| Mode | Suffix | When Rendered | Best For |
|------|--------|---------------|----------|
| SSG | `+ssg` | Build time | Content sites, blogs, docs |
| SSR | `+ssr` | Every request | Dynamic, personalized pages |
| SPA | `+spa` | Client only | Dashboards, admin panels |
| API | `+api` | On request | HTTP endpoints |

## Setting Render Mode

### Per-page (file suffix)

```
app/
  index+ssg.tsx           Static (default, suffix optional)
  dashboard+spa.tsx       Client-only
  feed+ssr.tsx            Server-rendered
  api/health+api.ts       API endpoint
```

### Global default

```ts
// vite.config.ts
one({
  web: {
    defaultRenderMode: 'ssg', // default
  },
})
```

Pages without a suffix use the global default.

### Per-folder

A folder suffix applies to all routes inside:

```
app/
  dashboard+spa/
    _layout.tsx            SPA
    index.tsx              SPA (inherited)
    analytics.tsx          SPA (inherited)
```

### Layout render modes

Layouts can have their own render mode:

```
app/
  _layout+ssg.tsx          Static shell
  dashboard+spa/
    index.tsx              SPA content inside static shell
```

This is powerful: a `_layout+ssg.tsx` wrapping SPA pages gives you a fast static shell with client-rendered content.

## SSG (Static Site Generation)

**Default mode.** Pages are pre-rendered at build time into static HTML.

- Fastest time-to-first-byte (served from CDN)
- No server needed for static-only sites
- Loader runs at build time, data is baked into HTML
- Dynamic params need `generateStaticParams`:

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

## SSR (Server-Side Rendering)

Pages are rendered on the server for every request.

- Fresh data on every request
- Higher server cost, use with CDN caching
- Loader runs on every request with access to `request` object
- Set cache headers with `setResponseHeaders`:

```tsx
import { setResponseHeaders } from 'one'

export async function loader({ request }) {
  setResponseHeaders(headers => {
    headers.set('Cache-Control', 'public, max-age=60')
  })
  return fetchData()
}
```

## SPA (Single Page Application)

Pages are rendered entirely on the client. No server-side rendering.

- No loader runs on server (loaders still work client-side on native)
- Good for authenticated dashboards where SEO doesn't matter
- Smallest server footprint
- Combine with SSG layout for a static shell:

```
app/
  _layout+ssg.tsx        Static navigation shell
  admin+spa/
    index.tsx            Client-rendered admin
    users.tsx            Client-rendered users
```

## API Routes

HTTP endpoints that return data, not UI:

```ts
// app/api/health+api.ts
export function GET() {
  return Response.json({ status: 'ok' })
}
```

See the `one-api-routes` skill for full API route patterns.

## Decision Guide

| Scenario | Mode |
|----------|------|
| Marketing pages, blog, docs | SSG |
| User dashboard, admin panel | SPA |
| Personalized feed, real-time data | SSR |
| Webhook endpoint, data API | API |
| SPA with fast initial load | SPA pages + SSG layout |
