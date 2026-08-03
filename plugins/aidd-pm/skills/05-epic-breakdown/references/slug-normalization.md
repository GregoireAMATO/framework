# Feature slug normalization

Deterministic algorithm that derives a `feature_slug` from a Core Feature `<name>`. Used by `01-breakdown` (epic identity and filenames) and `02-coverage-check` (feature-set comparison). Both actions MUST derive identical slugs, so this is the single source of truth.

1. Lowercase the entire name.
2. Strip diacritics/accents to their ASCII equivalents (é→e, ê→e, è→e, à→a, â→a, ç→c, î→i, ô→o, û→u, ü→u, ï→i, ë→e, and equivalents for all other combining diacritics).
3. Replace every contiguous run of non-alphanumeric characters (spaces, apostrophes, hyphens, colons, punctuation) with a single hyphen.
4. Trim any leading or trailing hyphens.

Examples:

- `Centre d'aide en libre-service` → `centre-d-aide-en-libre-service`
- `Enquêtes de satisfaction` → `enquetes-de-satisfaction`
- `Orchestrateur` → `orchestrateur`
