# sympy/functions — Mathematical Functions

Central library of symbolic mathematical functions: elementary (trig, exp, log, etc.), combinatorial (factorials, binomials, number sequences), and special (Bessel, gamma, error functions, orthogonal polynomials, etc.).

## Root-Level Files

### `__init__.py`
Re-exports all public functions from the three sub-packages (combinatorial, elementary, special). Also defines `ln = log`.

---

## Combinatorial Functions

### `factorials.py`
Factorial, double factorial, rising/falling factorials, and binomial coefficients.

- **`CombinatorialFunction`** — base class; `_eval_simplify` delegates to `combsimp`.
- **`factorial`** — `n!` via Prime-Swing algorithm for large n, lookup table for small n, GMPY when available. Rewrites as `gamma`, `Product`.
- **`subfactorial`** — derangement count `!n`; cached recursive evaluation. Rewrites as `uppergamma`.
- **`factorial2`** — double factorial `n!!`; handles negative odd integers. Rewrites as `gamma` with a `Piecewise`.
- **`MultiFactorial`** — stub base class (unused).
- **`RisingFactorial` (`rf`)** — Pochhammer symbol `x(x+1)...(x+k-1)`. When `x` is a polynomial of degree > 1, factors through each shifted root. Rewrites as `gamma`, `FallingFactorial`, `factorial`, `binomial`.
- **`FallingFactorial` (`ff`)** — `x(x-1)...(x-k+1)`. Mirrors `RisingFactorial` logic.
- **`binomial`** — `C(n,k)` via prime-sieve algorithm for integer args; generalized for non-integer `n`. Rewrites as `factorial`, `gamma`, `FallingFactorial`.

### `numbers.py`
Integer/rational number sequences commonly used in combinatorics and series.

- **`fibonacci`** / **`lucas`** — Fibonacci numbers/polynomials and Lucas numbers. Uses `mpmath.ifib` for fast integer computation; polynomial variant via recurrence memoization.
- **`bernoulli`** — Bernoulli numbers/polynomials via Ramanujan's formula (cuts 2/3 of terms). Falls back to `mpmath.bernfrac` for n > 500. Specialized mod-6 caching scheme.
- **`bell`** — Bell numbers, Bell polynomials, and incomplete (partial) Bell polynomials of the second kind. Dobinski's formula rewrite.
- **`harmonic`** — Generalized harmonic numbers `H(n,m)`. Supports expansion at rational arguments via digamma/trigamma identities. Memoized per-order generating functions.
- **`euler`** — Euler numbers via `mpmath.eulernum`.
- **`catalan`** — Catalan numbers; continuous generalization via `gamma`. Rewrites as `binomial`, `gamma`, `hyper`, `Product`.
- **`genocchi`** — Genocchi numbers `G_n = 2(1 - 2^n) B_n`.
- **`nP`** / **`nC`** / **`nT`** — Multiset-aware counting of permutations, combinations, and partitions. Supports replacement mode.
- **`stirling`** — Stirling numbers of the first and second kind; reduced Stirling numbers.
- **`_AOP_product`** — All-one polynomial product for multiset combination counting.

---

## Elementary Functions

### `exponential.py`
Exponential and logarithmic functions.

- **`ExpBase`** — abstract base for `exp`/`exp_polar`. Handles `as_numer_denom`, power expansion, rationality checks.
- **`exp_polar`** — polar exponential that doesn't wrap at `2π`. `is_comparable = False`.
- **`exp`** — `e^x`. Auto-simplifies `exp(log(x))→x`, `exp(a+b)→exp(a)*exp(b)` when sub-terms simplify, integer multiples of `iπ`. Provides `_eval_nseries` critical for the Gruntz limit algorithm.
- **`log`** — natural logarithm with optional base. Auto-simplifies `log(exp(x))→x`, `log(p/q)→log(p)-log(q)`, perfect-power extraction. `_eval_expand_log` splits products/powers respecting sign.
- **`LambertW`** — Lambert W function `W(z)` (inverse of `w·e^w`). Multi-valued with branch index `k`. Evaluates special points on `k=0` and `k=-1` branches.

### `trigonometric.py`
Full suite of trigonometric and inverse trigonometric functions (~2690 lines).

- **`TrigonometricFunction`** — base class with algebraicity/rationality checks and complex expansion.
- **`_peeloff_pi`**, **`_pi_coeff`** — helpers to normalize arguments by extracting `π` multiples.
- **`sin`**, **`cos`** — auto-evaluate at rational multiples of π (denominators up to 120 via product decomposition into Fermat primes; cos(π/257) computed via nested radicals). Taylor series, trig expansion via Chebyshev polynomials. AccumBounds support.
- **`tan`**, **`cot`** — expressed via sin/cos with same rational-π evaluation. Symbolic trig expansion uses symmetric polynomials.
- **`ReciprocalTrigonometricFunction`** — base for `sec`, `csc`; delegates most logic to the reciprocal.
- **`sec`**, **`csc`** — reciprocals of `cos`, `sin`.
- **`sinc`** — unnormalized `sin(x)/x`; rewrites as spherical Bessel `jn(0, x)`.
- **`InverseTrigonometricFunction`** — base for inverse trig.
- **`asin`**, **`acos`**, **`atan`**, **`acot`**, **`asec`**, **`acsc`** — inverse trig functions with table-driven evaluation at algebraic values. All support rewrite as `log`.
- **`atan2`** — two-argument arctangent with full quadrant logic. Rewrites as `log`, `atan`, `arg`.

### `hyperbolic.py`
Hyperbolic and inverse hyperbolic functions.

- **`HyperbolicFunction`** — base class.
- **`sinh`**, **`cosh`**, **`tanh`**, **`coth`** — hyperbolic functions with imaginary-argument folding to trig counterparts. Full Taylor series, trig expansion, exp rewrites.
- **`ReciprocalHyperbolicFunction`** — base for `sech`, `csch`.
- **`sech`**, **`csch`** — reciprocals of `cosh`, `sinh`.
- **`asinh`**, **`acosh`**, **`atanh`**, **`acoth`**, **`asech`**, **`acsch`** — inverse hyperbolics with special-value tables and log rewrites.

**Caveat:** `acosh` uses a large table of known algebraic values mapping to multiples of `π` (for complex results).

### `complexes.py`
Complex number manipulation functions.

- **`re`**, **`im`** — real/imaginary part extraction with term-by-term analysis.
- **`sign`** — complex signum (`1`, `-1`, `0`, `I`, `-I`, or unevaluated). Rewrites as `Piecewise`, `Heaviside`.
- **`Abs`** — absolute value with intelligent handling of `Mul`, `Pow`, `exp`. Rewrites as `Heaviside`, `Piecewise`, `sign`.
- **`arg`** — complex argument (radians) via `atan2(im, re)`.
- **`conjugate`**, **`transpose`**, **`adjoint`** — linear algebra operations on expressions.
- **`polar_lift`**, **`periodic_argument`**, **`unbranched_argument`**, **`principal_branch`** — Riemann surface / branch-cut management for multi-valued functions.
- **`polarify`** / **`unpolarify`** — convert between polar and standard representations.

### `integers.py`
Rounding functions.

- **`RoundFunction`** — base class that separates integral, numerical, and symbolic parts.
- **`floor`** — greatest integer ≤ arg.
- **`ceiling`** — smallest integer ≥ arg.
- **`frac`** — fractional part `x - floor(x)`.

### `miscellaneous.py`
Root functions and min/max.

- **`IdentityFunction` (`Id`)** — singleton identity lambda.
- **`sqrt`**, **`cbrt`** — shortcuts for `Pow(x, 1/2)`, `Pow(x, 1/3)`.
- **`root`** — k-th n-th root of x.
- **`real_root`** — forces real principal root for negative bases with odd denominators.
- **`MinMaxBase`** — base class implementing lattice-based simplification with directional comparisons.
- **`Max`**, **`Min`** — symbolic max/min with assumption-driven simplification and `Heaviside` rewrites.

### `piecewise.py`
Piecewise-defined expressions.

- **`ExprCondPair`** — (expression, condition) tuple.
- **`Piecewise`** — piecewise function with condition evaluation, collapsing of nested piecewise, definite interval integration, `_eval_subs`, and `_sort_expr_cond` for interval arithmetic.
- **`piecewise_fold`** — distributes arithmetic operations into piecewise branches.

### `benchmarks/bench_exp.py`
Micro-benchmark for `exp(2)` creation.

---

## Special Functions

### `gamma_functions.py`
Gamma function and related.

- **`gamma`** — `Γ(x)`. Evaluates at integers and half-integers, reflects for negative arguments. Taylor series, product expansion, Stirling-series asymptotics.
- **`lowergamma`** — lower incomplete gamma `γ(s,x)`. Rewrites as `expint`, `Ei`.
- **`uppergamma`** — upper incomplete gamma `Γ(s,x)`. Rewrites as `expint`, `Ei`.
- **`polygamma`** — `ψ^(n)(z)` (n-th derivative of `log Γ`). Evaluates at positive integers and rational arguments. Reflection/recurrence formulae.
- **`loggamma`** — `log Γ(z)` on the principal branch. Handles sign correctly for negative reals.
- **`digamma`**, **`trigamma`** — convenience wrappers for `polygamma(0, x)` and `polygamma(1, x)`.

### `beta_functions.py`
Euler beta function.

- **`beta`** — `B(x,y) = Γ(x)Γ(y)/Γ(x+y)`. Differentiation via digamma; expand_func decomposes into gamma products.

### `error_functions.py`
Error functions, exponential/logarithmic integrals, trigonometric/hyperbolic integrals, and Fresnel integrals (~2445 lines).

- **`erf`** — Gauss error function. Rewrites as `erfc`, `erfi`, `uppergamma`, `hyper`.
- **`erfc`** — complementary error function `1 - erf(x)`.
- **`erfi`** — imaginary error function `-i·erf(ix)`.
- **`erf2`** — generalized two-argument error function `erf(x, y)`.
- **`erfinv`**, **`erfcinv`**, **`erf2inv`** — inverse error functions.
- **`Ei`** — exponential integral `Ei(z)`.
- **`expint`** — generalized exponential integral `E_n(z)`. Extensive branch-cut handling.
- **`E1`** — shorthand `expint(1, z)`.
- **`li`**, **`Li`** — logarithmic integral and offset logarithmic integral.
- **`TrigonometricIntegral`** — base for `Si`, `Ci`, `Shi`, `Chi`.
- **`Si`**, **`Ci`** — sine/cosine integrals.
- **`Shi`**, **`Chi`** — hyperbolic sine/cosine integrals.
- **`FresnelIntegral`** — base for Fresnel integrals.
- **`fresnels`**, **`fresnelc`** — Fresnel S and C integrals.
- **`_erfs`**, **`_eis`** — internal helper classes for series expansion of `erf` and `Ei`.

### `bessel.py`
Bessel functions, spherical Bessel functions, Hankel functions, and Airy functions (~1661 lines).

- **`BesselBase`** — abstract base with differentiation formula `2F' = -aF_{n+1} + bF_{n-1}`, conjugate, expand, simplify (delegates to `besselsimp`).
- **`besselj`**, **`bessely`** — Bessel J and Y (first/second kind). Branch handling via `unpolarify`/`extract_branch_factor`.
- **`besseli`**, **`besselk`** — modified Bessel I and K.
- **`hankel1`**, **`hankel2`** — Hankel functions `H^(1)`, `H^(2)`.
- **`SphericalBesselBase`** — base for spherical Bessel functions.
- **`jn`**, **`yn`** — spherical Bessel j and y. Expand to `sin`/`cos` rational functions for integer order.
- **`SphericalHankelBase`**, **`hn1`**, **`hn2`** — spherical Hankel functions.
- **`jn_zeros`** — zeros of spherical Bessel j via `mpmath.besseljzero` or SciPy.
- **`AiryBase`** — abstract base for Airy functions.
- **`airyai`**, **`airybi`** — Airy Ai and Bi functions. Rewrite as `besselj`, `besseli`, `hyper`. `_eval_expand_func` handles cube-root argument transformations.
- **`airyaiprime`**, **`airybiprime`** — derivatives of Airy functions.

### `hyper.py`
Hypergeometric and Meijer G-functions.

- **`hyper`** — generalized hypergeometric function `pFq(a; b; z)`. Convergence radius tracking, series expansion, `_eval_nseries`.
- **`meijerg`** — Meijer G-function `G_{p,q}^{m,n}`. Handles branch factors and period.
- **`HyperRep`** and subclasses (`HyperRep_power1`, `_power2`, `_log1`, `_log2`, `_atanh`, `_asin1`, `_asin2`, `_sqrts1`, `_sqrts2`, `_cosasin`, `_sinasin`) — internal representations for expressing hypergeometric functions in terms of elementary functions.

### `elliptic_integrals.py`
Complete and incomplete elliptic integrals.

- **`elliptic_k`** — complete elliptic integral of the first kind `K(m)`.
- **`elliptic_f`** — incomplete elliptic integral of the first kind `F(z, m)`.
- **`elliptic_e`** — elliptic integral of the second kind (complete and incomplete).
- **`elliptic_pi`** — elliptic integral of the third kind `Π(n, m)` or `Π(n, z, m)`.

### `delta_functions.py`
Distribution-like functions.

- **`DiracDelta`** — Dirac delta `δ(x)` and its derivatives `δ^(k)(x)`. Methods: `is_simple` (linear argument test), `simplify` for higher-order deltas.
- **`Heaviside`** — Heaviside step function `H(x)`. Rewrites as `Piecewise`, `sign`. Derivative yields `DiracDelta`.

### `singularity_functions.py`
Macaulay bracket notation for structural engineering.

- **`SingularityFunction`** — `<x - a>^n`; evaluates to `Piecewise` or `DiracDelta`/`Heaviside` depending on exponent sign. `_eval_rewrite_as_Piecewise`, `_eval_rewrite_as_Heaviside`.

### `polynomials.py`
Classical orthogonal polynomial families.

- **`OrthogonalPolynomial`** — base class with degree and weight function hooks.
- **`jacobi`** — Jacobi polynomials `P_n^{(a,b)}(x)`. Recurrence, Rodrigues-type evaluation.
- **`jacobi_normalized`** — normalized Jacobi polynomials.
- **`gegenbauer`** — Gegenbauer (ultraspherical) polynomials `C_n^a(x)`.
- **`chebyshevt`**, **`chebyshevu`** — Chebyshev T and U polynomials.
- **`chebyshevt_root`**, **`chebyshevu_root`** — k-th root of the n-th Chebyshev polynomial.
- **`legendre`**, **`assoc_legendre`** — Legendre and associated Legendre polynomials.
- **`hermite`** — Hermite polynomials `H_n(x)`.
- **`laguerre`**, **`assoc_laguerre`** — Laguerre and associated Laguerre polynomials.

### `spherical_harmonics.py`
Spherical harmonics.

- **`Ynm`** — spherical harmonic `Y_n^m(θ, φ)`. Rewrites as `cos`/`sin` via associated Legendre polynomials.
- **`Ynm_c`** — conjugate spherical harmonic (convenience function).
- **`Znm`** — real spherical harmonic `Z_n^m(θ, φ)`.

### `tensor_functions.py`
Discrete tensor utility functions.

- **`Eijk`** / **`eval_levicivita`** — evaluate the Levi-Civita symbol for given indices.
- **`LeviCivita`** — symbolic Levi-Civita tensor `ε_{ijk}`. Handles arbitrary number of indices, deferred evaluation.
- **`KroneckerDelta`** — `δ_{ij}`. Properties: `indices_contain_equal_information`, `preferred_index`, `killable_index` (for Einstein summation). Rewrites as `Piecewise`.

### `zeta_functions.py`
Zeta and polylogarithm functions.

- **`lerchphi`** — Lerch transcendent `Φ(z, s, a)`. Extensive `_eval_expand_func` with branching for integer `s` and special `z` values.
- **`polylog`** — polylogarithm `Li_s(z)`. Evaluates at `z=1` (zeta), `z=-1` (Dirichlet eta), `z=0`.
- **`zeta`** — Riemann/Hurwitz zeta `ζ(s, a)`. Evaluates at negative/even integers via Bernoulli numbers. Rewrites as `lerchphi`.
- **`dirichlet_eta`** — Dirichlet eta `η(s) = (1-2^{1-s})ζ(s)`.
- **`stieltjes`** — Stieltjes constants `γ_n`.

### `mathieu_functions.py`
Mathieu functions.

- **`MathieuBase`** — abstract base.
- **`mathieus`**, **`mathieuc`** — Mathieu sine and cosine functions `S(a, q, z)`, `C(a, q, z)`.
- **`mathieusprime`**, **`mathieucprime`** — their derivatives.

### `bsplines.py`
B-spline basis construction.

- **`bspline_basis(d, knots, n, x)`** — n-th B-spline of degree d as a `Piecewise` expression, built recursively via Cox-de Boor.
- **`bspline_basis_set(d, knots, x)`** — full set of B-splines for given knots.
- `_add_splines` — internal helper for combining two piecewise B-splines.

### `benchmarks/bench_special.py`
Micro-benchmark for special function creation.
