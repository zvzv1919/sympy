# sympy/series — Series & Limits

## Glossary

- **FPS** — Formal Power Series: a closed-form representation of power series coefficients.
- **MRV** — Most Rapidly Varying: the set of subexpressions that grow/decay fastest; central to the Gruntz algorithm.
- **RE** — Recurrence Equation: a relation expressing later sequence terms in terms of earlier ones.
- **Big-O / Order** — Asymptotic upper bound notation, `O(f(x))`, characterising truncation error in series.

---

## Limits

### limits.py
Top-level limit interface; dispatches to Gruntz algorithm with heuristic fast-paths.

- `limit(e, z, z0, dir)` — convenience wrapper that creates a `Limit` and calls `.doit()`.
- `heuristics(e, z, z0, dir)` — tries simple argument-wise limit evaluation before falling back to Gruntz.
- **`Limit`** (class, extends `Expr`) — unevaluated limit object.
  - `doit()` applies heuristics, rewrites factorials as gamma, attempts Gruntz, and falls back to `limit_seq` for sequences.
  - Handles `O(…)` terms and leading-term shortcuts for `Mul` expressions at infinity.

### gruntz.py
Implements the Gruntz algorithm for computing limits at infinity, based on Dominik Gruntz's PhD thesis.

- `gruntz(e, z, z0, dir)` — public entry point; reduces any limit to the x→∞ case then delegates to `limitinf`.
- `limitinf(e, x)` — core recursive limit-at-infinity computation via MRV lead-term analysis.
- `mrv(e, x)` — returns the MRV (most rapidly varying) subset of subexpressions and rewrites `e` in terms of them.
- `mrv_leadterm(e, x)` — extracts the leading term `(c0, e0)` of `e` in the MRV variable `w`.
- `rewrite(e, Omega, x, wsym)` — rewrites `e` in terms of a single MRV representative `w` going to zero.
- `sign(e, x)` — determines the sign of `e(x)` for sufficiently large `x`.
- `compare(a, b, x)` — compares growth rates of `a` vs `b` via `limitinf(log|a|/log|b|)`.
- `calculate_series(e, x, logx)` — computes at least one non-zero term of the series of `e`; common failure point.
- **`SubsSet`** (class) — dictionary mapping MRV expressions to dummy variables, tracking rewrite rules for correct substitution.
- `build_expression_tree(Omega, rewrites)` — orders MRV set for correct substitution sequence.

**Caveats:** Most algorithm bugs manifest in `rewrite()` or in SymPy's own series expansion used by `calculate_series`. Set `SYMPY_DEBUG=True` for detailed recursive trace output.

### limitseq.py
Limits of sequences at infinity, using iterated differencing (Kauers' algorithm).

- `limit_seq(expr, n, trials)` — finds the limit of an admissible term (rational functions, indefinite sums/products) as `n→∞` by repeated application of `difference_delta` and `dominant`.
- `difference_delta(expr, n, step)` — discrete difference operator: `expr(n+step) - expr(n)`.
- `dominant(expr, n)` — identifies the single most rapidly growing additive term; returns `None` if ambiguous.

---

## Series Expansion

### series.py
Thin wrapper around `Expr.series()`.

- `series(expr, x, x0, n, dir)` — calls `expr.series(x, x0, n, dir)`.

### series_class.py
Abstract base class for series objects (Fourier, FPS, etc.).

- **`SeriesBase`** (class, extends `Expr`) — provides `term(pt)`, iteration (`__iter__`), slicing (`__getitem__`), and abstract properties `interval`, `start`, `stop`, `length`.

### order.py
Big-O asymptotic notation.

- **`Order`** (class, extends `Expr`, aliased as `O`) — represents `O(f(x))` remainder terms.
  - Constructor auto-simplifies: extracts leading term, normalises power exponents, handles multivariate case.
  - `contains(expr)` — tests whether `expr` belongs to this Order class.
  - Supports arithmetic: power, substitution, derivative, conjugate, transpose.
  - Works at arbitrary limit points (0, ∞, or any constant), though mixing points raises `NotImplementedError`.

---

## Formal Power Series

### formal.py
Computes closed-form coefficient formulas for formal power series.

- `fps(f, x, x0, dir, hyper, order, rational, full)` — main entry point; returns a `FormalPowerSeries` object.
- `compute_fps(f, x, x0, dir, ...)` — orchestrates the two algorithms below, handling direction/centering transforms and `Add` decomposition.
- `rational_algorithm(f, x, k, order, full)` — derives coefficient formula when `f` or a derivative is rational; uses partial fractions (`apart`).
- `hyper_algorithm(f, x, k, order)` — generates a simple DE for `f`, converts to RE, and solves.
- `simpleDE(f, x, g, order)` — yields differential equations of increasing order (up to `order`) satisfied by `f` with rational-function coefficients.
- `solve_de(f, x, DE, order, g, k)` — solves a DE by trying hypergeometric RE conversion then constant-coefficient RE.
- `rsolve_hypergeometric(f, x, P, Q, k, m)` — solves RE of the form `Q(k)*a(k+m) - P(k)*a(k)` via scaling/shifting/integration transforms.
- `exp_re(DE, r, k)` / `hyper_re(DE, r, k)` — convert a DE into a recurrence equation (constant-coeff vs general).
- **`FormalPowerSeries`** (class, extends `SeriesBase`) — stores `(ak, xk, ind)` coefficient/power/independent-term sequences.
  - `polynomial(n)` / `truncate(n)` — truncated polynomial or series with `O(…)` term.
  - `integrate(x)` / `_eval_derivative(x)` — symbolic integration/differentiation of the FPS.
  - Supports `+`, `-`, `*` (scalar) between FPS objects.
  - `infinite` property — returns the `Sum(…)` representation.

Internal helpers: `_transformation_a/c/e`, `_apply_shift/scale/integrate`, `_compute_formula`, `rational_independent`.

---

## Fourier Series

### fourier.py
Fourier sine/cosine series computation and manipulation.

- `fourier_series(f, limits)` — computes Fourier series of `f` on a finite interval; exploits even/odd symmetry. Returns a `FourierSeries`.
- **`FourierSeries`** (class, extends `SeriesBase`) — stores `(a0, an, bn)` coefficient sequences.
  - `truncate(n)` — returns the first `n` non-zero terms.
  - `shift(s)` / `shiftx(s)` — translates value or argument: `f(x)+s`, `f(x+s)`.
  - `scale(s)` / `scalex(s)` — scales value or argument: `s*f(x)`, `f(s*x)`.
  - Supports `+`, `-` between `FourierSeries` of the same period.
- `fourier_cos_seq` / `fourier_sin_seq` — compute the cosine and sine coefficient sequences via integration.

---

## Sequences

### sequences.py
Symbolic sequence types and term-wise algebra.

- `sequence(seq, limits)` — factory that returns `SeqFormula` (expression-defined) or `SeqPer` (periodic tuple).
- **`SeqBase`** (abstract base) — defines interface: `coeff(pt)`, iteration, slicing, `find_linear_recurrence(n, d, gfvar)`, `coeff_mul`, `+`, `-`, `*`.
- **`SeqFormula`** (extends `SeqExpr`) — sequence whose n-th term is given by a symbolic formula.
- **`SeqPer`** (extends `SeqExpr`) — periodic sequence repeating a tuple of values.
- **`EmptySequence`** (singleton) — the empty sequence; identity for `+`, absorbing for `*`.
- **`SeqAdd`** / **`SeqMul`** (extend `SeqExprOp`) — represent unevaluated term-wise addition/multiplication of sequences; auto-reduce when constituent types know how to combine.

---

## Convergence Acceleration

### acceleration.py
Extrapolation methods to accelerate slowly converging series/sequences.

- `richardson(A, k, n, N)` — Richardson extrapolation: approximates `lim A(k)` using terms `A(n)` through `A(n+N+1)`.
- `shanks(A, k, n, m)` — Shanks transformation (optionally `m`-fold recursive) for accelerating alternating/slowly-converging series.

---

## Approximants

### approximants.py
Padé approximant generator for coefficient lists.

- `approximants(l, X, simplify)` — yields consecutive Padé rational approximants for a power series given as a list of coefficients. The final yielded value (when the generator exhausts) is the rational generating function, if one exists.

---

## Residues

### residues.py
Residue computation via series expansion.

- `residue(expr, x, x0)` — finds the residue of `expr` at `x = x0` (coefficient of `1/(x-x0)` in the Laurent expansion). Uses `nseries` with progressively more terms.

---

## Finite Differences (Kauers)

### kauers.py
Finite difference utilities for polynomial expressions and sums.

- `finite_diff(expression, variable, increment)` — forward difference: `f(x + increment) - f(x)`.
- `finite_diff_kauers(sum)` — given a `Sum` object, returns the single-step difference `S(n+1) - S(n)` by substituting upper bounds.

---

## Package Initialization

### \_\_init\_\_.py
Re-exports the public API: `Order` (aliased as `O`), `limit`, `Limit`, `gruntz`, `series`, `approximants`, `residue`, `EmptySequence`, `SeqPer`, `SeqFormula`, `sequence`, `SeqAdd`, `SeqMul`, `fourier_series`, `fps`, `difference_delta`, `limit_seq`.

---

## Benchmarks

### benchmarks/bench_limit.py
Benchmark: `limit(1/x, x, oo)`.

### benchmarks/bench_order.py
Benchmark: `Add` of 1000 power terms with an `O(x^1001)` term.
