# sympy/calculus — Catalog

> Part of [SymPy](../catalog.md). Calculus operations: finite differences, singularities, and continuity utilities.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package initializer that re-exports public API symbols from `euler`, `singularities`, `finite_diff`, and `util` submodules. |
| `euler.py` | Implements `euler_equations` to compute Euler-Lagrange equations for a given Lagrangian over specified functions and variables. |
| `finite_diff.py` | Generates finite difference weights for arbitrary-order derivatives on non-uniform grids and provides helpers (`apply_finite_diff`, `as_finite_diff`, `differentiate_finite`) for numerical differentiation. |
| `singularities.py` | Finds singularities of **rational functions only** (solves `1/expr` for poles) and provides monotonicity predicates (`is_increasing`, `is_strictly_increasing`, `is_decreasing`, `is_strictly_decreasing`, `is_monotonic`) over intervals. Does not handle irrational or transcendental expressions. |
| `util.py` | Calculus utilities: `continuous_domain` computes the set of points where an expression is continuous by detecting singularities with special-case branching for half-integer (rational power with denominator 2) exponents vs. general expressions, constraining domains for `log` and square-root sub-expressions; `function_range` finds the range of a function over a domain; `not_empty_in` checks set non-emptiness; `periodicity` and `lcim` compute periods; `AccumulationBounds` provides interval-arithmetic on limits. |
