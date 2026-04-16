# sympy/calculus — Catalog

> Part of [SymPy](../catalog.md). Calculus operations: finite differences, singularities, and continuity utilities.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Public API surface for the calculus package. Re-exports all user-facing symbols: `euler_equations`, monotonicity predicates (`is_increasing`, `is_strictly_increasing`, `is_decreasing`, `is_strictly_decreasing`, `is_monotonic`), `singularities`, finite-diff helpers, and `not_empty_in`/`AccumBounds`. Questions about the behavior or return values of these functions at the API level should reference this file. |
| `euler.py` | Implements `euler_equations` to compute Euler-Lagrange equations for a given Lagrangian over specified functions and variables. |
| `finite_diff.py` | Generates finite difference weights for arbitrary-order derivatives on non-uniform grids and provides helpers (`apply_finite_diff`, `as_finite_diff`, `differentiate_finite`) for numerical differentiation. |
| `singularities.py` | Implementation of `singularities` (finds poles of **rational functions only** via `1/expr`) and monotonicity predicates (`is_increasing`, `is_strictly_increasing`, `is_decreasing`, `is_strictly_decreasing`, `is_monotonic`) over intervals. Notable edge case: constant expressions (no free symbols, no symbol arg) return `True` for non-strict variants and `False` for strict variants. |
| `util.py` | Calculus utilities: `continuous_domain` computes the set of points where an expression is continuous by detecting singularities with special-case branching for half-integer (rational power with denominator 2) exponents vs. general expressions, constraining domains for `log` and square-root sub-expressions; `function_range` finds the range of a function over a domain; `not_empty_in` checks set non-emptiness; `periodicity` and `lcim` compute periods; `AccumulationBounds` provides interval-arithmetic on limits. |
