# 02 - Brainstorm

Delegates idea clarification to `aidd-refine:01-brainstorm`, routing the raw idea in and the
clarified idea out.

**No business logic.** Contains no brainstorming, probing, or idea-shaping logic of its own.

## Input

The user's raw product idea from `01-preflight`. Precondition: `01-preflight` returned
`go_no_go = go`; if preflight aborted, this action does not run.

## Output

`clarified_idea`, the clarified idea artifact produced by `aidd-refine:01-brainstorm`, held in
conversation context and passed to action 03.

## Process

1. **Delegate.** Pass `raw_idea` as the input to `aidd-refine:01-brainstorm`. Let the skill run
   its full internal flow, `01-capture → probe/integrate loop → 04-finalize`, without intervening
   in the clarification process.
2. **Receive.** Take the clarified idea artifact returned by `aidd-refine:01-brainstorm` action
   `04-finalize`.
3. **Present.** Show the clarified idea to the user.
4. **Gate.** Confirm before continuing:

   > The idea has been clarified. Do you confirm this is the right scope to take into the PRD?
   > Reply `yes` to continue or provide feedback to re-brainstorm.

   - On feedback, re-invoke `aidd-refine:01-brainstorm` with the updated context before
     re-proposing the gate.
   - On approval, store `clarified_idea` and proceed to `03-prd`.

## Test

- Confirm this action contains no brainstorming, probing, or idea-shaping logic — it delegates
  entirely to `aidd-refine:01-brainstorm`.
- Confirm the gate is presented after the delegate returns its artifact, not before.
- Confirm that feedback at the gate loops back into `aidd-refine:01-brainstorm`, not into
  self-written logic in this action.
- Confirm `clarified_idea` is not stored until the gate is approved.
- Confirm this action does not run if `01-preflight` returned `go_no_go = abort`.
