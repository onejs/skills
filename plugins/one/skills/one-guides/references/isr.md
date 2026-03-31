# Incremental Static Regeneration (ISR)

Official docs: [ISR Guide](https://onestack.dev/docs/guides-isr)

Cache SSR pages at the CDN edge for static-like performance.

## How It Works

1. First visitor → SSR renders → CDN caches
2. Subsequent visitors → cached version instantly
3. Cache expires → next visitor gets stale while CDN regenerates in background
4. New version replaces cached one

No one waits for rendering — CDN always serves something.

## Setup

Use `setResponseHeaders` in SSR loaders:

```tsx
// app/blog/[slug]+ssr.tsx
import { setResponseHeaders, useLoader } from 'one'

export async function loader({ params }) {
  await setResponseHeaders((headers) => {
    headers.set('Cache-Control', 'public, s-maxage=3600, stale-while-revalidate=86400')
  })
  return { post: await fetchPost(params.slug) }
}
```

## Cache Header Patterns

| Pattern | Use Case |
|---------|----------|
| `s-maxage=3600, stale-while-revalidate=86400` | Standard (1h cache, 1d stale) |
| `s-maxage=86400, stale-while-revalidate=604800` | Rarely changes (1d cache, 1w stale) |
| `s-maxage=300, stale-while-revalidate=3600` | Frequently updated (5m cache, 1h stale) |
| `private, no-store` | User-specific, never cache |

## Conditional Caching

```tsx
export async function loader({ request }) {
  const isAuthenticated = request?.headers.get('Cookie')?.includes('session=')

  await setResponseHeaders((headers) => {
    headers.set('Cache-Control', isAuthenticated
      ? 'private, no-store'
      : 'public, s-maxage=3600, stale-while-revalidate=86400'
    )
  })
}
```

## CDN Support

| Platform | `stale-while-revalidate` |
|----------|--------------------------|
| Vercel | Full |
| CloudFront | Full |
| Fastly | Full |
| Cloudflare | No — use `s-maxage` only |

## When to Use

| Strategy | Best For |
|----------|----------|
| SSG | Content known at build time |
| ISR (SSR + cache) | Content that changes occasionally, many pages |
| SSR (no cache) | User-specific or real-time content |
