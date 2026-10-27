# iOS and macOS companion app

SwiftUI app with a macOS menu bar and iOS project/preview management. Shared CLI
modules supply server access, domain models, and automation; keep those concerns
in the CLI modules rather than reimplementing them here.

Run from `app/`:

```bash
tuist generate --no-open
xcodebuild build -workspace TuistApp.xcworkspace -scheme TuistApp
xcodebuild test -workspace TuistApp.xcworkspace -scheme TuistApp
```

- Use SwiftUI for new UI components and follow CLI Swift conventions.
- Only add one-line comments when they explain something non-obvious.
- `TUIST_ENV` selects `development` (local server), `staging`, or `canary`;
  the default is production.
