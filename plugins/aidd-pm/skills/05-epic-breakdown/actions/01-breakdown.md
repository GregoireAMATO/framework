# 01 - Breakdown

Validate a PRD, parse its Core Features, derive one epic per feature, and write the results to
the chosen output target (file, ticket, or both). Enforces 100% coverage, zero invention, and
idempotence across runs.

## Input

- `prd_path` (required): relative or absolute path to an existing PRD that has passed validation.
- `output_target` (optional, default `file`): `file` writes epic markdown artifacts to
  `aidd_docs/tasks/<prd_yyyy_mm>/`; `ticket` creates one tracker issue per epic via
  `aidd-vcs:04-issue-create`, degrading to `file` with a warning when no ticketing tool is
  configured; `both` performs both.

## Output

One record per epic (`epic_id`, `title`, `source_feature`, `path`, `ticket_url` or null,
`status: created | skipped`), a `coverage_summary` (features/epics counts, skips, deduplicated
TBD count), and `notes` listing warnings, degrade notices, and unresolved questions.

## Process

1. **Preflight.** Gate every write behind four checks (HC1); on any failure, return the stated
   error and produce nothing.
   - Verify `prd_path` points to a file that exists on disk. If not: return
     `"PRD not found at <prd_path>. Provide the path to a validated PRD."` and halt.
   - Read the file. Confirm it contains a line where both the word `Status` and the word
     `Approved` appear, tolerant of bold markers (`**`), extra whitespace, and colon placement
     (e.g. matches `Status: Approved`, `**Status** : Approved`, `**Status**: Approved`). If no
     such line is found, and no equivalent validation marker is configured in project memory:
     return `"PRD at <prd_path> is not validated (no 'Status … Approved' line found). Validate
     the PRD before running epic-breakdown."` and halt.
   - Confirm the file contains a `## 4. Core Features` section. If not: return `"PRD at
     <prd_path> lacks a '## 4. Core Features' section. A structured PRD with at least one
     Feature subsection is required."` and halt.
   - Confirm at least one `### Feature N - <name>` subsection exists under `## 4. Core
     Features`. If not: return `"No Feature subsections found under '## 4. Core Features'. Add
     at least one '### Feature N - <name>' entry to the PRD."` and halt.
2. **Parse.** Extract from the PRD filename the values that anchor every derived filename:
   `prd_date` (e.g. `2026_06_30` from `2026_06_30-greenfield-gateway-prd.md`), `prd_yyyy_mm`
   (e.g. `2026_06`), and `prd_slug` (e.g. `greenfield-gateway`, the kebab-case name between the
   date and `-prd.md`). These derive from the source PRD, never from the current run date
   (enables idempotence across days).
3. **Extract.** Parse ONLY lines matching the exact heading pattern `### Feature N - <name>` (a
   level-3 markdown heading starting with `###`, the literal word `Feature`, a positive integer,
   a hyphen, and the feature name); ignore all other content under `## 4. Core Features`, intro
   prose, bullet lists, tables, and any sub-subsections that do not match the pattern. Produce an
   ordered list `[F1, F2, …, FN]`. Each entry records:
   - The heading text verbatim, and its 1-based feature order index `N` as written in the PRD.
   - A `feature_slug` derived from `<name>` by the shared deterministic slug normalization
     (same algorithm as `02-coverage-check`):

     ```text
     @../references/slug-normalization.md
     ```
   - The `FR<N>.<y>` identifiers found in the feature's subsection body.
   This list is the authoritative scope for coverage: every item must map to exactly one epic.
4. **Partition.** For each feature `Fi` in order, assign one epic `Ei` (1:1 by default; M:1
   grouping is permissible only when the PRD explicitly groups features under a single heading,
   which is then treated as a single `Fi`).
   - Derive the epic identity: `epic_id` = `EPIC-<feature-slug>` (deterministic, stable across
     runs); `epic_filename` = `<prd_yyyy_mm_dd>-<prd_slug>-epic-<NN>-<feature_slug>.md`
     (zero-padded `NN` = feature index); `epic_filepath` =
     `aidd_docs/tasks/<prd_yyyy_mm>/<epic_filename>`.
   - Extract from the feature's PRD section whatever is available (name, FR descriptions,
     acceptance criteria, notes) and fill every template field that can be derived directly.
   - Mark every field that cannot be determined from the PRD as `TBD: <precise question>`. Never
     invent a value. Typical triggers: missing objective wording, ambiguous scope boundary,
     unresolved acceptance criterion.
   - Ensure no implementation detail enters the epic: no library, framework, design pattern,
     file-layout reference, or technology choice. If the PRD itself contains such details, omit
     them from the epic and emit `TBD: The PRD mentions [X] as an implementation detail; confirm
     whether the epic's scope should be defined independently of this choice.`
5. **Render.** For each `(Fi, Ei)` pair, populate `assets/epic-template.md` with the derived
   values: frontmatter (`epic_id`, `source_prd`, `source_features` list of Feature headings and
   FR ids, `status: draft`); `# Epic: <title>`; `## Objective` (what/why prose from the PRD
   feature description, TBD if absent); `## Scope` (bulleted outcomes from FR items, TBD
   placeholders where FR items are absent); `## Out of scope` (copy explicit PRD exclusions when
   present; emit `TBD: <precise question>` when the scope boundary is genuinely ambiguous; leave
   the section empty, with no generic placeholder, when the PRD mentions no exclusion and the
   boundary is clear); `## Source PRD reference` (exact PRD path + section heading + FR
   identifiers); `## Source feature(s)` (the mapping table of Feature heading → FR ids); `##
   Open questions / TBDs` (the consolidated list of TBD markers for this epic); `## User stories
   (children)` (a boilerplate note directing to `aidd-pm:02-user-stories`).
6. **Idempotence.** Before writing any artifact, scan `aidd_docs/tasks/<prd_yyyy_mm>/` for files
   matching the derived `epic_filepath` pattern.
   - For each candidate file found, read its `epic_id` frontmatter field. If it matches the
     derived `EPIC-<feature-slug>`: mark the epic as `status: skipped`, do NOT overwrite the
     file, and record the skip in `coverage_summary.epics_skipped`.
   - If no match: proceed to write the file in the next step.
   - This applies per epic independently: a run may create some new epics and skip others.
7. **Write.** Determine the effective output target first: when `output_target` is `ticket` or
   `both`, check whether a ticketing tool is configured (project memory's `tracker_tool` or
   `git_remote`, else `git remote get-url origin` to infer the platform); if none is detected,
   log `"No ticketing tool configured. Degrading output_target to 'file' for this run."` and set
   the effective target to `file`. The effective target is now `file` or `both` (never bare
   `ticket` when degrade applies).
   - When the effective target includes `file`: create `aidd_docs/tasks/<prd_yyyy_mm>/` if it
     does not exist; write each non-skipped epic to its `epic_filepath` with the content rendered
     in the previous step. After all epics are written or skipped, write or overwrite the epic
     index file from `assets/epic-index-template.md` at
     `aidd_docs/tasks/<prd_yyyy_mm>/<prd_yyyy_mm_dd>-<prd_slug>-epic-index.md` (derived from the
     source PRD date+slug, never the run date), populating the feature→epic mapping matrix with
     one row per PRD feature (`epic_id`, `epic_filepath`, `ticket_url` or `—`). The index is
     always overwritten on each run since it is a summary artifact anchored to the PRD, not an
     idempotent content file.
   - When the effective target is `both` or `ticket` without degrade: for each non-skipped epic,
     invoke `aidd-vcs:04-issue-create` with `type: epic`, `title: [EPIC-<feature-slug>] <epic
     title>`, `body: <rendered epic content>`, `labels: ["epic"]`, and store the returned issue
     URL as `ticket_url`. In `both` mode, when a file artifact was already written and the issue
     create returns a URL, optionally append it as a comment at the bottom of the epic file
     (skip silently when no URL is returned). Before calling `aidd-vcs:04-issue-create`, check
     whether an open issue titled with the prefix `[EPIC-<feature-slug>]` already exists; if
     found, skip creation and record the existing URL, preventing duplicate issues on re-runs.
8. **Return.** Surface the `Output` block: one entry per epic with `status: created` or
   `status: skipped`; `coverage_summary` with counts, deduplicating TBD questions within each
   epic first (a question appearing in both `Scope` and `Open questions` of the same epic counts
   once) then summing across all epics; and `notes` listing degrade warnings, TBD counts, and
   unresolved preflight observations. Coverage and idempotence verification is the caller's
   responsibility → run action `02-coverage-check` on the produced epics directory to confirm
   100% coverage and zero duplicates. Quality validation (structure, TBD correctness, no
   implementation detail) is performed by the caller's reviewer using `assets/epic-validator.yml`.
   This action does NOT self-validate.

## Test

- **Preflight — file not found**: call with a non-existent `prd_path`. Expect an explicit error
  message. Confirm zero epic files are written.
- **Preflight — not validated**: call with a PRD file that contains no line with both `Status` and
  `Approved` (regardless of bold or spacing). Expect an explicit error. Confirm zero epic files
  are written.
- **Preflight — raw idea / no Core Features**: call with a plain text description or a PRD missing
  `## 4. Core Features`. Expect an explicit error. Confirm zero epic files are written.
- **100% coverage**: call with a valid N-feature PRD. Run `02-coverage-check` afterward.
  Confirm `missing[]` and `extra[]` are both empty and the matrix has exactly N rows.
- **Required fields**: for each produced epic file, confirm presence of `epic_id` frontmatter,
  `# Epic:` heading, `## Objective`, `## Scope`, `## Source PRD reference`, `## Source feature(s)`.
- **TBD for gaps**: call with a deliberately incomplete PRD (feature with no FR descriptions).
  Confirm produced epic contains `TBD:` markers and no invented values.
- **No implementation detail**: confirm no produced epic contains library/framework/file-path text
  that was not in the PRD; confirm any implementation text from the PRD is excluded.
- **Output mode — file**: confirm epic `.md` files exist under `aidd_docs/tasks/<prd_yyyy_mm>/`
  after a `file`-mode run.
- **Output mode — ticket**: confirm `aidd-vcs:04-issue-create` is invoked per epic; confirm degrade
  warning when no ticketing tool is configured and files are written instead.
- **Output mode — both**: confirm both file artifacts and ticket issues are produced.
- **Idempotence — file mode**: run `breakdown` twice on the same PRD. Confirm second run reports
  `status: skipped` for each epic. Confirm file count is unchanged. Run `02-coverage-check` and
  confirm `duplicates[]` is empty.
- **Idempotence — ticket mode**: run twice; confirm `aidd-vcs:04-issue-create` is NOT called for
  epics whose issue already exists. Confirm no duplicate issues.
- **Source traceability**: for each produced epic, confirm `source_prd` matches the input
  `prd_path` and `source_features` contains the verbatim `### Feature N - <name>` heading.
- **Ticket delegation**: confirm `aidd-vcs:04-issue-create` is used (not a reimplemented call)
  when `output_target` is `ticket` or `both`.
