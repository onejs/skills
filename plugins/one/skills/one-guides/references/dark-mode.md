# Dark Mode

Official docs: [Dark Mode](https://onestack.dev/docs/guides-dark-mode)

SSR-safe dark mode with `@vxrn/color-scheme` and Tamagui.

## Setup

```bash
npm add tamagui @tamagui/config @vxrn/color-scheme
```

```tsx
// app/_layout.tsx
import { Slot } from 'one'
import { TamaguiProvider, Theme } from '@tamagui/core'
import { config } from '@tamagui/config/v3'
import { MetaTheme, SchemeProvider, useUserScheme } from '@vxrn/color-scheme'

export default function Layout() {
  return (
    <SchemeProvider>
      <MetaTheme
        darkColor={config.themes.dark.color1.val}
        lightColor={config.themes.light.color1.val}
      />
      <TamaguiRoot>
        <Theme name="yellow">
          <Slot />
        </Theme>
      </TamaguiRoot>
    </SchemeProvider>
  )
}

const TamaguiRoot = ({ children }) => {
  const userScheme = useUserScheme()
  return (
    <TamaguiProvider disableInjectCSS config={config} defaultTheme={userScheme.value}>
      {children}
    </TamaguiProvider>
  )
}
```

## Toggle Button

```tsx
import { useUserScheme } from '@vxrn/color-scheme'

const schemeSettings = ['light', 'dark', 'system'] as const

function ThemeToggle() {
  const userScheme = useUserScheme()
  return (
    <Button onPress={() => {
      const next = schemeSettings[(schemeSettings.indexOf(userScheme.setting) + 1) % 3]
      userScheme.set(next)
    }}>
      {userScheme.setting}
    </Button>
  )
}
```

## Key Points

- `SchemeProvider` manages light/dark/system preference
- `MetaTheme` sets the `theme-color` meta tag SSR-safely
- `useUserScheme()` returns `{ value, setting, set }` — value is resolved scheme, setting is user choice
- No flicker on SSR — works with JS disabled
