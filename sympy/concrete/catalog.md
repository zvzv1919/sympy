# sympy/concrete — Concrete Mathematics (Sums & Products)

Symbolic summation, product evaluation, Gosper's hypergeometric algorithm, Kronecker delta handling, and sequence-guessing heuristics.

---

## Summation

### `summations.py`

The primary summation engine — defines `Sum`, `summation()`, and internal evaluation routines.

- **`class Sum(AddWithLimits, ExprWithIntLimits)`** — unevaluated symbolic sum with Karr-convention semantics (upper limit inclusive, empty-sum rules, reversed-range negation).
  - `doit()` — drives evaluation; delegates to `eval_sum`, falls back to zeta-function matching.
  - `eval_zeta_function()` — pattern-matches `(w*i+y)**(-z)` to return `Piecewise` with `zeta(s,q)`.
  - `is_convergent()` / `is_absolutely_convergent()` — battery of convergence tests: divergence, p-series, root, alternating series, comparison (log variants), integral, Dirichlet.
  - `euler_maclaurin(m, n, eps)` — Euler–Maclaurin approximation returning `(approximation, error_estimate)`.
  - `reverse_order(*indices)` — flip limit direction with sign change per Karr convention.
- **`summation(f, *symbols)`** — convenience wrapper: constructs `Sum` and calls `doit(deep=False)`.
- **`eval_sum(f, limits)`** — top-level dispatcher: handles zeros, constant terms, `Piecewise`, `KroneckerDelta`, then tries direct/symbolic/hyper evaluation.
- `eval_sum_direct(expr, limits)` — brute-force unrolling for small finite ranges (< 100 terms).
- **`eval_sum_symbolic(f, limits)`** — symbolic evaluation pipeline:
  - Linearity (factor out constants).
  - Telescoping via `telescopic()`.
  - Faulhaber's formula for polynomial terms (`i**n`).
  - Geometric series detection.
  - Gosper summation fallback.
  - Hypergeometric fallback via `eval_sum_hyper`.
- `_eval_sum_hyper(f, i, a)` / `eval_sum_hyper(f, i_a_b)` — convert summand to `hyper()` and `hyperexpand`; returns `Piecewise` with convergence conditions.
- `telescopic(L, R, limits)` / `telescopic_direct(L, R, n, limits)` — detect and evaluate telescoping sums.

---

## Products

### `products.py`

Symbolic product evaluation — mirrors the summation API.

- **`class Product(ExprWithIntLimits)`** — unevaluated symbolic product with Karr convention (reversed range → reciprocal).
  - `doit()` — evaluates via `_eval_product`; applies `powsimp`.
  - `_eval_product(term, limits)` — case analysis:
    - Constant term, single-point, KroneckerDelta.
    - Finite integer range → explicit `Mul`.
    - Polynomial → factored via `RisingFactorial`.
    - Add/Mul/Pow decomposition.
    - Nested `Product` denesting.
  - `_eval_rewrite_as_Sum` — rewrite as `exp(Sum(log(f)))`.
  - `is_convergent()` — reduces to convergence of `Sum(log(f))`.
  - `reverse_order(*indices)` — flip limits, invert function.
- **`product(*args)`** — convenience wrapper: `Product(...).doit(deep=False)`.

---

## Base Classes (Expressions with Limits)

### `expr_with_limits.py`

Foundation for any expression carrying `(var, lower, upper)` limit tuples — shared by `Sum`, `Product`, and `Integral`.

- **`_process_limits(*symbols)`** — normalise heterogeneous limit specs (bare symbols, tuples, `Idx`, `Interval`) into canonical `Tuple(symbol, lower, upper)` form; also tracks orientation.
- **`class ExprWithLimits(Expr)`** — base providing:
  - Properties: `function`, `limits`, `variables`, `free_symbols`, `is_number`.
  - `as_dummy()` — replace dummy variables with fresh `Dummy` symbols to expose scoping.
  - `_eval_subs()` — substitution respecting dummy-variable scoping rules.
  - Constructor denests nested calls of the same type.
- **`class AddWithLimits(ExprWithLimits)`** — orientation-aware variant (multiplies function by ±1); adds `_eval_adjoint`, `_eval_conjugate`, `_eval_transpose`, `_eval_factor`, `_eval_expand_basic`.

### `expr_with_intlimits.py`

Integer-specific limit operations for `Sum` and `Product`.

- **`class ReorderError(NotImplementedError)`** — raised when dependent limits cannot be swapped.
- **`class ExprWithIntLimits(ExprWithLimits)`**:
  - `change_index(var, trafo, newvar)` — linear index substitution `x → ax+b` (only `a = ±1` for numeric coefficients).
  - `index(x)` — return position of dummy variable `x` in limits list.
  - `reorder(*arg)` — permute limit tuples by variable name or index pairs.
  - `reorder_limit(x, y)` — swap two independent limit tuples (raises `ReorderError` if limits have free-symbol dependencies).

---

## Kronecker Delta Handling

### `delta.py`

Specialised evaluation of sums and products whose summand/factor contains `KroneckerDelta`.

- **`deltasummation(f, limit, no_piecewise=False)`** — sum involving KroneckerDelta: extracts the delta, solves for the substitution value, returns `Piecewise` (or direct substitution when `no_piecewise=True`).
- **`deltaproduct(f, limit)`** — product involving KroneckerDelta: recursively splits Add terms, applies delta-extraction and simplification.
- `_extract_delta(expr, index)` — pull out a simple `KroneckerDelta` from a `Mul`, returning `(delta, remainder)`.
- `_expand_delta(expr, index)` — distribute a `Mul` over the first `Add` containing a simple delta.
- `_has_simple_delta` / `_is_simple_delta` — predicates: delta is "simple" when it constrains the index to a single value (linear in the index).
- `_remove_multiple_delta(expr)` — collapse redundant delta products by solving the implied equations.
- `_simplify_delta(expr)` — rewrite delta indices into canonical form.

> **Caveat:** All public and private functions are `@cacheit`-decorated; cache invalidation follows SymPy's global cache policy.

---

## Gosper's Algorithm

### `gosper.py`

Gosper's algorithm for closed-form hypergeometric summation.

- **`gosper_sum(f, k)`** — top-level entry point. Given a hypergeometric term `f` and limits `(k, a, b)`, returns the closed-form sum or `None`. Handles both definite and indefinite summation.
- **`gosper_term(f, n)`** — compute the "Gosper term" `g_n` satisfying `g(n+1) − g(n) = f(n)`. Obtains the normal form, determines candidate polynomial degree, then solves the resulting linear system.
- **`gosper_normal(f, g, n)`** — Gosper's rational normal form: factor `f(n)/g(n)` into `Z·A(n)·C(n+1) / (B(n)·C(n))` with the three coprimality conditions. Returns `(Z*A, B, C)` as polynomials (or expressions when `polys=False`).

---

## Sequence Guessing

### `guess.py`

Heuristic algorithms for identifying recurrence relations and generating functions from numerical sequences.

- **`find_simple_recurrence(v, A, N)`** — detect a linear recurrence with constant coefficients from a list of rational terms; returns a symbolic expression like `a(n+2) − a(n+1) − a(n)`.
- `find_simple_recurrence_vector(l)` — internal: returns the coefficient vector of the recurrence (used by the above and by generating-function guessers).
- **`rationalize(x, maxcoeff=10000)`** — approximate a float/mpf as a rational via continued fractions; stops when a partial quotient exceeds `maxcoeff`.
- **`guess_generating_function(v, X, types, maxsqrtn)`** — try to guess a generating function for a rational sequence. Returns a dict keyed by type. Supported types:
  - `ogf` — ordinary generating function.
  - `egf` — exponential generating function.
  - `lgf` / `hlgf` — logarithmic / hyperbolic-logarithmic g.f.
  - `lgdogf` / `lgdegf` — logarithmic derivatives of ogf / egf.
  - Also detects n-th roots of rational functions (up to `maxsqrtn`).
- `guess_generating_function_rational(v, X)` — sub-routine: guess a purely rational g.f. using the recurrence vector as denominator.

---

## Package Init

### `__init__.py`

Re-exports `Product`, `product`, `Sum`, `summation`.
