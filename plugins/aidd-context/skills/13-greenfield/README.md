# greenfield

Pure orchestrator that drives a raw product idea to a backlog of SDLC-ready tickets, delegating
every step. See `SKILL.md` for the action chain, the delegates, and the transversal rules
(interactive gates, output-mode propagation, default-stop boundary).

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

## How to invoke

```
/greenfield <your raw idea>
```

or simply describe your idea; the skill picks it up from context.
