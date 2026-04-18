# Calculus Module Catalog

## Euler–Lagrange Equations

### [`euler.py`](euler.py)
Computes Euler–Lagrange differential equations from a Lagrangian.
- `euler_equations(L, funcs, vars)` — derives variational equations of motion for a given Lagrangian expression.

## Finite Differences

### [`finite_diff.py`](finite_diff.py)
Finite difference weight generation, derivative approximation, and symbolic conversion.
- `finite_diff_weights(order, x_list, x0)` — generates weight coefficients for finite difference formulas on arbitrarily spaced grids using the Fornberg algorithm.
  - Validates `order`: raises `ValueError` for negative or non-integer derivative orders.
- `apply_finite_diff(order, x_list, y_list, x0)` — computes a finite difference approximation of a derivative from discrete point values.
- `as_finite_diff(derivative, points, x0, wrt)` — rewrites a `Derivative` object as a finite difference expression.

## Singularities and Monotonicity

### [`singularities.py`](singularities.py)
Finds poles of rational functions (denominator zeros) and tests monotonicity on intervals.
- `singularities(expr, sym)` — finds poles of a rational function only (denominator zeros); raises `NotImplementedError` for non-rational inputs. Does not handle general continuity analysis or rational-power expressions (see `util.py:continuous_domain` for that).
- `is_increasing`, `is_strictly_increasing`, `is_decreasing`, `is_strictly_decreasing` — interval monotonicity predicates.
- `is_monotonic(f, interval, symbol)` — tests whether a function is monotonic on an interval.

## Continuity, Range, and Accumulation Bounds

### [`util.py`](util.py)
Domain/range analysis and interval-arithmetic accumulation bounds.
- `continuous_domain(f, symbol, domain)` — returns intervals where a function is continuous; special-cases rational powers (half-integer exponents) by constraining domains and using a distinct singularity-search strategy vs. general expressions.
  - Raises `NotImplementedError` when the internal singularity solver (solveset) throws an exception.
- `function_range(f, symbol, domain)` — computes the range of a function over a domain.
- `not_empty_in(finset_intersection, *syms)` — finds domains where a finite-set intersection is non-empty.
- `AccumulationBounds` (alias `AccumBounds`) — represents a closed interval [a, b] for bounding accumulation points.
  - Supports arithmetic (+, -, *, /, **) and elementary functions (sin, exp, log).
  - Supports comparison operators (`<`, `>`, `<=`, `>=`) and containment (`__contains__`/`in`).
  - Caveat: `__contains__` treats ±oo as paired — if *either* bound is infinite, both oo and -oo are considered contained.
  - Used for limit computations with indeterminate forms, not for derivative-order validation.
