---
name: one-deployment
description: Deploy One framework apps to Node servers, Vercel, Cloudflare Workers, or as static sites. Covers build, serve, cluster mode, security scanning, and platform-specific configuration.
version: 1.0.0
license: MIT
---

# Deployment

Official docs: [one build](https://onestack.dev/docs/one-build), [one serve](https://onestack.dev/docs/one-serve), [Configuration](https://onestack.dev/docs/configuration), [FAQ](https://onestack.dev/docs/faq)

## Build

```bash
# build for web (default)
one build

# build for specific platform
one build web
one build ios
one build android
```

Set `ONE_SERVER_URL` for production:

```bash
ONE_SERVER_URL=https://myapp.com one build
```

Output goes to `dist/` with `client/`, `server/`, and `api/` folders.

### Parallel Builds

One uses worker threads to build static pages across CPU cores. Disable if needed:

```ts
one({ build: { workers: false } })
// or: ONE_BUILD_WORKERS=0 npx one build
```

### Security Scanning

One automatically scans client bundles for leaked secrets (API keys, tokens). Set to `'error'` for production:

```ts
one({
  build: {
    securityScan: 'error',  // 'warn' (default) | 'error' | false
  },
})
```

Detects Anthropic, OpenAI, Stripe, GitHub, AWS keys, bearer tokens, and generic secret patterns.

## Node (Default)

One includes a production Hono server.

```bash
one build
one serve
```

### Cluster Mode

For high-traffic deployments, use cluster mode to fork workers across CPU cores:

```bash
one serve --cluster        # use all CPU cores
one serve --cluster=4      # use 4 workers
```

Each worker handles requests independently with automatic restart on crash.

**When to use cluster:** 200+ concurrent connections, SSR-heavy workloads, multi-core servers.

### Programmatic Usage

```ts
import { serve } from 'one/serve'

await serve({
  port: 3000,
  compress: true,
  cluster: true,
  loadEnv: true,
  // app: customHonoApp  // bring your own Hono
})
```

### Docker

```dockerfile
FROM node:20-slim
WORKDIR /app
COPY package.json bun.lockb ./
RUN npm install --production
COPY dist/ dist/
EXPOSE 3000
CMD ["npx", "one", "serve", "--port", "3000", "--cluster"]
```

## Vercel

### Configuration

```ts
one({
  web: {
    deploy: 'vercel',
  },
})
```

### Required: vercel.json

```json
{
  "cleanUrls": true
}
```

`cleanUrls: true` is required for SSG routes to work correctly.

### Deploy

```bash
# auto-deploy on git push, or:
one build
vercel deploy --prebuilt
```

Build generates `.vercel/output/` with static assets and serverless functions.

### How it works

- SSG pages → static files
- SSR pages → serverless functions
- API routes → serverless functions

## Cloudflare Workers

### Configuration

```ts
one({
  web: {
    deploy: 'cloudflare',
  },
})
```

### Deploy

```bash
one build
cd dist
wrangler deploy
```

Build generates `dist/worker.js` and `dist/wrangler.jsonc`.

### How it works

- Routes are lazy-loaded (loaded on-demand, not all upfront)
- Static assets served from Workers KV or R2
- SSR/API routes run in the Worker

To customize, create `wrangler.jsonc` in the project root — One merges its config with yours.

## Static Export

For SPA or SSG sites with no server-side loaders, just serve `dist/client/`:

```bash
one build
npx serve dist/client
# or any static host: Netlify, GitHub Pages, S3, etc.
```

No `one serve` needed.

## Environment Variables

```bash
# required for loaders and API routes in production
ONE_SERVER_URL=https://myapp.com one build
```

Other env vars: `PORT` (default 3000), `HOST` (default localhost).

## Native Builds

One integrates with existing Expo/React Native build processes:

```bash
# generate native projects
one prebuild

# build and run
one run:ios
one run:android
```

For CI/CD, use [EAS Build](https://docs.expo.dev/build/introduction/) — see the EAS guide.

## Decision Guide

| Scenario | Deploy Target |
|----------|---------------|
| Full control, any cloud | `'node'` (default) |
| Serverless, auto-scaling | `'vercel'` |
| Edge computing, global | `'cloudflare'` |
| Static site, no server | Serve `dist/client/` |
