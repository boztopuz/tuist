# Noora

Phoenix LiveView component library published to Hex. The server uses a local path
dependency, so changes here affect `server/` directly. Component code lives in
`lib/`, hooks in `js/`, styles in `css/`, and built assets in `priv/static/`.

- Build: `mise run noora:build` (JS dependencies, assets, and Elixir compilation).
- JS tests: `mise run noora:test`.
- Formatting: `mise run noora:lint` or `mise run noora:lint --fix`.
- Preview components in `storybook/`. Its independent deployment is defined by
  `infra/helm/noora-storybook/` and `.github/workflows/noora-storybook-deployment.yml`.
- Releases read the version from `mix.exs` and use `cliff.toml` for changelogs.
