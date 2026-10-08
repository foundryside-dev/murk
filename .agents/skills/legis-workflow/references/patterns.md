# Legis workflow patterns

Relocated from `SKILL.md` (convention C-20 budget).

## Workflow patterns

### Evaluate a policy cell, then submit a graded override
```
policy_explain {policy, entity}        # which cell governs, is it enabled, what move is next
# read explanation.cell and available_moves (already filtered to agent-callable tools)
override_submit {policy, entity, rationale [, file_fingerprint, ast_path, idempotency_key]}
```
- **chill** → `ACCEPTED_SELF`; you are done, the human reviews the trail async.
- **coached/protected** → if `BLOCKED`, do not retry verbatim — `REVISE_CODE` or
  `REVISE_RATIONALE` per `next_actions`; the judge cannot be talked past and the
  blocked attempt costs you nothing on the override-rate.
- **structured** → `ESCALATED_PENDING`; poll `signoff_status_get {seq}` until
  `cleared: true`. Do not proceed on the gated change until then.
- **protected** → if `NEED_INPUTS`, supply `file_fingerprint` + `ast_path` (the
  bytes and AST node the judge binds its verdict to) and resubmit.

### Check the override-rate gate in CI
The gate measures **operator force-pasts**, not agent retries — a high rate means
the policy is miscalibrated or an operator is breaking their own rules.
```
# in-session read:
override_rate_get {}                    # → {status, rate, sample_size}
# CI step (exit 1 on FAIL):
legis check-override-rate --db <governance-db>
#   or the general entry point:
legis governance-gate --db <governance-db>
```

### Read the git-rename feed for Loomweave
Legis is the (contract-locked) rename provider Loomweave's SEI re-binding matcher
consumes.
```
git_rename_feed_get {base, head?, include_worktree?}
#   committed renames over base..head, plus optional uncommitted working-tree renames
# lower-level evidence over an explicit range:
git_rename_list {rev_range}
```

### Gate a Filigree closure on verified binding evidence
Before closing a governed Filigree issue, confirm Legis holds verified, SEI-keyed
sign-off binding evidence for it.
```
filigree_closure_gate_get {issue_id}    # requires the binding ledger to be enabled
# only close in Filigree once this reports verified binding evidence;
# Filigree retains lifecycle authority — Legis only certifies the evidence.
```
If the ledger is not enabled you get `CELL_NOT_ENABLED` — ask the operator to wire
`LEGIS_BINDING_DB` / `--binding-db`.

### Route Wardline findings through governance
```
scan_route {scan}                       # routing is server-owned; pass only the scan
# → ROUTED (governed into the configured cell), or SKIPPED_DIRTY_TREE with
#   isError:true (MCP error_code WARDLINE_DIRTY_TREE; commit, or set
#   LEGIS_WARDLINE_ALLOW_DIRTY=1 in dev)
```

### Gate boundary evidence in CI
```
legis policy-boundary-check --root src --repo-root . --format json
#   exit 1 with findings when @policy_boundary metadata lacks current behavioural evidence
```
