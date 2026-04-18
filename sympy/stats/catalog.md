# Stats Module Catalog

## Architecture Overview
The stats module is organized in three tiers per variable type:
- **Infrastructure** (`rv.py`, `crv.py`, `drv.py`, `frv.py`): base classes for domains, probability spaces, and distributions.
- **Concrete distributions** (`crv_types.py`, `drv_types.py`, `frv_types.py`): named distribution classes and factory functions, each with a `pdf` method.
- **User-facing API** (`rv_interface.py`, `symbolic_probability.py`): query functions (P, E, variance, density) and symbolic probability expressions.

Variable-type key: **crv** = continuous, **drv** = discrete (infinite support), **frv** = finite (discrete, finite support).

---

## Core Infrastructure

### [`rv.py`](rv.py)
Base classes and core query functions for all random variable types.
- Base classes: `RandomDomain`, `SingleDomain`, `PSpace`, `SinglePSpace`, `RandomSymbol`, `ProductPSpace`, `ProductDomain`.
- `ProductPSpace`: merges independent probability spaces; its `integrate` decomposes integration by iterating constituent spaces and integrating only each space's own variables.
- `SinglePSpace.__new__`: constructs a probability space for a single variable; coerces string to `Symbol`, raises `TypeError` for non-string/non-Symbol input.
- `rv()` factory: creates a `RandomSymbol` from a name and distribution class.
- `Density` class + `density()`: compute probability density of a random expression. `Density.doit()` returns a DiracDelta-based Lambda for deterministic (non-stochastic) expressions.
- `cdf(expr, condition, evaluate=True)`: computes cumulative distribution function; delegates to `pspace().compute_cdf()`.
  - When a condition is supplied, rewrites expr via `given()` and recurses to reduce to the unconditional case.
  - When `evaluate` is true and the result has a `doit` method, calls `doit()`; otherwise returns the raw result unchanged.
- `where()`, `sample()`, `sample_iter()`: query domain of conditions and draw realizations.
- `given(expr, condition)`: conditions a random expression on an event; for single-variable equality conditions, solves via `solveset` and substitutes solutions — unwraps `Intersection` with `S.Reals` when the solver returns that form; otherwise builds a full conditional probability space.
- `expectation(expr, condition)`: computes expected value of a random expression; exploits linearity (decomposes `Add` into per-term expectations) for efficiency; delegates final integration to `pspace().integrate()`.
- `probability(condition, given_condition)`: computes probability that a condition holds; supports Monte Carlo sampling via `numsamples`.
  - Short-circuits without integration: returns `S.Zero` when `given_condition` is `False`, returns `S.One`/`S.Zero` when `condition` is trivially true/false.
- `sampling_E`, `sampling_P`, `sampling_density`: Monte Carlo approximations of expectation, probability, and density.
- `_value_check(condition, message)`: parameter validation utility used across all distribution types; uses `condition == False` (not `not condition`), so symbolic/unevaluable conditions silently pass.
- `NamedArgsMixin`: mixin providing attribute-style access to positional `args` via `_argnames` tuple.

### [`crv.py`](crv.py)
Infrastructure for continuous random variables.
- `ContinuousDomain`, `SingleContinuousDomain`, `ProductContinuousDomain` (continuous subclass of `ProductDomain`), `ConditionalContinuousDomain`.
- `ConditionalContinuousDomain.integrate`: integrates expressions over condition-restricted domains; dispatches by condition type — inequalities narrow integration limits, equalities inject a DiracDelta factor into the integrand.
- `ContinuousPSpace`: continuous probability space; holds a domain and a density (PDF).
  - `compute_density(expr)`: if expr is one of the space's variables, marginalizes others out of the joint PDF via integration; for non-trivial functions of random variables, uses DiracDelta as an integration kernel to derive the transformed density.
  - `compute_cdf(expr)`: integrates the density from the left bound; raises `ValueError` on multivariate domains.
- `SingleContinuousPSpace`: probability space for a single univariate continuous variable.
  - `compute_density(expr)`: derives the density of a transformed variable (function of X) via change-of-variables; uses `solveset` to find inverses and unwraps `Intersection` with `S.Reals`.
  - `compute_cdf(expr)`: delegates to the distribution's `compute_cdf` for the identity case; falls back to base class otherwise.
- `ContinuousDistributionHandmade`: internal distribution wrapper used by `ContinuousRV` (in `crv_types.py`); accepts a Lambda pdf and a set.
- `SingleContinuousDistribution`: base class for all named continuous distributions; provides default `compute_cdf` (integrates PDF from left bound) and `expectation` (integrates expr·PDF). Subclasses in `crv_types.py` override these with distribution-specific simplifications.
- Integration-based expectation and probability computation over continuous intervals.

### [`drv.py`](drv.py)
Infrastructure for discrete random variables with infinite support.
- `SingleDiscreteDistribution`: base class with `pdf()`, `cdf()`, `sample()`, `expectation()` (summation-based).
  - `compute_cdf()`: sums the PDF from the infimum of the support set up to z, then wraps the result in a Piecewise that forces zero for any argument below the left boundary.
- `SingleDiscretePSpace`: probability space for a single discrete variable.
  - `integrate()`: computes expected values by delegating to `distribution.expectation()`; catches broad `Exception` and falls back to a raw `Sum` expression.
  - `compute_cdf()`, `compute_density()`: delegate to the underlying distribution.

### [`frv.py`](frv.py)
Infrastructure for finite random variables (discrete, finite support).
- `FiniteDensity`, `FiniteDomain`, `SingleFiniteDomain`, `ProductFiniteDomain`, `ConditionalFiniteDomain`.
- `SingleFiniteDistribution`: base class whose subclasses define a `.dict` mapping outcomes to probabilities.
- `FinitePSpace`: probability space for finitely many outcomes; `integrate` computes expected values as the weighted sum Σ f(x)·P(x) over all domain elements.
- `compute_density`, `compute_cdf`, `sorted_cdf`: derive density/CDF dicts by iterating and accumulating over the finite domain's enumerated outcomes.

---

## Concrete Distribution Types

### [`crv_types.py`](crv_types.py)
All built-in continuous probability distributions (~28) plus a factory for user-defined ones.
- `ContinuousRV(symbol, density, set)`: user-facing factory for custom continuous random variables from an arbitrary density; defaults support to `(-oo, oo)` (entire real line) when `set` is omitted. Does **not** invoke `check()` for parameter validation.
- `rv(symbol, cls, args)`: internal factory used by all named distribution constructors; calls `dist.check(*args)` for parameter validation before creating the pspace.
- Named distributions: `Normal`, `Exponential`, `Beta`, `Gamma`, `Uniform`, `StudentT`, `Weibull`, `Cauchy`, `Chi`, `LogNormal`, `Pareto`, `Rayleigh`, and more.
- Each distribution class has a `pdf(x)` method returning the probability density function expression.
- Some distributions override `compute_cdf` or `expectation` with post-processing that simplifies symbolic results (e.g., `UniformDistribution.compute_cdf` substitutes `Min` expressions after generic integration to resolve symbolic boundary ordering).
- Some distributions override `sample()` to bypass the generic inverse-CDF sampling in the base class (e.g., `LogNormalDistribution` delegates to `random.lognormvariate`).

### [`drv_types.py`](drv_types.py)
Built-in discrete distributions with infinite support.
- `PoissonDistribution` / `Poisson()`: Poisson-distributed random variable.
- `GeometricDistribution` / `Geometric()`: Geometric-distributed random variable.
- Each has a `pdf(x)` returning the probability mass function expression.

### [`frv_types.py`](frv_types.py)
Built-in finite random variable distributions (discrete, finite support). Each has a `pdf(x)` returning probability mass.
- `DieDistribution` / `Die(name, sides)`: fair multi-faced die; `pdf` has three branches: numeric → Rational for valid integers, Zero otherwise; plain Symbol → Sum with KroneckerDelta; anything else (e.g. symbolic expressions) → raises `ValueError`.
- `DiscreteUniformDistribution` / `DiscreteUniform()`: uniform distribution over a finite set of symbolic or numeric items; `pdf(x)` returns `Rational(1, n)` if x is in the item set, `S.Zero` otherwise.
- `BernoulliDistribution` / `Bernoulli()`: two-outcome process with probability p.
- `Coin(name, p)`: fair or unfair coin toss (wrapper around Bernoulli with 'H'/'T').
- `BinomialDistribution` / `Binomial()`: number of successes in n independent Bernoulli trials. `__new__` validates n (nonneg integer) and p (0 ≤ p ≤ 1), raises `ValueError` on invalid args.
- `HypergeometricDistribution` / `Hypergeometric()`: draws without replacement.
- `FiniteDistributionHandmade` / `FiniteRV()`: user-specified density dict.

---

## User-Facing API

### [`rv_interface.py`](rv_interface.py)
Higher-level statistical convenience functions built on top of `rv.py`; re-exports `E`/`P`/`density`/`cdf`/`where`/`given`/`sample`/`pspace` as thin aliases (implementation lives in `rv.py`).
- `moment()`, `variance()`, `std()`, `covariance()`, `correlation()`, `cmoment()`, `smoment()`, `skewness()`.
- `covariance(X, Y)`: numerically computes `E((X-E(X))*(Y-E(Y)))` via integration; no algebraic decomposition of arguments (contrast `Covariance.doit()` in `symbolic_probability.py`).
- `variance(X)`: delegates to `cmoment(X, 2)` (second central moment).

### [`symbolic_probability.py`](symbolic_probability.py)
Symbolic (unevaluated) representations of probabilistic expressions — for algebraic manipulation and rewriting, not direct numeric evaluation.
- `Probability`, `Expectation`, `Variance`, `Covariance`: subclasses of `Expr`; remain unevaluated until `.doit()` or `.rewrite()` is called.
- `Expectation.__new__`: short-circuits when `condition is None` and expr contains no `RandomSymbol` — returns expr unwrapped. `Probability.__new__` always wraps.
- `Expectation._eval_rewrite_as_Probability`: converts expectation to Integral/Sum weighted by `Probability(Eq(rv, x))`; generates a fresh dummy symbol (lowercased or `_1`-suffixed) and dispatches by pspace type (continuous → Integral, discrete-infinite → Sum, finite → raises `NotImplemented` singleton instead of `NotImplementedError`, producing a confusing `TypeError` at runtime).
- `Variance.doit()`: algebraically expands variance — splits sums into individual variances + pairwise covariances; factors products by squaring deterministic coefficients; returns `self` unchanged for other compound expressions (e.g., function applications like `sin(X)`).
- `Covariance.doit()`: algebraic bilinear expansion — decomposes each argument into (scalar, RandomSymbol) pairs via `_expand_single_argument`, then Cartesian-products them; identical args delegate to `Variance`.
- `Covariance._expand_single_argument(expr)`: splits a linear combination of stochastic variables into a list of `(coefficient, RandomSymbol)` tuples; handles `Add`, `Mul`, bare `RandomSymbol`, and general stochastic expressions.

---

## Utilities

### [`error_prop.py`](error_prop.py)
Derivative-based arithmetic error (uncertainty) propagation for general expressions (not probability-space algebra).
- `variance_prop(expr, consts, include_covar)`: computes total variance via partial-derivative formula; all non-const symbols are treated as variant.

### [`__init__.py`](__init__.py)
Package entry point; assembles `__all__` by importing and re-exporting the public API from each submodule.
- Authoritative manifest of every exported name, grouped by category: finite (`frv_types`), continuous (`crv_types`), discrete-infinite (`drv_types`), query functions (`rv_interface`), symbolic probability.
- To determine which distributions or functions are available at the package level, consult this file.
