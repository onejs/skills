# Routing

One uses file system routing in the `app/` directory. Every file becomes a route unless it starts with `_` (layout/middleware).

## File Conventions

| File | Route |
|------|-------|
| `app/index.tsx` | `/` |
| `app/about.tsx` | `/about` |
| `app/blog/index.tsx` | `/blog` |
| `app/blog/[slug].tsx` | `/blog/:slug` |
| `app/docs/[...rest].tsx` | `/docs/*` (catch-all) |
| `app/+not-found.tsx` | 404 page |
| `app/api/route+api.ts` | `/api` (API endpoint) |

## Dynamic Routes

Single parameter:

```tsx
// app/users/[id].tsx
import { useParams } from 'one'

export default function User() {
  const { id } = useParams<{ id: string }>()
  return <Text>User {id}</Text>
}
```

Catch-all (rest) parameter:

```tsx
// app/docs/[...rest].tsx
import { useParams } from 'one'

export default function Docs() {
  const { rest } = useParams<{ rest: string[] }>()
  return <Text>Path: {rest.join('/')}</Text>
}
```

## Route Groups

Parentheses create groups that nest layouts without affecting the URL:

```
app/
  (auth)/
    _layout.tsx        Auth layout wrapper
    login.tsx          /login
    register.tsx       /register
  (main)/
    _layout.tsx        Main layout wrapper
    index.tsx          /
    settings.tsx       /settings
```

Groups are invisible in the URL. Use them to:
- Apply different layouts to different sets of routes
- Organize routes logically without changing URLs
- Share a layout across routes that aren't in the same directory

## Parallel Routes

Use `@slot` naming for parallel routes (rendering multiple pages simultaneously):

```
app/
  _layout.tsx            Renders @sidebar and default child
  @sidebar/
    index.tsx            Sidebar content
  index.tsx              Main content
```

Access named slots in the layout:

```tsx
// app/_layout.tsx
export default function Layout({ sidebar }) {
  return (
    <View style={{ flexDirection: 'row' }}>
      <View style={{ width: 250 }}>{sidebar}</View>
      <View style={{ flex: 1 }}><Slot /></View>
    </View>
  )
}
```

## Intercepting Routes

Intercept a route to show it in a different context (e.g., modal over current page):

| Convention | Intercepts From |
|------------|-----------------|
| `(.)route` | Same level |
| `(..)route` | One level up |
| `(...)route` | Root level |

```
app/
  feed/
    index.tsx              Feed page
    (.)photo/[id].tsx      Intercepts /photo/:id — shows as modal over feed
  photo/
    [id].tsx               Full photo page (direct navigation)
```

When navigating from `/feed` to `/photo/123`, the intercepting route `(.)photo/[id].tsx` renders (e.g., as a modal). Direct navigation to `/photo/123` renders the full page.

## Not-Found Routes

```tsx
// app/+not-found.tsx
export default function NotFound() {
  return <Text>Page not found</Text>
}
```

Place `+not-found.tsx` at any level to catch unmatched routes in that subtree.

## Render Mode Suffixes

Append render mode to the filename:

```
app/
  index+ssg.tsx        Static (default, suffix optional)
  dashboard+spa.tsx    Client-only
  feed+ssr.tsx         Server-rendered
  api/health+api.ts    API endpoint
```

## Platform-Specific Routes

Use platform extensions for platform-specific implementations:

```
app/
  settings.tsx           Shared (all platforms)
  settings.web.tsx       Web only
  settings.native.tsx    iOS + Android
  settings.ios.tsx       iOS only
  settings.android.tsx   Android only
```

The bundler resolves the correct file per platform. Never import platform-specific files directly.

## Typed Routes

One auto-generates route types to `app/routes.d.ts`. Regenerate with:

```bash
one generate-routes
```

This provides type-safe `href` values in `<Link>` and `useRouter()`.

## Ignored Route Files

Configure patterns to exclude from routing:

```ts
one({
  router: {
    ignoredRouteFiles: ['**/*.test.tsx', '**/_utils/**'],
  },
})
```
