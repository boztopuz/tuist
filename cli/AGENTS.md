# Tuist CLI (Swift)

## Boundaries
- The executable and runtime setup live in `Sources/tuist`. Add new commands and generation logic to feature-specific modules, not the legacy `TuistKit` and `TuistGenerator` monoliths.
- `ProjectDescription` is the public manifest DSL; `TuistLoader` evaluates it. `ProjectAutomation` is the serialized generated-project output, not the manifest DSL. Graph models are embedded under `Sources/XcodeGraph`.
- `TuistCacheEE` is a git submodule; keep its guidance inside the submodule.
- Preserve Noora logger setup for post-command completion/error output so verbose logs capture it.

## Build and test
Run from the repository root. Generate only the targets needed for the change:
```bash
tuist generate tuist ProjectDescription --no-open
xcodebuild build -workspace Tuist.xcworkspace -scheme tuist CODE_SIGNING_ALLOWED=NO CODE_SIGNING_REQUIRED=NO CODE_SIGN_IDENTITY=""
```

If a production or test target is missing, regenerate with it explicitly rather than falling back to SwiftPM:
```bash
tuist generate tuist <ProductionTarget> <TestTarget> ProjectDescription --no-open
xcodebuild test -workspace Tuist.xcworkspace -scheme Tuist-Workspace \
    -only-testing <TestTarget>/<SuiteName> \
    CODE_SIGNING_ALLOWED=NO CODE_SIGNING_REQUIRED=NO CODE_SIGN_IDENTITY=""
```
- `xcsiftbuild test` is also supported. Prefer suites or individual tests over whole targets; a single Swift Testing filter is `ModuleTests/SuiteName/function_name`.
- Do not use `swift build` or `swift test` for CLI validation. SwiftPM is allowed only when explicitly requested or when changing package-resolution behavior; always include `--replace-scm-with-registry` in those cases.
- Lint with `mise run cli:lint`; use `mise run cli:lint --fix` for automatic fixes.

## Test conventions
- Use Swift Testing. For temporary directories, import `FileSystemTesting`, annotate with `@Test(.inTemporaryDirectory)`, and obtain the path with `try #require(FileSystem.temporaryTestDirectory)`.
- Test helpers must not mutate the global process environment.
- Avoid one-line comments unless they add useful context.
