# Functions Module Catalog

Mathematical function classes (symbolic, unevaluated). Defines function behavior (eval, fdiff, rewrite), does NOT simplify expressions.

## Glossary

- **ultraspherical polynomials**: synonym for Gegenbauer polynomials (`gegenbauer` in `special/polynomials.py`).
- **unit step function**: the Heaviside function (`Heaviside` in `special/delta_functions.py`), NOT in `elementary/piecewise.py`.
- **generalized factorial function / upper incomplete gamma**: `uppergamma` in `special/gamma_functions.py`.
- **signum function**: `sign` class in `elementary/complexes.py`; Heaviside-to-sign rewriting is in `special/delta_functions.py`.
- **fdiff**: method on Function subclasses returning symbolic partial derivatives; lives alongside the class definition.
- **rewrite**: `_eval_rewrite_as_*` methods live on the source class, not the target class.

## Package Init

### [`__init__.py`](__init__.py)
Top-level namespace for the functions package. Re-exports all standard mathematical functions from submodules.
- Defines shorthand aliases: `ln = log` (natural logarithm alias).

## Submodules

### [`special/`](special/)
Special mathematical functions: gamma, error, Bessel, orthogonal polynomials, distributions, hypergeometric, zeta.

#### [`special/gamma_functions.py`](special/gamma_functions.py)
Gamma function family: complete, incomplete, polygamma, loggamma.
- `gamma` — complete gamma function Γ(x); evaluates special values; `fdiff` returns Γ(x)·ψ(x) for argindex 1, raises `ArgumentIndexError` otherwise. The `factorial` class (in `combinatorial/factorials.py`) handles n! computation and negative-integer edge cases.
- `lowergamma` — lower incomplete gamma function γ(s, x).
- `uppergamma` — upper incomplete gamma function Γ(s, x); `fdiff` uses Meijer G-function for derivative w.r.t. order parameter.
- `polygamma` — polygamma function ψ^(n)(z), includes `digamma` (n=0) and `trigamma` (n=1).
- `loggamma` — log-gamma function log Γ(x).

#### [`special/polynomials.py`](special/polynomials.py)
Orthogonal polynomial families. Base class `OrthogonalPolynomial`; each subclass has `eval` (special-value logic) and `fdiff`.
- `jacobi` — Jacobi polynomials P_n^(a,b)(x).
- `gegenbauer` — Gegenbauer (ultraspherical) polynomials C_n^a(x); `eval` handles special reductions (a=1/2→Legendre, a=1→Chebyshev U) and x=−1 branching.
- `chebyshevt`, `chebyshevu` — Chebyshev polynomials of first and second kind; `chebyshevt_root`, `chebyshevu_root` for roots.
- `legendre`, `assoc_legendre` — Legendre and associated Legendre polynomials P_n^m(x); `eval` converts negative order m to positive via factorial-ratio identity.
- `hermite` — Hermite polynomials H_n(x).
- `laguerre`, `assoc_laguerre` — Laguerre and generalized Laguerre polynomials.

#### [`special/delta_functions.py`](special/delta_functions.py)
Dirac delta and Heaviside step function classes.
- `DiracDelta` — Dirac delta distribution δ(x); `eval` simplifies for numeric args; `_eval_rewrite_as_Piecewise`, `_eval_rewrite_as_SingularityFunction`.
- `Heaviside` — unit step function H(x) with configurable value at origin (H0 parameter).
  - `_eval_rewrite_as_sign` — converts to `(sign(x)+1)/2` only when H0 is None or 1/2; returns None (no conversion) for H0=0.
  - `_eval_rewrite_as_Piecewise`, `_eval_rewrite_as_SingularityFunction`.

#### [`special/error_functions.py`](special/error_functions.py)
Error functions and related integrals (special cases of incomplete gamma).
- `erf`, `erfc`, `erfi`, `erf2`, `erfinv`, `erfcinv`, `erf2inv` — error function family.
- `erf2` — two-argument error function erf(x,y); carries its own conversion methods to `uppergamma`, `expint`, Fresnel, Meijer G, and hypergeometric forms.
- `Ei` — exponential integral Ei(x).
- `expint` — generalized exponential integral E_ν(z); `eval` handles analytic continuation by extracting winding number (branch factor) from z and applying distinct correction formulas for integer ν vs non-integer ν.
  - `_eval_nseries` — series expansion branches on order: ν=1 rewrites via trig integrals (Si/Ci), integer ν>1 rewrites via Ei, otherwise falls back to default.
- `li` — logarithmic integral li(z) = ∫₀ᶻ dt/ln(t); branch-cut aware `_eval_conjugate` excludes negative reals.
- `Li` — offset logarithmic integral Li(z) = li(z) − li(2). NOT the polylogarithm (that is `polylog` in `special/zeta_functions.py`).
- `Si` (sine integral), `Ci` (cosine integral), `Shi` (hyperbolic sine integral), `Chi` (hyperbolic cosine integral) — trigonometric/hyperbolic integrals (NOT elementary trig/hyperbolic functions from `elementary/`).
  - Each defines argument-transformation rules for negation and imaginary-unit rotation (`_minusfactor`, `_Ifactor`).
- `fresnels`, `fresnelc` — Fresnel integrals S(x), C(x).

#### [`special/bessel.py`](special/bessel.py)
Bessel functions and Airy functions.
- `besselj`, `bessely`, `besseli`, `besselk` — Bessel functions of first/second kind, modified.
- `hankel1`, `hankel2` — Hankel functions.
- `jn`, `yn` — spherical Bessel functions; `hn1`, `hn2` — spherical Hankel.
- `AiryBase` — abstract base for Airy functions (solutions of w''(z) − z·w(z) = 0); `as_real_imag` decomposes complex-argument evaluation into real/imaginary parts.
- `airyai`, `airybi`, `airyaiprime`, `airybiprime` — Airy functions and derivatives (subclasses of `AiryBase`).

#### [`special/hyper.py`](special/hyper.py)
Hypergeometric and Meijer G-functions.
- `TupleParametersBase` — base class for functions with tuple-valued arguments (e.g., numerator/denominator parameter lists); handles `_eval_derivative` by iterating over grouped parameters with tuple-indexed `fdiff`.
- `hyper` — generalized hypergeometric function pFq.
- `meijerg` — Meijer G-function.
- `HyperRep` subclasses — closed-form representatives for specific hypergeometric cases; each defines `_expr_small`, `_expr_big`, `_expr_small_minus`, `_expr_big_minus` classmethods for branch-region evaluation.
  - Named by the elementary function they represent: `HyperRep_atanh` (atanh(√z)/√z), `HyperRep_asin1`/`_asin2` (asin), `HyperRep_log1`/`_log2`, `HyperRep_power1`/`_power2`, `HyperRep_sqrts`, `HyperRep_cosasin`, `HyperRep_sinasin`.

#### [`special/elliptic_integrals.py`](special/elliptic_integrals.py)
Elliptic integral functions: `elliptic_k`, `elliptic_f`, `elliptic_e`, `elliptic_pi`.

#### [`special/zeta_functions.py`](special/zeta_functions.py)
Riemann zeta and related functions: `zeta`, `lerchphi`, `polylog` (Li_s(z)), `dirichlet_eta`, `stieltjes`.
- `polylog` — polylogarithm Li_s(z) = Σ z^k/k^s. `_eval_expand_func` reduces to elementary rational form for non-positive integer order s via iterated u·d/du on u/(1−u).

#### [`special/beta_functions.py`](special/beta_functions.py)
Euler beta function `beta(x, y)`. NOT probability beta distribution (that's `stats/`).

#### [`special/tensor_functions.py`](special/tensor_functions.py)
Discrete tensor index functions: `LeviCivita` (epsilon tensor), `KroneckerDelta`.

#### [`special/mathieu_functions.py`](special/mathieu_functions.py)
Mathieu functions — solutions to the Mathieu differential equation y'' + (a − 2q·cos(2x))·y = 0.
- `mathieus` — sine-type Mathieu solution S(a,q,z); odd parity (`eval` negates on z → −z). Reduces to sin(√a·z) when q=0.
- `mathieuc` — cosine-type Mathieu solution C(a,q,z); even parity (`eval` preserves sign on z → −z). Reduces to cos(√a·z) when q=0.
- `mathieusprime`, `mathieucprime` — derivatives of mathieus and mathieuc w.r.t. z.

#### [`special/spherical_harmonics.py`](special/spherical_harmonics.py)
Spherical harmonics (angular basis functions on the unit sphere): `Ynm` (complex), `Znm` (real).
- `Ynm` — Y_n^m(θ,φ); `eval` auto-simplifies angular symmetry relations (negated θ/φ) and negative order via conjugate identity.

#### [`special/bsplines.py`](special/bsplines.py)
B-spline basis functions constructed as Piecewise expressions via recursive Cox-de Boor algorithm.
- `bspline_basis(d, knots, n, x)` — n-th B-spline of degree d; recursively combines left/right branches with endpoint-closure propagation to ensure partition-of-unity.
- `bspline_basis_set` — returns the full set of `len(knots)-d-1` B-splines for given degree and knot vector.
- `_add_splines` — helper that combines two spline Piecewise expressions.

#### [`special/singularity_functions.py`](special/singularity_functions.py)
`SingularityFunction` — Macaulay bracket function <x−a>^n for beam/structural analysis; discontinuous, piecewise-like.
- `eval` — simplifies to `(x-a)**n*Heaviside(x-a)` for n≥0, `Derivative(DiracDelta(...))` for n<0.
- `fdiff` — derivative convention: n>0 applies power rule (n·<x−a>^(n−1)); n=0 or n=−1 decrements exponent without coefficient (step-like/distributional case).

### [`elementary/`](elementary/)
Elementary mathematical functions: trig, exponential, hyperbolic, piecewise, complex, rounding.

#### [`elementary/trigonometric.py`](elementary/trigonometric.py)
Trigonometric functions and their inverses.
- `sin`, `cos`, `tan`, `cot`, `sec`, `csc`, `sinc` — trig functions; `sin`/`cos` have `_eval_expand_trig` for multiple-angle expansion using Chebyshev T (odd n) and Chebyshev U (even n) polynomials from `special/polynomials.py`.
- `ReciprocalTrigonometricFunction` — base class for reciprocal trig forms (`sec`, `csc`, `cot`); delegates rewrites/eval to the underlying base function (e.g., cos for sec) and inverts.
  - `_rewrite_reciprocal` guards against trivial identity rewrites by returning None if the delegated result equals the original expression unchanged.
- `asin`, `acos`, `atan`, `acot`, `asec`, `acsc`, `atan2` — inverse trig.
- `_pi_coeff` — helper to normalize arguments by π.

#### [`elementary/exponential.py`](elementary/exponential.py)
Exponential and logarithmic functions: `exp`, `exp_polar`, `log`, `LambertW`.

#### [`elementary/hyperbolic.py`](elementary/hyperbolic.py)
Elementary hyperbolic functions and inverses (NOT hyperbolic integrals — those are `Chi`, `Shi` in `special/error_functions.py`).
- `sinh`, `cosh`, `tanh`, `coth` — primary hyperbolic functions; `eval` converts purely imaginary arguments to circular trig equivalents (e.g., sinh(ix)→i·sin(x), cosh(ix)→cos(x)).
  - `sinh`/`cosh` have `_eval_expand_trig` for addition-identity expansion; integer-multiple arguments n*t are split into t + (n−1)*t and recursively expanded.
- `ReciprocalHyperbolicFunction` — hyperbolic counterpart of `ReciprocalTrigonometricFunction` (in `trigonometric.py`); base class for `csch`, `sech`; same delegation/rewrite-guard pattern.
- `asinh`, `acosh`, `atanh`, `acoth`, `asech`, `acsch` — inverse hyperbolic functions.
  - `acosh.eval` — has a constant lookup table mapping known algebraic values (1/2, √3/2, etc.) to exact π-multiples; multiplies result by i when the argument is real (branch-cut convention).

#### [`elementary/complexes.py`](elementary/complexes.py)
Complex number component functions.
- `re`, `im` — real/imaginary parts.
- `sign` — signum function; `Abs` — absolute value.
- `arg`, `conjugate`, `transpose`, `adjoint`.
- `polar_lift`, `periodic_argument`, `principal_branch` — branch-cut handling.

#### [`elementary/piecewise.py`](elementary/piecewise.py)
`Piecewise` — piecewise-defined expressions with condition-expression pairs.
- `ExprCondPair` — single (expression, condition) pair.
- `_eval_integral`, `_eval_as_leading_term` — integration and series support.
- Caveats: Heaviside/DiracDelta rewrite TO Piecewise, but those classes live in `special/delta_functions.py`.

#### [`elementary/integers.py`](elementary/integers.py)
Rounding functions: `floor`, `ceiling`, `frac`.

#### [`elementary/miscellaneous.py`](elementary/miscellaneous.py)
Miscellaneous: `sqrt`, `Min`, `Max`, `IdentityFunction`.

### [`combinatorial/`](combinatorial/)
Combinatorial functions and number sequences.

#### [`combinatorial/factorials.py`](combinatorial/factorials.py)
Factorial-family functions: `factorial`, `subfactorial`, `factorial2`, `RisingFactorial`, `FallingFactorial`, `binomial`.
- `factorial` — n! computation class; `eval` returns ComplexInfinity for negative integers (consistent with gamma function poles), 1 for zero, and uses optimized algorithms for positive integers. Rewrites to `gamma(n+1)`.
- `RisingFactorial` — rising factorial (Pochhammer symbol) x·(x+1)·…·(x+k−1); rewrites to gamma ratio.
- `FallingFactorial` — descending product x·(x−1)·…·(x−k+1); `eval` handles ±Infinity with sign/parity logic for positive vs negative k, and polynomial-argument expansion via `poly_from_expr`.
- `binomial` — binomial coefficient C(n,k); `eval` handles integer, non-integer, and edge cases.

#### [`combinatorial/numbers.py`](combinatorial/numbers.py)
Combinatorial number sequences: `fibonacci`, `lucas`, `bernoulli`, `bell`, `harmonic`, `euler`, `catalan`, `genocchi`, `stirling`.
- `catalan` — Catalan number C_n = binomial(2n,n)/(n+1); `eval` returns gamma-based closed form for nonneg integers and negative non-integers; for negative integers returns 0 (n≤−2) or −1/2 (n=−1).
- `harmonic` — generalized harmonic number H(n,m) = Σ 1/k^m for k=1..n; `eval` handles n=∞ by returning NaN (m<0), ∞ (m≤1), or `zeta(m)` (m>1). Rewrites to `polygamma`.
  - `_eval_expand_func` — decomposes integer-shifted arguments: H(n+k) adds positive reciprocal terms, H(n−k) adds negative reciprocal terms; rational arguments expand via trigonometric digit-extraction sums.
- `stirling` — Stirling numbers S(n,k) of first or second kind; helpers `_stirling1`/`_stirling2` implement cached recursive computation with closed-form shortcuts for special k values (e.g., k=n−1, k=n−2, k=2).
