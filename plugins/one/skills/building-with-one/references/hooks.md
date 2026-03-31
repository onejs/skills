# Hooks

All hooks are imported from `'one'`.

## useRouter

Imperative navigation. Returns a static object (never updates):

```tsx
const router = useRouter()

router.push('/path')              // push onto stack
router.replace('/path')           // replace current screen
router.navigate('/path')          // smart navigation (push or pop)
router.back()                     // go back
router.canGoBack()                // check if back is possible
router.dismiss()                  // dismiss modal/sheet
router.dismissAll()               // dismiss all modals
router.canDismiss()               // check if dismiss is possible
router.setParams({ key: 'val' })  // update query params
router.subscribe(listener)        // listen to state changes
router.onLoadState(listener)      // listen to loading state
```

### Route Masking (web only)

Display a different URL in the browser while navigating to a specific route:

```tsx
router.push('/photos/5/modal', {
  mask: { href: '/photos/5' }
})
```

### Scroll Control

```tsx
router.push('/next', { scroll: false })  // don't scroll to top
```

## useParams

URL parameters for the focused route. Only updates when the route is focused:

```tsx
const { id, sort } = useParams<{ id: string; sort?: string }>()
```

For catch-all routes (`[...slug].tsx`):

```tsx
const { slug } = useParams<{ slug: string[] }>()
// slug = ['api', 'hooks', 'useParams']
```

Parameters are automatically URL-decoded.

### With createRoute (recommended for typed params)

```tsx
import { createRoute } from 'one'

const route = createRoute<'/posts/[id]'>()

export default function Page() {
  const { id } = route.useParams()  // id is typed as string
}
```

## useActiveParams

Like `useParams` but updates on every navigation, even when the route is not focused. Use for analytics, global UI, or breadcrumbs:

```tsx
const params = useActiveParams()
```

## useSearchParams

URL search parameters as a read-only `URLSearchParams` object:

```tsx
const searchParams = useSearchParams()
searchParams.get('q')       // string | null
searchParams.has('key')     // boolean
searchParams.getAll('tag')  // string[] (for repeated params)
```

Pass `{ global: true }` to update on any route change (not just focused).

To update search params, use the router:

```tsx
const params = new URLSearchParams(searchParams)
params.set('sort', 'date')
router.push(`/products?${params.toString()}`)
```

## usePathname

Current pathname without query or hash:

```tsx
const pathname = usePathname()
// URL: /blog/hello?ref=twitter#comments
// Returns: "/blog/hello"
```

## useSegments

Route segments as an array (matches file structure, not resolved URL):

```tsx
const segments = useSegments()
// File: app/users/[id]/settings.tsx
// Returns: ['users', '[id]', 'settings']  (not ['users', '123', 'settings'])
```

Useful for route matching and auth guards.

## useLoader

Access loader data. Must be in the same file as the loader:

```tsx
export function loader() {
  return { posts: await fetchPosts() }
}

export default function Page() {
  const { posts } = useLoader(loader)
}
```

Automatically refetches when pathname, params, or search params change.

## useLoaderState

Extended loader hook with manual refetch and loading state:

```tsx
// with loader — replaces useLoader
const { data, refetch, state } = useLoaderState(loader)
// state: 'idle' | 'loading'

// without loader — access refetch from any component in the tree
const { refetch, state } = useLoaderState()
```

All `useLoaderState` hooks on the same route share state. Calling `refetch()` from anywhere re-runs the loader for all subscribers.

Use cases: pull-to-refresh, polling, form revalidation, error retry.

## useMatches

All matched routes from root layout to current page:

```tsx
const matches = useMatches()
// Each match: { routeId, pathname, params, loaderData }

// access layout data from a page
const layoutData = matches[0]?.loaderData

// access page data from a layout
const pageData = matches[matches.length - 1]?.loaderData
```

Related: `useMatch(routeId)` finds a specific match, `usePageMatch()` gets the current page match.

## useBlocker

Block navigation when conditions are met (unsaved changes):

```tsx
const blocker = useBlocker(isDirty)

// blocker.state: 'unblocked' | 'blocked' | 'proceeding'
if (blocker.state === 'blocked') {
  blocker.reset()    // cancel navigation, stay
  blocker.proceed()  // allow navigation
  blocker.location   // where user tried to go
}
```

Function-based blocking:

```tsx
const blocker = useBlocker(({ currentLocation, nextLocation, historyAction }) => {
  return currentLocation.startsWith('/edit') && !nextLocation.startsWith('/edit')
})
```

Web: intercepts back/forward, pushState, beforeunload. Native: intercepts back navigation and screen removal.

## useFocusEffect

Like `useEffect` but only runs when the route is focused. Accepts a dependency array directly (no `useCallback` wrapper needed):

```tsx
useFocusEffect(() => {
  const sub = API.subscribe(userId, setUser)
  return () => sub.unsubscribe()
}, [userId])
```

## useIsFocused

Boolean — `true` if the current screen is active:

```tsx
const isFocused = useIsFocused()
```

## useLinkTo

Generate link props for custom components:

```tsx
const linkProps = useLinkTo({ href: '/profile' })
// returns: { href, role: 'link', onPress }

<Pressable {...linkProps}>
  <Text>Go to Profile</Text>
</Pressable>
```

## useNavigation

Low-level React Navigation access. Prefer `useRouter` for most cases:

```tsx
const navigation = useNavigation()

navigation.setOptions({ title: 'New Title' })
navigation.addListener('focus', () => { ... })
navigation.addListener('beforeRemove', (e) => { e.preventDefault() })
navigation.getState()
navigation.dispatch(action)
```

Access parent navigators: `useNavigation('/(tabs)')`.

## useScrollGroup

Register a scroll group in a layout. Child routes preserve scroll position when navigating between them:

```tsx
// app/dashboard/_layout.tsx
export default function Layout() {
  useScrollGroup()  // all /dashboard/* routes share scroll position
  return <Slot />
}
```

Web only — no-op on native.
