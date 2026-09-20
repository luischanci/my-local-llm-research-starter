# Verification before reporting a task complete

Read at the end of every task, before writing the report. A task is not complete until
these checks pass or their failure is reported verbatim.

## R script

```powershell
Rscript Data_and_Estimation/Code/NN_name.R
```
- The log ends with `PASS`.
- Every expected output file exists and was modified by this run:
  `Get-ChildItem Data_and_Estimation/Outcomes/Tables | Sort-Object LastWriteTime -Descending | Select-Object -First 5`
- Estimates contain no `NA`, `NaN`, or `Inf`; sample sizes match the plan.

## Julia script

```powershell
julia --project=. Data_and_Estimation/Code/NN_name.jl
```
- Same checks as R. Every optimisation reports convergence.

## Stata do-file

```powershell
StataMP-64.exe /e do Data_and_Estimation/Code/NN_name.do
Select-String -Path NN_name.log -Pattern "^r\(\d+\);"
```
- Stata exits silently. A line like `r(111);` in the log is an error; no match means no error.

## LaTeX manuscript

```powershell
cd Edition_[PAPER]; latexmk -pdf [paper].tex
```
If `latexmk` fails because Perl is missing (common with MiKTeX), run instead:
`pdflatex [paper]; bibtex [paper]; pdflatex [paper]; pdflatex [paper]`.

Then check the log:
```powershell
Select-String -Path [paper].log -Pattern "undefined|Overfull|Rerun"
Select-String -Path [paper].blg -Pattern "Warning|error"
```
- The PDF exists and was written by this run.
- No undefined citations or references; no "Rerun" message left.
- Report every Overfull hbox wider than 10pt.

## Failure handling

- At most two attempts to fix a failing run. After that, stop and report the error verbatim
  with what was tried. Do not loop.
- Never weaken a check, specification, or sample to make a run pass.
