# CSS Loading

Official docs: [Configuration](https://onestack.dev/docs/configuration), [Features](https://onestack.dev/docs/features)

## Automatic Optimization

One separates CSS into two categories in production:

- **Layout CSS** (from `_layout` files) — loaded before JavaScript to prevent FOUC
- **Page CSS** (from pages) — loaded after scripts

No configuration needed.

## Inline CSS (`.inline.css`)

For CSS that must render before first paint, use `.inline.css`:

```tsx
// app/_layout.tsx
import './critical-styles.inline.css'  // inlined as <style> in HTML
import './non-critical.css'             // loaded via <link> tag
```

Use for: layout structure, above-the-fold colors/fonts, small files (<10KB).
Don't use for: large files, component CSS, post-interaction styles.

## `inlineLayoutCSS` Config

Inlines ALL CSS as `<style>` tags:

```ts
one({ web: { inlineLayoutCSS: true } })
```

| Approach | Effect |
|----------|--------|
| Default | Layout CSS before scripts, page CSS after |
| `.inline.css` | Only marked files inlined |
| `inlineLayoutCSS: true` | Everything inlined |
