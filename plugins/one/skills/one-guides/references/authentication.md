# Authentication

Official docs: [FAQ](https://onestack.dev/docs/faq), [Middlewares](https://onestack.dev/docs/routing-middlewares), [Loaders](https://onestack.dev/docs/routing-loader)

## Approaches

| Approach | Best For |
|----------|----------|
| **Self-hosted** (Better Auth, Auth.js) | Most apps, full control, own database |
| **Hosted** (Clerk, Supabase Auth) | Fastest setup, managed infrastructure |
| **Custom** | Specific requirements only |

## Session State

One doesn't prescribe a session management pattern — use whatever your auth library provides (Better Auth's `useSession`, Clerk's `useAuth`, Supabase's `useUser`, etc). The examples below use a generic `useSession()` hook to show One-specific integration points.

## Protected Routes

### With `<Protected />`

Routes don't exist when unauthorized — hidden from tab bars, navigation blocked:

```tsx
// app/_layout.tsx
import { Stack, Protected } from 'one'

export default function Layout() {
  const { session } = useSession()
  return (
    <Stack>
      <Stack.Screen name="login" />
      <Protected guard={!!session}>
        <Stack.Screen name="dashboard" />
      </Protected>
    </Stack>
  )
}
```

### With Redirect Guard

Routes exist but redirect — supports deep linking → login → return:

```tsx
// app/(app)/_layout.tsx
import { Redirect, Slot } from 'one'

export default function Layout() {
  const { session, isLoading } = useSession()
  if (isLoading) return null
  if (!session) return <Redirect href="/login" />
  return <Slot />
}
```

## Loader Authentication

Protect SSR pages server-side. No data reaches the client on redirect:

```tsx
// app/dashboard+ssr.tsx
import { redirect, useLoader } from 'one'

export async function loader({ request }) {
  const session = await getSession(request)
  if (!session) throw redirect('/login')
  return { user: session.user }
}
```

Use `+ssr` routes for protected pages so loaders check auth on every request.

## Middleware Authentication

Protect API routes:

```tsx
// app/api/_middleware.ts
import { createMiddleware } from 'one'

export default createMiddleware(async ({ request, next, context }) => {
  const token = request.headers.get('authorization')?.replace('Bearer ', '')
  if (!token) return Response.json({ error: 'Unauthorized' }, { status: 401 })
  context.user = await validateToken(token)
  return await next()
})
```

## OAuth (Better Auth Example)

```tsx
// features/auth/server.ts
import { betterAuth } from 'better-auth'

export const auth = betterAuth({
  database: process.env.DATABASE_URL,
  socialProviders: {
    github: {
      clientId: process.env.GITHUB_CLIENT_ID,
      clientSecret: process.env.GITHUB_CLIENT_SECRET,
    },
  },
})

// app/api/auth/[...rest]+api.ts
import { auth } from '~/features/auth/server'
export const GET = (req) => auth.handler(req)
export const POST = (req) => auth.handler(req)
```
