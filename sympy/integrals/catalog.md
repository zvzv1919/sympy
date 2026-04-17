# Integrals Module Catalog

## Core Integration

### [`integrals.py`](integrals.py)
Core symbolic integration engine and public API.
- `Integral` — unevaluated integral expression with limits; supports `.doit()` evaluation
- `Integral.transform(x, u)` — change of variable (u-substitution) on definite integrals; recomputes bounds, reverses limits if needed
- `integrate(*args, **kwargs)` — main entry point for symbolic definite and indefinite integration
- `line_integrate(field, curve, vars)` — line integral of a vector field over a curve

### [`manualintegrate.py`](manualintegrate.py)
Step-by-step integration emulating by-hand techniques (substitution, parts, trig rules, etc.).
- `manualintegrate(f, var)` — integrate using manual rule-based strategies
- `integral_steps(integrand, symbol)` — returns the rule tree describing the integration steps
- Rule infrastructure: `Rule()` factory, `@evaluates` decorator, pattern matchers for trig/exp/power forms

### [`trigonometry.py`](trigonometry.py)
Integration of products of trigonometric functions sin^n(x)·cos^m(x).
- `trigintegrate(f, x)` — integrates trigonometric products using reduction formulas

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
- Laplace: `laplace_transform`, `inverse_laplace_transform`, `LaplaceTransform`, `InverseLaplaceTransform`
- Fourier: `fourier_transform`, `inverse_fourier_transform`, `FourierTransform`, `InverseFourierTransform`
- Sine/Cosine: `sine_transform`, `cosine_transform` and their inverses
- Hankel: `hankel_transform`, `inverse_hankel_transform` with order parameter ν

---

## Algebraic Integration Algorithms

### [`risch.py`](risch.py)
Risch algorithm for integration of transcendental elementary functions.
- `risch_integrate(f, x)` — main entry point for the Risch decision procedure
- `DifferentialExtension` — represents a tower of differential field extensions
- `NonElementaryIntegralException` — raised when integral is provably non-elementary
- Sub-algorithms: `hermite_reduce`, `polynomial_reduce`, `residue_reduce`, `laurent_series`

### [`rde.py`](rde.py)
Risch Differential Equation solver: Dy + f·y = g in a differential field.
- `rischDE(fa, fd, ga, gd, DE)` — main RDE solver
- Helper cases: `no_cancel_b_large`, `no_cancel_b_small`, `cancel_primitive`, `cancel_exp`

### [`prde.py`](prde.py)
Parametric Risch Differential Equation solver (extension of RDE with undetermined constants).
- `param_rischDE` — main parametric RDE solver
- `limited_integrate` — solves f = Dv + Σ(ci·wi) via constraint-matrix nullspace analysis; raises NonElementaryIntegralException on empty or degenerate nullspace
- `is_deriv_k`, `is_log_deriv_k_t_radical` — structure-theorem tests for derivatives and logarithmic derivatives

### [`heurisch.py`](heurisch.py)
Heuristic (pattern-based) integration for expressions not covered by the Risch algorithm.
- `heurisch(f, x)` — main heuristic integrator using Bernstein/Bronstein approach
- `heurisch_wrapper(f, x)` — wrapper with retry logic for edge cases

### [`rationaltools.py`](rationaltools.py)
Integration of rational functions p(x)/q(x) via partial fractions and logarithmic parts.
- `ratint(f, x)` — main rational function integrator
- `ratint_ratpart`, `ratint_logpart` — rational and logarithmic part sub-routines
- `log_to_atan(f, g)` — converts complex logarithms to real arctangent form

---

## Meijer G-Function Integration

### [`meijerint.py`](meijerint.py)
Integration by rewriting integrands as Meijer G-functions and applying known convolution formulas.
- `meijerint_indefinite(f, x)` — indefinite integral via G-function rewriting
- `meijerint_definite(f, x, a, b)` — definite integral via G-function lookup tables
- `meijerint_inversion(f, x, t)` — inverse Laplace transform via G-function rewriting; extracts exponential/power shifts from the integrand

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
