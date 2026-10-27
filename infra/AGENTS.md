# Infrastructure

## Cluster access for agents

- Default to read-only `kubectl --context tuist-k8s-<env>` through Pomerium. Use `get`, `describe`, and `logs` for diagnosis; infrastructure CRs, nodes, and RBAC objects are readable too. Setup: [Engineer read access](k8s/onboarding.md#engineer-read-access-pomerium-kubeconfig).
- Do not read Kubernetes Secrets or retrieve admin kubeconfigs from 1Password, including asking a human to fetch one for you. The default `view` tier deliberately excludes Secrets. Do not bypass it through another credential path.
- Mutations require explicit human approval. **Never invoke `/elevate` autonomously. Never click Approve on any elevation request**, even one initiated by the human you are assisting. Ask the human to drive both the request and approval.
- A human-issued elevation applies to the identity, so agents may inherit writes during its window. Use it only for the stated intent and time window; do not expand scope.
- Access enforcement lives in [Pomerium RBAC](helm/pomerium/templates/access-tiers.yaml), [kube-impersonator](../kube-impersonator/), and [tuist-ops JIT policy](../tuist-ops/lib/tuist_ops/jit/). Tailscale ACLs are static, code-reviewed policy, not the runtime elevation mechanism.

## Development contracts

- Keep `helm/tuist/` provider-agnostic and self-host-safe by default. Preserve `managedSecrets` / `externalSecrets` gating: self-hosters must not require ESO, 1Password, or managed-provider wiring. Managed deployments layer `values-managed-common.yaml` with the per-environment overlay.
- Managed-only `helm/k8s-monitoring/` must remain independent of self-hosted observability dependencies in the main chart. See its [README](helm/k8s-monitoring/README.md).
- Independently released apps such as Slack and Noora Storybook keep their own charts and workflows, not workloads inside the main server release.
- Kura endpoint/gateway lifecycle belongs to `kura-controller/`, not CAPI node providers. Shared regional Kura ingress is separate from the web ingress; dedicated customer gateways come from `KuraGateway` reconciliation, not customer-specific platform chart aliases.
- Keep CNPG rendering enabled for either `postgresql.cnpg.enabled` or `postgresql.mode == "cnpg"`. Operator-run grants belong in [cnpg/](cnpg/README.md).
- Public `ops.tuist.dev` ingress exposes only Slack-signed webhook routes; the JIT policy endpoint remains tailnet-only. Do not expose the policy API alongside the public webhooks.
- Host-controlled runner VNC credentials stay in `tart-kubelet/` host state. Open the relay only for a host-local request or the server's runner-session Pod annotation.
- Registry routing must preserve `registry.tuist.dev` as the request host; `registry-origin.tuist.dev` is for origin resolution only.
- Grafana Git Sync files use `dashboard.grafana.app/v1` wrappers with raw dashboard JSON under `spec`; `metadata.name` must equal the dashboard UID. UI saves must go through PR review, not direct commits to `main`. Self-host imports use the extracted `spec`.

## Validation and operations references

- Render Helm changes locally with `helm template` before any explicitly approved cluster validation; use leaf instructions for chart-specific placeholder values.
- Put reproducible cluster setup/operator procedures in [k8s/onboarding.md](k8s/onboarding.md), not AGENTS files. [ClusterClass authoring](k8s/clusters/README.md), [platform chart](helm/platform/README.md), and [VM image builders](vm-image-builder.md) cover their respective procedures. Runbooks do not override the agent access restrictions above.
- Managed release orchestration is [server-production-deployment.yml](../.github/workflows/server-production-deployment.yml); pinned single-environment promotions/rollbacks use [server-deployment.yml](../.github/workflows/server-deployment.yml).
- Bootstrap add-on values do not automatically reach existing clusters. HCCM and CSI changes have dedicated [HCCM](../.github/workflows/hccm-deployment.yml) and [CSI](../.github/workflows/csi-deployment.yml) re-apply workflows sharing the server's per-environment deployment concurrency lane.
