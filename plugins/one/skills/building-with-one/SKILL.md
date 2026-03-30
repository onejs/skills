---
name: building-with-one
description: Complete guide for building full-stack React apps with One framework. Covers routing, layouts, navigation, styling, configuration, and cross-platform (web + native) development.
version: 1.0.0
license: MIT
---

# One Framework Guide

One is a full-stack React framework targeting both web and React Native with a single codebase. Built on Vite with file system routing, server-side loaders, and flexible render modes (SSG, SSR, SPA).

## References

Consult these resources as needed:

```
references/
  routing.md              File conventions, dynamic routes, groups, parallel routes, intercepting routes, catch-all, not-found
  layouts.md              _layout.tsx, Stack, Tabs, Drawer, Slot, nested layouts, layout loaders
  navigation.md           Link component, useRouter, typed routes, programmatic navigation, protected routes
  configuration.md        vite.config.ts, one() plugin options, web/native/global settings, env vars, devtools
  render-modes.md         SSG, SSR, SPA per-page and global, folder suffixes, layout render modes
```

## Quick Start

```bash
npx one
```

This scaffolds a new project. Then:

```bash
bun install
bun run dev
```

Access at http://localhost:8081. For native, use Expo Go or build with `one prebuild` and `one run:ios`.

## Project Structure

```
app/
  _layout.tsx          Root layout (Stack, Tabs, Drawer, or Slot)
  index.tsx            Home route (/)
  about.tsx            Static route (/about)
  blog/
    [slug].tsx         Dynamic route (/blog/:slug)
    _layout.tsx        Nested layout
  api/
    route+api.ts       API endpoint (/api)
vite.config.ts         One configuration
```

## Key Rules

- Routes live in the `app/` directory only
- Never co-locate components, types, or utilities in `app/` — keep them in `src/`, `components/`, etc.
- Always have a route that matches `/` (even inside a group route)
- Use `_layout.tsx` for layouts, never name a route `_layout`
- Use `+api.ts` suffix for API routes, not regular route files

## Configuration

All config goes through the Vite plugin in `vite.config.ts`:

```ts
import { one } from 'one/vite'

export default {
  plugins: [
    one({
      web: {
        defaultRenderMode: 'ssg',
        deploy: 'node',
      },
    }),
  ],
}
```

See `./references/configuration.md` for all options.

## Running the App

**Web development:**
```bash
one dev
```

**Native development:**
```bash
# use Expo Go (recommended for quick iteration)
one dev
# scan QR code with Expo Go

# or build native project
one prebuild
one run:ios
one run:android
```

**Production:**
```bash
one build
one serve
```

## Code Style

- Use `process.env.EXPO_OS` instead of `Platform.OS` for platform checks
- Use `import.meta.env.VITE_ENVIRONMENT` for environment checks ('client', 'ssr', 'ios', 'android')
- Prefix client-exposed env vars with `VITE_`, `EXPO_PUBLIC_`, or `ONE_PUBLIC_`
- Add `/// <reference types="one/env" />` to a `.d.ts` file for env var types
- Platform-specific files: `.web.tsx`, `.native.tsx`, `.ios.tsx`, `.android.tsx`

## Navigation

Use the `<Link>` component for navigation:

```tsx
import { Link } from 'one'

<Link href="/about">About</Link>

// dynamic routes
<Link href={`/blog/${slug}`}>Read Post</Link>
```

Use `useRouter()` for programmatic navigation:

```tsx
import { useRouter } from 'one'

const router = useRouter()
router.push('/settings')
router.replace('/login')
router.back()
```

See `./references/navigation.md` for full API.

## Data Loading

Use loaders for server-side data fetching (tree-shaken from client):

```tsx
import { useLoader } from 'one'

export async function loader({ params }) {
  const post = await db.posts.find(params.slug)
  return { post }
}

export default function PostPage() {
  const { post } = useLoader(loader)
  return <Text>{post.title}</Text>
}
```

See the `one-loaders` skill for complete loader patterns.

## Render Modes

Set per-page with file suffixes or globally in config:

- `page+ssg.tsx` — static generation (default)
- `page+ssr.tsx` — server-side rendering
- `page+spa.tsx` — client-side only
- `route+api.ts` — API endpoint

See `./references/render-modes.md` for details.

## Hooks Reference

| Hook | Purpose |
|------|---------|
| `useRouter()` | Programmatic navigation |
| `useParams()` | Dynamic route params |
| `useSearchParams()` | URL query parameters |
| `usePathname()` | Current pathname |
| `useSegments()` | Route segments array |
| `useLoader(loader)` | Access loader data |
| `useLoaderState(loader)` | Loader data with refetch |
| `useMatches()` | All route match data |
| `useNavigation()` | Navigation state |
| `useIsFocused()` | Screen focus state |
| `useActiveParams()` | Active route params |
| `useBlocker()` | Block navigation |
| `useFocusEffect()` | Focus-aware side effects |

## Components Reference

| Component | Purpose |
|-----------|---------|
| `<Link>` | Declarative navigation |
| `<Stack>` | Native stack navigator |
| `<Tabs>` | Tab navigator |
| `<Drawer>` | Drawer navigator |
| `<Slot>` | Render child route (no frame) |
| `<Head>` | Set page head/meta tags |
| `<Redirect>` | Declarative redirect |
| `<SafeAreaView>` | Safe area wrapper |
| `<Protected>` | Auth-gated routes |
| `<LoadProgressBar>` | Loading indicator |
| `<ScrollBehavior>` | Scroll restoration |
