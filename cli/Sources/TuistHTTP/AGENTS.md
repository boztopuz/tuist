# TuistHTTP

- Treat non-2xx file-transfer responses as fatal errors; keep explicit request/resource timeouts.
- Resolve default proxy mode from runtime HTTP settings. Shared sessions must be lazy, reused process-wide, and invalidated when those settings change.
