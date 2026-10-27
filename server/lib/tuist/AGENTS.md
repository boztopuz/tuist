# Tuist (Business Logic)

- Keep domain rules in contexts here; HTTP handling, controllers, and LiveView belong in `server/lib/tuist_web`.
- Put environment/runtime configuration in `server/config`, not domain modules.
- Keep schemas here and migrations in `server/priv/repo/migrations` (PostgreSQL) or `server/priv/ingest_repo/migrations` (ClickHouse).
