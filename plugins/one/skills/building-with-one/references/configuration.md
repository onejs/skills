# Configuration

Official docs: [Configuration](https://onestack.dev/docs/configuration)

All One configuration goes through the Vite plugin in `vite.config.ts`:

```ts
import { one } from 'one/vite'

export default {
  plugins: [
    one({
      // options here
    }),
  ],
}
```

## Web Options

```ts
one({
  web: {
    // render mode for pages without a suffix (default: 'ssg')
    defaultRenderMode: 'ssg' | 'ssr' | 'spa',

    // collapse hydration error logs in dev
    looseHydration: true,

    // server-level redirects
    redirects: [
      { source: '/old', destination: '/new', permanent: true },
    ],

    // deployment target (default: 'node')
    deploy: 'node' | 'vercel' | 'cloudflare',

    // link prefetching strategy (default: 'intent')
    linkPrefetch: 'intent' | 'viewport' | 'hover' | false,

    // inline CSS to reduce requests
    inlineLayoutCSS: true,

    // auto-generate sitemap.xml
    sitemap: true,

    // experimental script loading strategy
    experimental_scriptLoading: 'defer-non-critical' | 'after-lcp',
  },
})
```

## Native Options

```ts
one({
  native: {
    // bundler (default: 'metro', recommended)
    bundler: 'metro' | 'vite',

    // AppRegistry component name
    key: 'MyApp',

    // enable react-native-css-interop
    css: true,
  },
})
```

## Global Options

```ts
one({
  // run before app starts (supports per-environment)
  setupFile: './setup.ts',
  // or: setupFile: { client: './setup.client.ts', ssr: './setup.server.ts' }

  // module aliases (supports per-environment)
  alias: {
    '@/': './src/',
  },

  // node module compatibility patches
  patches: {
    'some-package': true,
  },

  // route file exclusions
  router: {
    ignoredRouteFiles: ['**/*.test.tsx', '**/_utils/**'],
  },

  // react compiler
  react: {
    compiler: true, // or platform/regex/function
  },

  // auto-scan entry points
  optimization: {
    autoEntriesScanning: 'flat' | true | false,
  },

  // built-in devtools
  devtools: true, // enables all, or configure individually

  // secret detection in builds
  build: {
    securityScan: 'warn' | 'error' | false,
  },

  // auto-optimize problematic deps for SSR
  ssr: {
    autoDepsOptimization: true,
  },

  // enforce 'server-only' / 'client-only' guards
  environmentGuards: true,
})
```

## Environment Variables

One loads `.env` files and exposes variables on `process.env` and `import.meta.env`.

**Client-safe prefixes** (exposed to browser/native bundles):
- `VITE_*`
- `EXPO_PUBLIC_*`
- `ONE_PUBLIC_*`

Variables without these prefixes are server-only.

**Built-in variables:**

| Variable | Value |
|----------|-------|
| `ONE_CACHE_KEY` | Per-build random key |
| `ONE_APP_NAME` | From config |
| `ONE_SERVER_URL` | Auto in dev, set manually in prod |
| `ONE_DEFAULT_RENDER_MODE` | From config |
| `VITE_ENVIRONMENT` | 'client', 'ssr', 'ios', 'android' |
| `VITE_NATIVE` | '1' on native, '' on web |
| `EXPO_OS` | 'web', 'ios', 'android' |

**Type support:**

Add to a `.d.ts` file:

```ts
/// <reference types="one/env" />
```

## Devtools

Access devtools in development:

| Shortcut | Panel |
|----------|-------|
| `Alt+Space` | Spotlight menu |
| `Alt+S` | SEO Preview |
| `Alt+R` | Route Debug |
| `Alt+L` | Loader Timing |
| `Alt+P` | Route Preload |
| `Alt+E` | Error Panel |

## Environments

One supports four environments: `client`, `ssr`, `ios`, `android`. Use platform extensions (`.web.tsx`, `.native.tsx`, `.ios.tsx`, `.android.tsx`) for platform-specific code. The bundler resolves the correct file automatically.
