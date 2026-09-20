# R econometrics patterns (fixest and friends)

Read this before writing R estimation code. Every call below is a verified signature;
do not add arguments that are not shown here without checking `?function` in the installed version.
Scripts run from the repository root; all paths are relative to it.

## Setup

```r
library(data.table)
library(fixest)
set.seed(12345)                      # state the seed in the report

dt <- fread("Data_and_Estimation/Data/Processed_Data/panel.csv")
out_tab <- "Data_and_Estimation/Outcomes/Tables"
out_fig <- "Data_and_Estimation/Outcomes/Figures"
```

Do not call `install.packages()`. If a package is missing, stop and report it.

## data.table essentials

```r
dt[, y_pct := y * 100]                               # new variable
dt[, mean_y := mean(y, na.rm = TRUE), by = group]    # group operation
dt[, .(mean_y = mean(y), n = .N), by = group]        # summarise
merged <- other[dt, on = .(id, year)]                # join: keeps all rows of dt
setorder(dt, id, year)
dt[, lag_y := shift(y, 1), by = id]                  # lag within id

# After every restriction or merge, record the row count
cat("rows after merge:", nrow(merged), "\n")
stopifnot(!anyDuplicated(merged, by = c("id", "year")))
```

## Fixed effects

```r
m1 <- feols(y ~ x + controls | id + year, data = dt, cluster = ~id)
m2 <- feols(y ~ x | id + year, data = dt, cluster = ~id + year)   # two-way clustering
m3 <- feols(y ~ x | id^year, data = dt, cluster = ~id)            # interacted FE
summary(m1, cluster = ~state)                                     # re-cluster, no re-estimation
```

- Fixed effects go after `|`. Clustering is `cluster = ~var`. There is **no** `clust.var`.
- Pass formula objects. Do not paste strings into `as.formula()`.
- Multiple estimation: `feols(c(y1, y2) ~ x + csw0(c1, c2) | id + year, data = dt)`.
- Large data: add `lean = TRUE`.

## Instrumental variables

```r
iv <- feols(y ~ controls | id + year | endog ~ z1 + z2, data = dt, cluster = ~id)
summary(iv, stage = 1)                 # first stage
fitstat(iv, ~ ivwald + ivf)            # first-stage Wald F (uses the model's vcov) and F
```

Report the first-stage F computed with the **same** clustered variance as the second stage
(`ivwald`). F > 10 is not by itself sufficient evidence against weak instruments; report
the statistic and let the PI decide on weak-IV-robust inference.

## Difference-in-differences and event studies

```r
# 2x2 / single-timing TWFE
did <- feols(y ~ treat_post | id + year, data = dt, cluster = ~id)

# Event study; never-treated units coded rel_time = -1000 and excluded as a reference
dt[, rel_time := year - treat_year]
dt[is.na(rel_time), rel_time := -1000]
es <- feols(y ~ i(rel_time, ref = c(-1, -1000)) | id + year, data = dt, cluster = ~id)
iplot(es)
```

With **staggered** treatment timing, TWFE is biased under heterogeneous effects. Use:

```r
# Sun-Abraham: never-treated cohort coded outside the sample period (e.g. 10000)
sa <- feols(y ~ sunab(cohort, year) | id + year, data = dt, cluster = ~id)
summary(sa, agg = "att")

# Callaway-Sant'Anna: gname = 0 for never-treated
library(did)
cs <- att_gt(yname = "y", tname = "year", idname = "id", gname = "cohort", data = dt)
aggte(cs, type = "dynamic")
```

## Regression discontinuity

```r
library(rdrobust); library(rddensity)
rd <- rdrobust(y = dt$y, x = dt$running, c = 0)     # data-driven bandwidth, robust bias-corrected CI
summary(rd)
summary(rddensity(X = dt$running, c = 0))           # manipulation test
rdplot(y = dt$y, x = dt$running, c = 0)
```

## Few clusters

With fewer than about 30-40 clusters, conventional cluster-robust inference over-rejects.
Report the wild cluster bootstrap that the protocol specifies (for example Webb weights),
using the package installed in this project, and state the number of clusters.

## Tables

```r
etable(m1, m2,
  dict      = c(x = "Treatment", y = "Outcome"),
  fitstat   = ~ n + r2,
  style.tex = style.tex("aer"),
  tex       = TRUE,
  file      = file.path(out_tab, "tab_main.tex"),
  replace   = TRUE)
fwrite(as.data.table(coeftable(m1), keep.rownames = "term"),
       file.path(out_tab, "tab_main.csv"))          # machine-readable copy for the report
```

## Figures

```r
library(ggplot2)
ct <- as.data.table(coeftable(m1), keep.rownames = "term")
ct[, `:=`(lo = Estimate - 1.96 * `Std. Error`, hi = Estimate + 1.96 * `Std. Error`)]
p <- ggplot(ct, aes(term, Estimate)) +
  geom_pointrange(aes(ymin = lo, ymax = hi)) +
  geom_hline(yintercept = 0, linetype = "dashed") +
  labs(x = NULL, y = "Coefficient (95% CI)") +
  theme_bw()
ggsave(file.path(out_fig, "fig_coefs.pdf"), p, width = 7, height = 4.5)
```

## Conventions

- Packages loaded at the top with `library()`, never `require()` (which fails silently).
- `dir.create(path, recursive = TRUE, showWarnings = FALSE)` before writing outputs.
- `snake_case`, verb-noun function names; named constants instead of magic numbers.
- Figures: `ggsave(..., width = 6.5, height = 4.5, dpi = 300, bg = "white")`.
- Save expensive objects with `saveRDS()` to `Processed_Data/`, so tables and figures can
  be regenerated without re-estimating.
- One pipe style per project (`|>` or `%>%`). Lines up to about 100 characters, except
  formulas that read better on one line (add a comment).

## Ending a script

```r
cat("PASS\n")        # last line; the report cites it
```
