# macOS + Xcode Base Image

Layer 1 shared by runner and xcresult-processor Tart images. Keep Xcode/dev-tool installation here, not duplicated in downstream runtime layers.

- Exactly one Xcode per image at `/Applications/Xcode_<version>.app`. For patch versions, preserve the major-minor symlink for repositories pinning `.xcode-version`. Tags use the full version with dots replaced by dashes.
- Keep full Xcode, platform runtimes, and Apple signing certificates available; Command Line Tools alone cannot provide `xcresulttool`. Do not preinstall Tuist CLI: customers pin it themselves.
- CI pulls `.xip` artifacts from `ghcr.io/tuist/xcode-xips:<version>` using oras. Do not introduce Apple ID/session/2FA state on rotating builder hosts. Human mirror publication uses `mise run xcode-mirror:upload <version>` on a maintainer Mac; implementation/usage is in [the upload task](../../mise/tasks/xcode-mirror/upload.sh).
- Build inputs and publication are defined in [macos-xcode-image.yml](../../.github/workflows/macos-xcode-image.yml). It requires a bare-metal builder with a live GUI session; Tart guests cannot nest Virtualization.framework. See [builder runbook](../vm-image-builder.md).
- Publishing this base does not promote customer runners. Keep [profiles.json](../runner-image/profiles.json) and `runnersFleet.xcodeVersions` in managed common values aligned. The first profile is the default; keep the xcresult-processor's `XCODE_VERSION` in [the release workflow](../../.github/workflows/server-production-deployment.yml) at least as new as every active runner profile.
