# Build Doc — Phoenix Current Software (PCS)
**Owner:** GIT-PHOENIX-HUB | **Last Updated:** 2026-03-27

## Objectives

1. Fix the plugin `.mcp.json` path — update to point at the built `current/packages/mcp-server/dist/index.js` so the plugin works end-to-end without manual path surgery.
2. Establish a build step in CI that compiles the TypeScript and validates the dist is up to date before merge.
3. Expand the Vitest test suite to cover all 23 active tools, the rate limiter, and the cache with meaningful coverage.
4. Wire the HTTP transport mode to the Phoenix Gateway so the MCP server is accessible to non-Claude-Code consumers.
5. Resolve the 35 deprecated tool stubs — either confirm endpoints exist in a future SF API version or remove the stubs after a formal deprecation period.

## End State

PCS is a production-grade TypeScript MCP server that:
- Compiles cleanly with `npm run build` and passes `npm test` in CI on every PR
- Runs as a stdio transport under the Claude Code plugin and as an HTTP transport behind the Phoenix Gateway
- All 23 active tools are covered by automated tests with mock SF responses
- The `service-fusion` plugin repo's `.mcp.json` points at the compiled dist from this repo (or the plugin is co-located)
- Deprecated stubs are either promoted to active (if SF re-enables those endpoints) or removed with a documented decision log entry
- A `CHANGELOG.md` tracks every tool addition, removal, or behavior change

## Stack Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Language | TypeScript ESM | Type safety on SF API shapes; MCP SDK types; ESM for Node 20+ compatibility |
| MCP SDK | @modelcontextprotocol/sdk ^1.25.1 | Official SDK; handles stdio + HTTP transport abstractions |
| Schema validation | Zod | Runtime validation of tool inputs; zod-to-json-schema for MCP schema generation |
| Secrets | Azure Key Vault + OIDC | No secrets in env files or repo; consistent with rest of Phoenix stack |
| Monorepo | npm workspaces | Shared types and logger between mcp-server and shared packages |
| Test runner | Vitest | Fast ESM-native; compatible with TypeScript without transpile step |
| Build | tsc only | Sufficient for server-side TS; no bundling needed |
| HTTP framework | Express | Thin wrapper for StreamableHTTP transport; no API surface of its own |

## Architecture Targets

- **Plugin path fix:** Update `plugin/.mcp.json` to reference a stable, version-pinned path (e.g., resolved relative to a symlink or install convention rather than a hardcoded absolute path into a staging repo).
- **Gateway HTTP integration:** The `createMCPServer({ transport: 'http' })` factory exists. Wire it to the Gateway's Express app so the 23 tools are accessible over HTTP to the Gateway backend.
- **Tool expansion:** As SF enables additional v1 endpoints, promote deprecated stubs to active tools. Add any new endpoint categories (e.g., memberships, recurring services, bookings) following the existing category pattern in `tools/index.ts`.
- **Shared package publishing:** `@phoenix/shared` is currently workspace-only. If a Gateway service needs it, consider publishing to a private registry or extracting to a standalone package.
- **Rate limiter observability:** Expose rate limiter state via the `health` tool response so Gateway dashboards can surface quota status without hitting the SF API.

## Success Criteria

- [ ] `npm run build` completes without errors from a clean clone (no manual path edits required)
- [ ] `npm test` passes with >80% coverage on tool handlers, rate limiter, and cache
- [ ] Plugin `.mcp.json` path resolved — plugin installs and tools are accessible in a new Claude Code session without manual surgery
- [ ] HTTP transport tested end-to-end with the Gateway (at least one tool call over HTTP returns a valid SF response)
- [ ] All 35 deprecated stubs have a documented disposition (promote or remove) in `docs/deprecated-tools.md`
- [ ] CI pipeline (GitHub Actions) runs build + typecheck + tests on every PR to main
- [ ] No secrets, credentials, or vault identifiers committed to the repo at any point

## Dependencies & Blockers

| Dependency | Status | Owner |
|-----------|--------|-------|
| Azure Key Vault access on build machine | Required for tests that hit live SF API | Shane (az login) |
| Phoenix Gateway HTTP surface | Needed for HTTP transport integration | Echo / Gateway team |
| SF API v1 endpoint status for deprecated stubs | Unknown — needs re-discovery run | Echo (api-discovery.sh) |
| npm workspaces build order | @phoenix/shared must build before mcp-server | Resolved via tsc project references or build script |
| service-fusion plugin repo .mcp.json sync | Repo is separate — changes here require manual sync there | NEEDS SHANE INPUT — consider co-locating |

## Change Process

All changes to this repository follow the Phoenix Electric governance model:

1. **Branch:** Create feature branch from `main`
2. **Develop:** Make changes with clear, atomic commits
3. **PR:** Open pull request with description of changes
4. **Review:** Required approval from `@GIT-PHOENIX-HUB/humans-maintainers`
5. **CI:** All status checks must pass (when configured)
6. **Merge:** Squash merge to `main`
7. **No force push.** No direct commits to `main`. No deletion without `guardian-override-delete` label.

**Additional rules specific to PCS:**
- Tool additions or removals require a `docs/api-surface.md` update in the same PR
- Any change to the SF OAuth flow or secret fetching pattern requires explicit sign-off from Shane
- Deprecated stubs may not be removed without a documented decision in `docs/deprecated-tools.md`

## NEEDS SHANE INPUT

- **Plugin co-location vs split repos:** Should the `plugin/` directory live in `current` (one repo) or remain in the separate `service-fusion` repo? Currently both repos contain partial copies. This creates a sync burden.
- **Deprecated tool disposition:** 35 stubs exist for SF endpoints that returned 404 during discovery. Should these be re-tested against a fresh SF tenant, or removed now with a deprecation notice?
- **HTTP transport priority:** Is wiring the HTTP transport to the Gateway a near-term priority, or is stdio-only sufficient for the next phase?
- **CI budget:** GitHub Actions minutes are free for public repos, paid for private. Confirm whether this repo should remain private and whether paid CI is approved.
- **Vitest test fixtures:** Live SF API tests require credentials. Should tests use mocked fixtures (no credentials needed) or live integration tests (requires az login in CI)? Mock fixtures are recommended but need Shane's confirmation.
