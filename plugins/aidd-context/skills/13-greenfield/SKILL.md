---
name: 13-greenfield
description: Take a product idea to a ready backlog of epics and user stories, every step delegated. Use when the user wants to start a greenfield project from a raw idea, or turn an idea into a ticketed backlog before building. Not for implementing the backlog (that is the dev lifecycle).
argument-hint: preflight | brainstorm | prd | bootstrap | epic-breakdown | user-stories | handoff
---

# Skill: greenfield

Take a raw product idea to a ticket-ready backlog by composing seven actions in a fixed, interactive chain. Holds no business logic; every step resolves to a named delegation.

## Actions

| #   | Action           | Role                                              | Delegate                                                                    |
| --- | ---------------- | -------------------------------------------------- | --------------------------------------------------------------------------- |
| 01  | `preflight`      | Verify AIDD installed; capture output mode        | none (orchestration gate)                                                   |
| 02  | `brainstorm`     | Clarify the raw idea                              | `aidd-refine:01-brainstorm`                                                 |
| 03  | `prd`            | Produce the PRD (§8 Technical Architecture left TBD) | `aidd-pm:03-prd`                                                         |
| 04  | `bootstrap`      | Derive the stack from the PRD                     | `aidd-context:01-bootstrap` → action `06-gather-from-prd` then `02 → 03 → 04 → 05` |
| 05  | `epic-breakdown` | Break the PRD into epics                          | `aidd-pm:05-epic-breakdown` → actions `01-breakdown` + `02-coverage-check` |
| 06  | `user-stories`   | Generate user stories per epic (loop)             | `aidd-pm:02-user-stories` (one run per epic)                         |
| 07  | `handoff`        | Confirm boundary; default = stop; opt-in offer to start SDLC on the first epic | none by default; `aidd-dev:00-sdlc` on explicit opt-in |

Run `01 → 07` in strict order; no step is skipped, merged, or reordered. Run each action's `## Test` before moving to the next.

## Transversal rules

- **Pure orchestrator.** Never brainstorm, write PRDs, choose a stack, split epics, or draft user stories directly; every substantive step resolves to exactly one named delegation (`plugin:skill[:action]`).
- **Prerequisite.** AIDD must be installed (`aidd_docs/` present at the repo root). Absent → `01-preflight` aborts the entire run immediately; produce nothing, create nothing.
- **Interactive only.** Pauses at every key-artifact gate (after brainstorm, after PRD, after bootstrap, after epic breakdown, after user stories, and the opt-in offer at handoff); there is no auto mode.
- **Output mode propagated unchanged.** `output_mode` (`file | ticket | both`, default `file`) is captured once in `01-preflight` and passed as-is to `05-epic-breakdown` (as `output_target`) and to `06-user-stories`; never re-asked, never reimplemented here.
- **Runtime tracking.** Materialize the chain as a task list at entry, before running `01-preflight`; a task closes only when its delegate returns its validated artifact and the gate is approved.
- **Contract handoff.** Each step starts only from the previous step's validated artifact. The PRD must carry `Status: Approved` before `04-bootstrap` or `05-epic-breakdown` run; each epic from `05-epic-breakdown` feeds exactly one `06-user-stories` run.
- **Default-stop boundary.** Stops once stories are materialized; `aidd-dev:00-sdlc` is never started automatically. **Opt-in only:** `07-handoff` offers an explicit yes/no (default = stop) to start `aidd-dev:00-sdlc` on `epics[0]` after a branch-discipline check.
- **Consumed as-is.** Never modify a delegated skill.
- **Language.** All artifacts authored in English.
