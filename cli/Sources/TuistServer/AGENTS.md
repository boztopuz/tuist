# TuistServer

- `TUIST_URL` overrides the configured server URL and must be valid.
- Do not hand-edit `OpenAPI/server.yml`, `OpenAPI/Types.swift`, or `OpenAPI/Client.swift`. Update the server controller schemas, then run `mise run generate-api-cli-code` from `server/` to export the spec and regenerate the Swift client.
