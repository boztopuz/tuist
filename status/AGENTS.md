# Status page

Stateless Hono/TypeScript Cloudflare Worker backed by Grafana IRM. It serves HTML,
JSON, RSS, and Atom; there is no database or hardcoded component list.

## Development

Run from `status/`:

- `mise run dev` starts Wrangler against real Grafana data by default. Put
  `USE_FAKE_DATA="true"` in gitignored `.dev.vars` for the runtime fixture in
  `src/dev/fake-status.ts`.
- Validate with `aube run typecheck`, `aube run test`, and `mise run build`
  (dry-run deploy only).
- Configuration defaults live in `wrangler.jsonc`. Keep the Grafana token in
  `.dev.vars` locally or a Wrangler secret in production, never in source.
- Use the stack proxy URL (`https://<stack>.grafana.net/api/plugins/grafana-irm-app/resources`),
  not regional `incident-prod-*` URLs, with stack-scoped service-account tokens.
- Debug endpoints require `ENABLE_DEBUG_ROUTES="true"` and are disabled in fake
  mode. `/api/debug/fields.json` diagnoses empty component lists;
  `/api/debug/incidents.json` exposes raw upstream field names.

## Contracts

- Components come from the select options of `GRAFANA_COMPONENT_LABEL_KEY`
  (default `affected_service`). Match incidents by both label key and known value;
  use the worst active severity for each component.
- `critical` → major outage, `major` → partial outage, `minor`/`pending` → degraded.
  A `maintenance` label overrides upstream severity.
- Feeds contain one entry per incident update, newest first, with an incident
  anchor link; retain RSS/Atom autodiscovery and the 60-second cache policy.
- Plain HTML must match Noora class names and `data-*` attributes. Keep
  `src/views/noora-css.ts` a verbatim concatenation of Noora's `tokens`, `card`,
  `badge`, `banner`, and `line_divider` CSS, in that order. Update copied brand/status
  icons with Noora; custom layout glue belongs in `src/views/styles.ts`.

CI checks and deployment are defined in `.github/workflows/status.yml` and
`.github/workflows/status-deploy.yml`. The Grafana token is a separately provisioned
Wrangler secret, not injected by the deployment's Cloudflare credentials.
