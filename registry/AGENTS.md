# Package registry

Stateless read frontend. Swift package sync belongs to the server's
`Tuist.Registry.Swift.*` modules in `TUIST_MODE=swift_registry_sync`, not this app.
See `registry/README.md` for routes, configuration, and serving behavior.

Run from `registry/`: `mise run install`, `mise run test`,
`mise run format --check`, and `mise run credo`. Tests need no database.

## Contracts

- Keep this service free of databases, sync jobs, S3 writes, and disk caches.
  Hot metadata reads use ETag-revalidated in-memory Cachex entries.
- Mount ecosystem-specific APIs under their ecosystem prefix (`/swift/*`).
  Preserve the legacy `/api/registry/swift/*` surface with `Deprecation` and
  `Sunset` headers; do not add those headers to canonical routes.
- Source archives use `303` redirects to presigned S3 URLs. Serve manifests
  in-process under the registry origin; default `Package.swift` responses include
  alternate-manifest `Link` headers.
- Use `tuist_common/lib/tuist_common/registry/swift/` for S3 key normalization,
  repository URL parsing, and the metadata contract shared with the writer.
- Deploy through `infra/helm/tuist/` and
  `.github/workflows/server-deployment.yml`. Consult current per-environment values
  before changing writer enablement; never run competing scheduled writers against
  the same bucket. Do not assume a rollout or cutover has finished from this file.
- The read frontend requires read access to its registry bucket. The separate
  writer also requires write access and GitHub credentials.

For smoke checks, request `/up`, `/swift/availability` (expect `Content-Version: 1`),
and a known package with `Accept: application/vnd.swift.registry.v1+json`.
