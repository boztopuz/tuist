# TuistWeb (Controllers, LiveView, API)

- Keep request handling, rendering, and UI state here; delegate domain operations to `server/lib/tuist` contexts. JS/CSS belongs in `server/assets`.
- Authorize channel access via `Tuist.Authorization`.

## Route metadata

- `server/lib/tuist_web/router.ex` route metadata feeds the runtime `robots.txt` Content-Usage and Disallow entries.
- When adding or changing public marketing/docs routes, keep `metadata: %{type: :marketing}` or `:docs` accurate.
- For public `GET` routes that should appear in `robots.txt` Content-Usage, define that configuration in the router with `metadata: %{robots_txt: [train_ai: true, search: true]}`.
- Routes without `:robots_txt` metadata default to `Disallow` entries derived from the router.
- If a route should not contribute any `robots.txt` entry, opt it out explicitly with `metadata: %{robots_txt: false}`.

## OpenAPI generation

- Update controller schemas, then run `mise run generate-api-cli-code` from `server/` to export the spec and regenerate the Swift client.
- Do not hand-edit `cli/Sources/TuistServer/OpenAPI/server.yml` or the generated `Types.swift` / `Client.swift`.
