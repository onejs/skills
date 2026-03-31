# Components

## Stack

Native stack navigator. Use in `_layout.tsx`:

```tsx
import { Stack } from 'one'

export default function Layout() {
  return (
    <Stack>
      <Stack.Screen name="index" options={{ title: 'Home' }} />
      <Stack.Screen name="[id]" options={{ title: 'Detail' }} />
      <Stack.Screen name="modal" options={{ presentation: 'formSheet' }} />
    </Stack>
  )
}
```

Wraps React Navigation Native Stack. The `name` must match the filename (without extension).

### Stack Header Composition API

Declarative header configuration:

```tsx
<Stack>
  <Stack.Screen name="index">
    <Stack.Header blurEffect="regular">
      <Stack.Header.Title large>Articles</Stack.Header.Title>
      <Stack.Header.SearchBar placeholder="Search..." />
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

**Stack.Header** props:
- `hidden` — hide the header
- `blurEffect` — iOS blur (`'regular'`, `'prominent'`, `'systemMaterial'`, etc.)
- `asChild` — render a custom header component
- `style` — `backgroundColor`, `shadowColor` (set `'transparent'` to hide shadow)
- `largeStyle` — style for large title mode

**Stack.Header.Title** props:
- `children` — title text
- `large` — enable iOS large title
- `style` — `fontWeight`, `fontSize`, `color`, `textAlign`

**Stack.Header.BackButton** props:
- `hidden` — hide back button
- `displayMode` — `'default'`, `'generic'`, `'minimal'`
- `withMenu` — enable long-press menu on iOS
- `children` — custom back button text

**Stack.Header.Left / Stack.Header.Right**:
- `asChild` — required, renders your custom component

**Stack.Header.SearchBar** props:
- `placeholder`, `autoCapitalize` (`'none'`, `'words'`, etc.)
- `placement` — `'automatic'` or `'stacked'`
- `hideWhenScrolling`, `obscureBackground`

Set a default header for all screens:

```tsx
<Stack>
  <Stack.Header blurEffect="regular">
    <Stack.Header.BackButton displayMode="minimal" />
  </Stack.Header>
  <Stack.Screen name="index" options={{ title: 'Home' }} />
</Stack>
```

## Tabs

Bottom tab navigator. Wraps React Navigation Bottom Tabs:

```tsx
import { Tabs } from 'one'

export default function Layout() {
  return (
    <Tabs>
      <Tabs.Screen name="home" options={{ title: 'Home', href: '/home' }} />
      <Tabs.Screen name="profile" options={{ title: 'Profile', href: '/profile' }} />
    </Tabs>
  )
}
```

Note: Set the `href` option on each Tabs.Screen to match the route file name.

## NativeTabs

Platform-native tab bars (iOS UITabBarController, Android Material BottomNavigation):

```bash
npx expo install @bottom-tabs/react-navigation react-native-bottom-tabs
```

```tsx
import { NativeTabs } from 'one'
// or: import { NativeTabs } from 'one/native-tabs'

export default function Layout() {
  return (
    <NativeTabs>
      <NativeTabs.Screen
        name="feed"
        options={{
          title: 'Feed',
          tabBarIcon: () => ({ sfSymbol: 'newspaper' }),
        }}
      />
      <NativeTabs.Screen
        name="profile"
        options={{
          title: 'Profile',
          tabBarIcon: () => ({ sfSymbol: 'person' }),
        }}
      />
    </NativeTabs>
  )
}
```

Supports `NativeTabs.Protected` for auth-gated tabs:

```tsx
<NativeTabs>
  <NativeTabs.Screen name="login" />
  <NativeTabs.Protected guard={isAuthed}>
    <NativeTabs.Screen name="dashboard" />
  </NativeTabs.Protected>
</NativeTabs>
```

## Drawer

Slide-out navigation panel. Requires additional dependencies:

```bash
npm install @react-navigation/drawer react-native-gesture-handler react-native-reanimated
```

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

`drawerType` options: `'front'` (default), `'back'`, `'slide'`, `'permanent'`.

## Slot

Simplest layout — renders the child route directly with no navigation frame:

```tsx
import { Slot } from 'one'

export default function Layout() {
  return <Slot />
}
```

## Head

Renders elements into the document `<head>` on web. On iOS, integrates with Apple Handoff and Spotlight Search:

```tsx
import { Head } from 'one'

<Head>
  <title>My Page</title>
  <meta name="description" content="Page description" />
  <meta property="og:title" content="My Page" />
  <meta property="og:image" content="https://example.com/og.png" />

  {/* iOS: enable Apple Handoff */}
  <meta property="expo:handoff" content="true" />

  {/* iOS: index in Spotlight Search */}
  <meta property="expo:spotlight" content="true" />
</Head>
```

Supports `<title>`, `<meta>`, `<link>`, `<script>`, `<style>`. Multiple `Head` components can be used across the app. On Android, `Head` currently has no effect.

## Protected

Declaratively hide routes based on a condition. When `guard` is false, wrapped screens are completely removed from the navigator:

```tsx
import { Stack, Protected } from 'one'

export default function Layout() {
  const { session } = useAuth()

  return (
    <Stack>
      <Stack.Screen name="login" />
      <Protected guard={!!session}>
        <Stack.Screen name="dashboard" />
        <Stack.Screen name="settings" />
      </Protected>
    </Stack>
  )
}
```

Works with Stack, Tabs, Drawer, and any navigator. Supports nesting — routes require ALL parent guards to be true.

**Protected vs Redirect:**
- `<Protected>` — routes don't exist when unauthorized (hidden from tab bars, navigation blocked)
- `<Redirect>` — routes exist but redirect (better for deep linking → login → return flows)

## Redirect

Declarative redirect. Fires once per mount:

```tsx
import { Redirect } from 'one'

export default function OldPage() {
  return <Redirect href="/new-page" />
}
```

## SafeAreaView

Cross-platform safe area handling. Web uses a lightweight implementation (`@vxrn/safe-area`), native uses `react-native-safe-area-context`:

```tsx
import { SafeAreaView } from 'one'

<SafeAreaView edges={['top', 'bottom']} style={{ flex: 1 }}>
  {/* content */}
</SafeAreaView>
```

Props: `mode` (`'padding'` | `'margin'`), `edges` (array of `'top'`, `'right'`, `'bottom'`, `'left'`).

Hook: `useSafeAreaInsets()` returns `{ top, right, bottom, left }`.

One automatically sets up `SafeAreaProvider` — no additional setup needed.

## ScrollBehavior

Customizes web scroll restoration. One handles this automatically by default (scroll to top on navigation, restore on back/forward, hash scrolling). Add only if you need to change the behavior:

```tsx
import { ScrollBehavior, Slot } from 'one'

export default function Layout() {
  return (
    <>
      <ScrollBehavior />
      <Slot />
    </>
  )
}
```

Props: `disable` (`boolean | 'restore'`).

Disable scroll for individual links: `<Link href="/path" scroll={false}>`.

## LoadProgressBar

Web-only loading progress bar. Add to root layout:

```tsx
import { LoadProgressBar, Slot } from 'one'

export default function Layout() {
  return (
    <>
      <LoadProgressBar startDelay={200} />
      <Slot />
    </>
  )
}
```

Props: `startDelay`, `finishDelay`, `initialPercent`, `updateInterval`, `sporadicness`, `style`.

## withLayoutContext

Create custom layouts from any React Navigation navigator:

```tsx
import { createNativeBottomTabNavigator } from '@bottom-tabs/react-navigation'
import { withLayoutContext } from 'one'

const NativeTabsNavigator = createNativeBottomTabNavigator().Navigator
export const CustomTabs = withLayoutContext(NativeTabsNavigator)
```

Now use `CustomTabs` in any `_layout.tsx`.
