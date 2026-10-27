# Android companion app

Kotlin/Compose app with Hilt, Retrofit, and OAuth PKCE through Chrome Custom Tabs.

## Development

Run from `android/`: `./gradlew assembleDebug`. Release builds use
`./gradlew bundleRelease` and require signing configuration; publishing is handled
by the `release-android` job in `.github/workflows/app-release.yml`.

Debug builds default to production. Select an environment with:

```bash
adb shell am force-stop dev.tuist.app
adb shell am start -n dev.tuist.app/.MainActivity --es environment staging
```

Valid environments are `development`, `staging`, `canary`, and `production`;
their URLs and OAuth client IDs belong in the environment configuration, not here.
The selection persists across launches. Switching environments signs out the user
and restarts the process to recreate Hilt singletons. Release builds always use
production.

## Icons

Keep adaptive background, foreground, and monochrome layers aligned. Render source
SVGs with `rsvg-convert` at mdpi=108, hdpi=162, xhdpi=216, xxhdpi=324, xxxhdpi=432.
Only add one-line comments when they explain something non-obvious.
