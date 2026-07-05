# 06 - User Stories

Loops over the epics produced by action 05 and delegates one user-story generation run to
`aidd-pm:02-user-stories` per epic. Routes each epic and `output_mode` in, and stories
or tickets out. Contains no story-writing, INVEST-checking, or ticket-creation logic.

## Inputs

```yaml
epics: <the list of epic artifacts approved at action 05's gate>
output_mode: <captured by 01-preflight: file | ticket | both>
```

Precondition: `epics` is a non-empty list (gate at action 05 was approved). `output_mode` was
captured at action 01 and is propagated unchanged — it is not re-asked here.

## Outputs

```yaml
stories: <user stories / tickets produced per epic by aidd-pm:02-user-stories>
```

## Process

For each epic in `epics` (one run of `aidd-pm:02-user-stories` per epic):

1. Delegate to **`aidd-pm:02-user-stories`**:
   - Pass the current epic artifact as `feature_description`.
   - Pass `output_mode` as the save target for the produced stories (propagated unchanged from
     `01-preflight`; not re-asked).
   - Let `aidd-pm:02-user-stories` run its full flow: clarify scope (at most 3 questions),
     draft INVEST-compliant user stories, validate with the user, save to the configured target.
     Do not intervene in story drafting or validation.
   - Receive the produced stories for this epic.

2. Accumulate stories across all epics.

After all epics have been processed:

**Gate — confirm the stories:**

> User stories generated for all `<N>` epics. Stories/tickets are ready.
> Do you confirm the backlog before handoff?
> Reply `yes` to continue or identify specific epics to revise.

On feedback: re-invoke `aidd-pm:02-user-stories` for the identified epic(s) before
re-proposing the gate.

On approval: store `stories` (the full set of produced stories/tickets across all epics) and
proceed to `07-handoff`.

## Test

- Confirm `aidd-pm:02-user-stories` is the named delegate.
- Confirm the Process states an explicit per-epic iteration: one run of
  `aidd-pm:02-user-stories` per epic in `epics` (not one batched run for all epics).
- Confirm `output_mode` is passed to each delegate run unchanged, not re-asked.
- Confirm this action contains no story-drafting, INVEST-checking, or ticket-creation logic.
- Confirm `stories` is not stored until the gate is approved.
- Confirm that feedback at the gate targets only the identified epic(s) for re-generation, not
  the full loop.
