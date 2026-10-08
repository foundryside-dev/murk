<!-- filigree:instructions:v3.1.0:c1c023c3 -->
<!-- filigree:last-writer:filigree install -->
## Filigree Issue Tracker

`filigree` tracks this project's work. Use it to find, claim, update and close
issues: `filigree session-context` at session start, then
`filigree start-next-work --assignee <name>`.

Full reference: the **filigree-workflow** skill (patterns, priorities,
observations, error codes), `filigree --help`, and the `mcp__filigree__*` tool
schemas. Prefer the MCP tools when available; fall back to the CLI.

Two rules `--help` will not tell you:

1. Claim atomically: `work_start` / `work_start_next` (MCP) or `start-work` /
   `start-next-work` (CLI). Never chain a claim with a separate status update;
   that two-step form races other agents.
2. On `SCHEMA_MISMATCH` the installed filigree is older than the project
   database. Surface it to the user; do not retry.
<!-- /filigree:instructions -->

<!-- wardline:instructions:v1:bcd19330 -->
<!-- wardline:last-writer:wardline install -->
This project uses **wardline** as its trust-boundary gate. Before handing back code that touches external input, run `wardline scan . --fail-on ERROR` (exit 0 = clean, 1 = gate tripped, 2 = wardline error) and fix findings at the boundary, not the sink. The full scan -> explain -> fix -> rescan loop and the baseline-vs-waiver discipline live in the `wardline-gate` skill and in `docs/agents.md`.
<!-- /wardline:instructions -->

<!-- legis:instructions:v1.5.0:37065fbc -->
## Legis (git/CI + governance)

Legis is the git/CI and governance layer of the Weft suite: graded policy
enforcement over branch/commit/PR/check context, recorded in an append-only
audit trail keyed to stable code identity (SEI), so it survives rename/move.

Reach for it when a policy fires at the CI/git boundary, when a change needs a
recordable override or human sign-off, or when you need git/CI context.

- Prefer the `mcp__legis__*` MCP tools; fall back to the `legis` CLI.
- Clear a fired policy through `override_submit` (MCP-only), which grades it
  (self-clear / judged / escalated) and records it — routing around it leaves
  no trail.

Full reference: the `legis-workflow` skill, `legis --help`, MCP schemas.
<!-- /legis:instructions -->

<!-- loomweave:instructions:v1.5.0:39edbf6d -->
<!-- loomweave:last-writer:loomweave install -->
## Loomweave (code structure + SEI identity)

Loomweave pre-extracts this repo into a queryable map — entities, their
call/reference/import/relation edges, and subsystems — each carrying a Stable
Entity Identity (SEI). Ask its `mcp__loomweave__*` tools, not grep, for "what
calls X", "what subclasses X", "where is X defined", "find the thing that
does Y".

- Never hand-construct an entity id: take it from `entity_find` / `entity_at` /
  `entity_resolve`, and bind cross-tool records on the `sei`, not the `id`.
- If `project_status_get` reports stale, re-index before answering.

Full reference: `loomweave-workflow` skill, `loomweave --help`, MCP schemas.
<!-- /loomweave:instructions -->

<!-- warpline:instructions:v1.3.0 -->
## Warpline (temporal change-impact)

`warpline` answers "if I touch X, what breaks, and what must I re-verify?".
Prefer the MCP tools (`mcp__warpline__*`); fall back to the `warpline` CLI.

Call `warpline_change_list` (shim: `changed`) for a rev range first, then follow
its `next_actions` into `reverify` / `blast_radius`. A `completeness` of
`NO_SNAPSHOT` means warpline cannot see, NOT that nothing is affected.

Enrich-only, local-only, advisory: warpline never gates. The `warpline-workflow`
skill carries the full tool set, the closed vocabularies, and the loop.
<!-- /warpline:instructions -->
