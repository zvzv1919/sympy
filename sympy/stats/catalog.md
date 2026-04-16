# sympy/stats — Catalog

> Part of [SymPy](../catalog.md). Symbolic probability and statistics: random variables, distributions, and expectation.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package entry point for the stats module. Re-exports all public symbols (distributions, query functions, symbolic probability classes) from submodules. |
| `rv.py` | Core random variable framework and computational engine. Defines abstract types for random domains, probability spaces (`PSpace`), and random symbols. Contains the actual implementations of `expectation`, `probability`, `density`, `where`, and `sample` that perform real computation — including efficiency optimizations such as exploiting linearity of expectation (distributing over sums), Monte Carlo sampling dispatch, conditional expectation handling, and delegating to `PSpace.integrate` for evaluation. Does not contain distribution-specific CDF/PDF logic or CDF inversion — those live in the type-specific modules (`crv.py`, `drv.py`, `frv.py`). |
| `rv_interface.py` | User-facing interface that wraps the core `rv` functions into convenient aliases (`P`, `E`, `variance`, `std`, `skewness`, `covariance`, `correlation`, `moment`, `cmoment`, `smoment`). |
| `crv.py` | Continuous random variable infrastructure. Implements `ContinuousDomain`, `SingleContinuousDistribution`, `ContinuousPSpace`, and single/product continuous probability spaces with integration-based computation of densities, CDFs, CDF inversion for sampling (including handling of `solveset` intersection results), and expectation. |
| `crv_types.py` | Prebuilt continuous distribution types. Provides constructors and PDF definitions for over 35 named distributions (Normal, Exponential, Gamma, Beta, Uniform, Cauchy, Chi-squared, etc.). |
| `drv.py` | Discrete random variable infrastructure. Defines `SingleDiscreteDistribution` and `SingleDiscretePSpace` with summation-based CDF computation, sampling, and expectation. |
| `drv_types.py` | Prebuilt discrete distribution types. Provides constructors and PDF definitions for the Poisson and Geometric distributions. |
| `frv.py` | Finite discrete random variable infrastructure. Implements `FiniteDomain`, `FinitePSpace`, `FiniteDensity` (a dict-based density lookup), and related domain/space classes for random variables with a finite set of outcomes. Handles enumeration over finite sample spaces, conditional domains, and product spaces — but not individual distribution definitions or their PDF logic. |
| `frv_types.py` | Prebuilt finite distribution types. Each distribution class implements its own `pdf` method with type-specific logic (e.g., handling symbolic vs numeric arguments, branching on input type). Provides constructors and distribution classes for Die, Coin, Bernoulli, Binomial, DiscreteUniform, Hypergeometric, Rademacher, and custom `FiniteRV`. |
| `symbolic_probability.py` | Symbolic (unevaluated) probability expression containers. Defines `Probability`, `Expectation`, `Variance`, and `Covariance` as SymPy `Expr` subclasses for display and symbolic manipulation (e.g., rewriting to integrals/sums, `expand`). Their `doit()` methods delegate to the real computational functions in `rv.py` rather than implementing computation directly. Not where computational optimizations like linearity of expectation over sums are implemented. |
| `error_prop.py` | Arithmetic error propagation. Implements `variance_prop` to symbolically propagate variance through mathematical expressions using first-order Taylor expansion, optionally including covariance terms. |
