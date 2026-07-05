# 04 - Bootstrap

Delegates stack derivation to `aidd-context:01-bootstrap`, entering at action
`06-gather-from-prd` (PRD-driven entry — not `01-gather-needs`). Routes the validated PRD path
in and `aidd_docs/INSTALL.md` out. Contains no stack-choosing, candidate-comparison, or
architecture logic.

## Inputs

```yaml
prd_path: <path to the validated PRD; must carry Status: Approved>
```

Precondition: `prd_path` must point to a file that exists and contains a line where both `Status`
and `Approved` appear. If not, halt and return an error before invoking the delegate.

## Outputs

```yaml
install_md_path: aidd_docs/INSTALL.md    # produced by aidd-context:01-bootstrap
stack_summary: <the chosen stack as returned by the delegate>
```

## Process

Delegate to **`aidd-context:01-bootstrap`** entering at action **`06-gather-from-prd`**
(not `01-gather-needs`):

1. Verify the precondition: `prd_path` must exist on disk and contain a line where both `Status`
   and `Approved` appear (tolerant of bold markers and whitespace). If not: halt with message
   `"PRD at <prd_path> is not validated (no 'Status … Approved' line found). Approve the PRD
   before running bootstrap."` Produce nothing.

2. Pass `prd_path` to `aidd-context:01-bootstrap` action `06-gather-from-prd`. This action
   derives the bootstrap architecture checklist from the PRD without Q&A interaction, producing
   the same filled checklist as `01-gather-needs` would have produced.

3. Let `aidd-context:01-bootstrap` run its standard chain from the filled checklist:
   - `02-propose-candidates` — derive 2-3 candidate stacks, render comparison table.
   - `03-audit-candidates` — spawn parallel agents to validate each candidate, emit verdict.
   - `04-pick-and-design` — user picks the winner; generate folder tree + Mermaid diagram.
   - `05-write-install-md` — produce `aidd_docs/INSTALL.md`.
   Do not intervene in stack selection or candidate evaluation.

4. Receive `aidd_docs/INSTALL.md` as the stack artifact.

**Gate — confirm the stack:**

> Stack derived from the PRD. `aidd_docs/INSTALL.md` is ready.
> Do you confirm this stack before running epic breakdown?
> Reply `yes` to continue or provide feedback to revise the stack.

On feedback: re-invoke `aidd-context:01-bootstrap` from the appropriate step with the revision
context before re-proposing the gate.

On approval: store `install_md_path = aidd_docs/INSTALL.md` and proceed to `05-epic-breakdown`.

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
