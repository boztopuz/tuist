# Runners Controller

Reconciles `RunnerPool` Pods/per-Pod SAs and desired replicas. Run `go test ./...` here; keep API changes aligned with [chart CRDs](../helm/tuist/crds/).

## Lifecycle and capacity

- Only idle Pods (without `tuist.dev/runner-pool-owner`) may be reaped for scale-down/image replacement. Preserve the idle guard for stale Pending Pods: Linux can be Pending while polling or just after a claim.
- Keep `tuist.dev/runner-pool-drain` finalization: pool deletion/rename must reap idle Pods and wait for busy jobs before releasing the CR, rather than letting owner-reference GC kill them.
- Billing ends at the `runner` container's `terminated.finishedAt`, not poller/sidecar lifetime. Before reaping, preserve exit code/reason and abnormal-end runner log tails in durable controller logs; clean runner exit is not the workflow's success status.
- Autoscaling is Pod-level only. Monthly bare-metal host capacity remains operator-managed through CAPI topology, not auto-ordering or patching management-cluster MachineDeployments from this controller.
- Allocate shared `(OS, FleetSelector)` capacity to real load first, warm floors second, speculative buffer last. Real queued/claimed load remains inviolable even over capacity (excess stays Pending); idle capacity may fall below warm floors under contention.
- Linux capacity is allocatable memory minus reserve, costed by Pod memory; macOS is one VM per Mac mini, shared across Xcode pools. Fleet-view failures must fall back to per-pool targets, never mass scale-down; signal-fetch failures leave replicas unchanged.
- Preserve target-versus-allocation/warm-deficit metrics and clear series on deletion/autoscaling opt-out. Keep phase counts so Pending cold boots remain observable.

## Linux isolation and Docker

- Disable default SA token automount. Only trusted poller/metrics containers mount the projected `tuist-runners-dispatch` token; customer `runner` gets job-scoped JIT via shared emptyDir, **never** the pool-scoped dispatch token.
- Poller runs only trusted dispatch code as root, stages JIT, and exits before `run-job.sh` runs customer code. No staged JIT is a successful drain/abort. Ship/pin split-capable runner images before the controller's split-Pod shape; older pollers could otherwise execute the job while holding the token.
- Privileged dind must fail closed unless `spec.runtimeClass == kata-qemu`; never allow host-runc fallback. Runner stays unprivileged, dind image stays digest-pinned, and native sidecars require Kubernetes >=1.29.
- Dind's `docker info` startup probe gates poller and runner startup. Keep shared socket GID `123` and identical workspace paths on both sides for Docker bind mounts.
- Docker overlay2 cannot use virtio-fs as its upper layer. Keep `/var/lib/docker` on a loop-mounted ext4 sparse disk backed by node-disk emptyDir, not directly on virtio-fs or RAM-consuming tmpfs. Do not pass C virtiofsd `--xattrmap` to Rust virtiofsd.
- Keep both dockerd's shell `ulimit -n 1048576` and spawned-container `--default-ulimit nofile=1048576:1048576`; either alone leaves a low FD limit. Preserve guest `psi=1` for vitals sampling.
- Machine metrics are sampled inside the VM, not kubelet Summary API. Linux uses a trusted metrics sidecar holding the token and waits for JIT; macOS samples inside its single-container Tart VM. Sampling must never block jobs.
- Keep `runnersFleetLinux.shapes` aligned with `:runner_linux_shapes` in [server config](../../server/config/config.exs).

Host provisioning belongs to [Hetzner inventory/caph integration](../hetzner-robot-controller/AGENTS.md) and [onboarding](../k8s/onboarding.md), not this controller. Parent access restrictions apply to operator procedures too.
