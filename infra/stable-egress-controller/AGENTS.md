# Stable Egress Controller

Owns active/standby election for the server's Hetzner Floating IP and gateway label; Cilium policy and the host-configurer follow that label.

- Adopt a healthy current active node even outside the candidate pool. Steady state must not churn the Floating IP; fail over only when no healthy active exists, choosing a Ready candidate deterministically.
- Move the Floating IP and active label together and clear stale cluster-wide active labels. Only one gateway may be active.
- Fail closed when the Floating IP is outside the configured egress-IP allowlist. Keep that allowlist aligned with the customer network guide.
- Preserve leader election for multiple replicas and eligibility-only Node event filtering: kubelet heartbeats must not cause Cloud API polling storms.
- Deployment/RBAC is gated by `ciliumEgressGateway.server.failoverController.enabled` in [the platform template](../helm/platform/templates/stable-egress-controller.yaml). Release tag resolution belongs to [k8s:install-platform](../mise/tasks/k8s/install-platform.sh) (run with `mise -C infra` from the repository root), not a hard-coded chart tag.

Run `go test ./...` here. Fake Kubernetes and Floating IP clients cover election, allowlist, event filtering, and reconciliation without live infrastructure.
