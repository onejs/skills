---
name: building-with-one
description: Complete guide for building full-stack React apps with One framework. Covers routing, layouts, navigation, components, hooks, typed routes, styling, configuration, devtools, and cross-platform (web + native) development.
version: 1.0.0
license: MIT
---

# One Framework Guide

One is a full-stack React framework targeting both web and React Native with a single codebase. Built on Vite with file system routing, server-side loaders, and flexible render modes (SSG, SSR, SPA). For native builds, One supports Metro (recommended) and an experimental Vite bundler.

## References

Consult these resources as needed:

```
references/
  routing.md              File conventions, dynamic routes, groups, parallel routes, intercepting routes, typed routes
  layouts.md              _layout.tsx, Stack (+ Header Composition API), Tabs, Drawer, Slot, NativeTabs, nested layouts
  navigation.md           Link (mask, scroll, asChild), useRouter, useLinkTo, Protected, Redirect, route masking
  components.md           Head (+ iOS Handoff/Spotlight), SafeAreaView, ScrollBehavior, LoadProgressBar, withLayoutContext
  hooks.md                All hooks: useParams, useSearchParams, useActiveParams, useSegments, useMatches, useBlocker, etc.
  configuration.md        vite.config.ts, one() plugin options, web/native/global settings, env vars
  render-modes.md         SSG, SSR, SPA, API per-page and global, folder suffixes, layout render modes
  middleware.md           _middleware.ts, createMiddleware, auth, CORS, redirects, context sharing
  devtools.md             Alt+Space spotlight, SEO preview, route debug, loader timing, source inspector, input recording
```

## Quick Start

```bash
npx one
```

This scaffolds a new project with starter templates. Then:

```bash
bun install
bun run dev
```

Access at http://localhost:8081. For native, use Expo Go or build with `one prebuild` and `one run:ios`.

## Project Structure

```
app/
  _layout.tsx          Root layout (Stack, Tabs, Drawer, or Slot)
  _middleware.ts       Server middleware (optional)
  index.tsx            Home route (/)
  about.tsx            Static route (/about)
  blog/
    [slug].tsx         Dynamic route (/blog/:slug)
    _layout.tsx        Nested layout
  dashboard+spa/       SPA folder (all children are SPA)
    index.tsx
  api/
    route+api.ts       API endpoint (/api)
    users/[id]+api.ts  Dynamic API route
  +not-found.tsx       404 page
vite.config.ts         One configuration
app/routes.d.ts        Auto-generated route types
```

## Key Rules

- Routes live in the `app/` directory only
- Never co-locate components, types, or utilities in `app/` — keep them in `src/`, `components/`, etc.
- Always have a route that matches `/` (even inside a group route)
- Use `_layout.tsx` for layouts — it must render `<Slot>`, `<Stack>`, `<Tabs>`, or `<Drawer>`
- Use `+api.ts` suffix for API routes, not regular route files
- Platform-specific files: `.web.tsx`, `.native.tsx`, `.ios.tsx`, `.android.tsx` — never import these directly, the bundler resolves them
- Pin the `one` version for production stability

## CLI

```bash
one dev [--clean] [--host] [--port] [--debug]  # dev server (web + native)
one build [web | ios | android]                 # production build
one serve [--cluster] [--port]                  # production server (Hono)
one prebuild                                    # generate native Xcode/Android projects
one run:ios                                     # build and run iOS app
one run:android                                 # build and run Android app
one generate-routes [--typed=runtime|type]       # regenerate route types
one patch [--force]                             # apply package patches
```

## Configuration

All config goes through the Vite plugin in `vite.config.ts`:

```ts
import { one } from 'one/vite'

export default {
  plugins: [
    one({
      web: {
        defaultRenderMode: 'ssg',
        deploy: 'node',          // 'node' | 'vercel' | 'cloudflare'
        linkPrefetch: 'intent',  // trajectory-based prefetching
        sitemap: true,
      },
      native: {
        bundler: 'metro',        // recommended for stability
      },
    }),
  ],
}
```

See `./references/configuration.md` for all options.

## Running the App

**Web:**
```bash
one dev
```

**Native with Expo Go** (recommended for quick iteration):
```bash
one dev
# press q, r in terminal to show QR code
# scan with Expo Go app
```

**Native with custom build** (for custom native dependencies):
```bash
one prebuild
one run:ios    # or one run:android
```

**Production:**
```bash
ONE_SERVER_URL=https://myapp.com one build
one serve --cluster  # cluster mode for high traffic
```

## Code Style

- Use `process.env.EXPO_OS` instead of `Platform.OS` for platform checks
- Use `import.meta.env.VITE_ENVIRONMENT` for environment checks ('client', 'ssr', 'ios', 'android')
- Prefix client-exposed env vars with `VITE_`, `EXPO_PUBLIC_`, or `ONE_PUBLIC_`
- Add `/// <reference types="one/env" />` to a `.d.ts` file for typed env vars
- Use `createRoute<'/path/[param]'>()` for fully typed route params and loaders

## Routing Exports

Route files support special exports:

```tsx
// loader — server-side data fetching (tree-shaken from client)
export async function loader({ params, path, request }) { ... }

// generateStaticParams — expand dynamic SSG routes at build time
export async function generateStaticParams() { return [{ slug: 'hello' }] }

// sitemap — control sitemap.xml entry for this route
export const sitemap = { priority: 0.8, changefreq: 'daily', exclude: false }
```

## Devtools

In development, press these shortcuts:

| Shortcut | Panel |
|----------|-------|
| `Alt+Space` | Spotlight menu |
| `Alt+S` | SEO Preview |
| `Alt+R` | Route Debug |
| `Alt+L` | Loader Timing |
| `Alt+P` | Route Preload |
| `Alt+E` | Error Panel |

## Environment Variables

| Variable | Value |
|----------|-------|
| `VITE_ENVIRONMENT` | 'client', 'ssr', 'ios', 'android' |
| `VITE_NATIVE` | '1' on native, '' on web |
| `EXPO_OS` | 'web', 'ios', 'android' |
| `ONE_SERVER_URL` | Auto in dev, set manually in prod |
| `ONE_CACHE_KEY` | Per-build random key |

Client-safe prefixes: `VITE_*`, `EXPO_PUBLIC_*`, `ONE_PUBLIC_*`.

## Platform Support

- **Web**: Browser via Vite
- **iOS**: React Native via Metro (recommended) or Vite
- **Android**: React Native via Metro (recommended) or Vite
- **Not supported**: Windows, Bun runtime for development
