# Dev Tools

One includes built-in development tools. Open with **Alt+Space** (Option+Space on Mac).

## Keyboard Shortcuts

| Shortcut | Panel |
|----------|-------|
| `Alt+Space` | Spotlight menu |
| `Alt+S` | SEO Preview |
| `Alt+R` | Route Info |
| `Alt+L` | Loader Timing |
| `Alt+P` | Route Preload |
| `Alt+E` | Error Panel |
| `Escape` | Close panel |

## SEO Preview

Live preview of Google search result appearance, meta tags, and Open Graph tags. Warnings for missing or too-long tags.

## Route Info

Debug current route: pathname, segments, route params, search params.

## Loader Timing

Waterfall visualization of loader performance:
- Module load time (importing the loader)
- Execution time (running the loader function)
- Total time with visual bars

## Route Preload

Shows which routes have been preloaded via hover intent, with loading/loaded/error status and timing.

Note: Only works in production builds (`one build && one serve`).

## Error Panel

Tracks errors from error boundaries and loaders in real-time with error type, route, message, and timestamp.

## Source Inspector

Hold **Option** (Mac) or **Alt** (Windows/Linux) for 0.8 seconds and hover over any element to see its source location. Click to open a picker with:

- **Open** — open topmost component in your editor
- **Path** — copy all source paths to clipboard
- **Selector** — copy a smart CSS selector
- **Record** — start recording user input (for bug reports)
- **Parent list** — click any parent to open that file

## Input Recording

Record user interactions for bug reports or AI debugging. Start from Source Inspector picker or Alt+Space spotlight. 2-second countdown, then recording begins. Tap Option to stop and copy to clipboard.

Output is a compact line-based format with clicks, keyboard input, scroll, focus, and more.

## Configuration

```ts
one({
  devtools: true,   // enable all (default)
  devtools: false,  // disable all
  devtools: {
    inspector: true,
    seoPreview: true,
    routeDebug: true,
    loaderTiming: true,
    routePreload: true,
    errorPanel: true,
  },
})
```
