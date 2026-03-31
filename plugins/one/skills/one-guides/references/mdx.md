# MDX

## Setup

```bash
yarn add @vxrn/mdx mdx-bundler
```

`@vxrn/mdx` bundles Rehype plugins with mdx-bundler for:
- Syntax highlighting (refractor)
- GitHub Flavored Markdown
- Smart typography
- Auto-linked headings with slugified IDs
- Image metadata with blur placeholders
- Reading time calculation
- Frontmatter with heading extraction

## Route with MDX

```tsx
// app/docs/[slug].tsx
import { getMDXComponent } from 'mdx-bundler/client'
import { useMemo } from 'react'
import { useLoader } from 'one'

export async function generateStaticParams() {
  const { getAllFrontmatter } = await import('@vxrn/mdx')
  return getAllFrontmatter('data').map(({ slug }) => ({
    slug: slug.replace(/.*docs\//, ''),
  }))
}

export async function loader({ params }) {
  const { getMDXBySlug } = await import('@vxrn/mdx')
  const { frontmatter, code } = await getMDXBySlug('data', params.slug)
  return { frontmatter, code }
}

export default function DocsPage() {
  const { code, frontmatter } = useLoader(loader)
  const Component = useMemo(() => getMDXComponent(code), [code])

  return (
    <View style={{ padding: 20 }}>
      <Text style={{ fontSize: 48, fontWeight: 'bold' }}>{frontmatter.title}</Text>
      <Component components={components} />
    </View>
  )
}
```

Note: Use `await import('@vxrn/mdx')` (dynamic import) because some rehype/remark dependencies are ESM-only.

## Vite Config

```ts
export default defineConfig({
  ssr: {
    noExternal: true,
    external: ['@vxrn/mdx'],
  },
  plugins: [one({ web: { defaultRenderMode: 'ssg' } })],
})
```

## Hot Reload

`@vxrn/mdx` has built-in HMR — MDX file changes auto-refresh without `watchFile`. For custom file reading, use `watchFile()` from `one`.
