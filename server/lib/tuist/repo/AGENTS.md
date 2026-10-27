# Repo Monitoring

- Shared repo pool metrics belong in `tuist_common/`; configure the server wrappers here.
- Keep repo labels and telemetry prefixes aligned with PromEx wiring.
- Keep repo startup and supervision wiring in `server/lib/tuist/application.ex`.
