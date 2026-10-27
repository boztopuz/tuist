# Linux Runner Image

One ephemeral GitHub Actions runner per Pod/job. Pod isolation and lifecycle are owned by [runners-controller](../runners-controller/AGENTS.md).

- Never bake registration credentials into the image. Exchange the projected dispatch token for a job-scoped JIT at runtime; keep runner auto-update disabled so the reviewed image is what runs.
- Customer code must never see the pool-scoped dispatch SA token. The root-only poller init container stages JIT into the shared volume and exits; the unprivileged `runner` consumes it via `run-job.sh` without a token. The trusted metrics sidecar also holds the token; it must not execute customer code.
- `run-job.sh` must exit successfully when no JIT was staged (drain/abort). Warm Linux Pods remain Pending in init until claimed; container exit, not VM shutdown logic, is this image's terminal signal.
- Publish/pin images supporting poller-mode `dispatch-poll.sh` and `run-job.sh` **before** deploying a split-Pod controller. Keep the env-unset compatibility path while older controllers remain supported; reversing rollout order can execute customer code in the token-bearing poller.
- Docker tooling here is client-only. Dockerd belongs to the controller's Kata-isolated native sidecar; preserve Docker group GID `123` to match its socket and shared workspace paths for bind mounts.
- Vitals/log capture and structured metrics are best-effort and must not block jobs. Metrics wait for a staged JIT before posting; collect inside the VM, not through kubelet Summary API.
- Keep `RUNNER_VERSION` Renovate-pinned in the Dockerfile. Release semver/`:latest` tags belong to the release pipeline; branch/ad-hoc workflow builds must not overwrite them.

Local build from this directory:

```bash
docker build --pull -t ghcr.io/tuist/tuist-linux-runner:dev .
```
