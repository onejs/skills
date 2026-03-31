# Skew Protection

Official docs: [Configuration](https://onestack.dev/docs/configuration)

Handles stale client bundles after deployments.

## The Problem

After deploy, users with open tabs have old JS bundles. Dynamic imports reference chunks that no longer exist → 404s, broken UI, blank pages.

## Error Recovery (Default)

`skewProtection: true` (default) — detects chunk load failures and auto-reloads:

- Catches `vite:preloadError` events
- Catches dynamic import failures (Chrome, Firefox, Safari patterns)
- SessionStorage guard prevents infinite reload loops (10s cooldown)

Always on unless explicitly disabled.

## Proactive Mode

```ts
one({ web: { skewProtection: 'proactive' } })
```

Adds version polling on top of error recovery:

1. Build emits `version.json` with cache key
2. Client polls every 2 minutes
3. When version doesn't match → app marked stale
4. Next navigation does full page load instead of SPA transition

### Events

```ts
window.addEventListener('one-version-update', (e) => {
  console.log('New version:', e.detail.version)
  // show "update available" banner
})

// or check the flag
if (window.__oneVersionStale) { ... }
```

## CDN Edge Caching

With skew protection, One sets CDN cache headers for SSG/SPA pages:

```
cache-control: public, s-maxage=60, stale-while-revalidate=120
```

SSR pages get `no-cache`. If a cached page loads a missing chunk, skew protection reloads → fresh HTML from origin.

## Disabling

```ts
one({ web: { skewProtection: false } })
```

## Configuration

```ts
one({
  web: {
    skewProtection: true,         // error recovery only (default)
    skewProtection: 'proactive',  // + version polling
    skewProtection: false,        // disabled
  },
})
```
