# [PROJECT NAME]

[One paragraph: what the paper is about, the countries or units, the data, the period.]

## Research question

[The question in one or two sentences.]

[The competing hypotheses, stated two-sided. Say explicitly that the direction is not assumed.]

## Where the project stands — [DD Month YYYY]

Update this section at every phase close. One short paragraph per phase, newest last.
State the result, its interval, and the decision it supports; point to the report.

**Phase A — [name]: [not started / in progress / complete].** [Status and key facts.
See `quality_reports/report_A1.md`.]

**Phase B — [name]: [status].** [...]

[What is out of scope and why, in one line each.]

## Governing documents

| Document | Role |
|---|---|
| `[Project_Design].md` | Question, hypotheses with decision rules, data, design. **Design authority.** |
| `[RA_Protocol].md` | Step-by-step protocol, each step with its report and stop. **Operating authority.** |
| `AGENTS.md` | Standing instructions for the research assistant (human or AI). |
| `MEMORY.md` | Durable corrections. |
| `quality_reports/decision_log.md` | Every data and design decision, dated, one line each. |
| `Edition_[PAPER]/[paper].tex` | Working paper draft. |

Archival documents: [list, or "none"]. Where they differ from the governing documents, the
governing documents apply.

## Repository structure

```
[PROJECT]/
├── AGENTS.md  MEMORY.md  README.md
├── [Project_Design].md  [RA_Protocol].md
├── Data_and_Estimation/
│   ├── Code/                  # NN_verb_object.{R,jl,do}
│   ├── Data/
│   │   ├── Raw_Data/          # never written; not in git
│   │   └── Processed_Data/
│   └── Outcomes/
│       ├── Logs/  Tables/  Figures/  Validation/
├── quality_reports/           # decision_log.md, plans/, report_<STEP>.md
├── Edition_[PAPER]/
├── refs/  master_supporting_docs/  tmp/
```

## Pipeline

Run in this order from the repository root. Each script ends with a `PASS`/`FAIL` line in its log.

| Script | Input | Output | Purpose |
|---|---|---|---|
| `Code/01_[...]` | `Raw_Data/[...]` | `Processed_Data/[...]` | [...] |

## Data rules

- Software: [R x.y / Julia x.y / Stata xx]. [Which language is primary and for what.]
- Outcome variables: [exact source variables and why].
- Key definitions: [primary definition; alternatives].
- Sample restrictions: [trimming, thresholds, minimum cell size].
- Raw data are never overwritten. Every step writes new files.

## Coding hazards already encountered

Each of these cost time. Listed here so nobody repeats them.

- [hazard -> what to do instead]

## Working method

The PI decides; the RA computes. One protocol step at a time, each ending in a report in
`quality_reports/` and a stop for review. Sample restrictions, variable definitions and model
choices belong to the PI and are logged. Results near a decision boundary are reported as
near the boundary, not relabelled.
