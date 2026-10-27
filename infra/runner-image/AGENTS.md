# macOS Runner Image

Tart Layer 2 over `macos-tahoe-xcode`: one VM per Pod, one ephemeral GitHub Actions job per VM. Keep Xcode/tool installation in [Layer 1](../macos-xcode-image/AGENTS.md).

## Runtime contracts

- Runtime identity/home/workspace remain `runner`, `/Users/runner`, `/Users/runner/work`; absolute paths are shared with GitHub-hosted cache artifacts. `admin` is Packer's provisioning identity, not a runtime service/auto-login target.
- Never bake runner registration credentials. Dispatch supplies a job-scoped JIT; keep `--disableupdate` and the Renovate-managed runner version pin.
- Preserve auto-login, Setup Assistant suppression, and the runner LaunchAgent's `zsh -lc` entrypoint. The live desktop session starts the agent and the login shell supplies the base image's development environment.
- `inject-env.sh` imports the read-only host env share at `/Volumes/My Shared Files/env/tuist.env`; runtime configuration must not be baked into the image.
- Preserve dispatch's exit trap that halts the VM after the single job: tart-kubelet observes VM exit to finish the Pod, enabling GC, billing, and warm-pool refill. Metrics collection is in-VM and best-effort, never a job dependency.
- Account cache branches arrive empty and are materialized only after dispatch identifies the account. Wait for host `cache-ready` **before** inventory/cache access; bound the wait and detach to a private cold cache on timeout so late host materialization cannot corrupt a job. Preserve host byte budgets, post-job `cache-dirty` reporting, and `TUIST_CACHE_SIGNING_GRANT` account-scoped signing across VMs.

## Profiles and validation

- [profiles.json](profiles.json) is the active Xcode build matrix; first entry is default. Keep it aligned with `runnersFleet.xcodeVersions` in [managed common values](../helm/tuist/values-managed-common.yaml). Keep the xcresult processor's `XCODE_VERSION` in [the release workflow](../../.github/workflows/server-production-deployment.yml) at least as new as every active runner profile.
- Release images publish immutable `:macos-<profile>-<semver>` tags alongside rolling profile tags. Deployment resolves the release semver; do not replace reproducible pins with rolling tags. Retired profile tags can remain available but do not receive active-matrix rebuilds.
- Tart builds need bare-metal macOS with a live GUI session, not a hosted/nested VM. Host provisioning and tool ownership: [builder runbook](../vm-image-builder.md).

From this directory:

```bash
packer init runner.pkr.hcl
packer build runner.pkr.hcl
```
