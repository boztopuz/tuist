# Grafana data source

Go backend and React editors for Tuist build/test duration metrics. Aggregation
belongs in the server; this plugin shapes requests and Grafana frames.

## Development

Run from `grafana-datasource/`:

- Backend: `mage -v` to build; `go test ./...` to test.
- Frontend: `npm install`, then `npm run typecheck`, `npm run lint`,
  `npm run test:ci`, and `npm run build`.
- Keep the plugin's own `go.work` separate from the root infra workspace; its
  Grafana SDK dependency graph must not force upgrades on the infra controllers.
- `.config/` is managed by `@grafana/create-plugin`; update with
  `npx @grafana/create-plugin@latest update` rather than editing it manually.
- `CHANGELOG.md` is generated and gitignored. The prebuild hook supplies a local
  stub; releases generate the real file. Do not commit it.

## API and credentials

- `TuistWeb.API.MetricsController` is the API source of truth. Keep
  `pkg/plugin/models.go` and `pkg/plugin/client.go` aligned with its routes and
  response shapes; these endpoints are not part of the generated CLI client.
- Time ranges and response dates are Unix seconds; duration values are
  milliseconds. The server chooses bucket granularity; do not send an interval.
- Store account tokens only in `secureJsonData`, never return them to the browser.
  Dropdowns use backend `CallResource` requests, not direct frontend API calls.
- Keep plugin ID `tuist-metrics-datasource` and executable
  `gpx_tuist_datasource` aligned with packaging. Release automation lives in
  `.github/workflows/grafana-datasource-release.yml`.
