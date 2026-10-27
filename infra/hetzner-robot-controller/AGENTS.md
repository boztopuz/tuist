# Hetzner Robot Controller

Reflects Robot inventory into `HetznerBareMetalHost` CRs and fills disk hints/provider IDs. caph owns rescue, installimage, and kubeadm join; do not duplicate that lifecycle here.

- Touch only CRs labeled `app.kubernetes.io/managed-by=hetzner-robot-controller`; leave hand-authored/unmanaged CRs alone.
- Never reap an inventory-missing CR while `spec.consumerRef.name` is set: caph still owns the binding. Operator recovery must drain/unbind the MachineDeployment before inventory cleanup.
- Keep shared-account environment isolation: derive partitions from Cluster `bareMetalCluster` topology variables, match operator `tuist-bm-<env>-…` and caph-rewritten `bm-<clusterCRName>-…` names, longest-prefix first. Stamp `tuist.dev/cluster` and preserve matching host selectors; only fall back to the legacy name-prefix gate when no partition is configured.
- Populate RAID-1 hints only when root-device hints are empty and at least two distinct WWNs are available. Never overwrite operator-selected disk hints; single-disk hosts require explicit hints.
- Keep inventory polling leader-elected. Provider-ID repair targets the bound workload Node, not arbitrary hosts.
- Deployment/RBAC integration: [management manifest](../k8s/mgmt/hetzner-robot-controller.yaml). Operator procedures belong in [onboarding](../k8s/onboarding.md); parent agent access rules still apply.

Run `go test ./...` from this directory. Tests use fake Kubernetes and Robot clients, not live infrastructure.
