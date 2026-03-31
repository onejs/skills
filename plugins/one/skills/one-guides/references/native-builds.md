# Native Builds

Official docs: [one dev](https://onestack.dev/docs/one-dev), [iOS Native Guide](https://onestack.dev/docs/guides-ios-native), [EAS Guide](https://onestack.dev/docs/guides-eas), [Metro Mode](https://onestack.dev/docs/metro-mode)

## Quick Development (Expo Go)

No build needed — install Expo Go on your device:

```bash
one dev
# press q, r in terminal to show QR code
# scan with Expo Go
```

Works for most `expo-*` packages without custom native code.

## Custom Native Builds

When you need custom native dependencies:

### Prebuild

```bash
one prebuild    # generate Xcode/Android projects
one run:ios     # build and run iOS
one run:android # build and run Android
```

### Xcode (Manual)

```bash
open ios/*.xcworkspace
```

Select simulator/device → click Run.

### Production Archive

1. Open `.xcworkspace` in Xcode
2. Set up signing (Signing & Capabilities → select team)
3. Product → Archive
4. Distribute App → upload to App Store/TestFlight

## EAS Build

Cloud builds without local Xcode/Android Studio:

```bash
eas build:configure
eas build
eas submit
```

### Setup

1. Install `expo` in your project
2. Add `react-native.config.cjs`:
   ```js
   module.exports = {
     commands: [...require('vxrn/react-native-commands')],
   }
   ```
3. Add to `app.json`:
   ```json
   { "expo": { "plugins": ["vxrn/expo-plugin"] } }
   ```
4. Add postinstall to `package.json`:
   ```json
   { "scripts": { "postinstall": "one patch" } }
   ```

### Node Version

EAS defaults to old Node. Fix in `eas.json`:

```json
{
  "build": {
    "shared": { "node": "20.15.0" },
    "development": { "extends": "shared" },
    "production": { "extends": "shared" }
  }
}
```

## Metro Mode

Recommended for production native builds:

```ts
one({ native: { bundler: 'metro' } })
```

Maximum compatibility with all React Native packages. Lazy startup available:

```ts
one({ native: { bundler: 'metro', bundlerOptions: { startup: 'lazy' } } })
```

## Troubleshooting

**Missing `.xcworkspace`:** Run `cd ios && pod install`

**`node: No such file or directory` in Xcode:** Delete `ios/.xcode.env.local`

**`No Metro config found`:** Ensure `react-native.config.cjs` exists and `vxrn/expo-plugin` is in `app.json`

**`Application has not been registered`:** Set `native.key` in config to match your app container name
