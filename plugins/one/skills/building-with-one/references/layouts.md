# Layouts

Layout files named `_layout.tsx` wrap child routes. They nest when placed in subdirectories.

## Layout Types

A layout must render exactly one of these:

| Component | Import | Use Case |
|-----------|--------|----------|
| `<Slot />` | `'one'` | No navigation frame, just renders the child |
| `<Stack />` | `'one'` | Native stack navigation (push/pop) |
| `<Tabs />` | `'one'` | Bottom tab navigation |
| `<Drawer />` | `'one/drawer'` | Slide-out drawer navigation |
| `<NativeTabs />` | `'one'` or `'one/native-tabs'` | Platform-native tab bars |

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
import { Stack } from 'one'

export default function Layout() {
  return (
    <Stack>
      <Stack.Screen name="index" options={{ title: 'Home' }} />
      <Stack.Screen name="settings" options={{ title: 'Settings' }} />
      <Stack.Screen name="modal" options={{ presentation: 'modal' }} />
      <Stack.Screen
        name="sheet"
        options={{
          presentation: 'formSheet',
          gestureDirection: 'vertical',
          animation: 'slide_from_bottom',
          headerShown: false,
        }}
      />
    </Stack>
  )
}
```

The `name` must match the filename (without extension, but including groups). Options pass directly to React Navigation NativeStack.

### Header Composition API

For declarative header configuration, use `Stack.Header` and its children inside `Stack.Screen`:

```tsx
<Stack>
  <Stack.Screen name="index">
    <Stack.Header blurEffect="regular">
      <Stack.Header.Title large>Articles</Stack.Header.Title>
      <Stack.Header.SearchBar placeholder="Search..." placement="stacked" />
    </Stack.Header>
  </Stack.Screen>

  <Stack.Screen name="[id]">
    <Stack.Header>
      <Stack.Header.Title>Post</Stack.Header.Title>
      <Stack.Header.BackButton displayMode="minimal" />
      <Stack.Header.Right asChild>
        <ShareButton />
      </Stack.Header.Right>
    </Stack.Header>
  </Stack.Screen>
</Stack>
```

Set a default header for all screens:

```tsx
<Stack>
  <Stack.Header blurEffect="regular">
    <Stack.Header.BackButton displayMode="minimal" />
  </Stack.Header>
  <Stack.Screen name="index" options={{ title: 'Home' }} />
</Stack>
```

See `components.md` for full Stack.Header sub-component props.

### Configuring Screens from Inside Pages

You can render `Stack.Screen` inside individual pages to access page-level data:

```tsx
// app/blog/[slug].tsx
import { Stack } from 'one'

export default function BlogPost() {
  const { post } = useLoader(loader)

  return (
    <>
      <Stack.Screen options={{ title: post.title }} />
      <Text>{post.content}</Text>
    </>
  )
}
```

The tradeoff: layout-level config applies before stack animation; page-level config can use page data.

## Tab Layout

```tsx
import { Tabs } from 'one'

export default function TabLayout() {
  return (
    <Tabs>
      <Tabs.Screen name="home" options={{ title: 'Home', href: '/home' }} />
      <Tabs.Screen name="profile" options={{ title: 'Profile', href: '/profile' }} />
    </Tabs>
  )
}
```

## NativeTabs Layout

Platform-native tab bars (iOS UITabBarController, Android Material BottomNavigation):

```tsx
import { NativeTabs } from 'one'

export default function Layout() {
  return (
    <NativeTabs>
      <NativeTabs.Screen
        name="feed"
        options={{ title: 'Feed', tabBarIcon: () => ({ sfSymbol: 'newspaper' }) }}
      />
      <NativeTabs.Screen
        name="profile"
        options={{ title: 'Profile', tabBarIcon: () => ({ sfSymbol: 'person' }) }}
      />
    </NativeTabs>
  )
}
```

Requires: `@bottom-tabs/react-navigation` and `react-native-bottom-tabs`.

## Drawer Layout

```tsx
import { GestureHandlerRootView } from 'react-native-gesture-handler'
import { Drawer } from 'one/drawer'

export default function Layout() {
  return (
    <GestureHandlerRootView style={{ flex: 1 }}>
      <Drawer>
        <Drawer.Screen name="index" options={{ drawerLabel: 'Home' }} />
        <Drawer.Screen name="settings" options={{ drawerLabel: 'Settings' }} />
      </Drawer>
    </GestureHandlerRootView>
  )
}
```

Requires: `@react-navigation/drawer`, `react-native-gesture-handler`, `react-native-reanimated`.

## Protected Routes

Use `<Protected>` inside any navigator to conditionally show/hide routes:

```tsx
import { Stack, Protected } from 'one'

export default function Layout() {
  const { session } = useAuth()

  return (
    <Stack>
      <Stack.Screen name="login" />
      <Protected guard={!!session}>
        <Stack.Screen name="dashboard" />
        <Protected guard={session?.role === 'admin'}>
          <Stack.Screen name="admin" />
        </Protected>
      </Protected>
    </Stack>
  )
}
```

When `guard` is false, routes are completely removed from the navigator — they won't appear in tab bars, and navigation attempts are blocked.

## Nested Layouts

Layouts nest automatically based on directory structure:

```
app/
  _layout.tsx              Root (Stack)
  index.tsx                /
  (tabs)/
    _layout.tsx            Tab layout (Tabs)
    home.tsx               /home
    profile.tsx            /profile
  settings/
    _layout.tsx            Settings layout (Stack)
    index.tsx              /settings
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

## Accessing Child Page Data from Layout

Use `useMatches()` to read loader data from any route:

```tsx
import { useMatches, Slot } from 'one'

export default function Layout() {
  const matches = useMatches()
  const pageData = matches[matches.length - 1]?.loaderData
  return <Slot />
}
```

## Layout Render Modes

Layouts support render mode suffixes:

```
app/
  _layout+ssg.tsx          Static shell (fast initial load)
  dashboard+spa/
    index.tsx              SPA content inside static shell
```

A `_layout+ssg.tsx` wrapping SPA pages gives a fast static shell with client-rendered content.

## Custom Layouts with withLayoutContext

Make any React Navigation navigator work as a One layout:

```tsx
import { createNativeBottomTabNavigator } from '@bottom-tabs/react-navigation'
import { withLayoutContext } from 'one'

const Navigator = createNativeBottomTabNavigator().Navigator
export const CustomTabs = withLayoutContext(Navigator)
```
