# Functions Module Catalog

Mathematical function classes (symbolic, unevaluated). Defines function behavior (eval, fdiff, rewrite), does NOT simplify expressions.

## Glossary

- **ultraspherical polynomials**: synonym for Gegenbauer polynomials (`gegenbauer` in `special/polynomials.py`).
- **unit step function**: the Heaviside function (`Heaviside` in `special/delta_functions.py`), NOT in `elementary/piecewise.py`.
- **generalized factorial function**: the gamma function Γ(n)=(n−1)!; `gamma` in `special/gamma_functions.py`. Its logarithm is `loggamma` in the same file.
- **upper incomplete gamma**: `uppergamma` in `special/gamma_functions.py`.
- **signum function**: `sign` class in `elementary/complexes.py`; Heaviside-to-sign rewriting is in `special/delta_functions.py`.
- **trigonometric integral** / **cosine integral** / **sine integral**: `Ci`, `Si` in `special/error_functions.py` — NOT the elementary trig functions in `elementary/trigonometric.py`.
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
- `loggamma` — log-gamma function log Γ(x); `eval` returns closed-form expressions for integer, half-integer (denominator=2), and general rational arguments.

#### [`special/polynomials.py`](special/polynomials.py)
Orthogonal polynomial families. Base class `OrthogonalPolynomial`; each subclass has `eval` (special-value logic) and `fdiff`.
- `jacobi` — Jacobi polynomials P_n^(a,b)(x).
- `gegenbauer` — Gegenbauer (ultraspherical) polynomials C_n^a(x); `eval` handles special reductions (a=1/2→Legendre, a=1→Chebyshev U) and x=−1 branching.
- `chebyshevt`, `chebyshevu` — Chebyshev polynomials of first and second kind; `chebyshevt_root`, `chebyshevu_root` for roots.
- `legendre`, `assoc_legendre` — Legendre and associated Legendre polynomials P_n^m(x) (the polynomial component of spherical harmonics); `eval` converts negative order m to positive via (-1)^m · factorial(m+n)/factorial(n-m) identity.
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
- `expint` — generalized exponential integral E_ν(z); `eval` simplifies non-positive integer orders and half-integer orders (checked via `(2*nu).is_Integer`) to expressions involving `uppergamma`, reducing to error functions at half-integers.
  - Also handles analytic continuation by extracting winding number (branch factor) from z and applying distinct correction formulas for integer ν vs non-integer ν.
  - `_eval_nseries` — series expansion branches on order: ν=1 rewrites via trig integrals (Si/Ci), integer ν>1 rewrites via Ei, otherwise falls back to default.
- `li` — logarithmic integral li(z) = ∫₀ᶻ dt/ln(t); branch-cut aware `_eval_conjugate` excludes negative reals.
- `Li` — offset logarithmic integral Li(z) = li(z) − li(2). NOT the polylogarithm (that is `polylog` in `special/zeta_functions.py`).
- `Si` (sine integral), `Ci` (cosine integral), `Shi` (hyperbolic sine integral), `Chi` (hyperbolic cosine integral) — trigonometric/hyperbolic integrals (NOT elementary trig/hyperbolic functions from `elementary/`).
  - Each defines argument-transformation rules for negation and imaginary-unit rotation (`_minusfactor`, `_Ifactor`).
- `FresnelIntegral` — base class for Fresnel integrals; `eval` extracts factors of −1 and I from the argument using a subclass `_sign` attribute (+1 for cosine, −1 for sine) to differentiate simplification of f(i·z).
- `fresnels`, `fresnelc` — Fresnel integrals S(x), C(x); subclasses of `FresnelIntegral`.

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
- `HyperRep` subclasses — closed-form representatives for specific hypergeometric cases; each defines `_expr_small`, `_expr_big`, `_expr_small_minus`, `_expr_big_minus` classmethods for branch-region evaluation (small/big × positive/negative argument).
  - Named by the elementary function they represent: `HyperRep_atanh`, `HyperRep_asin1`/`_asin2`, `HyperRep_log1`/`_log2`, `HyperRep_power1`/`_power2`, `HyperRep_cosasin`, `HyperRep_sinasin`.
  - `HyperRep_sqrts1` — represents ((1−√z)^{2a}+(1+√z)^{2a})/2; `_expr_big_minus` uses trig form with atan(√z).
  - `HyperRep_sqrts2` — represents √z·((1−√z)^{2a}−(1+√z)^{2a})/2; `_expr_big_minus` handles large negative-branch case with sin(2a·atan(√z)).

#### [`special/elliptic_integrals.py`](special/elliptic_integrals.py)
Elliptic integral functions: `elliptic_k`, `elliptic_f`, `elliptic_e`, `elliptic_pi`.

#### [`special/zeta_functions.py`](special/zeta_functions.py)
Riemann zeta and related functions: `zeta`, `lerchphi`, `polylog` (Li_s(z)), `dirichlet_eta`, `stieltjes`.
- `polylog` — polylogarithm Li_s(z) = Σ z^k/k^s. `_eval_expand_func` reduces to elementary rational form for non-positive integer order s via iterated u·d/du on u/(1−u).

#### [`special/beta_functions.py`](special/beta_functions.py)
Euler beta function `beta(x, y)`. NOT probability beta distribution (that's `stats/`).

#### [`special/tensor_functions.py`](special/tensor_functions.py)
Discrete tensor index functions for symbolic indices (NOT continuous distributions — those are in `delta_functions.py`).
- `LeviCivita` — Levi-Civita epsilon tensor ε_{i,j,...}.
- `KroneckerDelta` — discrete Kronecker delta δ_{i,j}; `eval` simplifies integer indices and enforces canonical ordering; supports fermi-level assumptions (above/below fermi).
  - `_eval_power` — idempotent for positive exponents (δ^n=δ); negative exponents ≠ −1 return 1/δ; exponent −1 falls through (returns None).

#### [`special/mathieu_functions.py`](special/mathieu_functions.py)
Mathieu functions — solutions to the Mathieu differential equation y'' + (a − 2q·cos(2x))·y = 0.
- `mathieus` — sine-type Mathieu solution S(a,q,z); odd parity (`eval` negates on z → −z). Reduces to sin(√a·z) when q=0.
- `mathieuc` — cosine-type Mathieu solution C(a,q,z); even parity (`eval` preserves sign on z → −z). Reduces to cos(√a·z) when q=0.
- `mathieusprime`, `mathieucprime` — derivatives of mathieus and mathieuc w.r.t. z.

#### [`special/spherical_harmonics.py`](special/spherical_harmonics.py)
Spherical harmonics (angular basis functions on the unit sphere): `Ynm` (complex), `Znm` (real).
- `Ynm` — Y_n^m(θ,φ), 4 args: (n, m, θ, φ); `eval` auto-simplifies angular symmetry relations (negated θ/φ) and negative order via conjugate identity (phase factor, no factorials). Polynomial component P_n^m delegated to `assoc_legendre` in `special/polynomials.py`.
  - `fdiff` supports differentiation w.r.t. angular args θ (argindex 3) and φ (argindex 4); raises `ArgumentIndexError` for discrete parameters n (1) and m (2).

#### [`special/bsplines.py`](special/bsplines.py)
B-spline basis functions constructed as Piecewise expressions via recursive Cox-de Boor algorithm.
- `bspline_basis(d, knots, n, x)` — n-th B-spline of degree d; recursively combines left/right branches with endpoint-closure propagation to ensure partition-of-unity.
- `bspline_basis_set` — returns the full set of `len(knots)-d-1` B-splines for given degree and knot vector.
- `_add_splines` — helper that combines two spline Piecewise expressions.

#### [`special/singularity_functions.py`](special/singularity_functions.py)
`SingularityFunction` — Macaulay bracket function <x−a>^n for beam/structural analysis; discontinuous, piecewise-like.
- `eval` — simplifies to `(x-a)**n*Heaviside(x-a)` for n≥0, `Derivative(DiracDelta(...))` for n<0.
- `fdiff` — derivative convention: n>0 applies power rule (n·<x−a>^(n−1)); n=0 or n=−1 decrements exponent without coefficient (step-like/distributional case); n=−2 (minimum) silently returns None (unhandled).

### [`elementary/`](elementary/)
Elementary mathematical functions: trig, exponential, hyperbolic, piecewise, complex, rounding.

#### [`elementary/trigonometric.py`](elementary/trigonometric.py)
Trigonometric functions and their inverses (NOT trigonometric integrals Si, Ci — those are in `special/error_functions.py`).
- `sin`, `cos`, `tan`, `cot`, `sec`, `csc`, `sinc` — trig functions; `sin`/`cos` have `_eval_expand_trig` for multiple-angle expansion using Chebyshev T (odd n) and Chebyshev U (even n) polynomials from `special/polynomials.py`.
- `ReciprocalTrigonometricFunction` — base class for reciprocal circular trig forms (`sec`, `csc`, `cot`); delegates rewrites to the underlying base function (e.g., cos for sec) and inverts.
  - `_rewrite_reciprocal` guards against trivial identity rewrites by returning None if the delegated result equals the original expression unchanged.
- `asin`, `acos`, `atan`, `acot`, `asec`, `acsc`, `atan2` — inverse trig; `eval` converts purely imaginary arguments to inverse hyperbolic equivalents (e.g., atan(ix)→i·atanh(x)).
- `_pi_coeff` — helper to normalize arguments by π.

#### [`elementary/exponential.py`](elementary/exponential.py)
Exponential and logarithmic functions: `exp`, `exp_polar`, `log`, `LambertW`.
- `log` — natural logarithm (and general-base logarithm). `eval` handles complex-domain decomposition:
  - Negative real args → π·i + log(|arg|); purely imaginary args → ±π·i/2 + log of the real coefficient.
  - Base conversion, rational factoring, and special-value shortcuts (0→ComplexInfinity, 1→0, e→1).
- `exp` — exponential function e^x; `_eval_refine` simplifies `exp(k·π·i)` under assumptions to 1, −1, i, or −i based on integer/half-integer parity of the coefficient.
- `exp_polar` — branch-aware polar exponential (does not wrap at 2π).
- `LambertW` — Lambert W function (product-log), the inverse of x·e^x.

#### [`elementary/hyperbolic.py`](elementary/hyperbolic.py)
Elementary hyperbolic functions and inverses (NOT hyperbolic integrals — those are `Chi`, `Shi` in `special/error_functions.py`).
- `sinh`, `cosh`, `tanh`, `coth` — primary hyperbolic functions; `eval` converts purely imaginary arguments to circular trig equivalents (e.g., sinh(ix)→i·sin(x), cosh(ix)→cos(x)).
  - `sinh`/`cosh` have `_eval_expand_trig` for addition-identity expansion; integer-multiple arguments n*t are split into t + (n−1)*t and recursively expanded.
- `ReciprocalHyperbolicFunction` — base class for reciprocal hyperbolic forms (`csch`, `sech`, `coth`); `eval` checks argument sign symmetry via `_is_even`/`_is_odd` parity flags, delegates to `_reciprocal_of.eval()`, and inverts the result.
- `asinh`, `acosh`, `atanh`, `acoth`, `asech`, `acsch` — inverse hyperbolic functions.
  - Odd-symmetry inverses (`asinh`, `atanh`, `acoth`, `acsch`): `eval` uses `_coeff_isneg` to detect negative leading coefficient → returns `−f(−arg)`.
  - `acosh.eval` — has a constant lookup table mapping known algebraic values (1/2, √3/2, etc.) to exact π-multiples; multiplies result by i when the argument is real (branch-cut convention).
  - `acsch.eval` — constant table for purely imaginary arguments (maps to π-fraction multiples of i); returns exact log expressions for ±1.

#### [`elementary/complexes.py`](elementary/complexes.py)
Complex number component functions.
- `re`, `im` — real/imaginary parts.
- `sign` — signum function; `Abs` — absolute value.
- `arg`, `conjugate`, `transpose`, `adjoint` — generic operator classes; function-specific conjugate/adjoint logic (`_eval_conjugate`) lives in each function's own file.
- `polar_lift`, `periodic_argument`, `principal_branch` — branch-cut handling.

#### [`elementary/piecewise.py`](elementary/piecewise.py)
`Piecewise` — piecewise-defined expressions with condition-expression pairs.
- `ExprCondPair` — single (expression, condition) pair.
- `_eval_integral`, `_eval_as_leading_term` — integration and series support.
- Caveats: Heaviside/DiracDelta rewrite TO Piecewise, but those classes live in `special/delta_functions.py`.

#### [`elementary/integers.py`](elementary/integers.py)
Rounding functions: `floor`, `ceiling`, `frac`.

#### [`elementary/miscellaneous.py`](elementary/miscellaneous.py)
Miscellaneous elementary functions and extrema.
- `sqrt` — principal square root (shorthand for `Pow(x, S.Half)`).
- `root(arg, n, k=0)` — principal or k-th nth-root of arg.
- `real_root(arg, n=None)` — returns the real nth-root of arg; when n is omitted, converts all `(-a)**(1/odd)` to `-(a**(1/odd))` via pattern-matching transform.
- `Min`, `Max` — symbolic minimum/maximum over arguments; base class `MinMaxBase`.
- `IdentityFunction` — identity function f(x)=x.

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
- `fibonacci` — Fibonacci numbers F(n) (0,1,1,2,3,5,8,...); also generates Fibonacci polynomials when called with two args. `eval` returns ∞ for n=∞.
- `lucas` — Lucas numbers L(n) (2,1,3,4,7,11,...); companion sequence to Fibonacci with initial values L₀=2, L₁=1. `eval` returns ∞ for n=∞; integer n delegates to `fibonacci(n+1)+fibonacci(n-1)`.
- `catalan` — Catalan number C_n = binomial(2n,n)/(n+1); `eval` returns gamma-based closed form for nonneg integers and negative non-integers; for negative integers returns 0 (n≤−2) or −1/2 (n=−1).
- `harmonic` — generalized harmonic number H(n,m) = Σ 1/k^m for k=1..n; `eval` handles n=∞ by returning NaN (m<0), ∞ (m≤1), or `zeta(m)` (m>1). Rewrites to `polygamma`.
  - `_eval_expand_func` — decomposes integer-shifted arguments: H(n+k) adds positive reciprocal terms, H(n−k) adds negative reciprocal terms; rational arguments expand via trigonometric digit-extraction sums.
- `genocchi` — Genocchi numbers G_n, the integer sequence with generating function 2t/(eᵗ+1); related to Bernoulli numbers via G_n = 2(1−2ⁿ)B_n.
  - Assumption predicates (`_eval_is_negative`, `_eval_is_positive`) determine sign of even-indexed terms by parity of n/2; odd-indexed G_n (n>1) are zero.
  - `_eval_is_prime` — only n=8 yields a prime (G_8=17); negatives are not considered prime (so G_6=−3 is excluded).
- `stirling` — Stirling numbers S(n,k) of first or second kind; helpers `_stirling1`/`_stirling2` implement cached recursive computation with closed-form shortcuts for special k values (e.g., k=n−1, k=n−2, k=2).
