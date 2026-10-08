# Legis MCP tool catalogue

Relocated from `SKILL.md` (convention C-20 budget). Tool schemas advertised by the
running MCP server are authoritative.

## MCP tool catalogue

All tools return a `structuredContent` JSON payload. Names are exact.

### Governance / policy
| Tool | Purpose |
|---|---|
| `policy_explain` | Explain which governance cell controls a policy/entity pair, whether that cell is enabled here, and which move the agent may make next. Reports `matched_rule` — the routing pattern that matched, or `null` when the policy fell through to `default_cell` (distinguishes a configured-but-disabled policy from an unconfigured name). |
| `policy_list` | List the policy-to-cell routing table (`default_cell` + the configured pattern `rules`) and every governance cell's **real** enabled state on this server. The complex tier (structured/protected) reports `enabled: false` without `LEGIS_HMAC_KEY`. No arguments. |
| `policy_evaluate` | Evaluate a policy against a target **without recording an override**. Returns outcome, detail, and any `provenance_gap`. |
| `override_submit` | Submit an override as the launch-bound agent. Routes to the governing cell and returns a discriminated outcome envelope (`ACCEPTED_SELF` / `ACCEPTED_BY_JUDGE` / `BLOCKED` / `ESCALATED_PENDING` / `NEED_INPUTS`). |
| `signoff_status_get` | Poll whether a **structured** sign-off request (by `seq`) has been cleared. |
| `override_rate_get` | Read the fixed operator force-past override-rate gate (status / rate / sample_size). Measures operator force-pasts; **not** movable by agent retries. |
| `scan_route` | Route Wardline scan findings through one cell, a `severity_map`, or a cell + `fail_on` threshold. Returns `ROUTED` on success; dirty unsigned artifacts surface as `SKIPPED_DIRTY_TREE` with `isError: true` unless the dev dirty opt-in is enabled. MCP preserves `WARDLINE_DIRTY_TREE` as the structured `error_code`. |

### Git
| Tool | Purpose |
|---|---|
| `git_branch_list` | List local git branches and upstream divergence facts. |
| `git_commit_get` | Read one git commit by SHA or safe ref. |
| `git_rename_list` | List git rename evidence for a revision range (`rev_range`). |
| `git_rename_feed_get` | Loomweave-ready rename feed: committed renames over `base..head` plus optional uncommitted working-tree renames (`include_worktree`). |

### Pulls / checks
| Tool | Purpose |
|---|---|
| `pull_request_get` | Read recorded pull-request metadata (`number`) with joined check outcomes. |
| `check_list` | Read recorded CI/check outcomes for a `target_type` of `commit`, `branch`, or `pr` plus a `target`. |

### Filigree binding
| Tool | Purpose |
|---|---|
| `filigree_closure_gate_get` | Read whether legis holds **verified binding evidence** for closing a Filigree issue (`issue_id`). Requires the binding ledger to be enabled. |

### Override-submit outcomes (by cell)
- **chill** → `ACCEPTED_SELF` — self-cleared; human reviews asynchronously.
- **coached** / **protected** → `ACCEPTED_BY_JUDGE` (may be re-judged later) or `BLOCKED`. A `BLOCKED` verdict carries a `blocked_reason_code` (`RATIONALE_INSUFFICIENT` / `CODE_VIOLATION` / `POLICY_HARD_BLOCK` / `UNCLASSIFIED`), `self_clearable: false`, and `next_actions: [REVISE_CODE, REVISE_RATIONALE]`. A blocked attempt **does not count toward your override-rate** — you cannot self-clear past the judge.
- **structured** → `ESCALATED_PENDING` — human sign-off required; poll `signoff_status_get` with the returned `seq`.
- **protected** with missing inputs → `NEED_INPUTS` — supply the listed fields (e.g. `file_fingerprint`, `ast_path`) and resubmit.

Pass an `idempotency_key` on `override_submit` to make retries safe: a repeat with
the same request returns the original outcome; a reused key with a *different*
request is rejected (`INVALID_ARGUMENT`).
