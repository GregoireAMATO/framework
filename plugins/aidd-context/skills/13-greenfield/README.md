# greenfield

Pure orchestrator that drives a raw product idea to a backlog of SDLC-ready tickets.

The chain: **brainstorm → prd → bootstrap(from-prd) → epic-breakdown → user-stories**.

Every step delegates to a named AIDD skill. This skill contains no brainstorming, PRD-writing,
stack-choosing, epic-splitting, or story-drafting logic of its own.

## When to use

- Starting a brand-new project from a fuzzy idea.
- You want to arrive at a ticket-ready backlog in one guided session.
- AIDD is already installed (`aidd_docs/` exists at the repo root).

## When NOT to use

- AIDD is not installed — run `aidd-context:00-onboard` first, then return here.
- A validated PRD already exists — start from `aidd-pm:05-epic-breakdown` directly.
- Epics already exist — start from `aidd-pm:02-user-stories` directly.
- You need to run the SDLC (spec / plan / implement / review / ship) — use `aidd-dev:00-sdlc`
  directly (or accept the opt-in offer at handoff to start it on the first epic).
- You want an unattended / auto run — this skill is interactive only by design.

## Prerequisites

- AIDD is installed: `aidd_docs/` is present at the repo root.

## How to invoke

```
/greenfield <your raw idea>
```

or simply describe your idea; the skill picks it up from context.

## The chain at a glance

| Step | Action          | Delegate                                                              | Artifact produced                   |
| ---- | --------------- | --------------------------------------------------------------------- | ----------------------------------- |
| 01   | preflight       | (none — guard)                                                        | `go/no-go` + `output_mode`          |
| 02   | brainstorm      | `aidd-refine:01-brainstorm`                                           | clarified idea                      |
| 03   | prd             | `aidd-pm:03-prd`                                                      | validated PRD (`Status: Approved`)  |
| 04   | bootstrap       | `aidd-context:01-bootstrap` → action `06-gather-from-prd`            | `aidd_docs/INSTALL.md` (stack)      |
| 05   | epic-breakdown  | `aidd-pm:05-epic-breakdown` → `01-breakdown` + `02-coverage-check`   | epics list + coverage verdict       |
| 06   | user-stories    | `aidd-pm:02-user-stories` (one run per epic)                   | stories / tickets per epic          |
| 07   | handoff         | (none — boundary)                                                     | end message                         |

## Output modes

At preflight you choose once where epics and user stories are saved:

| Mode     | Effect                                                       |
| -------- | ------------------------------------------------------------ |
| `file`   | Written to `aidd_docs/` (default — no external tools needed) |
| `ticket` | Created in the configured ticketing tool                     |
| `both`   | Written to file AND created as tickets                       |

The choice is captured once in `01-preflight` and propagated unchanged to `05-epic-breakdown`
(as `output_target`) and to `06-user-stories`. It is never re-asked.

## Interactive gates

The skill pauses after every key artifact and waits for explicit approval before continuing:

1. After brainstorm — confirm the clarified idea.
2. After PRD — confirm `Status: Approved` and §8 Technical Architecture is `TBD`.
3. After bootstrap — confirm the stack (`aidd_docs/INSTALL.md`).
4. After epic breakdown — confirm the epics list and coverage verdict.
5. After user stories — confirm stories are materialized.
6. At handoff — explicit opt-in offer to start `aidd-dev:00-sdlc` on the first epic (default = stop).

Push back at any gate to refine that step before continuing.

## End boundary

This skill **stops** once stories are materialized by default. It does not start the SDLC
automatically, does not implement code, and does not open a pull request.

At `07-handoff`, an explicit opt-in offer is presented (yes/no, default = stop): start
`aidd-dev:00-sdlc` on the **first epic only**? Decline or no reply keeps the default behavior —
stop here. On explicit accept, a branch-discipline check runs before delegation; HEAD must be on
a non-default branch (not `main`/`master`/detached) or the action warns and guides without
delegating.

The next step — `aidd-dev:00-sdlc` — remains a deliberate, explicit choice.
