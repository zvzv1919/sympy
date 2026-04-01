# sympy/calculus — Catalog

> Part of [SymPy](../catalog.md). Calculus operations: finite differences, singularities, and continuity utilities.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package initializer that re-exports public API symbols from `euler`, `singularities`, `finite_diff`, and `util` submodules. |
| `euler.py` | Implements `euler_equations` to compute Euler-Lagrange equations for a given Lagrangian over specified functions and variables. |
| `finite_diff.py` | Generates finite difference weights for arbitrary-order derivatives on non-uniform grids and provides helpers (`apply_finite_diff`, `as_finite_diff`, `differentiate_finite`) for numerical differentiation. |
| `singularities.py` | Finds singularities of rational functions and provides monotonicity predicates (`is_increasing`, `is_strictly_increasing`, `is_decreasing`, `is_strictly_decreasing`, `is_monotonic`) over intervals. |
| `util.py` | General calculus utilities including `continuous_domain`, `function_range`, `not_empty_in`, `periodicity`, `lcim`, and the `AccumulationBounds` class for interval-arithmetic on limits. |
