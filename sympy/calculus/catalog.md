# sympy/calculus — Calculus Utilities

A collection of calculus-related algorithms: Euler-Lagrange equations, finite difference approximations, singularity detection, monotonicity tests, continuous-domain analysis, and interval arithmetic via accumulation bounds.

---

## Variational Calculus

### `euler.py`

Derives Euler-Lagrange equations from a Lagrangian.

- **`euler_equations(L, funcs=(), vars=())`** — Given a Lagrangian `L`, returns a list of `Eq` objects representing the Euler-Lagrange differential equations.
  - Auto-detects unknown functions and independent variables when not supplied.
  - Handles arbitrary derivative order and multivariate Lagrangians (e.g. `f(x,y)`) by iterating over `combinations_with_replacement` of the independent variables.

---

## Finite Differences

### `finite_diff.py`

Generates finite difference weights for arbitrarily-spaced grids and applies them to approximate derivatives (or interpolate).

- **`finite_diff_weights(order, x_list, x0=0)`** — Computes FD weight tables for derivatives 0 through `order` on a 1-D grid `x_list`, evaluated at `x0`.
  - Uses the Fornberg (1988) recursive algorithm.
  - Returns a 3-D list `delta[m][n][nu]`: derivative order × grid-subset size × grid-point index. The most accurate weights are in `delta[m][-1]`.
  - Lower-order derivative weights and subset-based weights come "for free."

- **`apply_finite_diff(order, x_list, y_list, x0=0)`** — Evaluates a finite-difference approximation by combining precomputed weights with function values `y_list`. Thin wrapper around `finite_diff_weights`.

- **`as_finite_diff(derivative, points=1, x0=None, wrt=None)`** — Converts a symbolic `Derivative` instance into a finite-difference expression.
  - Accepts either an explicit point sequence or a step-size (generates an equidistant stencil centered at `x0`).
  - Supports partial derivatives via the `wrt` parameter.

---

## Singularities & Monotonicity

### `singularities.py`

Finds singularities of rational functions and tests monotonicity of univariate expressions over intervals.

- **`singularities(expr, sym)`** — Returns the set of singularities (poles) of a univariate rational function by solving `1/expr = 0`.

- **Monotonicity predicates** — Each differentiates `f` and checks whether the derivative's sign condition holds over the entire interval via `solveset`. All accept `(f, interval=S.Reals, symbol=None)`:
  - `is_increasing` — `f' ≥ 0` on interval.
  - `is_strictly_increasing` — `f' > 0` on interval.
  - `is_decreasing` — `f' ≤ 0` on interval.
  - `is_strictly_decreasing` — `f' < 0` on interval.
  - `is_monotonic` — returns `fuzzy_or([is_increasing, is_decreasing])`.

**Caveats:** Only rational functions are supported by `singularities`; other kinds raise `NotImplementedError`. The monotonicity helpers do not yet support general multivariate expressions.

---

## Analysis Utilities & Interval Arithmetic

### `util.py`

Domain/range analysis for univariate functions and the `AccumulationBounds` interval-arithmetic class.

- **`continuous_domain(f, symbol, domain)`** — Returns the subset of `domain` where `f` is continuous. Handles `sqrt` (base ≥ 0) and `log` (argument > 0) constraints, then removes singularities found via `solveset(1/f)`.

- **`function_range(f, symbol, domain)`** — Computes the range of `f` over `domain` by evaluating at critical points (roots of `f'`) and boundary limits. Returns an `Interval` (or `Union` of intervals).

- **`not_empty_in(finset_intersection, *syms)`** — Given a `FiniteSet ∩ Set` intersection, finds the domain of the symbol(s) for which the intersection is non-empty. Solves membership inequalities per element per interval.

- **`AccumulationBounds(min, max)`** (alias `AccumBounds`) — Represents a closed interval `⟨min, max⟩` for bounding accumulation points of a function at a limit.
  - Full arithmetic: `+`, `-`, `*`, `/`, `**`, `abs`, with careful handling of infinities and zero-crossings.
  - Comparisons (`<`, `<=`, `>`, `>=`) return `True`/`False`/`None` (indeterminate when ranges overlap).
  - `__contains__` — point-in-interval test; `±∞` are considered members when either endpoint is infinite.
  - `intersection(other)` / `union(other)` — set-like operations with other `AccumBounds` or `FiniteSet`.
  - Properties: `min`, `max`, `delta`, `mid`.

**Caveats:** `AccumulationBounds` is **not** a floating-point interval arithmetic library — use `mpmath.iv` for numerical interval computations. The `union` method is approximate and may not be fully correct for disjoint ranges.

---

## Package Initialization

### `__init__.py`

Re-exports the public API:

`euler_equations`, `singularities`, `is_increasing`, `is_strictly_increasing`, `is_decreasing`, `is_strictly_decreasing`, `is_monotonic`, `finite_diff_weights`, `apply_finite_diff`, `as_finite_diff`, `not_empty_in`, `AccumBounds`.
