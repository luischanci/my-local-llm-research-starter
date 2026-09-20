# Replication check before modifying existing estimation code

Read before editing any script whose output a report or the manuscript already cites.
Principle: prove the pipeline reproduces the known results **before** changing it.

## 1. Baseline

Before touching the code, copy the current outputs that will be affected into
`quality_reports/baseline_<STEP>.md`, citing the source file for each number:

| Quantity | Source file | Baseline value |
|---|---|---|

## 2. Re-run unchanged

Run the unmodified script. Compare every baseline quantity.

| Quantity | Tolerance |
|---|---|
| Point estimates | < 1e-6 |
| Standard errors (analytic) | < 1e-6 |
| Standard errors (bootstrap/simulation, same seed) | < 1e-4 |
| Sample sizes, cluster counts | exact |
| Significance at the reported level | exact |

If anything is outside tolerance, **stop**. Do not modify the code. Report which quantity
differs and by how much, and isolate the step where the difference first appears.

## 3. Modify and compare

Only after step 2 passes: make the change, re-run, and report old versus new for every
quantity, stating which differences are intended by the change and which are not.

## 4. Report

In the report's verification ledger:

| Quantity | Baseline | Unchanged re-run | After change | Status |
|---|---|---|---|---|

Record the software versions (R, Julia, Stata, key packages) used for the run.
