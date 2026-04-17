# Integrals Module Catalog

## Core Integration

### [`integrals.py`](integrals.py)
Core symbolic integration engine and public API.
- `Integral` — unevaluated integral expression with limits; supports `.doit()` evaluation
- `Integral._eval_integral` — strategy cascade for antiderivative computation:
  - Fast paths: polynomial, piecewise, constant integrands
  - Inline power rule for `(a*x+b)^c`: returns log when c==-1, general power otherwise; in `conds='piecewise'` mode emits a Piecewise distinguishing exp==-1 from general case
  - Rational functions via `ratint`, trig products, delta/singularity functions
  - Calls `risch_integrate` with `separate_integral=True`; if non-elementary remainder is returned, recursively evaluates it with other methods
  - Falls back to heuristic Risch, then Meijer G, then manual integration in order
- `Integral.transform(x, u)` — change of variable (u-substitution) on definite integrals; recomputes bounds, reverses limits if needed
- `integrate(*args, **kwargs)` — main entry point for symbolic definite and indefinite integration
- `line_integrate(field, curve, vars)` — line integral of a vector field over a curve

### [`manualintegrate.py`](manualintegrate.py)
Step-by-step integration emulating by-hand techniques (substitution, parts, trig rules, etc.).
- `manualintegrate(f, var)` — integrate using manual rule-based strategies
- `integral_steps(integrand, symbol)` — returns the rule tree describing the integration steps
- `power_rule` — handles base^exp and base^symbol (exponential) forms; returns piecewise when base==1 is indeterminate
- Trig sub-rules: `trig_sincos_rule` (sin·cos), `trig_tansec_rule` (tan·sec), `trig_cotcsc_rule` (cot·csc); each normalizes reciprocal forms (e.g. 1/sin→csc, cos/tan→cot) before pattern matching
- Rule infrastructure: `Rule()` factory, `@evaluates` decorator, `trig_rewriter`, substitution/parts strategies

### [`trigonometry.py`](trigonometry.py)
Integration of pure sin^n(x)·cos^m(x) products only (no tan/sec/cot/csc).
- `trigintegrate(f, x)` — integrates sin/cos power products via u-substitution and reduction formulas
  - When both exponents are odd, selects the smaller exponent for substitution to minimize result complexity
  - Handles piecewise output when the frequency coefficient may be zero

---

## Numerical Quadrature

### [`quadrature.py`](quadrature.py)
Gaussian quadrature rules: computes nodes and weights for numerical integration using roots of orthogonal polynomials.
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
- `_rewrite_gamma` — rewrites gamma/trig products into Meijer G-function parameters for inverse Mellin; raises NotImplementedError if numerator gamma poles partially overlap the fundamental strip
- Laplace: `laplace_transform`, `inverse_laplace_transform`, `LaplaceTransform`, `InverseLaplaceTransform`
- `_inverse_laplace_transform` — backend for inverse Laplace; tries inverse Mellin transform first (change of variables), falls back to `meijerint_inversion` if that fails
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
- `DifferentialExtension` — represents a tower of differential field extensions; `increment_level`/`decrement_level` adjust the working extension depth (raises ValueError at boundary)
- `NonElementaryIntegralException` — raised when integral is provably non-elementary
- Sub-algorithms: `hermite_reduce`, `polynomial_reduce`, `laurent_series`
- `residue_reduce` — Lazard-Rioboo-Rothstein-Trager resultant reduction for the logarithmic part of an antiderivative; returns (s_i, S_i) pairs for RootSum-log terms and a Boolean indicating whether the remaining integral is elementary

### [`rde.py`](rde.py)
Risch Differential Equation solver: solves Dy + f·y = g for y in a differential field (no undetermined constants, no structure theorems).
- `rischDE(fa, fd, ga, gd, DE)` — main RDE solver
- Helper cases: `no_cancel_b_large`, `no_cancel_b_small`, `cancel_primitive`, `cancel_exp`

### [`prde.py`](prde.py)
Parametric Risch Differential Equation solver (extension of RDE with undetermined constants).
- `param_rischDE` — main parametric RDE solver
- `limited_integrate` — solves f = Dv + Σ(ci·wi) via constraint-matrix nullspace analysis; raises NonElementaryIntegralException on empty or degenerate nullspace
- `prde_no_cancel_b_large` — parametric no-cancellation case when deg(b) ≥ deg(D); iterates degree-by-degree to build solution basis
- `prde_no_cancel_b_small` — parametric no-cancellation case when deg(b) < deg(D)−1; branches on deg(b)>0 vs ≤0 (latter raises NotImplementedError, needs recursive param_rischDE)
- `prde_linear_constraints` — generates linear constraints on undetermined constants; computes LCM denominator, divides scaled terms, returns empty Matrix when all remainders are zero (no constraints)
- `prde_spde` — parametric Special Polynomial Differential Equation; reduces degree bound via Diophantine step
- `is_deriv_k` — structure-theorem test for derivatives in a differential extension
- `is_log_deriv_k_t_radical` — verifies if an expression is the log-derivative of a radical in a tower of transcendental extensions:
  - Checks elementary extension validity; raises NotImplementedError if hypertangent monomials or unaccounted primitive extensions cause monomial count ≠ transcendence degree
  - Builds linear system from monomial derivatives, solves via `constant_system`
  - Verifies rationality of solution coefficients; raises NotImplementedError for non-rational coefficients
  - Computes multiplicative constant correction between exp(f) and the radical
- `is_log_deriv_k_t_radical_in_field` — field-level variant; checks if f is Du/u for some k(t)-radical u; uses `splitfactor` for denominator simplicity, then `residue_reduce`
- `parametric_log_deriv_heu` — heuristic for n·f = Dv/v + m·Dθ/θ (n,m∈ℤ, v∈k(t)*); branches on whether deg(q) exceeds a derivation-degree bound B, solving coefficient equations in each branch

### [`heurisch.py`](heurisch.py)
Heuristic (parallel) Risch integration using Bernstein/Bronstein "Poor Man's Integrator" approach. Supports transcendental elementary and special functions (Airy, Bessel, Whittaker, Lambert).
- `heurisch(f, x)` — main heuristic integrator; builds candidate antiderivative from undetermined coefficients over a monomial basis
- `heurisch_wrapper(f, x)` — wrapper with retry logic for edge cases
- `DiffCache` — caches derivatives during integration; for cylindrical (Bessel-type) functions, simultaneously stores derivatives for orders n and n−1 to avoid introducing a third algebraically dependent transcendental
- `components(f, x)` — collects the functional building blocks (atoms) of an expression that depend on x

### [`rationaltools.py`](rationaltools.py)
Integration of rational functions p(x)/q(x) via partial fractions and logarithmic parts.
- `ratint(f, x)` — main rational function integrator; auto-detects real/complex context from atom assumptions and selects between real arctangent/log forms vs RootSum over algebraic roots for the logarithmic part
- `ratint_ratpart`, `ratint_logpart` — rational and logarithmic part sub-routines (Horowitz-Ostrogradsky decomposition)
- `log_to_real`, `log_to_atan` — convert complex logarithmic terms to real arctangent/logarithm forms

---

## Meijer G-Function Integration

### [`meijerint.py`](meijerint.py)
Integration by rewriting integrands as Meijer G-functions and applying known convolution formulas.
- `meijerint_indefinite(f, x)` — indefinite integral via G-function rewriting; tries multiple splitting-point shifts, rewrites hyperbolic→exponential if no G-function match found
- `meijerint_definite(f, x, a, b)` — definite integral via G-function lookup tables
- `meijerint_inversion(f, x, t)` — inverse Laplace transform via G-function rewriting; extracts exponential/power shifts from the integrand
- `_has(res, *f)` — checks if a result contains unresolved target expressions; for Piecewise results, requires ALL branches to contain the target (not just any)

### [`meijerint_doc.py`](meijerint_doc.py)
Auto-generates Sphinx documentation for the Meijer G-function lookup table. No runtime logic.

---

## Special-Case Integrators

### [`deltafunctions.py`](deltafunctions.py)
Integration of expressions involving Dirac delta and Heaviside step functions.
- `deltaintegrate(f, x)` — integrates DiracDelta/Heaviside expressions by case analysis

### [`singularityfunctions.py`](singularityfunctions.py)
Integration of SingularityFunction expressions (beam/structural mechanics notation).
- `singularityintegrate(f, x)` — integrates SingularityFunction by rewriting to Heaviside/DiracDelta
