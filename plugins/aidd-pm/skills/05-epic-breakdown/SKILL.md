---
name: 05-epic-breakdown
description: Break a validated PRD into N distinct epics covering 100% of its Core Features. Use when the user says "break down prd into epics", "create epics from prd", or "epic breakdown". Do NOT use for writing user stories, running the SDLC, or generating source code.
argument-hint: breakdown | coverage-check
---

# Epic Breakdown

Turns a validated PRD into a set of traceable epics, each scoped to one or more Core Features, ready for downstream user-story generation and SDLC planning.

## Actions

| #   | Action           | Role                                                                                   | Input                                               |
| --- | ---------------- | --------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| 01  | `breakdown`      | Validate PRD, parse Core Features, derive N epics, write to chosen output target        | prd_path (required), output_target (default: file)   |
| 02  | `coverage-check` | Read-only: compare PRD features against produced epics, emit coverage matrix + verdict  | prd_path (required), epics_location (required)        |

Dispatch by input: `prd_path` alone → `breakdown`; `prd_path` with `epics_location` for audit purposes → `coverage-check`.

## Transversal rules

- **Input gate**: refuse any PRD that is absent, lacks a line containing both `Status` and `Approved` (tolerant of bold markers and whitespace, e.g. `Status: Approved` or `**Status** : Approved`), or lacks `## 4. Core Features`. Return an explicit error message. Produce nothing.
- **What/why only**: epics describe the problem and the value to deliver. Never include a stack choice, library name, file path, or implementation pattern.
- **Zero invention**: every gap or missing field is emitted as `TBD: <precise question>`. Never guess.
- **100% coverage**: every `### Feature N` subsection under `## 4. Core Features` maps to exactly one epic. No orphan features. No epics without a source feature.
- **Idempotence**: epic identity derives from the source PRD's date+slug and the feature slug, never from the run date. A second run on the same PRD skips already-produced epics rather than duplicating them.
- **Ticket delegation**: when `output_target` is `ticket` or `both`, delegate each epic creation to `aidd-vcs:04-issue-create`. Degrade to `file` mode with a warning when no ticketing tool is configured.
- **Source traceability**: every epic artifact explicitly references the source PRD path and the `### Feature N` heading(s) it derives from.
- **No self-validation**: action `01-breakdown` does NOT self-validate after writing. The caller is responsible for two distinct verification steps. First, run action `02-coverage-check` to verify 100% PRD coverage and idempotence (the in-skill structural verifier). Second, spawn a reviewer with `assets/epic-validator.yml` to validate quality (structure, TBD correctness, no implementation detail). Reviewer findings return through `breakdown` for correction or explicit TBD.
- **Language**: all skill artifacts are authored in English regardless of the source PRD's language.

## References

- `references/slug-normalization.md`: the deterministic feature-slug algorithm shared by `01-breakdown` and `02-coverage-check`.

## Assets

- `assets/epic-template.md`: canonical epic artifact body.
- `assets/epic-validator.yml`: reviewer checklist for validating produced epics.
- `assets/epic-index-template.md`: optional feature→epic mapping index written by `breakdown` in file/both mode; filename derived from the source PRD date+slug (deterministic, never run-date).
