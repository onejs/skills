# Layouts

Layout files named `_layout.tsx` wrap child routes. They nest when placed in subdirectories.

## Layout Types

A layout must render exactly one of these:

| Component | Use Case |
|-----------|----------|
| `<Slot />` | No navigation frame, just renders the child |
| `<Stack />` | Native stack navigation (push/pop screens) |
| `<Tabs />` | Tab-based navigation |
| `<Drawer />` | Drawer/sidebar navigation |

## Basic Layout

```tsx
// app/_layout.tsx
import { Slot } from 'one'

export default function RootLayout() {
  return <Slot />
}
```

## Stack Layout

```tsx
// app/_layout.tsx
import { Stack } from 'one'

export default function RootLayout() {
  return (
    <Stack
      screenOptions={{
        headerShown: true,
      }}
    />
  )
}
```

Configure individual screens:

```tsx
<Stack>
  <Stack.Screen name="index" options={{ title: 'Home' }} />
  <Stack.Screen name="settings" options={{ title: 'Settings' }} />
  <Stack.Screen name="modal" options={{ presentation: 'modal' }} />
</Stack>
```

## Tab Layout

```tsx
// app/(tabs)/_layout.tsx
import { Tabs } from 'one'

export default function TabLayout() {
  return (
    <Tabs>
      <Tabs.Screen name="index" options={{ title: 'Home' }} />
      <Tabs.Screen name="profile" options={{ title: 'Profile' }} />
    </Tabs>
  )
}
```

## Nested Layouts

Layouts nest automatically based on directory structure:

```
app/
  _layout.tsx              Root layout (Stack)
  index.tsx                / — inside root Stack
  (tabs)/
    _layout.tsx            Tab layout (Tabs)
    home.tsx               /home — inside Tabs, inside Stack
    profile.tsx            /profile — inside Tabs, inside Stack
  settings/
    _layout.tsx            Settings layout (Stack)
    index.tsx              /settings — nested Stack inside root Stack
    account.tsx            /settings/account
```

## Layout Loaders

Layouts can export their own loader:

```tsx
// app/_layout.tsx
import { Stack, useLoader } from 'one'

export async function loader() {
  const user = await getUser()
  return { user }
}

export default function RootLayout() {
  const { user } = useLoader(loader)

  return (
    <UserContext.Provider value={user}>
      <Stack />
    </UserContext.Provider>
  )
}
```

## Accessing Child Page Data

Use `useMatches()` to read loader data from any route in the tree:

```tsx
import { useMatches } from 'one'

export default function Layout() {
  const matches = useMatches()
  // access data from any matched route
  const pageData = matches[matches.length - 1]?.data
  return <Stack />
}
```

## Layout Render Modes

Layouts support render mode suffixes:

```
app/
  _layout+ssg.tsx          Static layout shell
  dashboard+spa/
    _layout.tsx            SPA layout (inherits from folder)
    index.tsx              SPA page
```

A `_layout+ssg.tsx` with SPA children gives you a statically rendered shell with client-rendered content — useful for fast initial load with dynamic pages.
