# 02 - Coverage Check

Read-only action. Compare the Core Features declared in a PRD against the set of produced epic
artifacts, emit a coverage matrix, and return a pass/fail verdict. Used by reviewers, orchestrators,
and idempotence tests — it does not create, modify, or delete any file.

## Input

- `prd_path` (required): the same PRD file used when `01-breakdown` was run.
- `epics_location` (required): directory (or glob) where the epic `.md` files produced by
  `01-breakdown` reside, typically `aidd_docs/tasks/<prd_yyyy_mm>/`. May also be a list of
  individual file paths.

## Output

A coverage matrix (one row per PRD feature, showing which `EPIC-<slug>` covers it or that it is
missing), the `missing`/`extra`/`duplicates` diagnostic lists, and a `verdict: pass | fail` that
passes only when all three lists are empty, with `notes` explaining any failure.

## Process

This action is read-only; it makes no writes.

1. **Extract.** Open `prd_path` and locate the `## 4. Core Features` section. Collect ONLY lines
   matching the exact heading pattern `### Feature N - <name>` (a level-3 markdown heading
   starting with `###`, the literal word `Feature`, a positive integer, a hyphen, and the feature
   name); ignore all other content under the section, intro prose, bullet lists, tables, and
   sub-subsections that do not match the pattern. The collected headings, in document order, form
   the authoritative feature set `F_PRD = {F1, F2, …, FN}`. Derive each feature's `feature_slug`
   from `<name>` by the shared slug normalization:

   ```text
   @../references/slug-normalization.md
   ```
2. **Collect.** Enumerate all `.md` files in `epics_location` whose filenames match the pattern
   `*-epic-<NN>-*.md` (or read each file provided directly). For each file, parse the YAML
   frontmatter and extract `epic_id` and `source_features`. Build the epic set
   `E = {E1, E2, …, EM}` where each `Ei` records its `source_features`, then build the multi-map
   `covered_by(Fi) = { Ej | Fi ∈ Ej.source_features }`.
3. **Compare.** Compute three disjoint diagnostic sets: `missing` = `{ Fi ∈ F_PRD | covered_by(Fi)
   = ∅ }` (PRD features with no epic); `extra` = `{ G ∈ ⋃ Ej.source_features | G ∉ F_PRD }`
   (feature references in epics that do not correspond to any PRD feature, indicating stale or
   invented feature names); `duplicates` = `{ Fi ∈ F_PRD | |covered_by(Fi)| > 1 }` (features
   covered by more than one epic, violating the 1:1 partition rule).
4. **Matrix.** Produce one row per PRD feature (`### Feature N - <name>` → `EPIC-<slug>` or `—
   (missing)` → covered/missing), and append a summary row: `<covered count> / <total count>
   features covered`.
5. **Verify.** When called after multiple `01-breakdown` runs on the same PRD,
   collect the set of distinct `epic_id` values found in `epics_location` after the most recent
   run.
   - If a prior snapshot is available (e.g. from a first-run output record): assert the set of
     `epic_id` values is identical, no ids added, none removed. When no prior snapshot is
     available (first invocation), record the current set as the baseline.
   - Assert independently that no `epic_id` appears in more than one file (`duplicates = []`).
   - Record the assertion result in `notes`: a pass here confirms the second run produced no new
     epics and dropped none, the idempotence invariant holds.
   - Note: the set size is NOT required to equal `|F_PRD|`. When M:1 grouping is used (one epic
     covering multiple PRD features as permitted by `01-breakdown` step 4), the epic count will
     be less than the feature count and that is correct. Coverage completeness is enforced by the
     compare step (`missing = []`), not by a count comparison.
6. **Verdict.** `verdict: pass` iff `missing = []`, `extra = []`, and `duplicates = []`;
   `verdict: fail` otherwise, with `notes` populated with the list of missing features and their
   count, the list of extra feature references and which epic files contain them, and the list of
   duplicated features and which epics each appears in. Return the full `Output` block. Make no
   writes.

## Test

- **Pass — perfect coverage**: run against a PRD and its produced epics after a successful
  `01-breakdown` run. Expect `verdict: pass`, `missing = []`, `extra = []`, `duplicates = []`,
  `coverage_matrix` has N rows all showing `covered`.
- **Fail — missing feature**: manually remove a feature's epic file, then run `coverage-check`.
  Expect `verdict: fail` and the omitted feature in `missing`.
- **Fail — extra reference**: manually add a feature name to an epic's `source_features` that
  does not appear in the PRD. Expect `verdict: fail` and the invented name in `extra`.
- **Fail — duplicate**: assign the same PRD feature to two epic files. Expect `verdict: fail` and
  the feature in `duplicates` with both epic IDs listed.
- **Idempotence after two runs**: run `01-breakdown` twice on the same PRD, then run
  `coverage-check`. Expect `verdict: pass` and the identical set of `epic_id` values as after
  the first run (no new ids, no dropped ids, no duplicates). The `notes` field should confirm:
  `"Epic id set unchanged across runs. No duplicates detected."`
- **Read-only**: confirm that `coverage-check` leaves the filesystem unchanged (no files created,
  modified, or deleted) regardless of whether the verdict is pass or fail.
- **PRD not found**: call with an invalid `prd_path`. Expect an explicit error message and no
  output block produced.
- **Empty epics_location**: call with a valid PRD but an empty or non-existent `epics_location`.
  Expect `verdict: fail` with all PRD features in `missing`.
