---
name: 13-greenfield
description: Pure orchestrator for the greenfield project flow. Takes a raw product idea to a backlog of SDLC-ready tickets by composing existing skills in a fixed chain — brainstorm → prd → bootstrap(from-prd) → epic-breakdown → user-stories. Interactive only. Holds no business logic of its own; every step is delegated.
---

# Skill: greenfield

Greenfield entry door. Drives a product idea to a ticket-ready backlog by composing five AIDD
skills in a fixed, sequential chain. Always interactive: brainstorm and PRD require human input by
nature. Contains zero business logic — every substantive step resolves to a named delegation.

## Iron rule

**You are the conductor, not a player.**

You orchestrate skills; you never brainstorm, write PRDs, choose a stack, split epics, or draft
user stories yourself.

Every substantive chain step resolves to exactly one named delegation (`plugin:skill[:action]`).
You route the previous step's validated artifact into the next delegate. Nothing more.

## Prerequisite

AIDD must be installed: `aidd_docs/` must exist at the repo root. If it is absent,
`01-preflight` stops the run immediately and produces nothing. This orchestrator does not scaffold
the framework or the memory bank.

## Available actions

| #   | Action           | Role                                              | Delegate                                                                    |
| --- | ---------------- | ------------------------------------------------- | --------------------------------------------------------------------------- |
| 01  | `preflight`      | Verify AIDD installed; capture output mode        | none (orchestration gate)                                                   |
| 02  | `brainstorm`     | Clarify the raw idea                              | `aidd-refine:01-brainstorm`                                                 |
| 03  | `prd`            | Produce the PRD (§8 Technical Architecture left TBD) | `aidd-pm:03-prd`                                                         |
| 04  | `bootstrap`      | Derive the stack from the PRD                     | `aidd-context:01-bootstrap` → action `06-gather-from-prd` then `02 → 03 → 04 → 05` |
| 05  | `epic-breakdown` | Break the PRD into epics                          | `aidd-pm:05-epic-breakdown` → actions `01-breakdown` + `02-coverage-check` |
| 06  | `user-stories`   | Generate user stories per epic (loop)             | `aidd-pm:02-user-stories` (one run per epic)                         |
| 07  | `handoff`        | Confirm boundary; default = stop at tickets; optionally offer to start `aidd-dev:00-sdlc` on the first epic (opt-in) | none by default; `aidd-dev:00-sdlc` on explicit opt-in |

Files: `@actions/01-preflight.md` ... `@actions/07-handoff.md`.

## Default flow

`01 → 02 → 03 → 04 → 05 → 06 → 07`. Strictly sequential; no step is skipped or merged.

- `01-preflight` may **halt** the entire flow if `aidd_docs/` is absent.
- `03-prd` passes an explicit instruction to leave §8 Technical Architecture as `TBD`; the stack
  is resolved afterwards by `04-bootstrap`, never frozen in the PRD.
- `04-bootstrap` enters `aidd-context:01-bootstrap` at action `06-gather-from-prd` (PRD-driven,
  not Q&A), then runs `02-propose-candidates → 03-audit-candidates → 04-pick-and-design →
  05-write-install-md` to produce `aidd_docs/INSTALL.md`.
- `06-user-stories` loops: one run of `aidd-pm:02-user-stories` per epic produced by
  `05-epic-breakdown`.
- `07-handoff` is the terminal boundary. Tickets are ready. Default behavior: stop here; the
  SDLC (`aidd-dev:00-sdlc`) is never started automatically (default = stop).
  **Tier-2 opt-in:** after the end boundary message, `07-handoff` presents an explicit yes/no
  offer to start `aidd-dev:00-sdlc` on the first epic only. Default on no input or decline =
  stop (unchanged behavior). On explicit `yes`, a branch-discipline check runs first; if HEAD
  is non-default, delegates to `aidd-dev:00-sdlc` with `epics[0]` as scope.

After each action, run its `## Test` before moving to the next.

## Interactive gates

This skill is **interactive only**. There is no auto mode: brainstorm and PRD require human input
by nature, so an unattended run is impossible.

Pause at each gate and wait for explicit human approval before proceeding to the next action.

1. **After `02-brainstorm`** — show the clarified idea; confirm before writing the PRD.
2. **After `03-prd`** — show the validated PRD path (`Status: Approved`); confirm before bootstrap.
3. **After `04-bootstrap`** — show the stack (`aidd_docs/INSTALL.md`); confirm before epic breakdown.
4. **After `05-epic-breakdown`** — show the epics list and coverage verdict; confirm before user stories.
5. **After `06-user-stories`** — show the materialized stories/tickets; confirm before handoff.
6. **At `07-handoff` (opt-in offer)** — after the end boundary message, explicit yes/no offer
   to start `aidd-dev:00-sdlc` on the first epic only; default = stop (no SDLC invocation).

If the human pushes back at a gate, route their feedback back into that action's delegate
(re-brainstorm, PRD refinement, re-bootstrap, re-breakdown, re-stories) before re-proposing the
gate. Never skip a gate.

## Runtime tracking

Materialize the chain as a task list at skill entry, before running `01-preflight`:

```
- [ ] 01-preflight  (AIDD check + output mode)
- [ ] 02-brainstorm (clarified idea)
- [ ] 03-prd        (validated PRD, §8 TBD)
- [ ] 04-bootstrap  (stack / aidd_docs/INSTALL.md)
- [ ] 05-epic-breakdown (epics + coverage)
- [ ] 06-user-stories   (stories / tickets per epic)
- [ ] 07-handoff    (end boundary)
```

A task closes only when its delegate returned its validated artifact and the interactive gate was
approved. Never close a task by skipping it.

## Transversal rules

- **Interactive only.** This skill always pauses at every key-artifact gate. There is no auto mode.
- **Prerequisite.** AIDD must be installed (`aidd_docs/` present). Absent → abort at `01-preflight`;
  produce nothing, create nothing.
- **Fixed chain.** Steps run in order `01 → 07`. No step is skipped, merged, or reordered.
- **Output mode propagated unchanged.** `output_mode` (file | ticket | both, default `file`) is
  asked once in `01-preflight` and passed as-is to `05-epic-breakdown` (as `output_target`) and to
  `06-user-stories`. This skill never reimplements ticket creation.
- **Contract handoff.** Each step starts only from the validated artifact of the previous step. The
  PRD must carry `Status: Approved` before `04-bootstrap` or `05-epic-breakdown` run. Each epic
  produced by `05-epic-breakdown` feeds exactly one `aidd-pm:02-user-stories` run.
- **End boundary.** By default, this skill stops once stories are materialized. It does not run
  the SDLC (`aidd-dev:00-sdlc`) automatically, does not start the first epic automatically, and
  does not produce spec, plan, code, review, or ship artifacts. **Opt-in only:** `07-handoff`
  presents an explicit offer (yes/no, default = stop) to start `aidd-dev:00-sdlc` on `epics[0]`
  only, after a branch-discipline check. No silent trigger; no SDLC logic reimplemented here.
- **Pure orchestrator.** Actions 02–06 contain zero brainstorming, PRD-writing, stack-choosing,
  epic-splitting, or story-drafting logic. All business logic lives in the named delegates.
- **Language.** All artifacts authored in English.

## Rules

- Never scaffold: if `aidd_docs/` is absent, stop with the explicit message in `01-preflight`;
  do not create the directory or any file.
- Never embed drafting or breakdown rules inside an action; each action resolves to one named
  delegate only.
- §8 Technical Architecture in the PRD must remain `TBD` after step 03. The stack is deduced by
  `04-bootstrap` from the PRD, never frozen in the PRD itself.
- `04-bootstrap` enters `aidd-context:01-bootstrap` at action `06-gather-from-prd`, not
  `01-gather-needs`. The validated PRD path is the sole input.
- Do not modify any delegated skill (they are consumed as-is).
- A dry-read of this SKILL.md and the seven action files must let a project-naive reader name the
  skill called at each step with zero ambiguity.

## References

- `aidd-refine:01-brainstorm` — idea clarification (action 02)
- `aidd-pm:03-prd` — PRD generation (action 03)
- `aidd-context:01-bootstrap` → `06-gather-from-prd` — PRD-to-stack derivation (action 04)
- `aidd-pm:05-epic-breakdown` — PRD-to-epics breakdown (action 05)
- `aidd-pm:02-user-stories` — per-epic story generation (action 06)
- `aidd-dev:00-sdlc` — the next step after this skill completes; optionally started on explicit
  opt-in at `07-handoff` (first epic only, after branch-discipline check)

## Assets

- None.
