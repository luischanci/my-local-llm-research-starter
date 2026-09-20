# Project Memory

Durable corrections: things the RA got wrong once and must not get wrong again.
This is not a results log. Findings belong in `quality_reports/report_<STEP>.md` and `README.md`.

## Format

One line per entry, appended at the bottom. Never rewrite or reorder.

```
[LEARN:category] what was wrong -> what is right. (YYYY-MM-DD)
```

Use one of these categories so entries can be searched with
`Select-String -Path MEMORY.md -Pattern "LEARN:<category>"`:

| Category | Covers |
|---|---|
| `data` | variable names, codes, units, merges, keys, source quirks |
| `measurement` | definitions, deflators, samples, thresholds |
| `estimation` | specifications, estimators, fixed effects, weights |
| `inference` | standard errors, clustering, bootstraps, multiple testing |
| `claims` | interpretation, wording, what a result does and does not show |
| `code` | language or package pitfalls (R, Julia, Stata, LaTeX) |
| `workflow` | process, authority, reporting, gates |

Add a new category only if none of these fits; record it in this table.

## Maintenance

At each phase close, the PI reviews this file:
- a correction that has become a standing rule moves into `AGENTS.md` and is deleted here;
- a data or coding hazard moves into the `README.md` hazards list and is deleted here;
- keep this file under about 40 entries.

---

<!-- Append new entries below. Most recent at bottom. -->
