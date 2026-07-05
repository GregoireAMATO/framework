# 07 - Handoff

Terminal action. Confirms that epics and user stories are materialized, states the explicit end
boundary, and names `aidd-dev:00-sdlc` as the next step. On the default path (no input or
anything other than an explicit "yes"), stops here — no delegation, no SDLC invocation.

After the end boundary, presents a one-time explicit opt-in offer to start `aidd-dev:00-sdlc`
on the first epic only. This offer is strictly additive: decline (or no reply) terminates
exactly as today. On explicit accept, this action performs a branch-discipline check, then
delegates to `aidd-dev:00-sdlc` — it reimplements no SDLC logic.

## Inputs

```yaml
epics: <the epics list approved at action 05's gate; order preserved from aidd-pm:05-epic-breakdown>
stories: <the materialized user stories / tickets approved at action 06's gate>
prd_path: <the validated PRD path from action 03>
output_mode: <the output mode captured at action 01: file | ticket | both>
```

Note: `epics` carries the ordering produced by `aidd-pm:05-epic-breakdown` and is not re-sorted
here. `epics[0]` is the first epic of that ordered list and is used deterministically in the
opt-in offer. When `output_mode = file`, `epics[0]` is the file path of the first epic artifact;
when `output_mode = ticket` or `both`, `epics[0]` is the ticket reference of the first epic.

## Outputs

No artifacts produced. End message only (on the default path). On explicit opt-in accept,
control transfers to `aidd-dev:00-sdlc`; its outputs are the SDLC's, not this action's.

## Process

### Step 1 — Confirm session deliverables

List what was produced during the session:

- PRD at `<prd_path>` (validated, `Status: Approved`).
- `<N>` epics derived from the PRD, saved as `<output_mode>`.
- User stories materialized as `<output_mode>` for each of the `<N>` epics.

### Step 2 — State the end boundary

Present the following message verbatim:

> **Greenfield chain complete. Tickets are ready.**
>
> - Epics: `<N>` epics derived from `<prd_path>`.
> - User stories: materialized as `<output_mode>` for each epic.
>
> This skill's job — turning an idea into a ready backlog — ends here. **The next phase is
> `aidd-dev:00-sdlc`** (spec → plan → implement → review → ship).
>
> By default it is a separate step you run yourself: invoke `/sdlc` when you are ready to begin
> implementation of the first epic. You can also start it now via the optional offer below.

### Step 3 — Offer the Tier-2 opt-in (additive gate)

After the end boundary message, present the following optional offer:

> **Optional — start `aidd-dev:00-sdlc` now on the first epic?**
>
> First epic: `<epics[0] name / path / ticket reference>` (the first of the `<N>` epics produced
> by epic-breakdown; this is the only epic offered here).
>
> Reply `yes` to start `aidd-dev:00-sdlc` on this epic now.
> Press Enter or reply anything else to **stop here (default)**.
>
> Note: to start the SDLC on a later epic, invoke `/sdlc` manually with that epic as scope.

**Default = stop.** No input, `no`, or anything other than an explicit affirmative → go to
Step 5 (terminate as today, identical to the behavior before this opt-in was added).

### Step 4 — On explicit accept: branch check, then delegate

Execute this step only if the user replied with an explicit affirmative (e.g., `yes`) in Step 3.

#### 4a — Branch-discipline check

Determine the current HEAD branch:

```
git rev-parse --abbrev-ref HEAD
```

- If HEAD is on a **non-default branch** (not `main`, not `master`, and not `HEAD` /
  detached) → proceed to Step 4b.
- If HEAD is on **`main`**, **`master`**, or is **detached** → emit the following warning and
  stop without delegating:

  > **Branch check failed — cannot delegate to `aidd-dev:00-sdlc` from here.**
  >
  > `aidd-dev:00-sdlc` never auto-branches (branch discipline is the caller's responsibility).
  > Running SDLC on `main` / `master` / detached HEAD would bypass the PR workflow.
  >
  > Create a feature branch first:
  > ```
  > git checkout -b <feature-branch-name>
  > ```
  > Then re-invoke this handoff (or `/sdlc` directly) once HEAD is on that branch.
  >
  > No delegation was started. Your backlog is complete and ready whenever you are.

  Do **not** create or switch branches automatically. Do not delegate until HEAD is non-default.

#### 4b — Delegate to `aidd-dev:00-sdlc`

With HEAD confirmed on a non-default branch, invoke **`aidd-dev:00-sdlc`** with the first
epic artifact as scope:

- Scope input: `epics[0]` — when `output_mode = file`, the file path of the first epic;
  when `output_mode = ticket` or `both`, the ticket reference of the first epic.
- Mode: default (`auto`). The SDLC's `interactive` mode is also available — pass
  `interactive` as an argument to the SDLC invocation if you prefer supervised gates.
- Do not pre-run any SDLC step (spec, plan, implement, review, ship); `aidd-dev:00-sdlc`
  owns the full flow from this point.

### Step 5 — Stop (default path)

If the opt-in was not accepted (Step 3 default), stop. Do not invoke `aidd-dev:00-sdlc`.
Do not run any spec, plan, implement, review, or ship action. Do not start or auto-trigger
any epic. Do not create any file or ticket.

## Test

- Confirm Step 2 opens with the heading "Greenfield chain complete. Tickets are ready." and
  frames `aidd-dev:00-sdlc` as the next phase (default = run separately; optionally startable
  via Step 3) — without asserting it cannot be started here.
- Confirm the default path (no reply or anything other than an explicit "yes" in Step 3)
  terminates at Step 5 without invoking `aidd-dev:00-sdlc`.
- Confirm the opt-in offer in Step 3 is explicit (yes/no), names `aidd-dev:00-sdlc`, and
  identifies `epics[0]` (the first of the ordered epic list) as the scope offered.
- Confirm `epics[0]` is deterministic: the first element of the ordered list from action 05,
  no re-sort, no user pick.
- Confirm the branch check in Step 4a warns and stops (without delegating) when HEAD is
  `main`, `master`, or detached; and that it does not auto-create or auto-switch branches.
- Confirm Step 4b names `aidd-dev:00-sdlc` as the explicit delegate and passes `epics[0]`
  as scope; confirm no SDLC sub-action (spec, plan, implement, review, ship) is reimplemented
  here.
- Confirm no spec, plan, code, review, or ship artifact is produced by this action.
- Confirm no file or ticket is created by this action (on the default path).
