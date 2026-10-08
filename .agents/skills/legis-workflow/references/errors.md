# Legis error handling

Relocated from `SKILL.md` (convention C-20 budget).

## Error handling

Tool errors carry `error_code`, `message`, `recoverable`, and a `next_action` hint.
Branch on `error_code`, not message text.

| `error_code` | Recoverable | `next_action` |
|---|---|---|
| `INVALID_ARGUMENT` | yes | Correct the tool arguments and retry. |
| `INVALID_CELL_SPEC` | yes | scan_route routing is server-owned and unconfigured by default. The operator sets `LEGIS_WARDLINE_CELL` (e.g. `=surface_only`) or `LEGIS_WARDLINE_CELL_BY_SEVERITY` out-of-band, then relaunches. (Request-side routing requires the `LEGIS_UNSAFE_WARDLINE_REQUEST_ROUTING` opt-in — discouraged.) The error message names which kind of cell spec was rejected. |
| `CELL_NOT_ENABLED` | yes | Two enablement tiers, by cell — both operator-enabled, out-of-band. Simple tier (chill/coached) is reachable WITHOUT a key: the operator maps the policy to a cell via `policy/cells.toml` or `LEGIS_POLICY_CELLS` (`LEGIS_DEV_DEFAULT_CELLS=1` selects the chill dev default), then relaunches. Complex tier (structured/protected and the binding ledger) additionally needs `LEGIS_HMAC_KEY` set by the operator out-of-band, then a relaunch. The error message names which cell is unenabled. |
| `NO_SUCH_REQUEST` | yes | Poll a known sign-off sequence returned by `override_submit`. |
| `NOT_FOUND` | yes | Refresh the target identifier and retry. |
| `UNKNOWN_TOOL` | yes | Call `tools/list` and use one of the advertised tool names. |
| `GIT_ERROR` | yes | Check the git ref or revision range and retry. |
| `SERVICE_ERROR` | yes | Inspect the error message before retrying. |
| `AUDIT_INTEGRITY_FAILURE` | **no** | Stop and ask an operator to inspect the governance trail. |
| `INTERNAL_ERROR` | **no** | Inspect the error message before retrying. |

`AUDIT_INTEGRITY_FAILURE` (raised on a failed hash-chain verification or a binding
ledger error) and `INTERNAL_ERROR` are **not recoverable** — do not retry; surface
them to a human. Everything else is recoverable by fixing the input or asking the
operator to enable a cell.

Two routing-specific notes for `scan_route`:
- Wardline routing is **server-owned**. Passing `cell` / `severity_map` / `fail_on`
  when the server already configures routing (`LEGIS_WARDLINE_CELL` /
  `LEGIS_WARDLINE_CELL_BY_SEVERITY`) returns `INVALID_CELL_SPEC`. Request-side
  routing is only honoured under the explicit `LEGIS_UNSAFE_WARDLINE_REQUEST_ROUTING=1`
  escape hatch.
- An unsigned dirty-tree dev artifact arriving where signed provenance is required
  is a typed recoverable failure, not a success: MCP returns `isError: true` with
  structured `error_code: WARDLINE_DIRTY_TREE` and message/reason
  `SKIPPED_DIRTY_TREE`; nothing is governed. Commit for a signed artifact, or set
  `LEGIS_WARDLINE_ALLOW_DIRTY=1` to govern it unsigned in dev.
