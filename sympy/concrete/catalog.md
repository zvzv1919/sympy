# concrete — Module Catalog

Finite and infinite sums, products, and related algorithms.

## Architecture Overview

- `ExprWithLimits` (in `expr_with_limits.py`) is the shared base for Integral, Sum, and Product.
- `ExprWithIntLimits` (in `expr_with_intlimits.py`) extends it for integer-bounded expressions (Sum, Product only).
- Limit-reordering, index-swapping, and change-of-index live in the **base classes**, not in Sum or Product.
- `Sum` and `Product` each implement their own `doit()` with type-specific evaluation and reversed-range handling.

---

## Base Classes

### [`expr_with_limits.py`](expr_with_limits.py)
Abstract base for any expression with limits (integrals, sums, products).

- `_process_limits(*symbols)` — canonicalize limit specifications into `(sym, lo, hi)` triples; coerces symbols and bounds. Returns `(limits, orientation)` where orientation is flipped (×−1) when upper bound is None but lower is present (e.g. `(x,5,None)` → `(x,None,5)` reversed).
- **`ExprWithLimits`** — base class providing `function`, `limits`, `variables`, `free_symbols`, `is_number`.
  - `__new__` — constructor: denests nested same-type calls (flattens by prepending inner limits), distributes over `Equality`, applies `piecewise_fold`, then validates all limits have 3 elements with no `None`.
  - `as_dummy()` — replace dummy variables with explicit dummies.
  - `_eval_interval`, `_eval_subs` — substitution helpers.
- **`AddWithLimits(ExprWithLimits)`** — base for Sum and Integral (oriented additions).
  - `_eval_adjoint`, `_eval_conjugate`, `_eval_transpose`, `_eval_factor`, `_eval_expand_basic`.

### [`expr_with_intlimits.py`](expr_with_intlimits.py)
Base class for expressions with **integer** limits — shared by Sum and Product.

- **`ReorderError`** — raised when dependent limits cannot be reordered.
- **`ExprWithIntLimits(ExprWithLimits)`**:
  - `change_index(var, trafo, newvar)` — substitute a new index variable via a linear mapping (`i → a*i+b`, `a=±1` only); rewrites bounds to match the new variable. Does **not** negate the summand/product.
  - `index(x)` — return the positional index of a dummy variable in the limits list.
  - `reorder(*arg)` — reorder limits by swapping pairs; each pair can mix numeric positions and symbolic variable names. Raises `ValueError` if any pair has length ≠ 2.
  - `reorder_limit(x, y)` — interchange two specific limit tuples.

---

## Sum and Product

### [`summations.py`](summations.py)
Unevaluated and evaluated finite/infinite summations.

- **`Sum(AddWithLimits, ExprWithIntLimits)`** — unevaluated summation.
  - `__new__` — constructor validates that every limit tuple has exactly 3 elements and no `None` bounds; raises `ValueError` if missing. Unlike integrals, discrete sums always require explicit lower and upper bounds.
  - `doit()` — evaluate the sum; handles reversed ranges by swapping bounds and negating the summand.
  - `is_convergent()` / `is_absolutely_convergent()` — convergence tests for infinite series.
  - `euler_maclaurin(m, n, eps, eval_integral)` — Euler–Maclaurin approximation.
  - `reverse_order(*indices)` — flip selected iteration bounds: swaps upper↔lower (each shifted by 1) and negates the summand once per flipped index; even flips cancel the sign.
  - `eval_zeta_function(f, limits)` — detect Riemann zeta function form.
  - `_eval_derivative`, `_eval_difference_delta`, `_eval_simplify`.
- `summation(f, *symbols)` — convenience wrapper that calls `Sum(...).doit()`.
- `eval_sum(f, limits)` — main evaluation dispatcher; handles Piecewise summands (folds when conditions are index-independent, bails out for index-dependent conditions with symbolic/large ranges), KroneckerDelta, finite direct, symbolic, and hypergeometric paths.
- `telescopic(L, R, limits)` — detect telescoping (collapsing/canceling) sums via pattern matching; matches `L(i+k)` against `-R` to find shift `k`, validates shift is integer and cancellation holds, falls back to `solve` if match fails.
- `telescopic_direct(L, R, n, limits)` — directly sum boundary terms of a confirmed telescoping sum.
- `_eval_sum_hyper(f, i, a)` / `eval_sum_hyper` — evaluate **infinite** hypergeometric sums (a to ∞) via `hypersimp` + `hyperexpand`.
  - Shifts index to start at 0; handles summands that vanish at the starting index (distinguishes identically-zero vs zero-only-at-start).

### [`products.py`](products.py)
Unevaluated and evaluated finite/infinite products.

- **`Product(ExprWithIntLimits)`** — unevaluated product.
  - `doit()` — evaluate the product; handles reversed ranges (upper < lower) by swapping bounds ±1 and inverting the term (`f → 1/f`).
  - `_eval_product(term, limits)` — core evaluation: polynomial factoring via `RisingFactorial`, direct expansion; delegates KroneckerDelta products to `delta.py`.
  - `is_convergent()` — convergence test for infinite products.
  - `reverse_order(*indices)` — flip selected iteration bounds: swaps upper↔lower (each shifted by 1) and inverts the term (`f → 1/f`) once per flipped index; even flips cancel the inversion so the function body is unchanged.
  - `_eval_rewrite_as_Sum()` — rewrite as `exp(Sum(log(f), ...))`.
  - `_eval_simplify`, `_eval_adjoint`, `_eval_conjugate`, `_eval_transpose`.
- `product(*args, **kwargs)` — convenience wrapper that calls `Product(...).doit()`.

---

## Algorithms and Helpers

### [`gosper.py`](gosper.py)
Gosper's algorithm for hypergeometric indefinite summation.

- `gosper_normal(f, g, n)` — compute Gosper's normal form of f/g.
- `gosper_term(f, n)` — compute Gosper's hypergeometric term.
- `gosper_sum(f, k)` — closed-form **definite** hypergeometric summation over finite ranges (returns result or None). Called as subroutine by `eval_sum_symbolic`; does **not** handle infinite series.
  - Falls back to symbolic `limit()` when substituting summation bounds into the antidifference produces an undefined (NaN) result.

### [`delta.py`](delta.py)
Simplification of sums and products containing Kronecker delta functions.

- `deltasummation(f, limit)` — evaluate summation with KroneckerDelta terms; simplifies deltas or returns piecewise results encoding delta conditions. Called by `eval_sum` only when KroneckerDelta is detected.
- `deltaproduct(f, limit)` — evaluate product with KroneckerDelta terms; splits additive expressions into delta and non-delta parts, handles integer bounds (direct sum) vs symbolic bounds (delegating to `deltasummation`).
- `_is_simple_delta`, `_has_simple_delta`, `_extract_delta`, `_expand_delta` — delta detection/extraction helpers.
- `_remove_multiple_delta`, `_simplify_delta` — simplification of delta products.

### [`guess.py`](guess.py)
Sequence identification: recurrence relations and generating functions.

- `find_simple_recurrence(v, A, N)` — detect linear recurrence relation from a sequence.
- `find_simple_recurrence_vector(l)` — find recurrence relation coefficient vector.
- `guess_generating_function(v, X, types)` — guess generating function (ogf, egf, lgf, hlgf, etc.).
- `guess_generating_function_rational(v, X)` — guess rational generating function.
- `rationalize(x, maxcoeff)` — identify rational number from float via continued fractions.
