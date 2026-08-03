# 03 - PRD

Delegates PRD generation to `aidd-pm:03-prd` with an explicit instruction to leave §8 Technical
Architecture as `TBD`, routing the clarified idea in and the validated PRD path out.

**No business logic.** Contains no PRD-writing, template-filling, or section-authoring logic of
its own.

## Input

`clarified_idea`, the clarified idea approved at action 02's gate. Precondition: `clarified_idea`
is present (the gate at action 02 was approved).

## Output

`prd_path`, the path to the saved PRD file; must carry `Status: Approved`.

## Process

1. **Delegate.** Pass `clarified_idea` as the `feature_description` input to `aidd-pm:03-prd`,
   together with the following instruction passed verbatim to the delegate:

   > **Greenfield run — §8 Technical Architecture must be left as `TBD`.**
   > This PRD is the input to a bootstrap step that deduces the technical architecture from the
   > product requirements (FR4.1). Do not freeze any stack, framework, hosting platform, or
   > database choice in §8 Technical Architecture. Leave the entire section as `TBD`. The stack
   > will be filled by the bootstrap step after this PRD is validated.

2. **Run.** Let `aidd-pm:03-prd` run its standard flow, parse input, draft per its template,
   iterate with the user, validate, save to
   `aidd_docs/tasks/<yyyy_mm>/<yyyy_mm_dd>-<feature_name>-prd.md`, without intervening in
   PRD drafting.
3. **Receive.** Take the saved PRD path from the delegate.
4. **Gate.** Confirm the validated PRD:

   > PRD saved at `<prd_path>`. Please confirm it carries `Status: Approved` and that §8 Technical
   > Architecture is `TBD`. Reply `yes` to continue or provide feedback to refine the PRD.

   - On feedback, re-invoke `aidd-pm:03-prd` with the refinement context before re-proposing
     the gate.
   - On approval, store `prd_path` (the path to the validated PRD carrying `Status: Approved`)
     and proceed to `04-bootstrap`.

## Test

- Confirm this action passes the verbatim §8 Technical Architecture TBD instruction to
  `aidd-pm:03-prd`.
- Confirm the produced PRD's §8 Technical Architecture section contains `TBD` (not a stack choice).
- Confirm this action contains no PRD-drafting, template-filling, or section-writing logic.
- Confirm `prd_path` is not stored until the gate is approved with `Status: Approved`.
- Confirm that feedback at the gate re-invokes `aidd-pm:03-prd`, not self-written refinement logic.
