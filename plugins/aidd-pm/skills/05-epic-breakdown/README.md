# 05 - Epic Breakdown

Transforms a validated PRD into a set of distinct, traceable epics — one per Core Feature or
logical grouping of features — covering 100% of the PRD's scope. Each epic is a structured
artifact ready for user-story generation downstream and for direct import into a ticket tracker.

The skill operates at the "what and why" level only. It never decides the stack, names a library,
or scaffolds a file layout. Gaps in the PRD surface as `TBD: <precise question>` markers in the
epic, never as invented values.

## When to use

- "break down the prd into epics", "create epics from prd", "epic breakdown".
- "generate epics for `<PRD path>`".
- Invoking `/epic-breakdown`.
- When an orchestrator (e.g., the greenfield-gateway) needs the PRD decomposed before triggering
  the user-stories skill or a planning run.

## When NOT to use

- To generate user stories from the epics → use `aidd-pm:02-user-stories` on each epic.
- To produce a new PRD → use `aidd-pm:03-prd`.
- To run or orchestrate the SDLC.
- To generate source code or define the technical stack.
- When the input is a raw idea or a non-validated PRD — the skill will refuse it with an error.

## Prerequisites

- A PRD file that:
  - exists at the supplied path,
  - contains `Status: Approved` (or equivalent validated marker),
  - has a `## 4. Core Features` section with one or more `### Feature N - <name>` subsections.
- Write access to `aidd_docs/tasks/<prd_yyyy_mm>/` for `file` or `both` output modes.
- A configured ticketing tool (detected automatically from project memory or git remote) for
  `ticket` or `both` output modes. When none is found the skill degrades to `file` mode and warns.

## How to invoke

### Break down a PRD into epics (file output — default)

```
Use skill aidd-pm:05-epic-breakdown breakdown for aidd_docs/tasks/2026_06/2026_06_30-greenfield-gateway-prd.md
```

### Break down a PRD and create tracker tickets

```
Use skill aidd-pm:05-epic-breakdown breakdown for <prd_path> with output_target: ticket
```

### Break down a PRD and produce both files and tickets

```
Use skill aidd-pm:05-epic-breakdown breakdown for <prd_path> with output_target: both
```

### Audit coverage of already-produced epics

```
Use skill aidd-pm:05-epic-breakdown coverage-check for <prd_path> with epics_location: aidd_docs/tasks/<yyyy_mm>/
```

## Outputs

### `breakdown` action

- One epic artifact per Core Feature (or explicit multi-feature grouping), written to:
  `aidd_docs/tasks/<prd_yyyy_mm>/<prd_yyyy_mm_dd>-<prd_slug>-epic-<NN>-<feature_slug>.md`
- When `output_target: ticket` or `both`: one tracker issue per epic, created via
  `aidd-vcs:04-issue-create`.
- A coverage summary: list of features processed, epics produced, any TBD markers emitted.
- No modifications to the source PRD.

### `coverage-check` action

- A coverage matrix: each PRD feature mapped to its corresponding epic.
- Lists of missing features (not covered by any epic) and extra features (referenced by epics but
  absent from the PRD).
- Duplicate detection: any feature referenced by more than one epic.
- A pass/fail verdict: passes only when every feature is covered by exactly one epic.

## Idempotence

Epic filenames and ticket titles embed the source PRD's date and slug (not the run date). Running
`breakdown` twice on the same PRD produces the same file set: existing epics whose `epic_id` and
source feature match are skipped rather than overwritten. Run `coverage-check` after two passes to
confirm the epic set is unchanged.

## Technical details

See [`SKILL.md`](SKILL.md) for the full action contract and transversal rules,
[`actions/01-breakdown.md`](actions/01-breakdown.md) for the primary producer action,
[`actions/02-coverage-check.md`](actions/02-coverage-check.md) for the read-only coverage
verifier, [`assets/epic-template.md`](assets/epic-template.md) for the canonical epic body, and
[`assets/epic-validator.yml`](assets/epic-validator.yml) for the reviewer checklist.
