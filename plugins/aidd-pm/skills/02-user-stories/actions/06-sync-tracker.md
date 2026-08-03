# 06 - Sync tracker

Gate the backlog on the Definition of Ready, get explicit approval, then save each story to the
resolved output target (file, ticket, or both).

## Input

```yaml
ranked_backlog: <the ranked backlog from 05-prioritize>   # required
output_target: file | ticket | both                        # optional; default: file
```

- `ranked_backlog`: the stories from `05-prioritize`, each carrying a priority rank.
- `output_target`:
  - `file` (default) — write the ranked stories to a markdown file under `aidd_docs/tasks/`.
  - `ticket` — create one ticket per story in the configured tracker; degrade to `file` with an
    explicit warning when no ticketing tool is configured.
  - `both` — perform both the `file` and `ticket` steps.

## Output

- `file` mode: one markdown file at `aidd_docs/tasks/<yyyy_mm>/<yyyy_mm_dd>-<feature_slug>-stories.md`
  containing every story from the ranked backlog.
- `ticket` mode: one ticket per story created in the configured tracker, each capturing the
  returned id and url.
- `both` mode: both of the above.
- In every mode, stories are returned in priority order.

## Process

1. **Resolve target.** Determine the effective output target:
   - If `output_target` is `ticket` or `both`: read the active ticketing tool from project
     memory. If none is declared, attempt to infer one (e.g. `git remote get-url origin`). If
     still none is found, log an explicit warning — `"No ticketing tool configured. Degrading
     output_target to 'file' for this run."` — and set the effective target to `file`.
   - Otherwise the effective target is `file` (the default).
   - The effective target is now one of `file` or `both` (never a bare `ticket` when degrade
     applies).
2. **Gate.** Check every story against the Definition of Ready in `@../references/rating.md`. Send
   any failing story back to its action. Do not proceed while one fails.
3. **Present.** Show the full ranked backlog. Wait for explicit user approval before any write.
4. **Save.** On approval, write per the resolved effective target:
   - **`file`** (or the `file` half of `both`): derive `feature_slug` from the confirmed scope
     statement (from `01-clarify-scope`), using the same deterministic slug normalization as
     `aidd-pm:05-epic-breakdown` (lowercase, strip diacritics to ASCII, collapse non-alphanumeric
     runs to a single hyphen, trim leading/trailing hyphens). Create
     `aidd_docs/tasks/<yyyy_mm>/` (current run date) if it does not exist, then render every story
     from the ranked backlog into `@../assets/user-story-template.md` and write them to
     `aidd_docs/tasks/<yyyy_mm>/<yyyy_mm_dd>-<feature_slug>-stories.md`. Overwrite the file when it
     already exists for this slug (the file is anchored to the confirmed scope, not append-only).
   - **`ticket`** (or the `ticket` half of `both`): create one ticket per story in the resolved
     tracker (the skill's existing tracker-creation mechanism). Capture the returned id and url for
     each story.
   - **`both`**: perform both steps above. When both artifacts are produced, no additional
     cross-linking is required.
   - **Degrade**: when `output_target` was `ticket` or `both` and no ticketing tool was found in
     step 1, only the `file` write happens; the degrade warning from step 1 is surfaced again in
     the report.
5. **Report.** Return the created/written stories with their ids and urls (when ticketed) and/or
   file path (when filed), in priority order, plus any degrade warning from step 1.

## Test

- **Definition of Ready**: the Definition of Ready holds for every saved story, in every mode.
- **Approval gate**: no write happens before explicit user approval, in every mode.
- **Output mode — file (default)**: call with `output_target: file` (or omitted). Confirm a single
  markdown file exists at `aidd_docs/tasks/<yyyy_mm>/<yyyy_mm_dd>-<feature_slug>-stories.md`
  containing every story from the ranked backlog rendered via `@../assets/user-story-template.md`.
  Confirm no ticket is created.
- **Output mode — ticket**: call with `output_target: ticket` and a configured ticketing tool.
  Confirm querying the tracker returns each saved story id with a matching title. Confirm no file
  is written.
- **Output mode — both**: call with `output_target: both` and a configured ticketing tool. Confirm
  both the stories file and the tracker tickets exist and match the ranked backlog.
- **Degrade — no tracker configured**: call with `output_target: ticket` (or `both`) when no
  ticketing tool is declared or inferable. Confirm the effective target degrades to `file`, the
  stories file is written, no ticket-creation call is attempted, and the report surfaces the
  explicit degrade warning.
