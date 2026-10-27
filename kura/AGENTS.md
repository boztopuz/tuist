# Kura

Rust distributed cache mesh. Read `docs/architecture.md` for subsystem interactions;
keep it and `README.md` aligned with protocol, configuration, and deployment changes.
Operational assets have additional guidance in `ops/AGENTS.md`.

## Development

Run from `kura/` after `mise install`. Bazel is the primary build/test path and CI
gate; use Cargo only when Bazel is unavailable.

| Check | Primary command | Cargo fallback |
| --- | --- | --- |
| Compile | `mise run compile` | `mise exec -- cargo build` |
| Unit tests | `mise run test-unit` | `mise exec -- cargo test` |
| Clippy (required) | `mise run clippy` | `mise exec -- cargo clippy --all-targets -- -D warnings` |
| Format check | `mise run format -- --check` | `mise exec -- cargo fmt --check` |

- `mise run format` fixes formatting; its `-- --check` path uses Bazel's rustfmt
  aspect. rules_rs resolves crates from `Cargo.toml`/`Cargo.lock`; update the lockfile
  normally when dependencies change.
- If authorized for `tuist/kura`, `tuist bazel setup` configures the nearest remote
  cache; rerun after changing location. Without access, local Bazel caching works.
- E2E: `docker compose build && mise exec -- shellspec`. Update `spec/e2e/` coverage
  for cache protocol changes, and keep `ops/` assets aligned with runtime config.
- Keep `LICENSE.md`, `CLA.md`, and `cla/` aligned with root contribution policy.

## Rollout and storage safety

- Mixed versions coexist during rolling updates. Preserve backward and forward
  compatibility across one version skew for clients, membership, replication,
  and on-disk formats; prefer additive or negotiated changes.
- Segment/blob files are append-only and reclaimed by unlink, never truncation or
  in-place rewrite. `src/mmap.rs` depends on this for memory safety: truncation
  can SIGBUS live mappings. Revisit that serving path before introducing
  `set_len`/`ftruncate`.
- Node-local optimizations must preserve response bytes/headers and degrade to a
  known-good path so partially rolled fleets remain consistent.
- New dependencies must build in the release Docker image without extra system
  requirements. Ship config/limit changes with matching Helm values in `ops/`.
