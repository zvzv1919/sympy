# sympy/series — Catalog

> Part of [SymPy](../catalog.md). Series expansions, limits (Gruntz algorithm), formal power series, Fourier series, and sequences.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package entry point; imports and re-exports the public API (limit, series, Order, fourier_series, fps, sequences, etc.). |
| `acceleration.py` | Convergence acceleration methods for series and sequences, including Richardson extrapolation and the Shanks transformation. |
| `approximants.py` | Generator for consecutive Pade approximants of a series, also usable for computing rational generating functions. Internally performs iterative coefficient-list inversion; terminates early (returns from the generator) when the remaining coefficient list consists entirely of zeros. |
| `formal.py` | Formal power series (FPS) implementation, including algorithms for computing coefficient formulas (rational, hypergeometric, simpleDE) and the `FormalPowerSeries` class. Also contains arithmetic operator overloads (add, subtract, multiply) on `FormalPowerSeries` objects, with logic for aligning mismatched coefficient starting indices (extracting gap terms into the independent/constant part) when combining two series expansions. |
| `fourier.py` | Fourier series representation and computation, providing the `FourierSeries` class and the `fourier_series` constructor with truncation, shifting, and scaling operations. |
| `gruntz.py` | Implementation of the Gruntz algorithm for computing symbolic limits. Contains the `gruntz()` entry-point function, which first converts all limit problems to z→∞ form: finite-point directional limits are transformed via variable substitution (e.g., `z0 - 1/z` for left-sided, `z0 + 1/z` for right-sided), and -∞ limits use a sign flip. The core algorithm then compares most-rapidly-varying (MRV) subexpressions, rewrites the expression in terms of those subexpressions, and performs series expansion. See `limits.py` for the public `limit()` dispatcher that calls `gruntz()`. |
| `kauers.py` | Finite difference operators for polynomials and sums, providing `finite_diff` and `finite_diff_kauers` utilities. |
| `limits.py` | Public `limit` function and `Limit` class for computing limits of expressions at a point. Applies fast heuristics first, then delegates to `gruntz()` in `gruntz.py` for the full algorithm. Does **not** itself perform the directional substitution for finite-point limits (left/right conversion to z→∞)—that transformation lives in `gruntz.py`. |
| `limitseq.py` | Limits of sequences, providing the `difference_delta` discrete difference operator and `limit_seq` for computing limits of sequences as n tends to infinity. |
| `order.py` | The `Order` (big-O) class representing the limiting behavior of a function, used to track truncation error in series expansions. |
| `residues.py` | Computes the residue of an expression at a point via Laurent series expansion, supporting the Residue Theorem. Uses an iterative retry loop with geometrically increasing expansion orders (0, 1, 2, 4, …, 32); includes a workaround that skips and retries when the series result contains only a truncation/Order term and all concrete terms vanish to zero. |
| `sequences.py` | Sequence classes (`SeqBase`, `SeqFormula`, `SeqPer`, `SeqAdd`, `SeqMul`, `EmptySequence`) and the `sequence` convenience constructor for defining and manipulating symbolic sequences. Handles raw term-wise sequence operations; does **not** handle series-level arithmetic like combining two infinite series expansions (that logic lives in `formal.py`). |
| `series.py` | Thin wrapper providing the top-level `series` function, which delegates to `Expr.series()` for Taylor/Laurent expansion. Contains no edge-case handling for vanishing terms or truncation—see `residues.py` for iterative Laurent expansion with retry logic, and `approximants.py` for coefficient-list processing with early termination. |
| `series_class.py` | Abstract base class `SeriesBase` for series representations, defining the common interface (interval, start, stop, length, term) used by `FourierSeries` and `FormalPowerSeries`. |
| `benchmarks/bench_limit.py` | Benchmark for the `limit` function, timing `limit(1/x, x, oo)`. |
| `benchmarks/bench_order.py` | Benchmark for `Order` addition, timing the creation of a large sum with an `O(x^1001)` term. |
