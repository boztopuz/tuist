# Cluster API Provider

Joins Scaleway Apple Silicon, Elastic Metal, Dedibox, and OVH workers to existing clusters. Keep node lifecycle separate from product workload controllers.

## Lifecycle and safety

- Adoption claims pre-ordered capacity; Dedibox/OVH boxes must already have Ubuntu, the fleet SSH key, sudo access, and the adoption marker. Do not add OS installation to their adoption path.
- Machine deletion/release can **wipe and reinstall the host**, destroying node-local volumes and rotating its SSH host key. Pool release is not provider contract termination; monthly/Apple minimum billing remains operator-owned. Do not bypass release finalizers. Dedibox retains its finalizer until reinstall is observed complete.
- Persist claims before bootstrap and exclude hosts claimed by other live CRs. Orphan reclaim is leader-gated and destructive: skip pool hosts, mid-adoption hosts, and hosts owned by live CR name or server ID; require the cluster's claim-name prefix and a fresh ownership recheck before release. Without a claim prefix it is report-only.
- For an explicitly approved Apple Silicon detach-without-wipe recovery: pause the infra CR **before** clearing `status.serverID` and `spec.providerID`, then delete its parent Machine. Pause blocks normal reconciliation, not deletion; deleting with a bound server ID still reinstalls it. Clearing the ID before pausing races adoption.
- Bootstrap must be idempotent and report Ready only after `Node.Ready`. Mint per-machine identities; Linux binds to `system:node`, macOS to `tart-kubelet`. Do not replace these with shared admin credentials.
- Host-config drift hashes must cover rendered scripts, embedded binaries, and fleet config while excluding per-host values. Updates to Ready hosts must preserve running Tart VMs. Tart itself stays version-pinned; upgrade existing hosts by Machine replacement, not hot-swapping the hypervisor under live VMs.
- Keep `fleetName` stable: it selects the fleet SSH key. Respect SSH TOFU pins; clear them only for a known reinstall/bootstrap transition, not arbitrary Ready-host key changes.

## Configuration contracts

- API/schema changes must stay aligned with [chart CRDs](../helm/tuist/crds/). Helm installs `crds/` only on first install; upgrades require an explicitly approved schema apply, not an assumed Helm upgrade.
- Provider credentials are per environment via ESO; SSH keys and kubelet bootstrap material are operator-managed. OVH tokens must match the configured API entity and permit naming/discovery as well as reinstall. Keep one 1Password item per title per vault to avoid ambiguous ESO/prep reads.
- Scaleway permissions include `AppleSiliconFullAccess`, `ElasticMetalFullAccess`, `PrivateNetworksFullAccess`, `IPAMReadOnly`, and `SSHKeysFullAccess` at project scope as applicable; `IAMManager` does not substitute for SSH-key permission.
- Bare-metal prep/mark tasks live in [mise/tasks/baremetal](../../mise/tasks/baremetal/). `PREP_NAMESPACE` selects environment; prep marks adoption eligibility last, and `PREP_SKIP_MARK=1` stages capacity without exposing it to adoption. Preserve explicit zero replicas for cold starts with no adoptable host, or `helm --wait` can block.
- Reproducible operator procedures belong in [onboarding](../k8s/onboarding.md); builder-host setup is in [the builder runbook](../vm-image-builder.md). These are human procedures, not permission to bypass parent access rules.

## Validation

From this directory: `go test ./...` and `go build ./...`. For image changes, the Docker build context is the repository root:

```bash
docker build -f Dockerfile -t ghcr.io/tuist/capi-provider-scaleway-applesilicon:dev ../..
```
