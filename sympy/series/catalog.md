# Series Module Catalog

## Architecture Overview
The `series` module handles series expansions, limits, sequences, and asymptotic analysis.
- **Limit computation**: `limits.py` (entry point + heuristic evaluation of composite expressions) → `gruntz.py` (full Gruntz algorithm when heuristics bail out); `limitseq.py` (discrete sequence limits).
- **Series expansions**: `series.py` (thin wrapper), `formal.py` (formal power series), `fourier.py` (Fourier series).
- **Sequence algebra**: `sequences.py` — discrete sequence objects and term-wise arithmetic operations.
- **Utilities**: `order.py` (big-O), `residues.py`, `approximants.py`, `acceleration.py` (convergence acceleration & sequence extrapolation), `kauers.py` (finite differences).
- **Base classes**: `series_class.py` provides `SeriesBase`; `sequences.py` provides `SeqBase`.

---

## Limit Computation

### [`limits.py`](limits.py)
Limit computation entry point and heuristic evaluator. Tries fast heuristic decomposition of composite expressions (Mul, Add, Pow, Function) first; falls back to Gruntz algorithm when heuristics bail out.
- `limit(e, z, z0, dir)` — compute limit of expression; main entry point.
- `Limit(Expr)` — unevaluated limit object; `.doit()` orchestrates a multi-layered fallback chain: rewrite factorials → reciprocal substitution → gruntz → heuristics (on PoleError/ValueError) → `limit_seq()` (on NotImplementedError when z0 is ∞).
- `heuristics(e, z, z0, dir)` — fast-path that evaluates sub-expressions individually and reconstructs the result.
  - Returns `None` (bailing out to Gruntz) if any sub-limit is unevaluated (`Limit`), indeterminate (infinite with unknown finiteness), `NaN`, or if the reconstructed expression is `NaN`.
  - For infinite target points, substitutes reciprocal variable and re-evaluates at zero.

### [`gruntz.py`](gruntz.py)
Gruntz algorithm for computing limits via most-rapidly-varying (MRV) subexpression analysis.
- `gruntz(e, z, z0, dir)` — main entry; converts all limits to z→∞: finite z0 via z→z0+1/z (right) or z→z0−1/z (left), −∞ via z→−z.
- `limitinf(e, x)` — compute limit as x → ∞ using MRV sets.
- `mrv(e, x)` — find most rapidly varying subexpressions in continuous-variable limits.
- `compare(a, b, x)` — compare growth rates of two subexpressions (continuous limits, not discrete additive-term dominance).
- `rewrite(e, Omega, x, wsym)` — rewrite expression in terms of MRV representative.
- `mrv_leadterm(e, x)` — extract leading term from MRV expansion.
- `SubsSet(dict)` — internal substitution tracking dictionary.

### [`limitseq.py`](limitseq.py)
Limits of discrete sequences (n → ∞).
- `limit_seq(expr, n, trials)` — compute limit of a sequence expression using dominant-term analysis.
- `difference_delta(expr, n, step)` — discrete difference operator: expr(n+step) − expr(n).
- `dominant(expr, n)` — split expression into additive terms, compare pairwise growth rates via ratio limits, return the single fastest-growing term; returns `None` if two or more terms grow at the same asymptotic rate (comparable).

---

## Series Expansions

### [`series.py`](series.py)
Thin wrapper delegating to `expr.series()`.
- `series(expr, x, x0, n, dir)` — compute truncated series expansion.

### [`formal.py`](formal.py)
Formal Power Series (FPS) computation and representation.
- `fps(f, x, x0, dir, ...)` — main interface; returns `FormalPowerSeries`.
- `FormalPowerSeries(SeriesBase)` — formal power series object; `.polynomial()`, `.truncate()`, `.infinite`.
  - Arithmetic (`__add__`, `__sub__`, `__mul__`, `__neg__`): combines two FPS symbolically, reconciling coefficient sequences with different starting indices by folding extra lower-order terms into the independent term.
  - `.integrate(x)` — symbolic integration of the FPS, adjusting coefficient formula and adding constant of integration.
- `rational_algorithm(f, x, k, order, full)` — derive closed-form FPS coefficient formulas when f or derivatives are rational functions of x.
- `hyper_algorithm(f, x, k, order)` — hypergeometric FPS solving.
- `solve_de()`, `compute_fps()` — internal FPS computation pipeline.

### [`fourier.py`](fourier.py)
Fourier series decomposition into sine/cosine components.
- `fourier_series(f, limits)` — main interface; returns `FourierSeries`.
- `FourierSeries(SeriesBase)` — Fourier series object; `.a0`, `.an`, `.bn` (constant/cosine/sine coefficients).
- `fourier_cos_seq()`, `fourier_sin_seq()` — compute coefficient sequences.

---

## Discrete Sequences

### [`sequences.py`](sequences.py)
Discrete sequence representations and term-wise arithmetic on raw index-based sequences (not formal power series — FPS arithmetic is in `formal.py`).
- `sequence(seq, limits)` — factory function to create sequence objects.
- `SeqBase(Basic)` — abstract base for all sequences; `.gen`, `.interval`, `.start`, `.stop`, `.length`, `.coeff(pt)`, `._ith_point(i)`.
- `_ith_point(i)` — inherited point-indexing helper (parallel to `SeriesBase._ith_point`).
- `EmptySequence(SeqBase)` — singleton trivial/empty sequence; interval is the empty set.
- `SeqFormula(SeqExpr)` — formula-defined sequence (e.g., n²).
- `SeqPer(SeqExpr)` — periodic sequence from a tuple of repeating values.
- `SeqAdd(SeqExprOp)` — term-wise addition of sequences; constructor filters out `EmptySequence` args before processing.
- `SeqMul(SeqExprOp)` — term-wise multiplication of sequences; detects `EmptySequence` via interval intersection (no explicit filtering).
- `SeqAdd.reduce(args)` / `SeqMul.reduce(args)` — simplify by iterating pairs and calling `_add` / `_mul` rules.
- `SeqBase.find_linear_recurrence(n, d, gfvar)` — discover shortest linear recurrence for a sequence via matrix determinant/LU-solve; optionally computes the ordinary generating function (rational) from the recurrence coefficients and initial terms.
- Caveat: `SeqAdd` and `SeqMul` handle `EmptySequence` differently — `SeqAdd` explicitly filters trivial args; `SeqMul` relies on interval intersection returning empty set.

---

## Asymptotic Analysis & Utilities

### [`order.py`](order.py)
Big-O notation for asymptotic expansions.
- `Order(Expr)` (alias `O`) — represents O(f(x)) as x → a; containment checks, arithmetic, simplification.
- `Order.__new__` — constructor extracts the leading term via `as_leading_term`; selects the dominant monomial depending on the limit point (lowest power near 0, highest power near ∞).
- Supports multivariate O with per-variable limit points; default limit point is zero when none specified.

### [`residues.py`](residues.py)
Residue computation via Laurent series coefficient extraction.
- `residue(expr, x, x0)` — compute residue at x=x0 using series expansion with adaptive precision.

### [`approximants.py`](approximants.py)
Generator for consecutive Padé approximants from a raw coefficient list (not from recurrence relations or sequence objects).
- `approximants(l, X, simplify)` — generator taking a coefficient list `l`; yields successive rational approximants via continued-fraction recurrence.
- Before yielding each result, normalizes by computing LCM of all coefficient denominators and scaling numerator/denominator polynomials to clear fractions.
- Terminates early when the internal coefficient list becomes all zeros (no further approximants producible).

### [`acceleration.py`](acceleration.py)
Convergence acceleration and extrapolation methods for slowly converging series and sequences.
- `richardson(A, k, n, N)` — Richardson extrapolation; approximates series limit using weighted finite differences of N+1 consecutive partial sums.
- `shanks(A, k, n, m)` — Shanks transformation (sequence extrapolation); supports m-fold recursive application (m > 1) where each pass builds on a table of intermediate values propagated from the previous pass.

### [`kauers.py`](kauers.py)
Finite difference operators for symbolic sums.
- `finite_diff(expression, variable, increment)` — discrete analogue of differentiation.
- `finite_diff_kauers(sum)` — finite difference for Sum objects.

---

## Base Classes

### [`series_class.py`](series_class.py)
Abstract base class for all series representations.
- `SeriesBase(Expr)` — canonical abstract base for ordered mathematical expansions (not discrete sequences); `.interval`, `.start`, `.stop`, `.length`, `.term(pt)`.
- `_ith_point(i)` — canonical definition: compute position of i-th element; reverses direction (iterates backward from stop) when start is negative infinity. Drives `__iter__` and `__getitem__`.

### [`__init__.py`](__init__.py)
Package initialization; aggregates public API exports from all submodules.
