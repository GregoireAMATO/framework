# 01 - Breakdown

Validate a PRD, parse its Core Features, derive one epic per feature, and write the results to
the chosen output target (file, ticket, or both). Enforces 100% coverage, zero invention, and
idempotence across runs.

## Inputs

```yaml
prd_path: <path to the validated PRD file>   # required
output_target: file | ticket | both           # optional; default: file
```

- `prd_path`: relative or absolute path to an existing PRD that has passed validation.
- `output_target`:
  - `file` (default) — write epic markdown artifacts to `aidd_docs/tasks/<prd_yyyy_mm>/`.
  - `ticket` — create one tracker issue per epic via `aidd-vcs:04-issue-create`; degrade to
    `file` with a warning when no ticketing tool is configured.
  - `both` — perform both `file` and `ticket` steps.

## Outputs

```yaml
epics:
  - epic_id: EPIC-<feature-slug>
    title: <epic title>
    source_feature: "### Feature N - <name>"
    path: aidd_docs/tasks/<prd_yyyy_mm>/<prd_yyyy_mm_dd>-<prd_slug>-epic-<NN>-<feature_slug>.md
    ticket_url: <issue URL or null>           # populated only when output_target is ticket or both
    status: created | skipped                 # skipped when idempotence check finds an existing epic
coverage_summary:
  features_in_prd: <N>
  epics_produced: <N>
  epics_skipped: <count of idempotent skips>
  tbd_count: <count of distinct TBD questions per epic (deduplicated within each epic), summed across all epics — a TBD question that appears in both Scope and Open questions of the same epic is counted once>
notes: <warnings, degrade notices, unresolved questions>
```

## Process

### Step 0 — Preflight validation (input gate, HC1)

1. Verify `prd_path` points to a file that exists on disk. If not: return error
   `"PRD not found at <prd_path>. Provide the path to a validated PRD."` and halt. Produce nothing.
2. Read the file. Confirm it contains a line where both the word `Status` and the word `Approved`
   appear — tolerant of bold markers (`**`), extra whitespace, and colon placement (e.g. matches
   `Status: Approved`, `**Status** : Approved`, `**Status**: Approved`). If no such line is found,
   and no equivalent validation marker is configured in project memory: return error
   `"PRD at <prd_path> is not validated (no 'Status … Approved' line found). Validate the PRD
   before running epic-breakdown."` and halt. Produce nothing.
3. Confirm the file contains a `## 4. Core Features` section. If not: return error
   `"PRD at <prd_path> lacks a '## 4. Core Features' section. A structured PRD with at least one
   Feature subsection is required."` and halt. Produce nothing.
4. Confirm at least one `### Feature N - <name>` subsection exists under `## 4. Core Features`.
   If not: return error `"No Feature subsections found under '## 4. Core Features'. Add at least one
   '### Feature N - <name>' entry to the PRD."` and halt. Produce nothing.

No files are written while any preflight check fails.

### Step 1 — Parse PRD metadata

Extract from the PRD filename:
- `prd_date` (e.g. `2026_06_30` from `2026_06_30-greenfield-gateway-prd.md`)
- `prd_yyyy_mm` (e.g. `2026_06`)
- `prd_slug` (e.g. `greenfield-gateway` — the kebab-case name between the date and `-prd.md`)

These values anchor all derived filenames. They derive from the **source PRD**, never from the
current run date (R7 — enables idempotence across days).

### Step 2 — Extract Core Feature list (HC2)

Parse ONLY lines that match the exact heading pattern `### Feature N - <name>` (a level-3
markdown heading starting with `###`, followed by the literal word `Feature`, a positive integer,
a hyphen, and the feature name). Ignore all other content under `## 4. Core Features` — intro
prose, bullet lists, tables, and any sub-subsections that do not match this pattern.

Produce an ordered list `[F1, F2, …, FN]`. Each entry records:
- Heading text (verbatim): `### Feature N - <name>`
- Feature order index `N` (1-based, as written in the PRD)
- Feature slug: derived from the `<name>` portion using this deterministic normalization:
  1. Lowercase the entire name.
  2. Strip diacritics/accents to their ASCII equivalents (é→e, ê→e, è→e, à→a, â→a, ç→c,
     î→i, ô→o, û→u, ü→u, ï→i, ë→e, and equivalents for all other combining diacritics).
  3. Replace every contiguous run of non-alphanumeric characters (spaces, apostrophes, hyphens,
     colons, punctuation, etc.) with a single hyphen (`-`).
  4. Trim any leading or trailing hyphens from the result.
  Examples: `Centre d'aide en libre-service` → `centre-d-aide-en-libre-service`;
            `Enquêtes de satisfaction` → `enquetes-de-satisfaction`;
            `Orchestrateur` → `orchestrateur`.
- FR identifiers: all `FR<N>.<y>` references found in the feature's subsection body

This list is the authoritative scope for coverage. Every item must map to exactly one epic.

### Step 3 — Partition features into epics (HC2, HC5, HC8)

For each feature `Fi` in order:

1. Assign one epic (`Ei`). Each feature maps to exactly one epic. One epic maps to exactly one
   feature (1:1 by default; M:1 grouping is permissible only when the PRD explicitly groups
   features under a single heading, which is treated as a single `Fi`).
2. Derive the epic identity:
   - `epic_id`: `EPIC-<feature-slug>` (deterministic from the source feature, stable across runs)
   - `epic_filename`: `<prd_yyyy_mm_dd>-<prd_slug>-epic-<NN>-<feature_slug>.md`
     (zero-padded `NN` = feature index, e.g. `01`, `02`)
   - `epic_filepath`: `aidd_docs/tasks/<prd_yyyy_mm>/<epic_filename>`
3. Extract from the feature's PRD section whatever information is available:
   - Feature name, FR descriptions, acceptance criteria, and any notes
   - Fill every template field that can be derived directly from the PRD
4. Mark every field that cannot be determined from the PRD as `TBD: <precise question>` (HC3).
   Never invent a value. Typical TBD triggers: missing objective wording, ambiguous scope
   boundary, unresolved acceptance criterion.
5. Ensure no implementation detail enters the epic (HC8): no library, framework, design pattern,
   file-layout reference, or technology choice. If the PRD itself contains such details, omit them
   from the epic and emit `TBD: The PRD mentions [X] as an implementation detail; confirm whether
   the epic's scope should be defined independently of this choice.`

### Step 4 — Render each epic from the template (M5, M6, M10)

For each `(Fi, Ei)` pair, populate `assets/epic-template.md` with the derived values:

- Frontmatter: `epic_id`, `source_prd` (prd_filepath), `source_features` (list of Feature headings
  and FR ids), `status: draft`
- `# Epic: <title>`
- `## Objective` — what/why prose from the PRD feature description; TBD if absent
- `## Scope` — bulleted outcomes from FR items; TBD placeholders where FR items are absent
- `## Out of scope` — apply this deterministic rule:
  - If the PRD explicitly lists exclusions for this feature: copy them here.
  - If the PRD's scope boundary for this feature is genuinely ambiguous (present but unclear):
    emit `TBD: <precise question about the ambiguous boundary>`.
  - If the PRD mentions no exclusion and the scope boundary is not ambiguous: leave the section
    empty — do NOT emit a generic TBD placeholder.
- `## Source PRD reference` — exact PRD path + section heading + FR identifiers
- `## Source feature(s)` — the mapping table (Feature heading → FR ids)
- `## Open questions / TBDs` — consolidated list of all TBD markers for this epic
- `## User stories (children)` — boilerplate note directing to `aidd-pm:02-user-stories`

### Step 5 — Idempotence check (HC7, DD4)

Before writing any artifact:

1. Scan `aidd_docs/tasks/<prd_yyyy_mm>/` for files matching the derived `epic_filepath` pattern.
2. For each candidate file found: read its `epic_id` frontmatter field.
3. If `epic_id` matches the derived `EPIC-<feature-slug>`: mark the epic as `status: skipped`
   and do NOT overwrite the file. Record the skip in `coverage_summary.epics_skipped`.
4. If no match: proceed to Step 6 to write the file.

The skip logic applies per-epic independently: a run may create some new epics and skip others.

### Step 6 — Resolve output target and write (HC4, DD5)

**Determine effective output target:**
1. If `output_target` is `ticket` or `both`:
   - Check whether a ticketing tool is configured (consult project memory for `tracker_tool`
     or `git_remote`; if absent, attempt `git remote get-url origin` to infer the platform).
   - If no tool is detected: log warning `"No ticketing tool configured. Degrading output_target
     to 'file' for this run."` and set effective target to `file`.
2. Effective target is now one of `file` or `both` (never bare `ticket` when degrade applies).

**Write file artifacts** (when effective target includes `file`):
- Create `aidd_docs/tasks/<prd_yyyy_mm>/` if it does not exist.
- Write each non-skipped epic to its `epic_filepath`.
- File content = rendered template from Step 4.
- After all epics are written (or skipped), write or overwrite the epic index file using
  `assets/epic-index-template.md`:
  - Index filename: `<prd_yyyy_mm_dd>-<prd_slug>-epic-index.md` (derived from the source PRD
    date+slug — deterministic across runs, never from the run date).
  - Index path: `aidd_docs/tasks/<prd_yyyy_mm>/<index_filename>`.
  - Populate the feature→epic mapping matrix with one row per PRD feature, recording `epic_id`,
    `epic_filepath`, and `ticket_url` (or `—` when not applicable).
  - The index is always overwritten on each run (it is a summary artifact, not an idempotent
    content file; its identity is anchored to the PRD, not to the run).

**Create tracker issues** (when effective target is `both` or `ticket` without degrade):
- For each non-skipped epic: invoke `aidd-vcs:04-issue-create` with:
  - `type: epic`
  - `title: [EPIC-<feature-slug>] <epic title>`
  - `body: <epic content from Step 4 rendered as the issue body>`
  - `labels: ["epic"]`
- Store the returned issue URL in the epic's output record (`ticket_url`).
- In `both` mode: if the file artifact was already written, optionally append the issue URL as a
  comment at the bottom of the epic file (C2 cross-link; emit if `aidd-vcs:04-issue-create`
  returns a URL, skip silently otherwise).
- Idempotence in ticket mode: before calling `aidd-vcs:04-issue-create`, check whether an open
  issue with title prefix `[EPIC-<feature-slug>]` already exists. If found: skip creation and
  record the existing URL. This prevents duplicate issues on re-runs.

### Step 7 — Return structured output

Return the `Outputs` block with:
- One entry per epic, including `status: created` or `status: skipped`
- `coverage_summary` with counts. For `tbd_count`: deduplicate TBD questions within each epic
  first (the same question text appearing in multiple sections counts once per epic), then sum
  across all epics. This matches the consolidated TBD list in the epic index.
- `notes` listing any degrade warnings, TBD counts, and unresolved preflight observations
- **Coverage and idempotence verification is the caller's responsibility**: run action `02-coverage-check`
  on the produced epics directory to confirm 100% coverage and zero duplicates (Done-when #1 and #5).
  Quality validation (structure, TBD correctness, no implementation detail) is performed by the caller's
  reviewer using `@assets/epic-validator.yml`. Action `01-breakdown` does NOT self-validate.

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
