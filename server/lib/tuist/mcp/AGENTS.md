# Tuist MCP

## Prompt/Skill Sync

- Keep `server/lib/tuist/mcp/components/prompts/fix_flaky_test.ex` prompt text aligned with the canonical `fix-flaky-tests` skill at `skills/skills/fix-flaky-tests/SKILL.md` in this repository.
- If the skill workflow changes (steps, heuristics, verification, checklist), update the MCP prompt in the same change.
- If MCP tools or tool payloads change, update the skill or document intentional divergence.

## Documentation Sync

- When MCP server behavior/capabilities change, update `server/priv/docs/en/guides/features/agentic-coding/mcp.md` in the same PR.

## Versioning

- When MCP client-visible behavior changes (tools, prompts, responses, auth/discovery behavior), bump `serverInfo.version` in `server/lib/tuist/mcp/server.ex` in the same PR.
- Do not bump `protocolVersion` unless the implementation is intentionally updated to a newer MCP specification revision and compatibility has been validated.
