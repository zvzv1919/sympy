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
Base classes for all random variable types: `RandomDomain`, `SingleDomain`, `PSpace`, `SinglePSpace`, `RandomSymbol`.
- Foundational probability-space and domain abstractions; handles conditioning and variable dependencies.
- `SinglePSpace.__new__`: constructs a probability space for a single variable; coerces string to `Symbol`, raises `TypeError` for non-string/non-Symbol input.
- `rv()` factory: creates a `RandomSymbol` from a name and distribution class.

### [`crv.py`](crv.py)
Infrastructure for continuous random variables.
- `ContinuousDomain`, `SingleContinuousDomain`, `ProductContinuousDomain`, `ConditionalContinuousDomain`.
- Integration-based expectation and probability computation over continuous intervals.

### [`drv.py`](drv.py)
Infrastructure for discrete random variables with infinite support.
- `SingleDiscreteDistribution`: base class with `pdf()`, `cdf()`, `sample()`, `expectation()`.
- Summation-based probability computation.

### [`frv.py`](frv.py)
Infrastructure for finite random variables (discrete, finite support).
- `FiniteDensity`, `FiniteDomain`, `SingleFiniteDomain`, `ProductFiniteDomain`, `ConditionalFiniteDomain`.
- `SingleFiniteDistribution`: base class whose subclasses define a `.dict` mapping outcomes to probabilities.
- Iteration-based probability computation over `FiniteSet`.

---

## Concrete Distribution Types

### [`crv_types.py`](crv_types.py)
All built-in continuous probability distributions (~28).
- Includes: `Normal`, `Exponential`, `Beta`, `Gamma`, `Uniform`, `StudentT`, `Weibull`, `Cauchy`, `Chi`, `LogNormal`, `Pareto`, `Rayleigh`, and more.
- Each distribution class has a `pdf(x)` method returning the probability density function expression.

### [`drv_types.py`](drv_types.py)
Built-in discrete distributions with infinite support.
- `PoissonDistribution` / `Poisson()`: Poisson-distributed random variable.
- `GeometricDistribution` / `Geometric()`: Geometric-distributed random variable.
- Each has a `pdf(x)` returning the probability mass function expression.

### [`frv_types.py`](frv_types.py)
Built-in finite random variable distributions (discrete, finite support). Each has a `pdf(x)` returning probability mass.
- `DieDistribution` / `Die(name, sides)`: fair multi-faced die; `pdf` branches on numeric vs symbolic input (returns Rational for valid integers, Zero otherwise; for symbols constructs a Sum with KroneckerDelta).
- `DiscreteUniformDistribution` / `DiscreteUniform()`: uniform distribution over a finite set of items.
- `BernoulliDistribution` / `Bernoulli()`: two-outcome process with probability p.
- `Coin(name, p)`: fair or unfair coin toss (wrapper around Bernoulli with 'H'/'T').
- `BinomialDistribution` / `Binomial()`: number of successes in n independent Bernoulli trials. `__new__` validates n (nonneg integer) and p (0 ≤ p ≤ 1), raises `ValueError` on invalid args.
- `HypergeometricDistribution` / `Hypergeometric()`: draws without replacement.
- `FiniteDistributionHandmade` / `FiniteRV()`: user-specified density dict.

---

## User-Facing API

### [`rv_interface.py`](rv_interface.py)
Convenience functions for probability and statistics queries.
- `P()`, `E()`, `density()`, `where()`, `given()`, `sample()`, `pspace()`.
- `moment()`, `variance()`, `std()`, `covariance()`, `correlation()`, `cmoment()`, `smoment()`, `skewness()`.

### [`symbolic_probability.py`](symbolic_probability.py)
Symbolic (unevaluated) representations of probabilistic expressions.
- `Probability`, `Expectation`, `Variance`, `Covariance`: subclasses of `Expr`.
- Support `rewrite()` to Integral/Sum forms and `doit()` for evaluation.

---

## Utilities

### [`error_prop.py`](error_prop.py)
Symbolic arithmetic error (uncertainty) propagation.
- `variance_prop(expr, consts, include_covar)`: propagates variance through symbolic expressions.

### [`__init__.py`](__init__.py)
Package init; re-exports public API (distributions, query functions) from submodules.
