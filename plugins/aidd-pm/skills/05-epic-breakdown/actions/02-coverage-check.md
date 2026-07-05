# 02 - Coverage Check

Read-only action. Compare the Core Features declared in a PRD against the set of produced epic
artifacts, emit a coverage matrix, and return a pass/fail verdict. Used by reviewers, orchestrators,
and idempotence tests — it does not create, modify, or delete any file.

## Inputs

```yaml
prd_path: <path to the validated PRD file>   # required
epics_location: <directory or glob pattern>   # required; e.g. aidd_docs/tasks/2026_06/
```

- `prd_path`: same PRD file used when `01-breakdown` was run.
- `epics_location`: directory (or glob) where the epic `.md` files produced by `01-breakdown`
  reside. Typically `aidd_docs/tasks/<prd_yyyy_mm>/`. May also be a list of individual file paths.

## Outputs

```yaml
prd_features:
  - heading: "### Feature N - <name>"
    feature_slug: <slug>
epics_found:
  - epic_id: EPIC-<feature-slug>
    path: <file path>
    source_features:
      - "### Feature N - <name>"
coverage_matrix:
  - feature: "### Feature N - <name>"
    covered_by: EPIC-<feature-slug> | null
missing: []          # PRD features with no covering epic
extra: []            # Features referenced by epics but absent from the PRD
duplicates: []       # Features referenced by more than one epic
verdict: pass | fail
notes: <human-readable summary of failures, if any>
```

- `verdict: pass` iff `missing`, `extra`, and `duplicates` are all empty lists.
- `verdict: fail` otherwise, with `notes` describing which condition(s) failed.

## Process

This action is **read-only**. It makes no writes.

### Step 1 — Extract PRD feature set

1. Open `prd_path` and locate the `## 4. Core Features` section.
2. Collect ONLY lines that match the exact heading pattern `### Feature N - <name>` (a level-3
   markdown heading starting with `###`, the literal word `Feature`, a positive integer, a hyphen,
   and the feature name). Ignore all other content under `## 4. Core Features` — intro prose,
   bullet lists, tables, and sub-subsections that do not match this pattern. The collected headings,
   in document order, form the authoritative feature set `F_PRD = {F1, F2, …, FN}`.
3. Derive each feature's `feature_slug` from the `<name>` portion using the same deterministic
   normalization as `01-breakdown` Step 2:
   1. Lowercase the entire name.
   2. Strip diacritics/accents to ASCII equivalents (é→e, ê→e, è→e, à→a, â→a, ç→c, î→i, ô→o,
      û→u, ü→u, ï→i, ë→e, and equivalents for all other combining diacritics).
   3. Replace every contiguous run of non-alphanumeric characters (spaces, apostrophes, hyphens,
      colons, punctuation, etc.) with a single hyphen (`-`).
   4. Trim any leading or trailing hyphens from the result.
   Examples: `Centre d'aide en libre-service` → `centre-d-aide-en-libre-service`;
             `Enquêtes de satisfaction` → `enquetes-de-satisfaction`.

### Step 2 — Extract epic feature references

1. Enumerate all `.md` files in `epics_location` whose filenames match the pattern
   `*-epic-<NN>-*.md` (or read each file provided directly).
2. For each file: parse the YAML frontmatter. Extract `epic_id` and `source_features` list.
3. Build the epic set `E = {E1, E2, …, EM}` where each `Ei` records its `source_features`.
4. Build the multi-map `covered_by(Fi) = { Ej | Fi ∈ Ej.source_features }`.

### Step 3 — Set comparison (coverage equality check)

Compute three disjoint diagnostic sets:

- **`missing`** = `{ Fi ∈ F_PRD | covered_by(Fi) = ∅ }` — features in the PRD with no epic.
- **`extra`** = `{ G ∈ ⋃ Ej.source_features | G ∉ F_PRD }` — feature references in epics that do
  not correspond to any PRD feature (indicates stale or invented feature names).
- **`duplicates`** = `{ Fi ∈ F_PRD | |covered_by(Fi)| > 1 }` — features covered by more than one
  epic (violates the 1:1 partition rule from HC2).

### Step 4 — Build coverage matrix

Produce one row per PRD feature:

| Feature | Covered by | Status |
| --- | --- | --- |
| `### Feature N - <name>` | `EPIC-<slug>` or `— (missing)` | covered / missing |

Append a summary row: `<covered count> / <total count> features covered`.

### Step 5 — Idempotence verification

When called after multiple `01-breakdown` runs on the same PRD:

1. Collect the set of distinct `epic_id` values found in `epics_location` after the most recent run.
2. If a prior snapshot is available (e.g. from a first-run output record): assert that the set of
   `epic_id` values is **identical** — no new ids added, no existing ids removed.
   When no prior snapshot is available (first invocation), record the current set as the baseline.
3. Assert independently that no `epic_id` appears in more than one file (`duplicates = []`).
4. Record the assertion result in `notes`. A pass here confirms the second run produced no new
   epics and dropped none — the idempotence invariant holds (Done-when #5).

Note: the set size is NOT required to equal `|F_PRD|`. When M:1 grouping is used (one epic
covering multiple PRD features as permitted by `01-breakdown` Step 3.1), the epic count will be
less than the feature count and that is correct. Coverage completeness (all features referenced)
is enforced by Step 3 (`missing = []`), not by a count comparison.

### Step 6 — Verdict

- `verdict: pass` iff `missing = []`, `extra = []`, and `duplicates = []`.
- `verdict: fail` otherwise. Populate `notes` with:
  - List of missing features and the count.
  - List of extra feature references and which epic files contain them.
  - List of duplicated features and which epics each appears in.

Return the full `Outputs` block. Make no writes.

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
  `"Epic id set unchanged across runs. No duplicates detected."` (Done-when #5).
- **Read-only**: confirm that `coverage-check` leaves the filesystem unchanged (no files created,
  modified, or deleted) regardless of whether the verdict is pass or fail.
- **PRD not found**: call with an invalid `prd_path`. Expect an explicit error message and no
  output block produced.
- **Empty epics_location**: call with a valid PRD but an empty or non-existent `epics_location`.
  Expect `verdict: fail` with all PRD features in `missing`.
