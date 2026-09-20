# Julia code review checklist

Use when the PI asks for a review of a Julia script. **Do not edit the script.** Produce a
report only.

## Procedure

1. Review one script at a time. If asked to review "all", list the scripts and ask which first.
2. Read the script. Read `refs/julia-econometrics.md` only if the script estimates models.
3. Run it with `julia --project=.` and read the output. A review of code that was not run
   must say so.
4. Write `quality_reports/review_<script_name>.md` with the sections below.
5. In chat, give only: number of issues by severity and the top three.

## Checks, in order of priority

**Critical: correctness**
- Objective, moments, or likelihood match the estimator stated in the plan or paper.
- Optimiser convergence is checked (`Optim.converged`), not assumed.
- Several starting values tried for nonconvex problems; local optima reported.
- Standard errors: correct formula (inverse Hessian of the *negative* log-likelihood;
  GMM sandwich; uncentered `S`), clustering where the data are clustered.
- Parallel code does not write to shared variables unsafely; results do not depend on
  thread count.

**High: reproducibility**
- Seeds set explicitly (per-draw RNGs in threaded loops); simulation draws held fixed across θ.
- `joinpath` and relative paths; never writes to `Raw_Data/`.
- Uses only packages in `Project.toml`.
- Outputs written to `Outcomes/`; ends with a `PASS` line.

**Medium: performance**
- Type stability in hot functions (`@code_warntype`); no non-`const` globals.
- Pre-allocation and in-place updates in loops; column-major iteration.

**Low: style**
- `snake_case` functions, `CamelCase` types; notation matches the paper.

## Report format

| # | Severity | Line | Issue | Suggested fix |
|---|---|---|---|---|

Close with: "Ran successfully: yes/no", and anything that could not be checked.
