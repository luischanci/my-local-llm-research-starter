# Julia econometrics and structural estimation

Read this before writing Julia estimation code. Run scripts with
`julia --project=. Data_and_Estimation/Code/NN_name.jl` from the repository root.
Use only packages already in `Project.toml`; if one is missing, stop and report it.

## Setup

```julia
using DataFrames, CSV, Statistics, StatsBase, LinearAlgebra, Random
using FixedEffectModels, Optim, ForwardDiff, Distributions

const SEED = 12345                                   # state it in the report
df = CSV.read(joinpath("Data_and_Estimation", "Data", "Processed_Data", "panel.csv"), DataFrame)
```

Use `joinpath`, never hard-coded separators.

## DataFrames

```julia
transform!(df, :x => (x -> x .* 100) => :x_pct)
subset!(df, :year => ByRow(>=(2000)))
combine(groupby(df, :id), :y => mean => :mean_y, nrow => :n)
leftjoin!(df, other, on = [:id, :year])
println("rows after join: ", nrow(df))               # record counts after merges
@assert nrow(unique(df, [:id, :year])) == nrow(df)
```

## Panel regressions (FixedEffectModels.jl)

```julia
m  = reg(df, @formula(y ~ x + fe(id) + fe(year)), Vcov.cluster(:state))
iv = reg(df, @formula(y ~ (endog ~ z) + fe(id) + fe(year)), Vcov.cluster(:state))
coef(m); vcov(m); nobs(m); r2(m)
```

## Maximum likelihood

Parameterise constrained parameters (σ = exp(logσ)) so the optimiser is unconstrained.

```julia
function nll(θ, y, X)                     # negative log-likelihood of the whole sample
    k = size(X, 2)
    β, σ = θ[1:k], exp(θ[k+1])
    u = y .- X * β
    return 0.5 * length(y) * log(2π * σ^2) + sum(abs2, u) / (2σ^2)
end

θ0  = zeros(size(X, 2) + 1)
res = optimize(θ -> nll(θ, y, X), θ0, LBFGS(); autodiff = :forward)
Optim.converged(res) || error("MLE did not converge")
θ̂  = Optim.minimizer(res)
V   = inv(ForwardDiff.hessian(θ -> nll(θ, y, X), θ̂))   # inverse Hessian of the NLL
se  = sqrt.(diag(V))                                    # SEs of the transformed θ
```

Try several starting values and report whether they reach the same optimum.

## GMM (linear IV example; same structure for nonlinear moments)

```julia
# moments g_i(θ) = Z_i * u_i(θ);   ḡ(θ) = Z'u / n
resid(θ)  = y .- X * θ
gbar(θ)   = Z' * resid(θ) / size(Z, 1)
Q(θ, W)   = (g = gbar(θ); dot(g, W * g))

n  = size(Z, 1)
θ1 = optimize(θ -> Q(θ, I), θ0, LBFGS(); autodiff = :forward) |> Optim.minimizer
Zu = Z .* resid(θ1)
S  = (Zu' * Zu) / n                      # uncentered; do NOT use cov(), which demeans
W  = inv(S)
θ2 = optimize(θ -> Q(θ, W), θ1, LBFGS(); autodiff = :forward) |> Optim.minimizer

G  = ForwardDiff.jacobian(gbar, θ2)      # ∂ḡ/∂θ'
V  = inv(G' * W * G) / n                 # efficient-GMM variance
J  = n * Q(θ2, W)                        # Hansen J, χ²(#moments − #params)
```

For clustered data, build `S` from cluster sums of `Zu` rows instead of individual rows.

## Simulation (MSM / indirect inference)

Hold simulation draws fixed across θ (common random numbers), or the objective is not smooth.

```julia
const DRAWS = randn(Xoshiro(SEED), N, S)             # drawn once, outside the objective
simulate(θ) = θ[1] .+ θ[2] .* Xsim .+ θ[3] .* DRAWS
```

## Bootstrap (threaded, reproducible)

```julia
function bootstrap(estimate, data, B; seed = SEED)
    n  = size(data, 1)
    k  = length(estimate(data))
    Θ  = Matrix{Float64}(undef, k, B)
    Threads.@threads for b in 1:B
        rng = Xoshiro(seed + b)                      # per-draw RNG: same result for any thread count
        idx = rand(rng, 1:n, n)
        Θ[:, b] = estimate(data[idx, :])
    end
    return Θ                                          # std(Θ; dims = 2) gives SEs
end
```

Resample clusters, not rows, when the data are clustered. Start Julia with
`julia --threads=auto --project=.` to use threads.

## Delta method

```julia
delta_se(g, θ, V) = (∇ = ForwardDiff.gradient(g, θ); sqrt(∇' * V * ∇))
```

## Performance

- Pre-allocate and update in place: `@. out = x^2 + y` writes into an existing `out`;
  `out = x.^2 .+ y` fuses the same way but allocates a new array.
- Concrete types in structs and hot loops; check with `@code_warntype`.
- Julia is column-major: loop over rows inside columns.
- Wrap hot code in functions; avoid non-`const` globals.

## Conventions

- Never install packages from a script; the environment is fixed by `Project.toml`/`Manifest.toml`.
- Estimation functions in one module file; a separate main script configures and calls them.
- Prefer `Threads.@threads` on one machine. Use `Distributed` (`addprocs`, `@everywhere`,
  `pmap`) only if the PI asks; then every shared function and dataset needs `@everywhere`.
- Check `Optim.converged` for every optimisation; use a grid or several starting values for
  nonconvex objectives and report whether they agree.
- `snake_case` functions and variables, `CamelCase` types; names follow the paper's
  notation (`sigma_v`, `rho`), as recorded in `refs/notation.md`.
- Lines up to about 92 characters, except equations that read better on one line (add a comment).

## Ending a script

```julia
println("PASS")
```
