# Platform-Adaptive Sections

When the platform is detected in Phase 1, merge these additions into the base templates during Phase 2 and 3 generation. These are **additions** to base templates, not replacements.

## Web

**architecture.md additions:**

```markdown
## Deployment
- Hosting: {provider} (Vercel / Netlify / AWS / self-hosted)
- CDN: {strategy}
- Environment strategy: development / staging / production

## Frontend Architecture
- Routing: {strategy} (file-based / config-based)
- State management: {library}
- CSS strategy: {approach} (Tailwind / CSS Modules / styled-components)
```

**services/api.md additions:**

```markdown
## Pagination
- Strategy: {cursor | offset}
- Default page size: {N}
- Max page size: {N}

## Rate Limiting
- Strategy: {token bucket | sliding window}
- Limits: {N} req/{period} per {scope}
```

## Mobile — Flutter

**architecture.md additions:**

```markdown
## Platform Targets
- iOS minimum: {version}
- Android minimum: {API level}
- Web support: {yes/no}

## Flutter Architecture
- State management: {Riverpod / Bloc / Provider}
- Navigation: {GoRouter / auto_route}
- DI: {get_it / injectable}
- Local storage: {Hive / shared_preferences / drift}
```

**New artifact — services/native.md:**

```markdown
## Native Capabilities
| Capability | Package | Platforms | Permissions |
|-----------|---------|-----------|-------------|
| Camera | camera | iOS, Android | camera |
| Location | geolocator | iOS, Android | location |

## Deep Linking
- Scheme: {app-scheme}://
- Universal links: {domain}/.well-known/apple-app-site-association
- App Links: {domain}/.well-known/assetlinks.json
```

## Mobile — React Native

**architecture.md additions:**

```markdown
## Platform Targets
- iOS minimum: {version}
- Android minimum: {API level}

## React Native Architecture
- Navigation: {React Navigation / Expo Router}
- State: {Zustand / Redux Toolkit / Jotai}
- Styling: {StyleSheet / NativeWind / Tamagui}
- Native modules: {Expo modules / TurboModules}
```

**New artifact — services/native.md:**

```markdown
## Native Capabilities
| Capability | Package | Platforms | Permissions |
|-----------|---------|-----------|-------------|
| Camera | expo-camera | iOS, Android | camera |

## OTA Updates
- Strategy: {Expo Updates / CodePush}
- Channel: {production / preview}
```

## Mobile — Native (iOS/Android)

**architecture.md additions:**

```markdown
## iOS Architecture
- UI: {SwiftUI / UIKit}
- Architecture: {MVVM / TCA / VIPER}
- Networking: {URLSession / Alamofire}
- Persistence: {SwiftData / Core Data / Realm}

## Android Architecture
- UI: {Jetpack Compose / XML Views}
- Architecture: {MVVM / MVI}
- DI: {Hilt / Koin}
- Networking: {Retrofit / Ktor}
```

## Hybrid (Web + Mobile)

Combines Web sections with the relevant mobile platform sections. Additional:

**architecture.md additions:**

```markdown
## Shared Code Strategy
- Shared layer: {API client / business logic / validation}
- Platform-specific: {UI / navigation / native features}
- Code sharing: {monorepo / shared package / API-only}
```
