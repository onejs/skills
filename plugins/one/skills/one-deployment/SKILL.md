---
name: one-deployment
description: Deploy One framework apps to Node servers, Vercel, Cloudflare Workers, or as static sites. Covers build, serve, and platform-specific configuration.
version: 1.0.0
license: MIT
---

# Deployment

## Build

```bash
# build for web
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

## Node (Default)

The simplest deployment. One includes a production Hono server.

### Quick start

```bash
one build
one serve
```

### Options

```bash
one serve --host 0.0.0.0 --port 3000 --cluster
```

`--cluster` forks one worker per CPU core for better throughput.

### Deploy anywhere

The `dist/` directory is self-contained. Deploy to any Node host:

```bash
# Railway, Render, Fly.io, etc.
one build
one serve

# or use Node directly
node dist/server/index.js
```

### Docker

```dockerfile
FROM node:20-slim
WORKDIR /app
COPY package.json bun.lockb ./
RUN npm install --production
COPY dist/ dist/
EXPOSE 3000
CMD ["npx", "one", "serve", "--port", "3000"]
```

## Vercel

### Configuration

```ts
// vite.config.ts
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

- SSG pages → static files in `.vercel/output/static/`
- SSR pages → serverless functions in `.vercel/output/functions/`
- API routes → serverless functions

## Cloudflare Workers

### Configuration

```ts
// vite.config.ts
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

### wrangler.jsonc

The build auto-generates this. To customize, create your own `wrangler.jsonc` in the project root — One will merge its output config with yours.

## Static Export

For SPA or SSG sites with no server-side loaders:

```bash
one build
```

Then serve `dist/client/` from any static host (Netlify, GitHub Pages, S3, etc.):

```bash
# example: serve locally
npx serve dist/client
```

No `one serve` needed — just static files.

## Environment Variables

Set `ONE_SERVER_URL` to your production URL before building:

```bash
# required for loaders and API routes to work in production
ONE_SERVER_URL=https://myapp.com one build
```

Other useful env vars:
- `PORT` — server port (default: 3000)
- `HOST` — server host (default: localhost)

## Decision Guide

| Scenario | Deploy Target |
|----------|---------------|
| Full control, any cloud | `'node'` (default) |
| Serverless, auto-scaling | `'vercel'` |
| Edge computing, global | `'cloudflare'` |
| Static site, no server | Serve `dist/client/` |
