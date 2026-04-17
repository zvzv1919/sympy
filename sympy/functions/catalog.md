# Functions Module Catalog

Mathematical function classes (symbolic, unevaluated). Defines function behavior (eval, fdiff, rewrite), does NOT simplify expressions.

## Glossary

- **ultraspherical polynomials**: synonym for Gegenbauer polynomials (`gegenbauer` in `special/polynomials.py`).
- **unit step function**: the Heaviside function (`Heaviside` in `special/delta_functions.py`), NOT in `elementary/piecewise.py`.
- **generalized factorial function / upper incomplete gamma**: `uppergamma` in `special/gamma_functions.py`.
- **signum function**: `sign` class in `elementary/complexes.py`; Heaviside-to-sign rewriting is in `special/delta_functions.py`.
- **fdiff**: method on Function subclasses returning symbolic partial derivatives; lives alongside the class definition.
- **rewrite**: `_eval_rewrite_as_*` methods live on the source class, not the target class.

## Submodules

### [`special/`](special/)
Special mathematical functions: gamma, error, Bessel, orthogonal polynomials, distributions, hypergeometric, zeta.

#### [`special/gamma_functions.py`](special/gamma_functions.py)
Gamma function family: complete, incomplete, polygamma, loggamma.
- `gamma` — complete gamma function Γ(x); evaluates special values. The `factorial` class (in `combinatorial/factorials.py`) handles n! computation and negative-integer edge cases.
- `lowergamma` — lower incomplete gamma function γ(s, x).
- `uppergamma` — upper incomplete gamma function Γ(s, x); `fdiff` uses Meijer G-function for derivative w.r.t. order parameter.
- `polygamma` — polygamma function ψ^(n)(z), includes `digamma` (n=0) and `trigamma` (n=1).
- `loggamma` — log-gamma function log Γ(x).

#### [`special/polynomials.py`](special/polynomials.py)
Orthogonal polynomial families. Base class `OrthogonalPolynomial`; each subclass has `eval` (special-value logic) and `fdiff`.
- `jacobi` — Jacobi polynomials P_n^(a,b)(x).
- `gegenbauer` — Gegenbauer (ultraspherical) polynomials C_n^a(x); `eval` handles special reductions (a=1/2→Legendre, a=1→Chebyshev U) and x=−1 branching.
- `chebyshevt`, `chebyshevu` — Chebyshev polynomials of first and second kind; `chebyshevt_root`, `chebyshevu_root` for roots.
- `legendre`, `assoc_legendre` — Legendre and associated Legendre polynomials.
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
- `expint` — generalized exponential integral E_n(x).
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
- `airyai`, `airybi`, `airyaiprime`, `airybiprime` — Airy functions and derivatives.

#### [`special/hyper.py`](special/hyper.py)
Hypergeometric and Meijer G-functions.
- `TupleParametersBase` — base class for functions with tuple-valued arguments (e.g., numerator/denominator parameter lists); handles `_eval_derivative` by iterating over grouped parameters with tuple-indexed `fdiff`.
- `hyper` — generalized hypergeometric function pFq.
- `meijerg` — Meijer G-function.
- `HyperRep` subclasses — closed-form representations of specific hypergeometric cases.

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
Mathieu functions: `mathieus`, `mathieuc`, `mathieusprime`, `mathieucprime`.

#### [`special/spherical_harmonics.py`](special/spherical_harmonics.py)
Spherical harmonics (angular basis functions on the unit sphere): `Ynm` (complex), `Znm` (real).
- `Ynm` — Y_n^m(θ,φ); `eval` auto-simplifies symmetry relations: negative order m, negated polar/azimuthal angles.

#### [`special/bsplines.py`](special/bsplines.py)
B-spline basis functions constructed as Piecewise expressions via recursive Cox-de Boor algorithm.
- `bspline_basis(d, knots, n, x)` — n-th B-spline of degree d; recursively combines left/right branches with endpoint-closure propagation to ensure partition-of-unity.
- `bspline_basis_set` — returns the full set of `len(knots)-d-1` B-splines for given degree and knot vector.
- `_add_splines` — helper that combines two spline Piecewise expressions.

#### [`special/singularity_functions.py`](special/singularity_functions.py)
`SingularityFunction` — generalized singularity function <x-a>^n for beam/structural analysis.

### [`elementary/`](elementary/)
Elementary mathematical functions: trig, exponential, hyperbolic, piecewise, complex, rounding.

#### [`elementary/trigonometric.py`](elementary/trigonometric.py)
Trigonometric functions and their inverses.
- `sin`, `cos`, `tan`, `cot`, `sec`, `csc`, `sinc` — trig functions.
- `asin`, `acos`, `atan`, `acot`, `asec`, `acsc`, `atan2` — inverse trig.
- `_pi_coeff` — helper to normalize arguments by π.

#### [`elementary/exponential.py`](elementary/exponential.py)
Exponential and logarithmic functions: `exp`, `exp_polar`, `log`, `LambertW`.

#### [`elementary/hyperbolic.py`](elementary/hyperbolic.py)
Elementary hyperbolic functions and inverses (NOT hyperbolic integrals — those are `Chi`, `Shi` in `special/error_functions.py`).
- `sinh`, `cosh`, `tanh`, `coth` — primary hyperbolic functions.
- `ReciprocalHyperbolicFunction` — base class for reciprocal forms (`csch`, `sech`); delegates rewrites to the underlying function via `_rewrite_reciprocal`, which guards against trivial identity rewrites by returning None if the result is unchanged.
- `asinh`, `acosh`, `atanh`, `acoth`, `asech`, `acsch` — inverse hyperbolic functions.

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
- `binomial` — binomial coefficient C(n,k); `eval` handles integer, non-integer, and edge cases.

#### [`combinatorial/numbers.py`](combinatorial/numbers.py)
Combinatorial number sequences: `fibonacci`, `lucas`, `bernoulli`, `bell`, `harmonic`, `euler`, `catalan`, `genocchi`.
