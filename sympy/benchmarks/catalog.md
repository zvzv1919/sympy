# sympy/benchmarks — Performance Benchmarks

## Package Init

### `__init__.py`
Empty package marker.

## Benchmark Suites

### `bench_meijerint.py`
Benchmarks Meijer G-function integration and integral-transform performance across ~170 symbolic expressions.

- **Setup**: defines shorthand aliases (`LT`, `FT`, `MT`, `IFT`, `ILT`, `IMT`) for the six integral transforms, plus symbolic constants with various assumptions (positive, real, integer, etc.).
- `normal(x, mu, sigma)` / `exponential(x, rate)` — PDF helpers used to build statistical-moment integrals.
- `E(expr)` — computes the expected value of `expr` over joint exponential × normal distributions via `meijerg=True` integration (both orderings).
- `bench` (list) — the main benchmark payload; string expressions covering:
  - Mellin transforms of powers, exponentials, logarithms, error functions, and various Bessel functions.
  - Laplace transforms of elementary and special functions.
  - Fourier / inverse-Fourier transforms.
  - Moments (0th–3rd) of normal, exponential, beta, chi, chi-squared, Dagum, F, Rice, and Laplace distributions.
  - Bessel-product and oscillatory integrals.
  - `hyperexpand` / `combsimp` of complicated Meijer G results.
  - Mellin / inverse-Mellin / Laplace transforms of special functions (`E1`, `Si`, `Ci`, `Shi`, `Chi`, `expint`).
  - Definite and indefinite integrals involving special functions.
- `__main__` block: iterates `bench`, `exec`-ing each string with cache clearing, then prints timings sorted slowest-first.

**Caveats**: Uses `exec("from sympy import *")` to hide the wildcard import from linters. Every benchmark expression is stored as a string and `exec`-ed at runtime.

### `bench_symbench.py`
A suite of symbolic-computation micro-benchmarks (R1–R11, S1) drawn from standard CAS benchmark suites.

- `bench_R1()` — deeply nested complex function evaluation (`f(f(…f(i/2)…))`, 10 levels), extracts real part.
- `bench_R2()` — recursive Hermite polynomial expansion up to degree 15.
- `bench_R3()` — repeated symbolic equality checks (`f == f`, 10 iterations).
- `bench_R5()` — symbolic expression blowup via repeated sums/products, then deduplication.
- `bench_R6()` — `simplify` over 100 trigonometric rational expressions.
- `bench_R7()` — 10 000 random-point substitutions into a degree-24 polynomial.
- `bench_R8()` — right-endpoint Riemann sum (10 000 rectangles) using symbolic arithmetic.
- `_bench_R9()` — `factor(x^20 - pi^5 * y^20)` (disabled by default, prefixed with `_`).
- `bench_R10()` — symbolic `srange(-pi, pi, 1/10)` list construction.
- `bench_R11()` — generate 1 000 random complex numbers.
- `bench_S1()` — expand `(x+y+z+1)^7 * ((x+y+z+1)^7 + 1)`.
- `__main__` block: runs selected benchmarks, prints name + docstring + elapsed time.

**Caveats**: `bench_R4` is a no-op stub (SymPy lacks the required `Tuples` feature). `_bench_R9` and `bench_S1` are commented out in the default run list.
