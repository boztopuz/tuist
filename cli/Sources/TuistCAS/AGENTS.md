# TuistCAS

- `CacheURLStore` resolves endpoints, honoring `TUIST_CACHE_ENDPOINT`; select the lowest-latency endpoint when several are returned.
- The `kura` client feature flag selects between the Swift per-project compilation-cache daemon (`CASService`/`KeyValueService`) and the Rust `cas-plugin/` transport with `tuist-cas-proxy`.
- Bazel REAPI probing belongs in `TuistREAPI`, not the compilation-cache protocol implementation here.
