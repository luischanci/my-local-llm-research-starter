# R code review checklist

Use when the PI asks for a review of an R script. **Do not edit the script.** Produce a
report only.

## Procedure

1. Review one script at a time. If asked to review "all", list the scripts and ask which first.
2. Read the script. Read `refs/r-econometrics.md` only if the script estimates models.
3. Run it with `Rscript` and read the log. A review of code that was not run must say so.
4. Write `quality_reports/review_<script_name>.md` with the sections below.
5. In chat, give only: number of issues by severity and the top three.

## Checks, in order of priority

**Critical: correctness**
- Estimand matches the plan or protocol: outcome, sample, FE, weights, clustering.
- Every function argument exists in the installed package version.
- Merges: keys asserted, row counts recorded, unmatched rows reported.
- No silent dropping of observations (NA handling, `fixest` singleton removal: check `nobs`).
- Standard errors clustered at the level the protocol specifies.

**High: reproducibility**
- Seed set before any random step; relative paths only; no `setwd()`.
- Reads only from `Processed_Data/` or `Raw_Data/`; never writes to `Raw_Data/`.
- Every table and figure is written by the script to `Outcomes/`.
- Ends with a `PASS` line; stops with an informative error otherwise.

**Medium: clarity**
- Header: purpose, inputs, outputs, date.
- Named constants for thresholds; no magic numbers.
- Comments explain why, not what.

**Low: style**
- Consistent naming; no dead code; no `print()` spam in the log.

## Report format

| # | Severity | Line | Issue | Suggested fix |
|---|---|---|---|---|

Close with: "Ran successfully: yes/no", and anything that could not be checked.
