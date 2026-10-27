# Kura Controller

Owns `KuraInstance` / `KuraGateway` product workload lifecycle, not CAPI node lifecycle. Run `go test ./...` here; align API changes with [chart CRDs](../helm/tuist/crds/).

## Routing and availability

- Public HTTP and Bazel REAPI share `spec.publicHost` and the runtime's HTTP/h2c port (`4000`); `grpcPublicHost` is backward-compatible but ignored. Backend Services remain ClusterIP, not per-account LoadBalancers.
- Single-host protocol splitting **requires ingress-nginx**: gRPC method-prefix paths use leading-slash regexes plus `use-regex` and per-location `backend-protocol: GRPC`. Plain Prefix paths do not match REAPI method names. Keep one public-host Certificate; do not restore a separate gRPC certificate/listener.
- cert-manager and the issuer must exist before enabling `--grpc-cluster-issuer`; the controller does not provision the issuer. Keep `KURA_GRPC_PORT=50051` compatibility env until no supported pinned runtime requires it to boot, even though new Services do not route that port.
- Public Services must select one sticky runtime-routable primary, not all replicas: async replication cannot guarantee read-your-writes under fan-out. Keep broad headless discovery for replication. Prefer runtime health (rollout serving, writer lock, ring visibility) and use Kubernetes Ready as fallback when probes are unavailable; hand off to a routable peer without flapping.
- Managed regional pools are single-AZ; hostname spread tolerates node loss, not location loss. Do not imply automatic cross-region failover.

## Isolation and storage

- Preserve peer-replication mTLS and per-instance NetworkPolicy: same-instance traffic, same-account mTLS peers, and cluster-namespaced cache clients only. No public peer gateway/LoadBalancer. Do not default to externally shared peer TLS material until its SANs match advertised `KURA_NODE_URL` discovery names.
- PVC retention is `whenDeleted: Delete`, `whenScaled: Retain`: destroying an instance releases volumes; scaling retains caches.
- Let the runtime derive FD/memory budgets from cgroups and `RLIMIT_NOFILE`; tune Pod memory limits, not hard-coded `KURA_FILE_DESCRIPTOR_POOL_SIZE` / `KURA_MEMORY_*` overrides.
- Preserve managed scrape/OTLP deployment-environment defaults, while `spec.extraEnv` remains authoritative for per-instance overrides.
