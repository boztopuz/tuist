# TuistREAPI

- This is Bazel REAPI (`build.bazel.remote.execution.v2`), not Xcode's compilation-cache protocol. `RemoteCacheProbeService` performs the `GetCapabilities` TLS/auth handshake before handing the URL to Bazel.
- Probe already-resolved URLs only; endpoint resolution/selection belongs to `TuistCAS.CacheURLStore`, command wiring to `TuistBazelCommand`.
- After editing `capabilities.proto`, regenerate checked-in `*.pb.swift`/`*.grpc.swift` with `mise run cli:generate-reapi-proto` from the repository root. The task strips the generator's unsupported `MethodDescriptor` `type:` argument and formats output; generated files are linted. Remove the strip step only when the runtime accepts that argument.
