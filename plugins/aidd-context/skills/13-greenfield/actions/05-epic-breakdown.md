# 05 - Epic Breakdown

Delegates PRD-to-epics breakdown to `aidd-pm:05-epic-breakdown`. Routes the validated PRD path
and `output_mode` in, and the epics list out. Contains no epic-splitting, feature-mapping, or
template-filling logic.

## Inputs

```yaml
prd_path: <path to the validated PRD; must carry Status: Approved>
output_mode: <captured by 01-preflight: file | ticket | both>
```

Precondition: `prd_path` carries `Status: Approved` (verified at action 03's gate). `output_mode`
was captured at action 01 and is propagated unchanged — it is not re-asked here.

## Outputs

```yaml
epics: <list of epic artifacts produced by aidd-pm:05-epic-breakdown>
coverage_verdict: pass | fail    # from 02-coverage-check
```

## Process

Delegate to **`aidd-pm:05-epic-breakdown`**:

1. Run action **`01-breakdown`** with:
   - `prd_path` = the validated PRD path from action 03.
   - `output_target` = `output_mode` (propagated unchanged from `01-preflight`; never re-asked).
   Let `01-breakdown` validate the PRD, parse its Core Features, derive N epics, and write the
   epic artifacts to the chosen output target. Do not intervene in epic derivation.

2. Run action **`02-coverage-check`** with:
   - `prd_path` = same validated PRD path.
   - `epics_location` = the output location returned by `01-breakdown`.
   Let `02-coverage-check` compare PRD features against produced epics and emit the coverage
   matrix and verdict. If coverage is not 100%, route the gap report back into `01-breakdown`
   for correction before re-running `02-coverage-check`. Do not proceed to the gate until the
   coverage verdict is `pass`.

3. Receive the epics list and the coverage verdict (`pass`).

**Gate — confirm the epics:**

> Epics produced: `<N>` epics covering `<M>` PRD features. Coverage: `<verdict>`.
> Do you confirm the epics before generating user stories?
> Reply `yes` to continue or provide feedback to revise the breakdown.

On feedback: re-invoke `aidd-pm:05-epic-breakdown` action `01-breakdown` with the revision context,
then re-run `02-coverage-check`, before re-proposing the gate.

On approval: store `epics` (the list of produced epic artifacts) and proceed to `06-user-stories`.

## Test

- Confirm `aidd-pm:05-epic-breakdown` is the named delegate.
- Confirm `output_target` is set to `output_mode` (not hardcoded to `file` or any other value).
- Confirm `02-coverage-check` is run after `01-breakdown` and before the gate; confirm the
  coverage verdict must be `pass` before the gate is proposed.
- Confirm this action contains no epic-splitting, feature-mapping, or template-filling logic.
- Confirm `epics` is not stored until the gate is approved with a passing coverage verdict.
- Confirm `output_mode` is propagated from `01-preflight` and not re-asked in this action.
