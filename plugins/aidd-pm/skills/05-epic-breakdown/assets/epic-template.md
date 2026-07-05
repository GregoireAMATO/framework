---
epic_id: EPIC-<feature-slug>
source_prd: <relative path to the source PRD, e.g. aidd_docs/tasks/2026_06/2026_06_30-greenfield-gateway-prd.md>
source_features:
  - "### Feature <N> - <Feature Name>"
  - FR<x>.<y>   # list every FR identifier that maps to this epic
status: draft
---

# Epic: <Title — what this epic delivers at the user/system level>

## Objective

<!-- Why this epic exists: the problem it solves and the value it delivers.
     What/why only — no stack, no library, no file layout. -->

TBD: <Describe the user or system problem this epic addresses, and the measurable value delivered.>

## Scope

<!-- What the epic covers: the capabilities and outcomes expected.
     One bullet per outcome, phrased as observable behaviour or deliverable.
     No implementation detail. -->

- TBD: <Outcome 1>
- TBD: <Outcome 2>

## Out of scope

<!-- Populate using this deterministic rule:
     - PRD explicitly lists exclusions → copy them here as bullets.
     - PRD scope boundary is genuinely ambiguous → emit TBD: <precise question>.
     - PRD mentions no exclusion and scope is clear → leave this section EMPTY (no TBD placeholder).
     A generic TBD placeholder must NOT appear here when no ambiguity exists. -->

## Source PRD reference

- **PRD path**: `<source_prd from frontmatter>`
- **PRD section**: `## 4. Core Features` → `### Feature <N> - <Feature Name>`
- **FR identifiers**: <list FR<x>.<y> ids from the PRD that belong to this epic>

## Source feature(s)

The following Core Feature(s) from the source PRD are covered by this epic:

| Feature heading | FR identifiers |
| --- | --- |
| `### Feature <N> - <Feature Name>` | FR<x>.1, FR<x>.2, … |

## Open questions / TBDs

<!-- Every gap discovered during breakdown is listed here as a TBD.
     Nothing is invented. Each TBD is a precise question for a human or a downstream agent. -->

- TBD: <Question 1 — what specific decision or information is missing?>
- TBD: <Question 2>

## User stories (children)

User stories for this epic are NOT generated here. They are produced downstream by the
`aidd-pm:02-user-stories` skill, which receives this epic file as its input.

When generated in file mode, the child user stories are written under `aidd_docs/tasks/` by the
`aidd-pm:02-user-stories` skill, which determines the exact filename.
