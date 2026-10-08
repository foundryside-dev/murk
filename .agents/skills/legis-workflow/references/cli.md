# Legis CLI reference

Relocated from `SKILL.md` (convention C-20 budget). Keep it faithful to the
installed `legis` — `legis --help` and `legis <command> --help` are authoritative.

## CLI reference

`legis <command> [flags]`. Most stores fall back to environment variables; flags
override.

### `legis serve` — run the Legis API server
- `--host` (default `127.0.0.1`), `--port` (default `8000`) — bind address.
- `--governance-db` — governance store URL (env `LEGIS_GOVERNANCE_DB`).
- `--check-db` — check store URL (env `LEGIS_CHECK_DB`).
- `--protected-policies` — comma-separated protected policy list (env `LEGIS_PROTECTED_POLICIES`).
- `--loomweave-url` — Loomweave identity API URL (env `LOOMWEAVE_API_URL`).
- `--filigree-url` — Filigree issue-tracker API URL (env `FILIGREE_API_URL`).
- `--binding-db` — sign-off binding ledger URL (env `LEGIS_BINDING_DB`).
- Judge flags (shared): `--judge-provider` (`openrouter`; omit to keep protected cells fail-closed), `--judge-model` (env `LEGIS_JUDGE_MODEL`), `--judge-max-tokens` (env `LEGIS_JUDGE_MAX_TOKENS`).

### `legis mcp` — run the MCP stdio server
- `--agent-id` (**required**) — launch-bound agent identity; the actor for all records this session.
- `--governance-db` (env `LEGIS_GOVERNANCE_DB`), `--check-db` (env `LEGIS_CHECK_DB`).
- `--policy-cells` — policy cell registry TOML path (env `LEGIS_POLICY_CELLS`).
- `--protected-policies` (env `LEGIS_PROTECTED_POLICIES`), `--loomweave-url` (env `LOOMWEAVE_API_URL`).
- Judge flags (shared): `--judge-provider`, `--judge-model`, `--judge-max-tokens`.

### `legis check-override-rate` — CI gate
Fails (exit 1) if the override-rate gate is `FAIL`. For CI use.
- `--db` — governance store URL (default mirrors the server's `LEGIS_GOVERNANCE_DB` / `DEFAULT_GOVERNANCE_DB`).

Prints `override-rate gate: <STATUS> (rate=…, sample=…)`. A missing SQLite DB under
`CI=true` (without `LEGIS_ALLOW_MISSING_GOVERNANCE_DB=1`) fails; otherwise it prints
`PASS_WITH_NOTICE` and exits 0. A failed hash-chain integrity check exits 1.

### `legis governance-gate` — run governance CI gates
Currently runs the override-rate gate (same implementation and `--db` semantics as
`check-override-rate`). Use this name for the general CI gate entry point.

### `legis sei-backfill` — resolve legacy locator-keyed records
Resolves legacy locator-keyed governance records through Loomweave batch resolve and
emits a JSON report.
- `--db` — governance store URL (env `LEGIS_GOVERNANCE_DB`).
- `--loomweave-url` (**required**) — Loomweave identity API URL.
- `--execute` — append backfill events (omit for a dry-run report).
- `--actor` (default `legis-sei-backfill`) — actor stamped on appended events.

### `legis policy-boundary-check` — boundary-evidence gate
Fails (exit 1) when `@policy_boundary` metadata lacks current behavioural evidence.
- `--root` (default `src`) — Python source root to scan.
- `--repo-root` (default `.`) — repo root for `test_ref` resolution.
- `--format` (`text` | `json`, default `text`) — human-readable lines vs machine-readable findings.

Prints `policy-boundary-check: PASS` (exit 0) when clean; otherwise one
`path:line: rule_id: qualname: reason` per finding (exit 1).
