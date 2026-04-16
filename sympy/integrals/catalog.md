# sympy/integrals — Catalog

> Part of [SymPy](../catalog.md). Symbolic integration: definite/indefinite integrals, transforms (Laplace, Fourier, Mellin), and the Risch algorithm.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package init; exports `integrate`, `Integral`, `line_integrate`, and all integral transform functions. |
| `integrals.py` | Core `Integral` class (unevaluated integral representation) and the `integrate` / `line_integrate` entry points that dispatch to various integration strategies. `Integral` also provides: the `transform` method for change-of-variables (u-substitution) on definite integrals; `_eval_derivative` for differentiating integrals with respect to variables (applying the Fundamental Theorem of Calculus for indefinite integrals, and using dummy-variable masking for definite integrals); and `_eval_integral` which orchestrates the full anti-derivative pipeline — including direct handling of linear-base powers `(a*x+b)**c` with conditional piecewise logic for exponent=-1 (log) vs general power rule, polynomial fast-paths, the full Risch algorithm (with `separate_integral=True`), sum-of-terms decomposition, manual integration, Meijer G-functions, and heuristic Risch as a last resort. |
| `risch.py` | Implementation of the Risch algorithm for transcendental function integration, including `DifferentialExtension`, `integer_powers`, and the main `risch_integrate` driver. |
| `rde.py` | Algorithms for solving the Risch Differential Equation (Dy + f*y == g), used as a sub-problem solver by the Risch algorithm. |
| `prde.py` | Algorithms for solving the Parametric Risch Differential Equation (Dy + f*y == Sum(ci*gi)), paralleling `rde.py`. Contains `constant_system` which enforces that solutions of a linear system over a differential field lie in the constant subfield — after RREF, it differentiates rows whose entries depend on the extension variable to generate additional constraint rows. Also contains `prde_linear_constraints` for generating linear constraints on constant parameters, `limited_integrate` / `limited_integrate_reduce` for the limited integration problem (f = Dv + Sum(ci*wi)), and `is_log_deriv_k_t_radical` / `is_log_deriv_k_t_radical_in_field` which use the structure theorem to test whether a derivative is the logarithmic derivative of a radical in a differential extension field. |
| `heurisch.py` | Heuristic Risch algorithm for indefinite integration; provides `heurisch` and `heurisch_wrapper` along with the `components` helper. |
| `manualintegrate.py` | Integration method emulating by-hand techniques with step-by-step rules (namedtuples); provides `integral_steps` and `manualintegrate`. Rules decompose an integrand into named computation steps (e.g., `SubstitutionRule`, `PartsRule`, `PowerRule`, `ExpRule`, `PiecewiseRule`) for solving integrals. This is a downstream strategy called by `_eval_integral` in `integrals.py` — the core conditional power-rule logic (linear base with symbolic exponent, log vs power rule ambiguity) lives in `integrals.py`, not here. |
| `meijerint.py` | Integration by rewriting integrands as Meijer G-functions; exposes `meijerint_indefinite`, `meijerint_definite`, and `meijerint_inversion`. This is the G-function computation backend — it does not define transform classes or handle transform-specific error logic (see `transforms.py` for that). Contains product decomposition utilities: `_mul_as_two_parts` enumerates all ways to split a product into exactly two sub-products for G-function matching (with a special case for two-factor products to preserve canonical ordering), and `_mul_args` which flattens integer powers. `meijerint_inversion` computes inverse Laplace transforms via G-functions. `meijerint_definite` handles definite integration using a multi-stage pipeline: expansion/simplification, G-function rewriting, and a linearity fallback for sums. |
| `meijerint_doc.py` | Auto-generates a Sphinx docstring listing all Meijer G-function lookup table entries for documentation purposes. |
| `transforms.py` | Unevaluated class representations and dispatch entry points for integral transforms: Mellin, inverse Mellin, Laplace, inverse Laplace, Fourier, inverse Fourier, sine, cosine, and Hankel. Contains `_simplifyconds`, a helper for naively simplifying convergence conditions in the Laplace transform backend — its inner `bigger` function compares expression magnitudes with respect to the transform variable, catching TypeError during recursive reciprocal comparisons. Also contains `_sine_cosine_transform`, the shared helper for half-range (0→∞) sine/cosine transforms, and `_rewrite_gamma`, which rewrites products of gamma functions into Meijer G-function parameters for inverse Mellin transforms. |
| `trigonometry.py` | Integration of products of trigonometric functions (sin, cos, tan, sec, csc, cot) via the `trigintegrate` function. |
| `rationaltools.py` | Tools for integrating rational functions: `ratint`, `ratint_ratpart`, and `ratint_logpart` implementing Hermite and Lazard-Rioboo-Trager methods. |
| `deltafunctions.py` | Integration support for Dirac delta functions: `deltaintegrate` and the `change_mul` helper for rearranging delta-containing products. |
| `singularityfunctions.py` | Indefinite integration of `SingularityFunction` expressions via `singularityintegrate`. |
| `intpoly.py` | Integration of uni/bi/trivariate polynomials over 2D and 3D polytopes using the method of Chin et al. (2015); provides `polytope_integrate`. |
| `quadrature.py` | Gaussian quadrature rules: Gauss-Legendre, Gauss-Laguerre, Gauss-Hermite, Gauss-Chebyshev (T and U), Gauss-Jacobi, generalized Gauss-Laguerre, and Gauss-Lobatto. |
| `benchmarks/bench_integrate.py` | Benchmarks for `integrate` with polynomial-times-sin integrands of increasing degree. |
| `benchmarks/bench_trigintegrate.py` | Benchmarks for `trigintegrate` with sin^3(x) and non-trig inputs. |
| `rubi/__init__.py` | Rubi sub-package init; documents the Rule-Based Integration (RUBI) module, its MatchPy dependency, and usage notes. |
| `rubi/rubi.py` | Main Rubi integration entry point; registers MatchPy operations for SymPy types and builds the `ManyToOneReplacer` discrimination net from all rule modules, exposing `rubi_integrate`. |
| `rubi/symbol.py` | Defines `matchpyWC` and the `WC` factory for creating MatchPy wildcard symbols compatible with SymPy's `Symbol` class. |
| `rubi/utility_function.py` | Large collection of utility/helper functions used by Rubi rules (predicates, simplifiers, normalizers, trig helpers, etc.) ported from Mathematica. |
| `rubi/parsetools/parse.py` | Parser for Mathematica `FullForm[DownValues[]]` output; converts Mathematica integration rules into MatchPy `ReplacementRule` format. |
| `rubi/rules/binomial_products.py` | Rubi transformation rules for integrands involving binomial product expressions (a + b*x^n)^p. |
| `rubi/rules/trinomial_products.py` | Rubi transformation rules for integrands involving trinomial product expressions (a + b*x^n + c*x^(2n))^p. |
| `rubi/rules/quadratic_products.py` | Rubi transformation rules for integrands involving quadratic product expressions (a + b*x + c*x^2)^p. |
| `rubi/rules/linear_products.py` | Rubi transformation rules for integrands involving linear expressions and basic algebraic forms. |
| `rubi/rules/integrand_simplification.py` | Rubi rules for simplifying integrands before applying other integration rules. |
| `rubi/rules/exponential.py` | Rubi transformation rules for integrands involving exponential functions F^(g*(e+f*x)). |
| `rubi/rules/logarithms.py` | Rubi transformation rules for integrands involving logarithmic functions. |
| `rubi/rules/sine.py` | Rubi transformation rules for integrands involving sine and cosine functions. |
| `rubi/rules/tangent.py` | Rubi transformation rules for integrands involving tangent and cotangent functions. |
| `rubi/rules/secant.py` | Rubi transformation rules for integrands involving secant and cosecant functions. |
| `rubi/rules/hyperbolic.py` | Rubi transformation rules for integrands involving hyperbolic functions (sinh, cosh, tanh, etc.). |
| `rubi/rules/inverse_trig.py` | Rubi transformation rules for integrands involving inverse trigonometric functions (asin, acos, atan, etc.). |
| `rubi/rules/inverse_hyperbolic.py` | Rubi transformation rules for integrands involving inverse hyperbolic functions (asinh, acosh, atanh, etc.). |
| `rubi/rules/piecewise_linear.py` | Rubi transformation rules for integrands involving piecewise-linear functions. |
| `rubi/rules/miscellaneous_algebraic.py` | Rubi transformation rules for miscellaneous algebraic integrands not covered by other rule modules. |
| `rubi/rules/miscellaneous_trig.py` | Rubi transformation rules for miscellaneous trigonometric integrands not covered by the primary trig rule modules. |
| `rubi/rules/miscellaneous_integration.py` | Rubi transformation rules for miscellaneous integration patterns (catch-all rules). |
