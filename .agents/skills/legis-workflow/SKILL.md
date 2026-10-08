---
name: legis-workflow
description: >
  Use when a project uses legis for git/CI governance and graded enforcement:
  evaluating or explaining a policy cell, submitting a graded override, polling a
  sign-off, checking the override-rate or governance CI gate, running
  policy-boundary-check, routing Wardline findings through governance, reading git
  branch/commit context or the Loomweave rename feed, reading recorded PR/check
  outcomes, gating a Filigree closure on binding evidence, or back-filling
  SEI-keyed governance records.
---

# Legis Workflow

Legis is the git/CI and **governance** side of the Weft suite. This skill is the
depth behind the lean `CLAUDE.md` block. Keep it faithful to the installed
`legis` — when in doubt, `legis --help` and `legis <command> --help` are
authoritative.

## What legis is

Legis answers *what changed, in which branch/commit/PR/check context, and what
governance or attestation state exists for that change?* It is an SEI **consumer**
(Loomweave remains the identity authority) and the suite's single governed judge —
**Wardline analyses trust; Legis governs it, one judge not two**. It does not own
issue state (Filigree) or code identity (Loomweave); it adds branch/commit/PR/check
context and a graded enforcement layer on top.

Enforcement is a **2×2** of policy *cells*, each agent-set, each a distinct
override flow:

| | Judge OFF | Judge ON |
|---|---|---|
| **Simple** | **chill** — agent self-reports a recordable override; human reviews async (`ACCEPTED_SELF`) | **coached** — an LLM wall evaluates the override *before* it records; `ACCEPTED_BY_JUDGE` or `BLOCKED` (not self-clearable) |
| **Complex** | **structured** — block + escalate; a human operator must sign off before the gate clears (`ESCALATED_PENDING`) | **protected** — full machinery: HMAC-signed verdicts, decay sweep, override-rate gate, operator override |

The operating invariant is **agent-first: humans on the loop, not in the loop.**
Every cell produces an append-only audit trail keyed on SEI, so the record survives
rename/move. The recorded override is the safety mechanism — an attributable audit
event, never a silent pass.

## Reaching the tools

Prefer the MCP tools (`mcp__legis__*`) when a Legis MCP server is attached; fall
back to the `legis` CLI otherwise. Each surface maps thinly over the same service
layer, so they agree on outcomes.

**Identity is launch-bound.** The MCP server is started with
`legis mcp --agent-id <name>`; that `--agent-id` is the actor for every override,
sign-off, and audit record the session produces. **No tool schema accepts an actor
argument** — you cannot spoof or override identity from a call. (Contrast the CLI's
`sei-backfill --actor`, which stamps appended backfill events from a one-shot
command, not an interactive session.)

The MCP transport is stdio JSON-RPC (one object per line). Tool errors come back as
`isError` results with a `structuredContent` envelope carrying `error_code`,
`message`, `recoverable`, and `next_action`.

## References

Load on demand — these carry the detail this file used to inline:

- `references/cli.md` — the governance/CI commands and their flags (`serve`,
  `mcp`, `check-override-rate`, `governance-gate`, `sei-backfill`,
  `policy-boundary-check`), env-var fallbacks, and exit codes. `legis --help`
  lists the remaining install/operator commands.
- `references/mcp-tools.md` — the MCP tool catalogue by area (governance/policy,
  git, pulls/checks, Filigree binding) and the `override_submit` outcome envelope
  per cell.
- `references/errors.md` — the `error_code` table (recoverable vs not, with the
  `next_action` for each) and the `scan_route` routing rules.
- `references/patterns.md` — worked patterns: evaluate-then-override, the CI
  override-rate gate, the Loomweave rename feed, gating a Filigree closure, routing
  Wardline findings, and the boundary-evidence gate.
