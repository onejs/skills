# Dynamic OpenGraph Images

Generate social sharing preview images with API routes and `@vercel/og`.

## Setup

```bash
npm install @vercel/og
```

## API Route

```tsx
// app/api/og+api.tsx
import { ImageResponse } from '@vercel/og'

export const GET = async (request: Request) => {
  const url = new URL(request.url)
  const title = url.searchParams.get('title') || 'One Framework'

  return new ImageResponse(
    <div style={{
      display: 'flex', alignItems: 'center', justifyContent: 'center',
      width: '100%', height: '100%',
      backgroundColor: '#000', color: '#fff', fontSize: 60, fontWeight: 700,
    }}>
      {title}
    </div>,
    { width: 1200, height: 630 }
  )
}
```

## Use in Pages

```tsx
import { Head, useParams } from 'one'

<Head>
  <meta property="og:image" content={`/api/og?title=${encodeURIComponent(title)}`} />
  <meta property="og:image:width" content="1200" />
  <meta property="og:image:height" content="630" />
  <meta name="twitter:card" content="summary_large_image" />
</Head>
```

## Custom Fonts

```tsx
import { readFile } from 'fs/promises'
import { join } from 'path'

const fontData = readFile(join(process.cwd(), 'public/fonts/Inter-Bold.ttf'))

return new ImageResponse(jsx, {
  width: 1200, height: 630,
  fonts: [{ name: 'Inter', data: await fontData, weight: 700 }],
})
```

## Caching

```tsx
import { setResponseHeaders } from 'one'

await setResponseHeaders((headers) => {
  headers.set('Cache-Control', 'public, s-maxage=86400, stale-while-revalidate=604800')
})
```

Test at `http://localhost:8081/api/og?title=Test`.
