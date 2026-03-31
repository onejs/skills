# Navigation

Official docs: [Navigation](https://onestack.dev/docs/routing-navigation), [useRouter](https://onestack.dev/docs/hooks-useRouter), [useNavigation](https://onestack.dev/docs/hooks-useNavigation)

## Link Component

The primary way to navigate between routes:

```tsx
import { Link } from 'one'

// basic
<Link href="/about">About</Link>

// dynamic route
<Link href={`/blog/${post.slug}`}>{post.title}</Link>

// replace instead of push
<Link href="/login" replace>Login</Link>

// wrap a custom component
<Link href="/settings" asChild>
  <Pressable><Text>Settings</Text></Pressable>
</Link>

// open in new tab (web)
<Link href="https://example.com" target="_blank">External</Link>

// download file (web)
<Link href="/report.pdf" download="report.pdf">Download</Link>
```

### Route Masking (web only)

Display a different URL in the browser while navigating to a specific route:

```tsx
<Link href="/photos/5/modal" mask="/photos/5">
  View Photo
</Link>
```

When the user navigates back/forward, the actual route is restored from history.

### Scroll Control

Disable scroll-to-top for individual navigations:

```tsx
<Link href="/next" scroll={false}>
  Navigate without scrolling
</Link>
```

### Link Props

| Prop | Type | Description |
|------|------|-------------|
| `href` | `Href` | Destination path (typed from file system routes) |
| `asChild` | `boolean` | Forward props to child component |
| `replace` | `boolean` | Replace instead of push |
| `push` | `boolean` | Force push |
| `mask` | `Href` | Display different URL in browser (web only) |
| `scroll` | `boolean` | Control scroll behavior |
| `target` | `string` | Where to open (`'_blank'`, etc.) — web only |
| `rel` | `string` | Link relationship — web only |
| `download` | `string` | Download filename — web only |
| `className` | `string` | HTML class (web) or CSS interop (native) |

## Link Prefetching

One prefetches links automatically. Configure strategy:

| Mode | Behavior |
|------|----------|
| `'intent'` | Trajectory-based prediction (default) |
| `'viewport'` | Prefetch when link enters viewport |
| `'hover'` | Prefetch on hover |
| `false` | Disable prefetching |

```ts
// vite.config.ts
one({ web: { linkPrefetch: 'intent' } })
```

## useRouter

Imperative navigation. Returns a static object (never re-renders):

```tsx
import { useRouter } from 'one'

const router = useRouter()

router.push('/path')              // push
router.replace('/path')           // replace
router.navigate('/path')          // smart push/pop
router.back()                     // go back
router.canGoBack()                // boolean

router.dismiss()                  // dismiss modal
router.dismissAll()               // dismiss all modals
router.canDismiss()               // boolean

router.setParams({ key: 'val' })  // update query params
```

### Route Masking with useRouter

```tsx
router.push('/photos/5/modal', {
  mask: { href: '/photos/5' }
})
```

### Scroll Control with useRouter

```tsx
router.push('/next', { scroll: false })
```

### Subscribe to State

```tsx
const unsub = router.subscribe((state) => {
  console.log('Route changed:', state)
})
```

## useLinkTo

Generate link props for fully custom link components:

```tsx
import { useLinkTo } from 'one'

const linkProps = useLinkTo({ href: '/profile', replace: true })
// returns: { href: string, role: 'link', onPress: Function }

<Pressable {...linkProps}>
  <Text>Profile</Text>
</Pressable>
```

## Typed Routes

One auto-generates route types to `app/routes.d.ts`. Use `createRoute` for fully typed params and loaders:

```tsx
import { createRoute } from 'one'

const route = createRoute<'/docs/[slug]'>()

export const loader = route.createLoader(async ({ params }) => {
  // params.slug is typed as string
  return { doc: await fetchDoc(params.slug) }
})

export default function Page() {
  const { slug } = route.useParams()  // typed
}
```

### Auto-Generation

Enable in config to auto-insert type helpers when route files are created:

```ts
one({
  router: {
    experimental: {
      typedRoutesGeneration: 'runtime',  // inserts createRoute
      // or: 'type' — inserts RouteType only
    },
  },
})
```

### Manual Type Usage

```tsx
import type { RouteType } from 'one'

type Route = RouteType<'/docs/[slug]'>
// Route['Params'] = { slug: string }
// Route['LoaderProps'] = { path: string; params: { slug: string }; request?: Request }
```

### href Helper

Type-check route strings at compile time:

```tsx
import { href } from 'one'

const link = href('/post/hello-world')  // type error if route doesn't exist
```

### Regenerate Types

```bash
one generate-routes                  # types only
one generate-routes --typed=runtime  # + inject createRoute helpers
one generate-routes --typed=type     # + inject RouteType helpers
```

## Protected Routes

Hide routes based on a condition:

```tsx
import { Stack, Protected } from 'one'

<Stack>
  <Stack.Screen name="login" />
  <Protected guard={!!session}>
    <Stack.Screen name="dashboard" />
    <Protected guard={isAdmin}>
      <Stack.Screen name="admin" />
    </Protected>
  </Protected>
</Stack>
```

Works with Stack, Tabs, Drawer. Routes are completely removed when `guard` is false.

## Redirect

### Declarative

```tsx
import { Redirect } from 'one'

if (!user) return <Redirect href="/login" />
```

Fires once per mount.

### From Loaders

```tsx
import { redirect } from 'one'

export async function loader({ request }) {
  const user = await getUser(request)
  if (!user) throw redirect('/login')
  return { user }
}
```

Both `throw redirect()` and `return redirect()` work. `throw` stops execution immediately. During client-side navigation, the redirect is intercepted before the protected page ever renders — no data leaks.

## Scroll Behavior

One handles scroll restoration automatically:

- New navigation → scroll to top
- Back/forward → restore saved position
- Hash in URL → scroll to element
- Session persistence via sessionStorage

Customize with `<ScrollBehavior />` or `useScrollGroup()` in layouts. See components reference.

## Navigation Blocking

Prevent navigation for unsaved changes:

```tsx
import { useBlocker } from 'one'

const blocker = useBlocker(hasUnsavedChanges)

if (blocker.state === 'blocked') {
  // show confirmation dialog
  blocker.reset()    // stay
  blocker.proceed()  // leave
}
```
