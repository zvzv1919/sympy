# sympy/stats — Statistics & Probability

Introduces random variable types into the SymPy expression system. Random variables are created via convenience constructors (e.g. `Normal`, `Die`, `Poisson`) and queried with operators like `P`, `E`, `density`, `variance`. Internally, each random variable is backed by a probability space (`PSpace`) that handles integration, sampling, and conditioning.

## Core Framework

### `rv.py`
Abstract base classes and core operators for the random variable system. Everything else in the module builds on these types.

- **`RandomDomain`** — base class representing a set of symbols and their possible values.
  - Subclasses: `SingleDomain` (one variable), `ConditionalDomain` (domain + condition), `ProductDomain` (Cartesian product of domains).
- **`PSpace`** — abstract probability space; defines `where`, `compute_density`, `sample`, `probability`, `integrate`.
  - `SinglePSpace` — wraps a single symbol + distribution.
  - `ProductPSpace` — merger of independent probability spaces; dispatches to `ProductFinitePSpace` or `ProductContinuousPSpace` based on constituent spaces.
- **`RandomSymbol(Expr)`** — SymPy expression node that ties a `Symbol` to a `PSpace`. Created by `PSpace.value`, not directly by users.
- **`Density(Basic)`** — lazy density object; `.doit()` computes the actual PDF/PMF via the underlying PSpace.
- Core query functions (used internally; public wrappers live in `rv_interface.py`):
  - `expectation(expr, condition, numsamples, evaluate)` — expected value; exploits linearity of `E` over `Add`.
  - `probability(condition, given_condition, numsamples, evaluate)` — probability of a relational condition.
  - `density(expr, condition)` — PDF/PMF as a `Lambda` (continuous) or `dict` (discrete/finite).
  - `cdf(expr, condition)` — cumulative distribution function.
  - `where(condition, given_condition)` — domain where a condition is true.
  - `given(expr, condition)` — creates a conditional random expression by building a new conditional PSpace.
  - `sample(expr, condition)` / `sample_iter(expr, condition, numsamples)` — realizations; tries `lambdify` first, falls back to `subs`.
  - `dependent(a, b)` / `independent(a, b)` — tests statistical dependence by comparing conditional vs. unconditional densities.
- Sampling helpers: `sampling_P`, `sampling_E`, `sampling_density` — Monte Carlo approximations.
- `random_symbols(expr)` — extracts all `RandomSymbol` atoms from an expression.
- `pspace(expr)` — infers the `PSpace` of an expression (returns `ProductPSpace` for multi-variable expressions).
- `NamedArgsMixin` — mixin giving attribute access to positional `args` via `_argnames`.
- `_value_check(condition, message)` — parameter validation helper used by distribution constructors.

### `rv_interface.py`
Public statistical query API — thin wrappers and higher-order statistics built on `rv.py`.

- `P` = `probability`, `E` = `expectation` — shorthand aliases.
- `moment(X, n, c=0)` — n-th raw moment about `c`: `E((X-c)**n)`.
- `variance(X)` — `cmoment(X, 2)`.
- `std(X)` / `standard_deviation(X)` — `sqrt(variance(X))`.
- `covariance(X, Y)` — `E((X-E(X))*(Y-E(Y)))`.
- `correlation(X, Y)` — Pearson correlation coefficient.
- `cmoment(X, n)` — n-th central moment: `E((X-E(X))**n)`.
- `smoment(X, n)` — n-th standardized moment: `cmoment / std**n`.
- `skewness(X)` — `smoment(X, 3)`.

### `__init__.py`
Re-exports the public API from `rv_interface`, `frv_types`, `crv_types`, `drv_types`, and `symbolic_probability`.

---

## Continuous Random Variables

### `crv.py`
Framework for continuous random variables — domains, distributions, and probability spaces backed by symbolic integration.

- **Domain classes** (all subclass `ContinuousDomain` → `RandomDomain`):
  - `SingleContinuousDomain` — one symbol + `Interval`; `integrate` produces a `sympy.Integral`.
  - `ProductContinuousDomain` — Cartesian product of continuous domains.
  - `ConditionalContinuousDomain` — restricts integration limits by intersecting with solved inequalities; handles equalities via `DiracDelta`.
- **Distribution classes**:
  - `ContinuousDistribution` — callable base (`__call__` → `pdf`).
  - `SingleContinuousDistribution` — parameterized single-variable distribution; provides `pdf`, `cdf` (via `compute_cdf`), `sample` (via inverse-CDF), `expectation`.
  - `ContinuousDistributionHandmade` — user-supplied PDF + set; used by `ContinuousRV()`.
- **Probability-space classes**:
  - `ContinuousPSpace` — integrates `pdf * expr` over the domain; computes density, CDF, probability (univariate via `where`, multivariate via `DiracDelta` trick), conditional spaces.
  - `SingleContinuousPSpace` — single-variable specialization; delegates CDF/density to the distribution when possible, else falls back to general integration.
  - `ProductContinuousPSpace` — product of independent continuous spaces; `pdf` is the product of individual PDFs.
- `reduce_rational_inequalities_wrap(condition, var)` — utility to convert relational conditions into `Interval` objects for integration limit adjustment.

### `crv_types.py`
Prebuilt continuous distribution constructors. Each follows the pattern: a `*Distribution(SingleContinuousDistribution)` class defining `pdf` (and optionally `set`, `check`, `sample`), plus a public factory function returning a `RandomSymbol`.

- `ContinuousRV(symbol, density, set)` — create an RV from an arbitrary PDF expression.
- `rv(symbol, cls, args)` — internal factory shared by all constructors below.
- **34 distributions** (alphabetical): `Arcsin`, `Benini`, `Beta`, `BetaPrime`, `Cauchy`, `Chi`, `ChiNoncentral`, `ChiSquared`, `Dagum`, `Erlang` (reuses `GammaDistribution`), `Exponential`, `FDistribution`, `FisherZ`, `Frechet`, `Gamma`, `GammaInverse`, `Gompertz`, `Kumaraswamy`, `Laplace`, `Logistic`, `LogNormal`, `Maxwell`, `Nakagami`, `Normal`, `Pareto`, `QuadraticU`, `RaisedCosine`, `Rayleigh`, `ShiftedGompertz`, `StudentT`, `Triangular`, `Uniform`, `UniformSum` (Irwin-Hall), `VonMises`, `Weibull`, `WignerSemicircle`.

**Caveat**: `Erlang` does not have its own distribution class; it delegates to `GammaDistribution` with `theta = 1/l`.

---

## Discrete Random Variables (Infinite Support)

### `drv.py`
Framework for discrete random variables over infinite (or large) integer-valued domains — mirrors `crv.py` but uses summation instead of integration.

- `SingleDiscreteDistribution` — base class; provides `pdf`, `cdf` (via `compute_cdf` using `summation`), `sample` (inverse-CDF), `expectation` (via `Sum`).
- `SingleDiscreteDomain` — trivial subclass of `SingleDomain`.
- `SingleDiscretePSpace` — discrete probability space; `integrate` delegates to `distribution.expectation`, falling back to explicit `Sum`.

### `drv_types.py`
Prebuilt discrete distribution constructors for infinite-support distributions.

- `Poisson(name, lamda)` — Poisson distribution; `pdf(k) = lamda**k * exp(-lamda) / k!`.
- `Geometric(name, p)` — Geometric distribution; `pdf(k) = p * (1-p)**(k-1)`.

---

## Finite Random Variables

### `frv.py`
Framework for finite discrete random variables — uses explicit enumeration instead of integration/summation.

- **Domain classes**:
  - `FiniteDomain` — backed by a `FiniteSet` of `(symbol, value)` pairs.
  - `SingleFiniteDomain` — one symbol over a `FiniteSet`.
  - `ProductFiniteDomain` — Cartesian product via `itertools.product`.
  - `ConditionalFiniteDomain` — filters elements by testing a condition predicate.
- `FiniteDensity(dict)` — callable dictionary mapping values to probabilities.
- `SingleFiniteDistribution` — base for finite distributions; builds `dict`, `pdf` (as `Lambda` with `Piecewise`/`KroneckerDelta`), `set` from the dictionary.
- **Probability-space classes**:
  - `FinitePSpace` — computes density/CDF/probability by iterating over all domain elements and summing probabilities. Sampling via sorted-CDF lookup.
  - `SingleFinitePSpace` — single-variable specialization.
  - `ProductFinitePSpace` — product of independent finite spaces; density built via `itertools.product` over constituent densities.

### `frv_types.py`
Prebuilt finite distribution constructors.

- `FiniteRV(name, density)` — arbitrary finite RV from a `{value: probability}` dict.
- `DiscreteUniform(name, items)` — equal probability over a finite set.
- `Die(name, sides=6)` — fair die; `pdf` supports both numeric and symbolic arguments (uses `KroneckerDelta` for symbolic `x`).
- `Bernoulli(name, p, succ=1, fail=0)` — two-outcome trial.
- `Coin(name, p=1/2)` — `Bernoulli` with outcomes `'H'`/`'T'`.
- `Binomial(name, n, p, succ=1, fail=0)` — number of successes in `n` Bernoulli trials; validates that `n` is a non-negative integer and `0 ≤ p ≤ 1`.
- `Hypergeometric(name, N, m, n)` — draws without replacement.
- `Rademacher(name)` — equally likely `{-1, +1}`.

---

## Symbolic Probability Expressions

### `symbolic_probability.py`
Unevaluated symbolic wrappers for probability, expectation, variance, and covariance — allowing algebraic manipulation before numeric evaluation.

- **`Probability(Expr)`** — symbolic `P(condition)`. Supports `rewrite(Integral)` and `evaluate_integral()`.
- **`Expectation(Expr)`** — symbolic `E(expr)`.
  - `doit()` exploits linearity: distributes over `Add`, factors constants out of `Mul`.
  - `rewrite(Probability)` expresses as `∫ x·P(X=x) dx`.
- **`Variance(Expr)`** — symbolic `Var(X)`.
  - `doit()` expands `Var(aX) = a²·Var(X)`, `Var(X+Y) = Var(X) + Var(Y) + 2·Cov(X,Y)`.
  - `rewrite(Expectation)` → `E(X²) - E(X)²`.
- **`Covariance(Expr, Expr)`** — symbolic `Cov(X, Y)`.
  - `doit()` expands linearity: `Cov(aX+bY, cZ+dW) = ac·Cov(X,Z) + …`; `Cov(X,X) = Var(X)`.
  - Arguments are auto-sorted for canonical form.

---

## Utilities

### `error_prop.py`
Symbolic error/uncertainty propagation via first-order variance expansion.

- `variance_prop(expr, consts=(), include_covar=False)` — recursively propagates `Variance` / `Covariance` symbols through arithmetic expressions:
  - `Add` → sum of variances (+ cross-covariances if `include_covar`).
  - `Mul` → relative-variance formula `(expr)² · Σ(Var(aᵢ)/aᵢ²)`.
  - `Pow` → power rule.
  - `exp` → `Var(arg) · exp(arg)²`.
  - Symbols in `consts` are treated as exact (zero variance).
