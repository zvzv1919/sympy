# sympy/functions — Catalog

> Part of [SymPy](../catalog.md). Elementary and special mathematical functions (trig, exponential, Bessel, combinatorial, etc.).

## Python Files

| File | Summary |
|------|----------|
| `__init__.py` | Top-level package init that imports and re-exports all public functions from the combinatorial, elementary, and special subpackages. |
| `combinatorial/__init__.py` | Subpackage init for combinatorial functions; imports the `factorials` and `numbers` modules. |
| `combinatorial/factorials.py` | Implements combinatorial factorial-related functions: `factorial`, `factorial2`, `subfactorial`, `RisingFactorial`, `FallingFactorial`, and `binomial`. |
| `combinatorial/numbers.py` | Implements combinatorial number sequences: Fibonacci, Lucas, harmonic, Bernoulli, Bell, Euler, Catalan, and Genocchi numbers/polynomials. |
| `elementary/__init__.py` | Subpackage init for elementary functions; imports complexes, exponential, hyperbolic, integers, trigonometric, and miscellaneous modules. |
| `elementary/complexes.py` | Implements complex-number operations: `re` (real part extraction), `im` (imaginary part extraction with partitioning of sum terms into three buckets: known real i-coefficients, non-real i-coefficients, and terms requiring `as_real_imag` expansion), `sign` (complex sign), `Abs` (absolute value), `conjugate`, `arg` (argument/phase), `polar_lift`, `periodic_argument`, `transpose`, `adjoint`, and related helpers. |
| `elementary/exponential.py` | Implements exponential and logarithmic functions (`exp`, `exp_polar`, `log`, `LambertW`) with series expansion support for the Gruntz limit algorithm. |
| `elementary/hyperbolic.py` | Implements hyperbolic functions (`sinh`, `cosh`, `tanh`, `coth`, `sech`, `csch`) and their inverses (`asinh`, `acosh`, `atanh`, `acoth`, `asech`, `acsch`). |
| `elementary/integers.py` | Implements rounding functions: `floor`, `ceiling`, and `frac` (fractional part). |
| `elementary/miscellaneous.py` | Implements miscellaneous elementary functions: `sqrt`, `cbrt`, `root`, `real_root`, `Min`, `Max`, and the `IdentityFunction`. |
| `elementary/piecewise.py` | Implements the `Piecewise` function for piecewise-defined expressions and the `piecewise_fold` simplification utility. |
| `elementary/trigonometric.py` | Implements trigonometric functions (`sin`, `cos`, `tan`, `cot`, `sec`, `csc`, `sinc`) and their inverses (`asin`, `acos`, `atan`, `acot`, `asec`, `acsc`, `atan2`). Includes base classes `TrigonometricFunction` and `ReciprocalTrigonometricFunction` that use delegation patterns to implement operations on reciprocal functions (e.g., `sec`, `csc`, `cot`); `ReciprocalTrigonometricFunction._rewrite_reciprocal` provides special handling for rewrite operations that suppresses results when the underlying paired function returns an unmodified expression. |
| `elementary/benchmarks/bench_exp.py` | Benchmark for `exp.subs()` performance on exponential expression substitution. |
| `special/__init__.py` | Subpackage init for special functions; imports gamma, error, zeta, tensor, delta, elliptic, beta, Mathieu, singularity, and polynomial modules. |
| `special/bessel.py` | Implements Bessel functions (`besselj`, `bessely`, `besseli`, `besselk`), Hankel functions, spherical Bessel functions (`jn`, `yn`), and Airy functions (`airyai`, `airybi`). |
| `special/beta_functions.py` | Implements the Euler beta function `B(x, y)` with differentiation, conjugation, and numerical evaluation support. |
| `special/bsplines.py` | Implements B-spline basis functions (`bspline_basis`, `bspline_basis_set`) as piecewise polynomials over a knot vector. |
| `special/delta_functions.py` | Implements the `DiracDelta` distribution and the `Heaviside` step function. |
| `special/elliptic_integrals.py` | Implements complete and incomplete elliptic integrals of the first (`elliptic_k`, `elliptic_f`), second (`elliptic_e`), and third kind (`elliptic_pi`). |
| `special/error_functions.py` | Implements error functions (`erf`, `erfc`, `erfi`, `erf2`, `erfinv`, `erfcinv`), exponential integrals (`Ei`, `expint`, `E1`, `li`, `Li`), trig integrals (`Si`, `Ci`, `Shi`, `Chi`), and Fresnel integrals (`fresnels`, `fresnelc`) with series expansion support via `taylor_term` methods. |
| `special/gamma_functions.py` | Implements the gamma function, upper/lower incomplete gamma functions, log-gamma, polygamma (`digamma`, `trigamma`), and related utilities. |
| `special/hyper.py` | Implements the generalized hypergeometric function `hyper` (pFq) and the Meijer G-function `meijerg`. Also contains `HyperRep_*` classes for closed-form representations of hypergeometric functions across different analytic continuation regions (small/large arguments, positive/negative real axis), with even/odd branch distinction for Riemann sheet winding numbers (via `_expr_big` and `_expr_big_minus` methods that handle even/odd `n` parameter). |
| `special/mathieu_functions.py` | Implements Mathieu functions (`mathieus`, `mathieuc`) and their derivatives (`mathieusprime`, `mathieucprime`) for the Mathieu differential equation. |
| `special/polynomials.py` | Implements special orthogonal polynomials: Jacobi, Gegenbauer, Chebyshev (T and U), Legendre, associated Legendre, Hermite, Laguerre, and associated Laguerre. |
| `special/singularity_functions.py` | Implements the `SingularityFunction` (Macaulay bracket notation) used in structural engineering beam analysis. |
| `special/spherical_harmonics.py` | Implements spherical harmonic functions `Ynm`, `Ynm_c` (conjugate), and `Znm` (real form). |
| `special/tensor_functions.py` | Implements the `KroneckerDelta` and `LeviCivita` (Levi-Civita symbol / `Eijk`) tensor-index functions. |
| `special/zeta_functions.py` | Implements the Riemann `zeta` function, `dirichlet_eta`, Lerch transcendent (`lerchphi`), `polylog`, and Stieltjes constants. |
| `special/benchmarks/bench_special.py` | Benchmark for spherical harmonics (`Ynm`) evaluation performance. |
