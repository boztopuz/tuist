# Xcresult Processor Image

Tart Layer 2 containing a full server release and macOS NIFs. It consumes `:process_xcresult`; the image, not a host checkout, is the deploy artifact.

- Inherit full Xcode from [Layer 1](../macos-xcode-image/AGENTS.md): `/usr/bin/xcrun xcresulttool` is unavailable with Command Line Tools alone. The processor must use at least as new an Xcode as every active [runner profile](../runner-image/profiles.json). Update its `XCODE_VERSION` in [the release workflow](../../.github/workflows/server-production-deployment.yml) when adding newer profiles.
- Keep one environment-neutral image. `inject-env.sh` imports the read-only host share into `/etc/tuist.env` before launchd starts the release; do not bake environment secrets or restore obsolete MASTER_KEY/encrypted-bundle assumptions. [Chart runtime wiring](../helm/tuist/templates/xcresult-processor-deployment.yaml) is authoritative, including `TUIST_MODE=xcresult_processor`, processor-role database access, license, and object-storage configuration.
- Preserve the one-Pod/one-VM shape and dedicated processor-fleet scheduling; customer runner jobs must not compete on these hosts. Host metrics-port uniqueness also requires at most one processor Pod per node during rollout.
- This image's baked code must remain compatible with the live database schema. [Release include paths](../../mise/tasks/release/components.json) and [git-cliff filters](cliff.toml) can skip server changes (notably `chore`/`ci`). For schema/runtime-affecting changes, verify a compatible processor release will actually be built; do not assume every server deploy rebuilds it.
- Builds need a bare-metal Mac with a live GUI session, Xcode, and Erlang; Tart cannot nest. See [builder runbook](../vm-image-builder.md).

Local build from repository root, using an already-published base version:

```bash
mise run xcresult-processor:build-image <xcode_version>
```

The [build task](../../mise/tasks/xcresult-processor/build-image.sh) compiles both NIFs, builds/packages the production release, and invokes Packer. It builds locally; publishing or workflow dispatch is a separate explicitly requested action.
