# sympy/series — Catalog

> Part of [SymPy](../catalog.md). Series expansions, limits (Gruntz algorithm), formal power series, Fourier series, and sequences.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package entry point; imports and re-exports the public API (limit, series, Order, fourier_series, fps, sequences, etc.). |
| `acceleration.py` | Convergence acceleration methods for series and sequences, including Richardson extrapolation and the Shanks transformation. |
| `approximants.py` | Generator for consecutive Pade approximants of a series, also usable for computing rational generating functions. |
| `formal.py` | Formal power series (FPS) of continuous functions, including algorithms for computing closed-form coefficient formulas (rational, hypergeometric, simpleDE) and the `FormalPowerSeries` class. Contains the concrete `_eval_term` implementation that evaluates individual terms by combining the coefficient sequence (`ak`) with the basis (`xk`), and uses range-based logic to selectively include singular/independent (`ind`) addends based on their power of x relative to the requested index. Also provides truncation, polynomial extraction, and derivative computation for FPS. Deals with representing analytic functions as power series, not with discrete sequence recurrences or generating functions. |
| `fourier.py` | Fourier series representation and computation, providing the `FourierSeries` class and the `fourier_series` constructor with truncation, shifting, and scaling operations. |
| `gruntz.py` | Implementation of the Gruntz algorithm for computing symbolic limits, based on comparing most-rapidly-varying subexpressions and series expansion in terms of those subexpressions. |
| `kauers.py` | Finite difference operators for polynomials and sums, providing `finite_diff` and `finite_diff_kauers` utilities. |
| `limits.py` | Public `limit` function and `Limit` class for computing limits of expressions at a point, dispatching to heuristics and the Gruntz algorithm. |
| `limitseq.py` | Limits of sequences at infinity. Provides `limit_seq` for computing sequence limits, `difference_delta` for discrete differences, and `dominant` for finding the most dominating term in a sum expression by pairwise ratio comparison (returns `None` when terms have comparable asymptotic growth rates). |
| `order.py` | The `Order` (big-O) class representing the limiting behavior of a function, used to track truncation error in series expansions. |
| `residues.py` | Computes the residue of an expression at a point via Laurent series expansion, supporting the Residue Theorem. |
| `sequences.py` | Discrete *sequence* classes (`SeqBase`, `SeqFormula`, `SeqPer`, `SeqAdd`, `SeqMul`, `EmptySequence`) and the `sequence` constructor. `SeqBase` is the base for sequences only (not series); provides coefficient access, arithmetic ops, and `find_linear_recurrence` which discovers the shortest linear recurrence relation from initial terms (via matrix determinant/LU-solve) and optionally constructs the rational ordinary generating function (OGF) numerator and denominator from the recurrence coefficients. |
| `series.py` | Thin wrapper providing the top-level `series` function, which delegates to `Expr.series()` for Taylor/Laurent expansion. |
| `series_class.py` | Abstract base class `SeriesBase` for *series* representations (parent of `FourierSeries`, `FormalPowerSeries`), defining the common abstract interface (interval, start, stop, length, `_eval_term` stub) and the `_ith_point` helper that maps iteration index to series index when lower bounds are negative infinity. Does not contain any concrete term-evaluation logic — subclasses in `formal.py` and `fourier.py` implement `_eval_term`. |
| `benchmarks/bench_limit.py` | Benchmark for the `limit` function, timing `limit(1/x, x, oo)`. |
| `benchmarks/bench_order.py` | Benchmark for `Order` addition, timing the creation of a large sum with an `O(x^1001)` term. |
