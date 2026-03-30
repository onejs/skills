# Navigation

## Link Component

The primary way to navigate between routes:

```tsx
import { Link } from 'one'

// basic
<Link href="/about">About</Link>

// dynamic route
<Link href={`/blog/${post.slug}`}>
  {post.title}
</Link>

// replace instead of push
<Link href="/login" replace>
  Login
</Link>

// wrap a custom component
<Link href="/settings" asChild>
  <Pressable>
    <Text>Settings</Text>
  </Pressable>
</Link>
```

## Link Prefetching

One prefetches links automatically based on the `linkPrefetch` config:

| Mode | Behavior |
|------|----------|
| `'intent'` | Trajectory-based prediction (default) |
| `'viewport'` | Prefetch when link enters viewport |
| `'hover'` | Prefetch on hover |
| `false` | Disable prefetching |

Configure in `vite.config.ts`:

```ts
one({
  web: {
    linkPrefetch: 'intent',
  },
})
```

## Programmatic Navigation

Use `useRouter()` for imperative navigation:

```tsx
import { useRouter } from 'one'

function LoginButton() {
  const router = useRouter()

  async function handleLogin() {
    await auth.login()
    router.push('/dashboard')
  }

  return <Button onPress={handleLogin} title="Login" />
}
```

### Router Methods

```tsx
const router = useRouter()

router.push('/path')         // push onto stack
router.replace('/path')      // replace current screen
router.back()                // go back
router.canGoBack()           // check if back is possible
router.navigate('/path')     // navigate (smart — push or pop as needed)
```

## Route Params

### Dynamic Params

```tsx
import { useParams } from 'one'

// in app/users/[id].tsx
function UserPage() {
  const { id } = useParams<{ id: string }>()
}
```

### Search Params

```tsx
import { useSearchParams } from 'one'

function SearchPage() {
  const [params, setParams] = useSearchParams<{ q: string }>()
  // params.q contains the query
  // setParams({ q: 'new query' }) updates URL
}
```

## Current Route Info

```tsx
import { usePathname, useSegments } from 'one'

function Breadcrumb() {
  const pathname = usePathname()    // "/blog/my-post"
  const segments = useSegments()    // ["blog", "my-post"]
}
```

## Protected Routes

Gate routes behind authentication:

```tsx
import { Protected, Redirect } from 'one'

// in _layout.tsx
export default function Layout() {
  const { user } = useAuth()

  return (
    <Protected
      isAuthed={!!user}
      fallback={<Redirect href="/login" />}
    >
      <Stack />
    </Protected>
  )
}
```

## Navigation Blocking

Prevent navigation (unsaved changes warning):

```tsx
import { useBlocker } from 'one'

function EditForm() {
  const [dirty, setDirty] = useState(false)

  useBlocker(dirty, {
    message: 'You have unsaved changes. Leave anyway?',
  })

  return <TextInput onChangeText={() => setDirty(true)} />
}
```

## Redirect Helper

Redirect from loaders:

```tsx
import { redirect } from 'one'

export async function loader({ request }) {
  const user = await getUser(request)
  if (!user) {
    throw redirect('/login')
  }
  return { user }
}
```

Or declaratively:

```tsx
import { Redirect } from 'one'

export default function OldPage() {
  return <Redirect href="/new-page" />
}
```

## Scroll Behavior

Control scroll restoration:

```tsx
import { ScrollBehavior } from 'one'

// in _layout.tsx
<ScrollBehavior />
```

Use scroll groups for independent scroll containers:

```tsx
import { useScrollGroup } from 'one'

function Feed() {
  const scrollRef = useScrollGroup('feed')
  return <ScrollView ref={scrollRef}>...</ScrollView>
}
```
