# 04 - Bootstrap

Delegates stack derivation to `aidd-context:01-bootstrap`, entering at action
`06-gather-from-prd` (PRD-driven, not `01-gather-needs`), routing the validated PRD path in and
`aidd_docs/INSTALL.md` out.

**No business logic.** Contains no stack-choosing, candidate-comparison, or architecture logic
of its own.

## Input

`prd_path`, the path to the validated PRD; must carry `Status: Approved`. Precondition: the file
must exist and contain a line where both `Status` and `Approved` appear; if not, halt and return
an error before invoking the delegate.

## Output

`install_md_path` (`aidd_docs/INSTALL.md`, produced by `aidd-context:01-bootstrap`) and
`stack_summary`, the chosen stack as returned by the delegate.

## Process

1. **Verify.** Check that `prd_path` exists on disk and contains a line where both `Status` and
   `Approved` appear (tolerant of bold markers and whitespace).
   - If not, halt with the message `"PRD at <prd_path> is not validated (no 'Status … Approved'
     line found). Approve the PRD before running bootstrap."` Produce nothing.
2. **Enter.** Pass `prd_path` to `aidd-context:01-bootstrap` action `06-gather-from-prd` (not
   `01-gather-needs`). This action derives the bootstrap architecture checklist from the PRD
   without Q&A interaction, producing the same filled checklist that `01-gather-needs` would
   have produced.
3. **Run.** Let `aidd-context:01-bootstrap` continue its standard chain from the filled
   checklist, without intervening in stack selection or candidate evaluation:
   - `02-propose-candidates` — derive 2-3 candidate stacks, render a comparison table.
   - `03-audit-candidates` — spawn parallel agents to validate each candidate, emit a verdict.
   - `04-pick-and-design` — the user picks the winner; generate the folder tree and Mermaid
     diagram.
   - `05-write-install-md` — produce `aidd_docs/INSTALL.md`.
4. **Receive.** Take `aidd_docs/INSTALL.md` as the stack artifact.
5. **Gate.** Confirm the stack:

   > Stack derived from the PRD. `aidd_docs/INSTALL.md` is ready.
   > Do you confirm this stack before running epic breakdown?
   > Reply `yes` to continue or provide feedback to revise the stack.

   - On feedback, re-invoke `aidd-context:01-bootstrap` from the appropriate step with the
     revision context before re-proposing the gate.
   - On approval, store `install_md_path = aidd_docs/INSTALL.md` and proceed to
     `05-epic-breakdown`.

## Test

- Confirm this action names `06-gather-from-prd` as the bootstrap entry point, not `01-gather-needs`.
- Confirm the precondition check halts if `prd_path` lacks a `Status … Approved` line, producing
  nothing.
- Confirm the action contains no stack-choosing, candidate-comparison, audit, or folder-tree logic.
- Confirm `aidd_docs/INSTALL.md` is produced by the delegate (`aidd-context:01-bootstrap`), not
  by this action.
- Confirm `install_md_path` is not stored until the gate is approved.
- Confirm that feedback at the gate re-invokes `aidd-context:01-bootstrap`, not self-written
  stack revision logic.
