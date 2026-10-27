# Tuist Server (Elixir/Phoenix)

PostgreSQL is the primary database; ClickHouse analytics writes use `IngestRepo`, reads use `ClickHouseRepo`.

## Development

- Run commands from `server/`. Use Mise and PostgreSQL 16 (`brew install postgresql@16` / `brew services start postgresql@16`).
- Bootstrap with `mise run clickhouse:start` and `mise install`; start with `mise run dev`. CLI integration uses `http://localhost:8080` by default; worktrees can have their own port.
- Dev secrets (OAuth, Stripe, etc.) are optional: `fnox.toml` sources the 1Password `Development` vault using an `op` session or `OP_SERVICE_ACCOUNT_TOKEN`. Without access the server boots with those integrations disabled.
- After dependency changes or in a fresh worktree, run `mise run install`; after pulling migrations, run `mix ecto.migrate`.
- Seed login: `tuistrocks@tuist.dev` / `tuistrocks`.

## Validation

- Prefer focused ExUnit runs: `mix test test/path/to/specific_test.exs`, `mix test test/path/to/test_file.exs:line_number`, or `mix test --only tag_name`.
- Tests compile only English and exclude `:locale` tests by default. Use `TUIST_DEV_ALL_LOCALES=1 mix test` (optionally with a focused path) to compile all locales and include those tests, as CI does.
- Dev compiles skip Boundary validation. Run `MIX_ENV=test mix compile` to check boundaries before pushing.
- Format with `mise run format`; lint with `mix credo`; security checks: `mix sobelow` or `mise run security`.
- For migrations, run `mix excellent_migrations.check_safety`; use `mix ecto.dump` when updating the database structure dump.

## Assets

- JS dependencies use aube. `mix assets.setup` installs server esbuild and local `noora` JS dependencies.
- `mix assets.build` builds local `noora` assets, then app/marketing/docs/apidocs bundles. Use `mix assets.deploy` for production assets.

## Elixir and tests

- Do not add typespecs (`@spec`, `@type`, etc.).
- Declare aliases at module level, never inside functions. Prefer aliases for reused modules; reserve imports for DSLs such as `Ecto.Query`.
- Use tagged tuples for recoverable failures.
- Timestamps: `:timestamptz` in migrations, `:utc_datetime` in `lib/`.
- Register mocked modules with `Mimic.copy` only in `server/test/test_helper.exs`, not in individual tests or functions.
- Tests default to `async: true`; avoid global state and change the architecture when necessary to support concurrency.
- Never mutate System environment variables in tests. Use mocks, stubs, or dependency injection for environment-dependent behavior.

## Translations

- Use `dgettext/2` for translatable text. Run `mix gettext.extract` after adding/changing strings and commit only `.pot` templates alongside code; Weblate/`tuistit` manages `.po` files.
- Never run `mix gettext.extract --merge` or edit `.po` files.
- Currency symbols and monetary amounts are not translatable; surrounding descriptions such as "and up" or "per unit" are. Example: `"0€ " <> dgettext("marketing", "and up")`.

## Data export documentation

Keep `server/data-export.md` current for all personal and organizational data so legal customer exports remain complete. Update it whenever a change:

- Adds or changes PostgreSQL/ClickHouse tables, customer-data columns, Ecto schemas/models, data relationships, or retention policies.
- Adds stored file types, categories or buckets; changes S3/object-storage keys or path structures, or upload/download processes.
- Collects new user fields, analytics/tracking, integration data, or customer-owned generated content.

Document data type and purpose, storage location (table or S3 path), relationships, retention, and export capabilities. Follow the existing format and verify the export process can actually retrieve the documented data.
