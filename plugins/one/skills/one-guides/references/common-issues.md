# Common Issues

## SSR Errors

If server rendering fails, try bundling all dependencies:

```ts
// vite.config.ts
{ ssr: { noExternal: true } }
```

This makes Vite transform all deps with esbuild. Downside: can break `__dirname`/`__filename` usage and slow things down. For individual packages, use `patches` instead.

## `SyntaxError: does not provide an export named 'default'`

Add the dependency to `optimizeDeps.exclude`:

```ts
{ optimizeDeps: { exclude: ['problem-package'] } }
```

## `require is not defined`

Try `vite-require` or `vite-plugin-commonjs` plugins.

## React Native Package Issues

Packages built only for Metro may have:
- Flow types in `.js` files
- JSX in `.js` files (not `.jsx`)
- ESM without proper `package.json` declarations
- Missing import extensions

Fix with patches:

```ts
one({
  patches: {
    'react-native-vector-icons': {
      '**/*.js': ['jsx', 'flow'],  // transpile JSX, remove Flow
    },
  },
})
```

## `fetch()` on Native

Relative URLs don't work on native. Use `ONE_SERVER_URL`:

```tsx
// BAD
const res = await fetch('/api/hello')

// GOOD
const res = await fetch(`${process.env.ONE_SERVER_URL}/api/hello`)
```

| Platform | Relative URL | localhost | Local IP |
|----------|-------------|-----------|----------|
| Web | works | works | works |
| iOS Simulator | fails | works | works |
| Real iOS Device | fails | fails | works |

## `Application has not been registered`

Set `native.key` in your vite config:

```ts
one({ native: { key: 'my-app' } })
```

Must match the key in your built native app container.
