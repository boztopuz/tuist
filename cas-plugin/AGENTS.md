# Xcode CAS plugin

Rust `cdylib` wrapping Apple's `libToolchainCASPlugin.dylib`, plus a per-machine
`tuist-cas-proxy` that owns remote REAPI traffic. Keep compiler-front-end work thin:
fixed per-process costs multiply across thousands of frontends.

## Development

Run `mise run build` from `cas-plugin/` to build both release artifacts. For a
source-built CLI, export absolute `TUIST_CAS_PLUGIN_PATH` and `TUIST_CAS_PROXY_PATH`
pointing to `target/release/libtuist_cas_plugin.dylib` and
`target/release/tuist-cas-proxy`; otherwise generation falls back to local caching.
Run `tuist cache-proxy --url https://tuist.dev`, then `tuist generate` in the project.

- Xcode plugin mode requires explicit Swift modules and no
  `COMPILATION_CACHE_REMOTE_SERVICE_PATH`. Do not benchmark the legacy remote mode
  as if it exercised this plugin.
- Proxy mode is the product path; direct REAPI mode is for debugging/benchmarks.
  With neither configured, the plugin is a local passthrough.
- Plugin options (`tuist-instance`, `tuist-upload`) override environment settings
  and reach Xcode-launched compilers. Consume `tuist-*` options locally, never
  forward them upstream. Upload-disabled builds must still serve read hits.
- Verify instance routing when a benchmark shows mostly misses. Plugin and builtin
  caching use separate local CAS directories; their warm-state numbers differ.

## Correctness and performance contracts

- Never let panics cross the C ABI. Async callbacks fire exactly once, and worker
  in-flight bookkeeping must complete even on panic. Drain workers before disposing
  the upstream CAS or freeing state.
- Allocate returned C strings in this crate; copy and release upstream strings
  immediately. Preserve fallbacks for optional symbols absent from Apple's plugin.
- Publish only on `llcas_actioncache_put_for_digest`, not on every object store.
  Record durable write-ahead state before notifying the proxy; bounded teardown
  must leave unfinished uploads recoverable.
- Publish missing blobs before `UpdateActionResult`. Never expose an action result
  whose graph is incomplete. Suppress identical replay puts without suppressing
  genuinely recomputed values.
- `RESOLVE` runs on llbuild's scheduling thread: return the digest without waiting
  for graph materialization. Register every node's fetch instructions before the
  reply, retain demand-fetch recovery after materialization, and recover from the
  snapshot after proxy restart. Missing objects can fail clang builds.
- Keep snapshots bounded, newest-first, and lower-priority than demand loads.
  Delta watermarks are inclusive and merges idempotent; periodic full refreshes
  must reapply eviction. Old servers without snapshots or inline blobs must keep
  working through per-key lookups and batched reads.
- Bump `src/proxy_proto.rs`'s protocol version for incompatible wire/semantic changes;
  reject stale proxies rather than misparse them.
- Use presence checks, not data loads, on the resolve path. Preserve the REAPI
  digest/frame contract in `src/reapi.rs`; existence probes must not fetch graphs.
- Retry `Internal`/`Cancelled` only for local h2/hyper transport failures, not
  trailer-borne server errors or the client's RPC timeout.
- Auth stays in the CLI (`tuist auth token`); the proxy caches/refetches the bearer.
  Keep analytics SQLite schema compatible with Swift `CASAnalyticsDatabase` and
  the server's xcactivitylog parser.
