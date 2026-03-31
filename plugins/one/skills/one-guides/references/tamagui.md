# Tamagui

Official docs: [Tamagui Guide](https://onestack.dev/docs/guides-tamagui), [Dark Mode](https://onestack.dev/docs/guides-dark-mode)

SSR-safe cross-platform styling with Tamagui and One.

## Setup

```bash
npm add tamagui @tamagui/config @tamagui/vite-plugin @vxrn/color-scheme
```

### Root Layout

```tsx
// app/_layout.tsx
import { TamaguiProvider } from 'tamagui'
import { SchemeProvider, useUserScheme } from '@vxrn/color-scheme'
import { Slot } from 'one'
import config from '../config/tamagui.config'

export default function Layout() {
  return (
    <SchemeProvider>
      <ThemeProvider>
        <Slot />
      </ThemeProvider>
    </SchemeProvider>
  )
}

const ThemeProvider = ({ children }) => {
  const userScheme = useUserScheme()
  return (
    <TamaguiProvider config={config} defaultTheme={userScheme.value}>
      {children}
    </TamaguiProvider>
  )
}
```

### Config

```tsx
// config/tamagui.config.ts
import { config as configBase } from '@tamagui/config/v3'
import { createTamagui } from 'tamagui'

export const config = createTamagui(configBase)
```

### Vite Plugin

```tsx
// vite.config.ts
import { tamaguiPlugin } from '@tamagui/vite-plugin'

export default {
  plugins: [
    tamaguiPlugin({
      optimize: true,
      components: ['tamagui'],
      config: './config/tamagui.config.ts',
    }),
  ],
}
```

### CSS Extraction (Optional)

Output CSS to a stylesheet for better caching:

```tsx
tamaguiPlugin({
  optimize: true,
  components: ['tamagui'],
  config: './config/tamagui.config.ts',
  outputCSS: './public/tamagui.css',
})
```

Then import it and add `disableInjectCSS`:

```tsx
import '../public/tamagui.css'

<TamaguiProvider config={config} defaultTheme={userScheme.value} disableInjectCSS>
```

## Web Bundle Size

Swap react-native-web for a lighter alternative:

```ts
one({
  alias: {
    web: { 'react-native-web': '@tamagui/react-native-web-lite' },
  },
})
```
