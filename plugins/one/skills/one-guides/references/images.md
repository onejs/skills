# Images

Official docs: [Features](https://onestack.dev/docs/features), [MDX Guide](https://onestack.dev/docs/guides-mdx)

## `?imagedata` Imports

Import images with automatic dimensions and blur placeholders at build time:

```bash
npm install sharp
```

```tsx
import heroImage from '/hero.jpg?imagedata'

// heroImage = {
//   src: '/hero.jpg',
//   width: 1920,
//   height: 1080,
//   blurDataURL: 'data:image/jpeg;base64,...'
// }

<img {...heroImage} alt="Hero" />
```

Works with public directory (`/images/hero.jpg`) and relative imports (`./avatar.png`).

## Blur Placeholders

The blur placeholder is a tiny 10px-wide base64 JPEG for LQIP (Low Quality Image Placeholder):

```tsx
import hero from '/hero.jpg?imagedata'

<div style={{ position: 'relative' }}>
  <img
    src={hero.blurDataURL}
    style={{ filter: 'blur(20px)', opacity: loaded ? 0 : 1, transition: 'opacity 0.3s' }}
  />
  <img {...hero} onLoad={() => setLoaded(true)} alt="Hero" />
</div>
```

## Programmatic Usage

```tsx
import { getImageData } from 'one/image'

const data = await getImageData('/images/hero.jpg')
// { src, width, height, blurDataURL }
```

## MDX Frontmatter

With `@vxrn/mdx`, frontmatter images are auto-processed:

```yaml
---
image: /images/post-hero.jpg
---
```

Returns `frontmatter.imageMeta` with `{ width, height, blurDataURL }`.

## Supported Formats

JPEG, PNG, WebP, AVIF, GIF, TIFF, SVG (dimensions only).

## Benefits

- Build-time processing, zero runtime cost
- Prevents layout shift (width/height for browser)
- Type-safe with `ImageData` type from `one`
- Works with any image component
