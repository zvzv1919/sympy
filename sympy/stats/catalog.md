# sympy/stats — Catalog

> Part of [SymPy](../catalog.md). Symbolic probability and statistics: random variables, distributions, and expectation.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package entry point for the stats module. Re-exports all public symbols (distributions, query functions, symbolic probability classes) from submodules. |
| `rv.py` | Core random variable framework. Defines abstract types for random domains, probability spaces (`PSpace`), and random symbols, plus fundamental operators like `probability`, `expectation`, `density`, `where`, and `sample`. Also contains boolean dependence/independence testing (`dependent`, `independent`, `pspace_independent`) that detects whether two stochastic expressions are statistically related by comparing conditional vs marginal densities. Includes helper utilities `rv_subs`, `NamedArgsMixin`, and `_value_check`. |
| `rv_interface.py` | User-facing interface that wraps the core `rv` functions into convenient aliases (`P`, `E`, `variance`, `std`, `skewness`, `covariance`, `correlation`, `moment`, `cmoment`, `smoment`). These compute numeric statistical measures (e.g. `correlation` returns a scalar coefficient). For boolean dependence/independence testing (whether knowing one variable changes another's distribution), see `rv.py`. |
| `crv.py` | Continuous random variable infrastructure. Implements `ContinuousDomain`, `ContinuousPSpace`, and single/product continuous probability spaces with integration-based computation of densities and CDFs. |
| `crv_types.py` | Prebuilt continuous distribution types. Provides constructors and PDF definitions for over 35 named distributions (Normal, Exponential, Gamma, Beta, Uniform, Cauchy, Chi-squared, etc.). |
| `drv.py` | Discrete random variable infrastructure for distributions over infinite/countable supports (e.g. integers). Defines `SingleDiscreteDistribution` and `SingleDiscretePSpace` with summation-based CDF computation, inverse-CDF sampling, and expectation. |
| `drv_types.py` | Prebuilt discrete distribution types. Provides constructors and PDF definitions for the Poisson and Geometric distributions. |
| `frv.py` | Finite discrete random variable infrastructure. Implements `FiniteDomain`, `FinitePSpace`, and related classes for random variables with a finite set of outcomes. Uses explicit enumeration for density, CDF (`compute_cdf`, `sorted_cdf`), sampling (uniform-random draw against sorted CDF with linear search), probability, and conditional spaces. See `drv.py` for infinite-support discrete variables. |
| `frv_types.py` | Prebuilt finite distribution types. Provides constructors for Die, Coin, Bernoulli, Binomial, DiscreteUniform, Hypergeometric, Rademacher, and custom `FiniteRV`. |
| `symbolic_probability.py` | Symbolic (unevaluated) probability expressions. Defines `Probability`, `Expectation`, `Variance`, and `Covariance` as SymPy `Expr` subclasses that support rewriting to integrals/sums and algebraic expansion via `doit()`. |
| `error_prop.py` | Arithmetic error propagation. Implements `variance_prop` to symbolically propagate variance through mathematical expressions using first-order Taylor expansion, optionally including covariance terms. |
