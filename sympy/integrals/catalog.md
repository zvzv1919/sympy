# Integrals Module Catalog

## Core Integration

### [`integrals.py`](integrals.py)
Core symbolic integration engine and public API.
- `Integral` — unevaluated integral expression with limits; supports `.doit()` evaluation
- `Integral.doit` — evaluates the integral using a strategy cascade over each limit set:
  - For definite integrals with infinite bounds, tries Meijer G via `meijerint_definite`; handles `conds` parameter for convergence conditions
  - When `meijerg=True` is explicitly set and the definite G-function attempt fails for infinite-bound integrals, gives up entirely (no fallback to other methods) to avoid nonsensical results from the indefinite Meijer G path
  - Raises `ValueError` when `conds='separate'` is used with multiple integration limits (multi-dimensional integrals)
  - Falls back to `_eval_integral` for antiderivative + interval evaluation
- `Integral._eval_integral` — strategy cascade for antiderivative computation:
  - Fast paths: polynomial, piecewise, constant integrands
  - Inline power rule for `(a*x+b)^c`: returns log when c==-1, general power otherwise; in `conds='piecewise'` mode emits a Piecewise distinguishing exp==-1 from general case
  - Rational functions via `ratint`, trig products, delta/singularity functions
  - Calls `risch_integrate` with `separate_integral=True`; if non-elementary remainder is returned, recursively evaluates it with other methods
  - Falls back to heuristic Risch, then Meijer G, then manual integration in order
  - When `manual=True` is explicitly set and `manualintegrate` returns a fully unevaluated `Integral`, that result is discarded and other strategies are tried
  - When `manualintegrate` partially succeeds (result still contains unevaluated `Integral` sub-expressions), recursively evaluates those remaining pieces with all other methods (manual disabled)
  - Last-resort fallback: when all strategies fail on a single-term integrand, expands the product (`mul=True, deep=False`) and retries recursively if the result becomes a sum; deferred because some expressions (e.g. x**x*(1+log(x))) are only solvable in unexpanded form
- `Integral.as_sum(n, method)` — approximates a definite integral as a finite sum using rectangle-based quadrature (left, right, midpoint, trapezoid); raises NotImplementedError for multidimensional integrals
- `Integral.transform(x, u)` — change of variable (u-substitution) on definite integrals; recomputes bounds, reverses limits if needed
- `integrate(*args, **kwargs)` — main entry point for symbolic definite and indefinite integration
- `line_integrate(field, curve, vars)` — line integral of a vector field over a curve

### [`manualintegrate.py`](manualintegrate.py)
Step-by-step integration emulating by-hand techniques (substitution, parts, trig rules, etc.). May return results containing unevaluated `Integral` sub-expressions when it can only partially evaluate; the caller in `integrals.py` handles those remainders.
- `manualintegrate(f, var)` — integrate using manual rule-based strategies
- `integral_steps(integrand, symbol)` — returns the rule tree describing the integration steps
- `power_rule` — handles base^exp and base^symbol (exponential) forms; returns piecewise when base==1 is indeterminate
- Trig sub-rules: `trig_sincos_rule` (sin·cos), `trig_tansec_rule` (tan·sec), `trig_cotcsc_rule` (cot·csc); pattern-matching dispatchers that classify the integrand and select a handler — substitution strategies and piecewise edge cases for sin^n·cos^m live in `trigonometry.py`
- `eval_trigsubstitution` — back-converts trig substitution results from angle parameter to original variable using triangle-side geometry (opposite/adjacent/hypotenuse ratios derived from the substitution relation)
- Rule infrastructure: `Rule()` factory, `@evaluates` decorator, `trig_rewriter`, substitution/parts strategies

### [`trigonometry.py`](trigonometry.py)
Integration of pure sin^n(x)·cos^m(x) products only (no tan/sec/cot/csc).
- `trigintegrate(f, x)` — integrates sin/cos power products via u-substitution and reduction formulas
  - When both exponents are odd, selects the smaller exponent for substitution to minimize result complexity
  - Handles piecewise output when the frequency coefficient may be zero
- `_sin_pow_integrate(n, x)` / `_cos_pow_integrate(n, x)` — recursive reduction formulas for sin^n / cos^n; base cases: n=1 (trig identity), n=0 (returns x), n=−1 (delegates to `trigintegrate(1/sin(x))` to break recursion)

---

## Numerical Quadrature

### [`quadrature.py`](quadrature.py)
Gaussian quadrature rules: computes nodes and weights (not sums) for numerical integration using roots of orthogonal polynomials. Does not approximate specific integrals — returns reusable quadrature tables.
- `gauss_legendre(n, n_digits)` — nodes/weights for ∫₋₁¹ f(x)dx
- `gauss_laguerre(n, n_digits)` — nodes/weights for ∫₀^∞ e^{-x} f(x)dx
- `gauss_hermite(n, n_digits)` — nodes/weights for ∫₋∞^∞ e^{-x²} f(x)dx
- `gauss_gen_laguerre(n, alpha, n_digits)` — nodes/weights for ∫₀^∞ x^α e^{-x} f(x)dx; α is a singularity exponent
- `gauss_chebyshev_t(n, n_digits)` — nodes/weights for ∫₋₁¹ f(x)/√(1−x²)dx (first kind)
- `gauss_chebyshev_u(n, n_digits)` — nodes/weights for ∫₋₁¹ f(x)√(1−x²)dx (second kind)
- `gauss_jacobi(n, alpha, beta, n_digits)` — nodes/weights for ∫₋₁¹ (1−x)^α(1+x)^β f(x)dx
- All functions return `(x, w)` tuples of node positions and weights as arbitrary-precision Floats.

---

## Integral Transforms

### [`transforms.py`](transforms.py)
Symbolic integral transforms — class-based API and dispatch layer (delegates heavy computation to `meijerint.py`).
- `IntegralTransform` — abstract base class for all transforms
- Mellin: `mellin_transform`, `inverse_mellin_transform`, `MellinTransform`, `InverseMellinTransform`
  - `InverseMellinTransform._compute_transform` validates input by traversing the expression and checking each function against a whitelist of allowed types (exp, gamma, sin, cos, tan, etc.); raises `IntegralTransformError` for unrecognized functions
- `_rewrite_gamma` — rewrites gamma/trig products into Meijer G-function parameters (an, ap, bm, bq) for inverse Mellin transform
  - `left(c, is_numer)` — determines whether a pole at c lies left of the fundamental strip (integration contour); handles None/infinite strip boundaries with heuristic inequality checks
    - When pole position is indeterminate: returns None for numerator factors; for denominator factors, returns None if strip bounds or pole contain free symbols, otherwise raises `MellinTransformStripError`
  - Poles are classified as left/right of contour to assign them to bm vs bq (or an vs ap) G-function parameter lists
  - Polynomial factors: degree-1 extracts linear root directly; degree>1 factors via `roots()`, falls back to `CRootOf.all_roots()` when roots() doesn't find all roots
  - Trig factors (sin, cos, tan, cot): rewrites as pairs of gamma functions via `_rewrite_sin` / analogous helpers
- `_rewrite_sin` — converts sin(m·s+n) into a gamma(…)·gamma(1−…) pair using the reflection formula; computes an integer shift via `ceiling` to keep both gamma arguments safely on the correct side of the integration strip (avoids poles inside the contour); uses `as_real_imag()[0]` instead of `re()` because `re()` does not expand symbolic expressions
  - Applies gamma multiplication theorem to normalize coefficient magnitudes ≠ 1 (in `_rewrite_gamma`)
  - Raises `MellinTransformStripError` if a pole falls inside the critical strip; raises NotImplementedError if numerator gamma poles partially overlap the strip
- Laplace: `laplace_transform`, `inverse_laplace_transform`, `LaplaceTransform`, `InverseLaplaceTransform`
- `_inverse_laplace_transform` — backend for inverse Laplace; tries inverse Mellin transform first (change of variables), falls back to `meijerint_inversion` if that fails
  - When fallback returns a Piecewise that still contains an unevaluated `Integral`, raises `IntegralTransformError` ('inversion integral of unrecognised form')
  - If result is still Piecewise after processing, returns early without Heaviside/exp simplification (booleans in args break those transforms)
- `_fourier_transform(f, x, k, a, b)` — backend computing generalized F(k) = a·∫exp(b·i·x·k)f(x)dx over (−∞,∞); extracts first branch if result is Piecewise
- Fourier: `fourier_transform`, `inverse_fourier_transform`, `FourierTransform`, `InverseFourierTransform`
- `_sine_cosine_transform` — backend for sine/cosine transforms; integrates over [0,∞), raises IntegralTransformError if result is not Piecewise or if first Piecewise branch still contains unevaluated Integral
- Sine/Cosine: `sine_transform`, `cosine_transform` and inverses — unitary half-range [0,∞) transforms with prefactor sqrt(2/π); odd-parity (sine) and even-parity (cosine)
- `_hankel_transform` — backend for Hankel transforms; integrates f·Jν(kr)·r over [0,∞), raises IntegralTransformError if result is not Piecewise or if first Piecewise branch still contains unevaluated Integral
- Hankel: `hankel_transform`, `inverse_hankel_transform` with order parameter ν

---

## Algebraic Integration Algorithms

### [`risch.py`](risch.py)
Risch algorithm for integration of transcendental elementary functions.
- `risch_integrate(f, x)` — main entry point for the Risch decision procedure
  - Iterates tower levels in reverse; skips levels where integrand is independent of current extension monomial
  - Dispatches to exp/primitive sub-algorithms per level; applies back-substitutions at termination
- `DifferentialExtension` — builds and represents a tower of differential field extensions; `increment_level`/`decrement_level` adjust the working extension depth (raises ValueError at boundary)
  - `_exp_part` — attempts to add an exponential monomial to the tower; uses `is_log_deriv_k_t_radical` to detect algebraic dependencies
    - Normalizes n==-1 radical degree to n==1 (inverts u, negates const/powers); restarts extension when algebraic radical avoidable
  - `_log_part` — attempts to add a logarithmic monomial to the tower; uses `is_deriv_k` to detect existing derivatives
- `NonElementaryIntegralException` — raised when integral is provably non-elementary
- `get_case(d, t)` — classifies derivation type: 'base' (d==1, no t), 'primitive' (d is constant but ≠1), 'exp' (d divisible by t), 'tan' (d divisible by 1+t²), or other_linear/other_nonlinear
- `derivation(p, DE)` — computes Dp for polynomial p in the differential extension tower; `coefficientD=True` computes the coefficient derivation (treats top-level variable as constant)
- Polynomial utilities: `gcdex_diophantine` (extended GCD solving s*a + t*b == c with degree bound; reduces s modulo b via degree comparison when s.degree() >= b.degree()), `frac_in`, `as_poly_1t`
- `hermite_reduce` — Mack's linear version of Hermite reduction; decomposes f = Dg + h + r (g rational, h simple, r reduced) by iteratively reducing denominator multiplicity via extended GCD
- `polynomial_reduce` — writes p = Dq + r with deg(r) < deg(Dt)
- `laurent_series` — contribution of a factor to the full partial fraction decomposition
- `recognize_log_derivative(a, d, DE)` — tests whether f=a/d is a logarithmic derivative (dv/v for some v in the function field) by computing the resultant, splitting it via `splitfactor_sqf`, and checking that all real roots of the special factors are integers; known limitation: ignores complex roots (TODO)
- `residue_reduce` — Lazard-Rioboo-Rothstein-Trager resultant reduction for the logarithmic part of an antiderivative; returns (s_i, S_i) pairs for RootSum-log terms and a Boolean indicating whether the remaining integral is elementary
- `integrate_primitive_polynomial(p, DE)` — iteratively reduces polynomial degree in a logarithmic (primitive) tower extension
  - Peels off leading coefficient each iteration, calls `limited_integrate` to solve for it, subtracts partial antiderivative's derivative from remainder
  - Raises `NonElementaryIntegralException` when the leading coefficient has no elementary antiderivative
- `integrate_primitive(a, d, DE)` — integrates primitive (logarithmic) functions; uses Hermite reduction + residue reduction + `integrate_primitive_polynomial` pipeline
- `integrate_hyperexponential_polynomial(p, DE, z)` — integrates Laurent polynomials in k[t, 1/t] for hyperexponential extensions; iterates over degrees (skips zero), calls `rischDE` per coefficient; on `NonElementaryIntegralException` sets b=False but continues processing remaining terms (does not bail out)
- `integrate_hyperexponential(a, d, DE)` — integrates hyperexponential functions (exponential monomials); uses Hermite reduction + residue reduction + polynomial integration pipeline
  - In piecewise mode, emits a Piecewise to handle the case where the exponential monomial equals 1 (zero exponent), avoiding division by zero by substituting t=1 and integrating separately
- `integrate_nonlinear_no_specials(a, d, DE)` — integrates rational functions in a nonlinear tower extension when no special irreducible factors exist
  - Applies Hermite + residue + polynomial reduction pipeline
  - Determines elementarity by checking whether the remainder polynomial still contains the tower variable (if so, non-elementary)
- `NonElementaryIntegral` — subclass of `Integral` that guarantees the integral is provably nonelementary; returned by `risch_integrate` with `risch=True`

### [`rde.py`](rde.py)
Risch Differential Equation solver: solves Dy + f·y = g for y in a differential field (no undetermined constants, no structure theorems).
- `rischDE(fa, fd, ga, gd, DE)` — main RDE solver
- `special_denom` — non-parametric special denominator computation; dispatches on exp/tan/primitive case to adjust denominator and polynomial order bound
  - For exp: checks `parametric_log_deriv` to tighten bound n when possible cancellation detected (nb==0)
  - For tan (hypertangent): gates on `recognize_log_derivative(2*beta)` of the real part before attempting `parametric_log_deriv` to tighten bound n
  - Returns transformed (A, B, C, h) tuple with p^N scaling
- `bound_degree(a, b, cQ, DE, case)` — computes upper bound on degree of polynomial solution to the RDE; dispatches on extension type:
  - base/primitive/exp/tan each have distinct formulas comparing degrees of a, b, and c
  - primitive case: when deg(b)==deg(a)−1, calls `limited_integrate` to check if α is an integer-shifted derivative; when deg(b)==deg(a), uses `is_log_deriv_k_t_radical_in_field`
  - exp case: when deg(a)==deg(b), uses `parametric_log_deriv` to potentially raise the bound
- `spde(a, b, c, n, DE)` — Rothstein's Special Polynomial Differential Equation; reduces RDE to equivalent equation with lower degree bound
- Helper cases: `no_cancel_b_large`, `no_cancel_b_small`, `cancel_primitive`, `cancel_exp`

### [`prde.py`](prde.py)
Parametric Risch Differential Equation solver (extension of RDE with undetermined constants).
- `param_rischDE` — main parametric RDE solver
- `limited_integrate` — solves f = Dv + Σ(ci·wi) via constraint-matrix nullspace analysis; raises NonElementaryIntegralException on empty or degenerate nullspace
- `prde_special_denom` — parametric variant of special denominator (operates on vector of RHS polynomials G=[g1,...,gm] instead of scalar c)
  - For primitive/base: short-circuits with `(a, ba.quo(bd), G, 1)` — no order computation needed since k<t>==k[t]
  - For exp: when order nb==0 (possible cancellation), checks `parametric_log_deriv` to tighten bound n
  - For tan: when nb==0, separates real/imaginary parts via `real_imag`, gates on `recognize_log_derivative(2*beta_imag)` before attempting `parametric_log_deriv` on both parts; adjusts n to s/2 only when both succeed
- `real_imag` — separates a rational function into real and imaginary parts evaluated at a complex root of t²+1
- `prde_no_cancel_b_large` — parametric no-cancellation case when deg(b) ≥ deg(D); iterates degree-by-degree to build solution basis
- `prde_no_cancel_b_small` — parametric no-cancellation case when deg(b) < deg(D)−1; branches on deg(b)>0 vs ≤0 (latter raises NotImplementedError, needs recursive param_rischDE)
- `prde_linear_constraints` — generates linear constraints on undetermined constants; computes LCM denominator, divides scaled terms, returns empty Matrix when all remainders are zero (no constraints)
- `prde_spde` — parametric Special Polynomial Differential Equation; reduces degree bound via Diophantine step
- `is_deriv_k` — checks if Df/f is the derivative of an element of k(t) using the structure theorem
  - Validates that log + hyperexp monomial count equals transcendence degree
  - Raises NotImplementedError for tangent-type (hypertangent) or non-elementary extensions
- `is_log_deriv_k_t_radical` — verifies if an expression is the log-derivative of a radical in a tower of transcendental extensions:
  - Checks elementary extension validity; raises NotImplementedError if hypertangent monomials or unaccounted primitive extensions cause monomial count ≠ transcendence degree
  - Builds linear system from monomial derivatives, solves via `constant_system`
  - Verifies rationality of solution coefficients; raises NotImplementedError for non-rational coefficients
  - Computes multiplicative constant correction between exp(f) and the radical
- `is_log_deriv_k_t_radical_in_field` — field-level variant; checks if f=fa/fd is Du/u for some k(t)-radical u; dispatches on case (exp/primitive/base/tan)
  - Base case: immediately returns None if fd is not square-free or deg(fa) >= deg(fd); otherwise computes n and u from residue terms
  - Uses `splitfactor` for denominator simplicity check, then `residue_reduce`; returns None if not all resultant roots are rational
- `parametric_log_deriv_heu` — heuristic for n·f = Dv/v + m·Dθ/θ (n,m∈ℤ, v∈k(t)*); branches on whether deg(q) exceeds a derivation-degree bound B, solving coefficient equations in each branch

### [`heurisch.py`](heurisch.py)
Semi-decision (heuristic) Risch integration using Bernstein/Bronstein "Poor Man's Integrator" approach. Supports transcendental elementary and special functions (Airy, Bessel, Whittaker, Lambert). Unlike the full Risch decision procedure in `risch.py`, cannot prove non-existence of antiderivatives.
- `heurisch(f, x)` — main heuristic integrator; builds candidate antiderivative from undetermined coefficients over a monomial basis
  - Substitutes subexpressions with placeholder symbols; tries all permutations of the substitution ordering until the result is rational in placeholders
  - Two-phase coefficient domain strategy: first solves the undetermined-coefficients system over the rationals ('Q'); if that fails, retries without a field restriction (general domain)
  - If both domain attempts fail, recursively retries with decremented retry count and different variable permutations
  - If no permutation yields a rational function, falls back to rewriting the integrand in terms of tan/tanh and retries
  - `_exponent` helper computes upper degree bound for the polynomial ansatz; handles fractional rational powers specially (p/q with q≠1 yields p+q−1 or |p+q|)
  - `_splitter` recursively decomposes polynomials via derivation and GCD for denominator factoring
- `heurisch_wrapper(f, x)` — wraps `heurisch`; detects new symbolic poles in the antiderivative's denominators (not present in original integrand), re-evaluates under each special-case substitution, and returns a Piecewise over those parameter conditions
- `DiffCache` — caches derivatives during integration; for cylindrical (Bessel-type) functions, simultaneously stores derivatives for orders n and n−1 to avoid introducing a third algebraically dependent transcendental
- `components(f, x)` — collects the functional building blocks (atoms) of an expression that depend on x; for power expressions: integer exponents yield only base components, rational non-integer exponents add base^(1/denominator), symbolic/irrational exponents add both full power and exponent components

### [`rationaltools.py`](rationaltools.py)
Integration of rational functions p(x)/q(x) via partial fractions and logarithmic parts.
- `ratint(f, x)` — main rational function integrator; auto-detects real/complex context from atom assumptions and selects between real arctangent/log forms vs RootSum over algebraic roots for the logarithmic part
- `ratint_ratpart(f, g, x)` — Horowitz-Ostrogradsky algorithm: decomposes f/g into A' + B where B has square-free denominator
  - Computes GCD of denominator g with its derivative to split g into repeated-factor part u and square-free part v
  - Sets up undetermined polynomial coefficients for numerators A and B, solves the resulting linear system
- `ratint_logpart(f, g, x)` — Lazard-Rioboo-Trager algorithm for the logarithmic part of rational integration; computes resultant-based decomposition into RootSum-log terms
  - Shortcuts when denominator degree equals the multiplicity from the resultant's square-free decomposition (appends (g, q) directly); otherwise normalizes via leading-coefficient inversion
- `log_to_real`, `log_to_atan` — convert complex logarithmic terms to real arctangent/logarithm forms

---

## Meijer G-Function Integration

### [`meijerint.py`](meijerint.py)
Integration by rewriting integrands as Meijer G-functions and applying known convolution formulas.
- `meijerint_indefinite(f, x)` — indefinite integral via G-function rewriting
  - Tries multiple splitting-point shifts; if a result still contains unevaluated `hyper`/`meijerg`, collects it as a fallback candidate rather than returning immediately
  - If all shifts yield unevaluated special functions and f contains HyperbolicFunction, rewrites hyperbolics as exponentials and retries
  - Returns the best (simplest) collected result if no clean closed-form is found
- `meijerint_definite(f, x, a, b)` — definite integral via G-function lookup tables
- `_check_antecedents_inversion(g, x)` — validates convergence conditions for inverse transform integrals:
  - Checks "condition A" (parameter differences must not be positive integers)
  - When p >= q: uses asymptotic Slater expansion directly
  - When p < q: applies multiple theorems from [L] §5.10 (angle/delta conditions, rho positivity, tau bounds) to verify convergence
- `meijerint_inversion(f, x, t)` — inverse Laplace transform via G-function rewriting
  - Pre-processes product-form integrands by filtering out `exp(a*x)` and `base^(a*x)` factors, accumulating their exponents into a cumulative shift applied to the final result
  - When coefficient extraction from a power exponent fails (`_CoeffExpValueError`), treats the factor as non-exponential (keeps it in the integrand unchanged)
- `_rewrite_saxena(fac, po, g1, g2, x)` — normalizes a product of two G-functions with different rational powers of x in their arguments so both become linear in x; harmonizes exponents via LCM-based inflation, flips negative exponents, applies principal branch, and absorbs the polynomial factor into one G-function
- `_split_mul(f, x)` — decomposes multiplicative integrand into (constant_factor, x_power, remainder); retries with `expand_mul` if base doesn't initially split as coeff*x
- `_condsimp` — simplifies boolean convergence conditions from G-function integration; applies pattern-based rewrite rules (e.g. Or(p<q, Eq(p,q))→p≤q); rewrites equalities involving `periodic_argument` with infinite period on non-polar args as positivity conditions (arg > 0)
- `_has(res, *f)` — checks if a result contains unresolved target expressions; for Piecewise results, requires ALL branches to contain the target (not just any)

### [`meijerint_doc.py`](meijerint_doc.py)
Auto-generates Sphinx documentation for the Meijer G-function lookup table. No runtime logic.

---

## Special-Case Integrators

### [`deltafunctions.py`](deltafunctions.py)
Integration of expressions involving Dirac delta and Heaviside step functions.
- `deltaintegrate(f, x)` — integrates DiracDelta/Heaviside expressions by case analysis
  - For higher-order DiracDelta (derivatives), performs repeated integration by parts; returns the largest non-zero hyperreal term to ensure correct results under nested integration
  - For products, extracts a simple DiracDelta term via `change_mul`, evaluates the remaining factor at the delta's root
- `change_mul(node, x)` — rearranges a multiplicative term to extract one simple DiracDelta factor
  - Sorts commutative args for deterministic collapse; when a factor is DiracDelta raised to a power, decrements the exponent by 1 and keeps the base for extraction
  - If no simple DiracDelta is found, falls back to expanding all DiracDelta terms with `diracdelta=True`

### [`singularityfunctions.py`](singularityfunctions.py)
Integration of SingularityFunction (Macaulay bracket) expressions used in beam/structural mechanics.
- `singularityintegrate(f, x)` — integrates SingularityFunction expressions with three branches:
  - Bare SingularityFunction(x,a,n): increments exponent (power rule); for n≥0 divides by n+1, for n∈{-1,-2} just increments
  - Product or power containing SingularityFunction: rewrites to DiracDelta/Heaviside, integrates, then converts result back to SingularityFunction
  - Otherwise returns None (not handled)
