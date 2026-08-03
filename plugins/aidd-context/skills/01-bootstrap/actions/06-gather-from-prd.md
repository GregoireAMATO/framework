# 06 - Gather from PRD

Derive the bootstrap architecture checklist from a validated PRD instead of interactive Q&A.
Produces the same artifact as `01-gather-needs` — a filled copy of `@../assets/checklist.md` held
in context — so the existing chain `02 → 03 → 04 → 05` runs unchanged. No disk writes, no Q&A,
no scaffolding.

**Mutually exclusive with `01-gather-needs`.** Use this action when a validated PRD already exists;
use `01-gather-needs` when starting from a free-form idea.

## Inputs

```yaml
prd_path: <path to a validated PRD file>   # required; relative or absolute; no other input
```

No free-form user request, no Q&A prompts, no additional parameters.

## Outputs

A filled copy of `@../assets/checklist.md` held in conversation context (NOT written to disk),
structurally identical to the artifact produced by `01-gather-needs`:

- **Block 1** (7 items) — Project (the what)
- **Block 2** (7 items) — Technical constraints
- **Block 3** (4 items) — Team preferences & constraints

Every item in blocks 1-3 is either a concrete value (replacing its `<...>` placeholder) or
`TBD: <precise question>`. No raw `<...>` placeholder remains in blocks 1-3.

- **Block 4** (6 items) — Derived choices: left entirely untouched, with all 6 raw `<...>`
  placeholders intact. Block 4 is the role of actions 02 and 04, not this action.

The action also returns a one-line fill/TBD summary (e.g. "14 filled, 4 TBD") and then hands off
to `02-propose-candidates` exactly as `01-gather-needs` does.

```markdown
## 📌 Block 1 - Project (the what)

- [x] **Project name** - Greenfield Gateway
- [x] **One-liner** - Developer platform for routing AI-generated code changes through automated quality gates
- [x] **Type** - TBD: Is the product aimed at B2B teams, individual developers (B2C), or internal tooling? The PRD personas suggest developer teams but does not state a pricing/positioning tier.
- [x] **Target users** - Platform engineers and full-stack developers using AI coding tools; TBD: no 6-month active-user count given in §10 or §7.
- [x] **Top 3-5 features** - Automated gateway routing, Quality gate pipeline, PRD-to-stack mapping, Epic breakdown orchestration
- [x] **External integrations** - GitHub (via aidd-vcs); TBD: §11 lists no additional external services explicitly.
- [x] **Target platform** - TBD: §9 is marked N/A in this PRD; no platform target stated.
... (all 18 items filled or TBD)
```

## Process

### Step 0 — Preflight validation (input gate)

Reuses the validated-PRD marker rule from `aidd-pm:05-epic-breakdown` action `01-breakdown`
Step 0 verbatim:

1. **File existence.** Verify `prd_path` points to a file that exists on disk. If not: return
   error `"PRD not found at <prd_path>. Provide the path to a validated PRD."` and halt.
   Produce nothing.

2. **Validation marker.** Read the file. Confirm it contains a line where both the word `Status`
   and the word `Approved` appear — tolerant of bold markers (`**`), extra whitespace, and colon
   placement (e.g. matches `Status: Approved`, `**Status** : Approved`, `**Status**: Approved`).
   If no such line is found: return error `"PRD at <prd_path> is not validated (no
   'Status … Approved' line found). Validate the PRD before running bootstrap from-PRD."` and halt.
   Produce nothing.

3. **PRD structure.** Confirm the file contains a `## 4. Core Features` section (a recognizable
   PRD structure). If not: return error `"PRD at <prd_path> lacks a '## 4. Core Features' section.
   A structured PRD is required — a raw idea or outline is not sufficient."` and halt. Produce nothing.

No checklist is emitted while any preflight check fails.

### Step 1 — Load the canonical checklist

Read `@../assets/checklist.md` (read-only; never modify the asset). Hold it in context as the
structural template whose item labels and block order are authoritative. The output checklist must
match these labels verbatim so that `02-propose-candidates` can consume it without adjustment.

### Step 2 — Map PRD sections to the 18 checklist items

For each item in blocks 1-3, locate the corresponding PRD content by **section heading text**
(never by hard-coded section numbers — PRD variants may renumber). For each item:

- If a deterministic value is derivable from the cited PRD section(s): fill it with that value.
- If the value is not derivable or the section is absent: emit `TBD: <precise question>`.
- **Boolean items** (`Real-time?`, `Multi-tenant?`, `SEO important?`, `Offline mode?`): require an
  explicit positive or explicit negative signal in the PRD. Silence ⇒ `TBD`. Never default to "no"
  from absence.

**Mapping table (heading text → checklist item):**

| Item | Checklist label | PRD section(s) to consult | Fill rule |
|------|----------------|--------------------------|-----------|
| 1 | Project name | H1 title / filename slug / §1 Executive Summary | Always derivable from PRD title |
| 2 | One-liner | §1 Executive Summary → Solution sentence | Condense to one sentence if multi-sentence; never invent a new claim |
| 3 | Type (B2B/B2C/internal/marketplace/other) | §2 User Personas + §1 Problem/Solution | Fill only if persona or positioning makes it explicit; else TBD |
| 4 | Target users (profile + volume @6 months) | Profile ← §2 Personas; volume ← §10 Success Metrics / §7 NFR | Profile usually derivable; "@6 months volume" ⇒ TBD if no figure stated |
| 5 | Top 3-5 features | §4 Core Features (`### Feature N` headings) | Direct copy of feature names |
| 6 | External integrations | §11 Dependencies (External) + §8 Integration Points | If PRD lists none ⇒ "none"; if unclear ⇒ TBD |
| 7 | Target platform | §9 User Experience (IA / Design System) | TBD by default — greenfield PRDs rarely fix platform; §9 = N/A ⇒ TBD |
| 8 | Real-time? | §4 feature descriptions + §7 NFR | Boolean — explicit signal only (chat / live updates / websockets); silence ⇒ TBD |
| 9 | Multi-tenant? | §1 / §4 + Data Model sub-section of §8 | Boolean — explicit signal (per-customer workspaces); silence ⇒ TBD |
| 10 | Data sensitivity | §7 Security (NFR) + §1 | Fill from Security NFR if present; else TBD |
| 11 | Volume at 6 months | §10 Success Metrics (KPIs) + §7 Performance NFR | Fill from stated figure; qualitative-only language ⇒ TBD for a concrete number |
| 12 | SEO important? | §9 UX + §4 (marketing / content signal) | Boolean — explicit signal; silence ⇒ TBD |
| 13 | Performance target (p95) | §7 Performance NFR | Capture stated target; if no numeric p95 ⇒ TBD: precise p95 target |
| 14 | Offline mode? | §7 / §9 / §4 (PWA / local sync signal) | Boolean — explicit signal; silence ⇒ TBD |
| 15 | Languages mastered by team | §2 Persona → Technical Environment (if stated) | Team capability, not a product fact ⇒ usually TBD |
| 16 | Hosting budget | §10 / §15 (cost constraint, if stated) | Usually absent from a greenfield PRD ⇒ TBD |
| 17 | Hosting preference | §8 Technical Architecture + §11 | See no-stack guard below — always TBD |
| 18 | Deal-breakers (excluded tech) | §6 Non-Goals + §15 Tier-3 "Never" | Copy only explicit technology exclusions; scope-only non-goals ⇒ not a deal-breaker ⇒ TBD |

### Step 3 — No-stack guard

**This action never reads PRD §8 (Technical Architecture) for a stack value and never fills
block 4.**

- Item 17 (Hosting preference): §8 is TBD by design in a greenfield PRD. Resolve to `TBD: What
  hosting platform does the team prefer (Vercel, AWS, self-hosted, or no opinion)?`. Never extract
  a stack product from §8.
- Block 4 (Architecture pattern, Front-end, Back-end, Database, Auth provider, Final hosting):
  left entirely as raw `<...>` placeholders. These are filled by actions 02 and 04, not here.
- If the PRD happens to mention a technology in §8 or elsewhere, do not copy it into any
  block-1-3 item or block-4 item. Stack decisions are the role of `02-propose-candidates` and
  `04-pick-and-design`.

This satisfies the hard constraint: "aucun choix de stack n'est extrait du PRD."

### Step 4 — Self-check structure

Before returning, verify:

1. Blocks 1-3 contain no remaining raw `<...>` placeholder — every item line holds a concrete
   value or `TBD: <precise question>`.
2. Block 4 is untouched — all 6 items still hold their raw `<...>` placeholders.
3. Item labels match `@../assets/checklist.md` verbatim (same wording, same order) — required for
   `02-propose-candidates` to consume the checklist without adjustment.

If any check fails, correct it before returning.

### Step 5 — Return

Return the filled checklist plus a one-line summary:

```
Checklist filled: <N> items from PRD, <M> items marked TBD. Ready for 02-propose-candidates.
```

Hand off to `02-propose-candidates` exactly as `01-gather-needs` does. No `INSTALL.md`, no Q&A,
no disk write.

## Test

- **Preflight — file not found**: call with a non-existent `prd_path`. Expect the explicit error
  `"PRD not found at <prd_path>…"`. Confirm zero checklist output.

- **Preflight — not validated**: call with a PRD file that contains no line with both `Status` and
  `Approved` (regardless of bold or spacing). Expect the explicit error `"…is not validated (no
  'Status … Approved' line found)…"`. Confirm zero checklist output.

- **Preflight — raw idea / no Core Features**: call with a plain-text description or a PRD missing
  `## 4. Core Features`. Expect the explicit error `"…lacks a '## 4. Core Features' section…"`.
  Confirm zero checklist output.

- **Happy path**: call with `aidd_docs/tasks/2026_06/2026_06_30-greenfield-gateway-prd.md`
  (Status: Approved). Expect a filled checklist where blocks 1-3 each hold a concrete value or
  `TBD: <precise question>` and no raw `<...>` placeholder remains in blocks 1-3.

- **Same artifact as `01-gather-needs`**: confirm item labels and block order in the produced
  checklist are byte-identical to `@../assets/checklist.md`; confirm block 4 still holds its 6
  raw `<...>` placeholders untouched.

- **Zero invention**: call with a deliberately lacunary PRD (missing several sections). Confirm
  uncovered items appear as `TBD: <precise question>` and no value is fabricated.

- **No stack leaked**: assert that block 4 is untouched and that no block-1-3 item contains a
  framework, library, database engine, hosting product, or auth-provider name sourced from PRD §8.
  Confirm §8 is not consulted for any stack value.
