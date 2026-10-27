# Helm Charts

- Model dependencies by capability (`objectStorage`), not implementation brand (`minio`); support embedded and external modes where practical.
- Keep independently released workloads in standalone charts rather than growing the main umbrella chart.
- Validate with `helm lint` and `helm template` using the chart's CI placeholder values, then production overlays. Cluster installs are not local validation and require the approval described in [infra guidance](../AGENTS.md#cluster-access-for-agents).
- Main chart usage and dependency modes: [Tuist chart README](tuist/README.md).
