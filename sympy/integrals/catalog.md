# sympy/integrals — Integration

Symbolic integration engine: indefinite/definite integration, integral transforms, and the underlying decision procedures (Risch algorithm, heuristic Risch, Meijer G-functions).

## Glossary

- **Risch algorithm** — a complete decision procedure for integrating elementary functions; proves an integral is elementary or non-elementary.
- **Heurisch** — heuristic (parallel) Risch algorithm; not a decision procedure but handles many cases including special functions.
- **Meijer G-function** — very general special function used to rewrite integrands so that integration theorems for G-functions can be applied.
- **Differential extension** — tower of transcendental field extensions (exp, log) over the rational function field, used internally by the Risch algorithm.
- **RDE / PRDE** — Risch Differential Equation / Parametric RDE, sub-problems solved as part of the Risch algorithm.

---

## Core Integration

### `integrals.py`

Main entry point for symbolic integration; defines the `Integral` class and the `integrate()` dispatcher.

- **`class Integral(AddWithLimits)`** — unevaluated integral expression.
  - `doit(**hints)` — evaluate the integral by dispatching through the algorithm cascade: polynomial → rational → trig → delta → singularity → Risch → Meijer G → manual → heurisch.
  - `transform(x, u)` — change of variables / u-substitution on a definite integral.
  - `as_sum(n, method)` — approximate a definite integral via rectangle/trapezoid rules.
  - `_eval_integral(f, x, ...)` — internal anti-derivative engine; orchestrates all back-end algorithms.
  - `_eval_derivative(sym)` — differentiation under the integral sign + fundamental theorem of calculus.
  - Series helpers: `_eval_lseries`, `_eval_nseries`, `_eval_as_leading_term`.
- **`integrate(*args, **kwargs)`** — top-level convenience function; constructs an `Integral` and calls `doit`.
- **`line_integrate(field, curve, vars)`** — compute a scalar line integral along a `Curve`.

### `manualintegrate.py`

Step-by-step integration that mimics hand techniques (substitution, parts, trig identities, etc.).

- **`integral_steps(integrand, symbol)`** — returns a nested namedtuple tree describing the sequence of rules applied (power rule, u-sub, parts, trig rewrite, etc.). Used by SymPy Gamma for step-by-step explanations.
- **`manualintegrate(f, var)`** — evaluate the steps tree to produce the antiderivative.
- Rule namedtuples: `ConstantRule`, `PowerRule`, `AddRule`, `URule`, `PartsRule`, `CyclicPartsRule`, `TrigRule`, `ExpRule`, `ReciprocalRule`, `ArcsinRule`, `ArctanRule`, `RewriteRule`, `TrigSubstitutionRule`, `HeavisideRule`, `DontKnowRule`, etc.
- Strategy helpers:
  - `find_substitutions(integrand, symbol, u_var)` — search for viable u-substitutions.
  - `_parts_rule(integrand, symbol)` — LIATE-based integration by parts.
  - `trig_sincos_rule`, `trig_tansec_rule`, `trig_cotcsc_rule` — trig power/product matchers.
  - `trig_substitution_rule` — Euler-type trig substitution for radicals of the form `a + b*x²`.
  - `alternatives`, `substitution_rule`, `partial_fractions_rule`, `cancel_rule` — strategy combinators.

---

## Algorithmic Backends

### `risch.py`

Implementation of the (transcendental) Risch algorithm for elementary integration.

- **`class DifferentialExtension`** — builds and holds the transcendental extension tower (log/exp monomials, derivations, back-substitution info).
  - `_exp_part(exps)` / `_log_part(logs)` — extend the tower with exponential / logarithmic monomials.
  - `increment_level()` / `decrement_level()` — navigate the tower.
- **`class DecrementLevel`** — context manager for temporarily lowering the extension level.
- **`class NonElementaryIntegral(Integral)`** — sentinel subclass; if `integrate(…, risch=True)` returns this, the integral is provably non-elementary.
- **`risch_integrate(f, x, ...)`** — top-level Risch integration driver; loops over extension levels dispatching to `integrate_hyperexponential`, `integrate_primitive`, or base-case `ratint`.
- Key sub-algorithms:
  - `hermite_reduce(a, d, DE)` — Mack's linear Hermite reduction.
  - `polynomial_reduce(p, DE)`, `residue_reduce(a, d, DE)` — polynomial and residue reduction steps.
  - `integrate_primitive(a, d, DE)`, `integrate_hyperexponential(a, d, DE)` — main integration routines per extension type.
  - `integrate_hypertangent_polynomial`, `integrate_nonlinear_no_specials` — less common cases.
  - `canonical_representation`, `splitfactor`, `splitfactor_sqf` — splitting factorization utilities.
  - `recognize_derivative`, `recognize_log_derivative` — structure theorem tests.
  - `is_deriv_k`, `is_log_deriv_k_t_radical` — decide if an element is a derivative or log-derivative of a field element (structure theorem approach).
- Utility: `integer_powers`, `frac_in`, `as_poly_1t`, `derivation`, `get_case`, `gcdex_diophantine`, `laurent_series`.

### `prde.py`

Parametric Risch Differential Equation solvers — sub-problem algorithms used by `risch.py`.

- `prde_normal_denom`, `prde_special_denom` — compute normal/special parts of the denominator for the parametric problem.
- `prde_linear_constraints` — generate linear constraints on the constant parameters.
- `constant_system(A, u, DE)` — reduce a system to one over the constant field.
- `prde_spde` — parametric version of the special polynomial DE algorithm.
- `prde_no_cancel_b_large`, `prde_no_cancel_b_small` — parametric no-cancellation cases.
- `limited_integrate_reduce`, `limited_integrate` — solve the limited integration problem (f = Dv + Σ cᵢwᵢ).
- `parametric_log_deriv_heu`, `parametric_log_deriv` — parametric logarithmic derivative heuristic.
- `is_deriv_k`, `is_log_deriv_k_t_radical`, `is_log_deriv_k_t_radical_in_field` — structure theorem decision procedures.
- `real_imag(ba, bd, gen)` — split a rational function at √(-1) into real/imaginary parts (for hypertangent case).

### `rde.py`

Risch Differential Equation solver — solves Dy + f·y = g in a differential field.

- `order_at(a, p, t)` / `order_at_oo(a, d, t)` — compute multiplicity of a polynomial at a point / at infinity.
- `weak_normalizer(a, d, DE)` — weak normalization of f = a/d.
- `normal_denom(fa, fd, ga, gd, DE)` — step 1: normal part of denominator.
- `special_denom(a, ba, bd, ca, cd, DE)` — step 2: special part of denominator.
- `bound_degree(a, b, cQ, DE)` — step 3: degree bound on polynomial solutions.
- `spde(a, b, c, n, DE)` — step 4: Rothstein's special polynomial DE algorithm.
- `no_cancel_b_large`, `no_cancel_b_small`, `no_cancel_equal` — polynomial RDE solvers (no cancellation cases).
- `cancel_primitive`, `cancel_exp` — polynomial RDE solvers (cancellation cases).
- `solve_poly_rde(b, cQ, n, DE)` — dispatcher for polynomial RDE solving (step 4+5).
- `rischDE(fa, fd, ga, gd, DE)` — top-level RDE driver combining all steps.

### `heurisch.py`

Heuristic (parallel) Risch algorithm — a non-deterministic fallback integrator.

- **`heurisch(f, x, ...)`** — compute indefinite integral via the extended heuristic Risch algorithm (Bronstein's "Poor Man's Integrator"). Supports elementary, Airy, Bessel, Whittaker, Lambert W functions.
- **`heurisch_wrapper(f, x, ...)`** — wraps `heurisch` to handle potential poles in the result, returning `Piecewise` when denominators might vanish.
- `components(f, x)` — collect all functional components (symbols, functions, fractional powers) of an expression.
- **`class BesselTable`** / **`class DiffCache`** — cache Bessel function derivative recurrences to keep the parallel algorithm well-posed.

### `meijerint.py`

Integration via Meijer G-function rewriting — handles both indefinite and definite integrals, including many special-function results.

- **`meijerint_indefinite(f, x)`** — indefinite integral by rewriting f as G-functions.
- **`meijerint_definite(f, x, a, b)`** — definite integral over [a, b] (may split at ±∞, use Heaviside tricks for finite bounds).
- **`meijerint_inversion(f, x, t)`** — inverse Laplace-type integral (c−i∞ to c+i∞).
- Lookup table: `_create_lookup_table(table)` — populates a large table mapping elementary/special functions to their Meijer G representations (hundreds of entries covering exp, trig, hyperbolic, Bessel, erf, Fresnel, elliptic, etc.).
- Rewriting helpers:
  - `_rewrite_single(f, x)` / `_rewrite1` / `_rewrite2` — rewrite an integrand as one or a product of two G-functions.
  - `_inflate_g`, `_flip_g`, `_inflate_fox_h` — manipulate G-function parameters (argument inflation, inversion, Fox H reduction).
  - `_split_mul`, `_mul_as_two_parts` — factor an integrand into pieces amenable to G-function rewriting.
- Antecedent checking: `_check_antecedents_1`, `_check_antecedents` — verify convergence conditions (Prudnikov/Luke theorems).
- Integration kernels: `_rewrite_saxena_1`, `_rewrite_saxena`, `_int0oo_1`, `_int0oo` — apply the integral theorems for one or two G-functions over (0, ∞).
- Inversion: `_rewrite_inversion`, `_check_antecedents_inversion`, `_int_inversion`.
- Internal helpers: `_mytype`, `_get_coeff_exp`, `_exponents`, `_functions`, `_find_splitting_points`, `_condsimp`, `_eval_cond`, `_dummy`, `_is_analytic`, `_guess_expansion`.

### `meijerint_doc.py`

Auto-generates a docstring (for Sphinx) listing all function ↔ Meijer G-function identities from the lookup table.

---

## Transforms

### `transforms.py`

Integral transforms: Mellin, Laplace, Fourier, sine/cosine, and Hankel — both forward and inverse.

- **`class IntegralTransform(Function)`** — abstract base for unevaluated transforms; handles linearity, `doit()`, and `as_integral`.
- **`class IntegralTransformError`** — raised when a transform cannot be computed.
- Mellin:
  - `mellin_transform(f, x, s)` / `MellinTransform`
  - `inverse_mellin_transform(F, s, x, strip)` / `InverseMellinTransform`
  - `_rewrite_gamma(f, s, a, b)` — rewrite a product as gamma functions for inverse Mellin.
  - `_rewrite_sin((m, n), s, a, b)` — rewrite sin as gamma pair compatible with strip.
- Laplace:
  - `laplace_transform(f, t, s)` / `LaplaceTransform`
  - `inverse_laplace_transform(F, s, t, plane)` / `InverseLaplaceTransform`
  - `_simplifyconds(expr, s, a)` — simplify convergence conditions given Re(s) > a.
- Fourier:
  - `fourier_transform(f, x, k)` / `FourierTransform`
  - `inverse_fourier_transform(F, k, x)` / `InverseFourierTransform`
- Sine / Cosine:
  - `sine_transform` / `inverse_sine_transform` / `SineTransform` / `InverseSineTransform`
  - `cosine_transform` / `inverse_cosine_transform` / `CosineTransform` / `InverseCosineTransform`
- Hankel:
  - `hankel_transform(f, r, k, nu)` / `HankelTransform`
  - `inverse_hankel_transform(F, k, r, nu)` / `InverseHankelTransform`
- Helpers: `_noconds_` decorator, `_simplify`, `_default_integrator`, `MellinTransformStripError`.

---

## Special-Case Integrators

### `trigonometry.py`

Integrates products of `sin(a*x)^n * cos(a*x)^m` using reduction formulas.

- **`trigintegrate(f, x, conds='piecewise')`** — match `sin(a*x)^n * cos(a*x)^m`, dispatch to odd-exponent substitution or even-exponent reduction.
- `_sin_pow_integrate(n, x)` / `_cos_pow_integrate(n, x)` — recursive power-reduction for sin^n / cos^n.

### `rationaltools.py`

Integration of rational functions via the Lazard-Rioboo-Trager algorithm.

- **`ratint(f, x)`** — indefinite integral of a rational function (polynomial part + Horowitz-Ostrogradsky + log part).
- `ratint_ratpart(f, g, x)` — Horowitz-Ostrogradsky algorithm; splits f/g into A' + B where B has square-free denominator.
- `ratint_logpart(f, g, x, t)` — Lazard-Rioboo-Trager resultant method for the logarithmic part.
- `log_to_atan(f, g)` — convert `I·log((f+Ig)/(f-Ig))` to real arctangents.
- `log_to_real(h, q, x, t)` — convert complex logarithms from the resultant method to real log + atan expressions.

### `deltafunctions.py`

Integration of expressions involving `DiracDelta`.

- **`deltaintegrate(f, x)`** — integrate an expression containing DiracDelta terms (simplifies, extracts simple deltas, evaluates via sifting property).
- `change_mul(node, x)` — rearrange a product to bring a simple `DiracDelta` to the front; if none exists, expand all delta terms.

### `singularityfunctions.py`

Integration of `SingularityFunction` expressions (used in beam mechanics).

- **`singularityintegrate(f, x)`** — integrates `SingularityFunction(x, a, n)` directly; for products/powers rewrites via DiracDelta/Heaviside, integrates, then rewrites back.

---

## Numerical Quadrature

### `quadrature.py`

Gaussian quadrature rules — compute nodes and weights for numerical integration.

- `gauss_legendre(n, n_digits)` — Gauss-Legendre quadrature on [-1, 1].
- `gauss_laguerre(n, n_digits)` — Gauss-Laguerre on [0, ∞) with weight e^(-x).
- `gauss_hermite(n, n_digits)` — Gauss-Hermite on (-∞, ∞) with weight e^(-x²).
- `gauss_gen_laguerre(n, alpha, n_digits)` — generalized Gauss-Laguerre with weight x^α·e^(-x).
- `gauss_chebyshev_t(n, n_digits)` — Chebyshev 1st kind on [-1, 1] with weight 1/√(1-x²).
- `gauss_chebyshev_u(n, n_digits)` — Chebyshev 2nd kind on [-1, 1] with weight √(1-x²).
- `gauss_jacobi(n, alpha, beta, n_digits)` — Gauss-Jacobi on [-1, 1] with weight (1-x)^α(1+x)^β.

---

## Package Plumbing

### `__init__.py`

Re-exports the public API: `integrate`, `Integral`, `line_integrate`, all transform functions and classes, and `singularityintegrate`.

---

## Benchmarks

### `benchmarks/bench_integrate.py`

Benchmarks for `integrate(x^k * sin(x), x)` for k = 0..3.

### `benchmarks/bench_trigintegrate.py`

Benchmarks for `trigintegrate`: `sin(x)^3` and a non-matching case (`x^2`).
