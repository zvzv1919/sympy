# sympy/integrals — Catalog

> Part of [SymPy](../catalog.md). Symbolic integration: definite/indefinite integrals, transforms (Laplace, Fourier, Mellin), and the Risch algorithm.

## Python Files

| File | Summary |
|------|----------|
| `__init__.py` | Package init; exports `integrate`, `Integral`, `line_integrate`, and all integral transform functions. |
| `integrals.py` | Core `Integral` class (unevaluated integral representation), the `transform` method for change-of-variable substitution (u-substitution) on definite/indefinite integrals with automatic bound reversal and integrand negation when mapped bounds are inverted, and the `integrate` / `line_integrate` entry points that dispatch to various integration strategies. |
| `risch.py` | Implementation of the Risch algorithm for transcendental function integration, including `DifferentialExtension`, `integer_powers`, the main `risch_integrate` driver, and rational function verification via `recognize_derivative` (checks if a rational function is the exact derivative of another by verifying divisibility conditions on squarefree factors) and `laurent_series` (computes principal parts of Laurent series expansions). |
| `rde.py` | Algorithms for solving the Risch Differential Equation (Dy + f*y == g), used as a sub-problem solver by the Risch algorithm. |
| `prde.py` | Algorithms for solving the Parametric Risch Differential Equation (Dy + f*y == Sum(ci*gi)), paralleling the methods in `rde.py`. Also includes `constant_system` for solving linear systems over differential fields with solutions restricted to the constant subfield, handling transcendental extensions via row reduction and derivative-based elimination. |
| `heurisch.py` | Heuristic Risch algorithm for indefinite integration; provides `heurisch` and `heurisch_wrapper` along with the `components` helper. |
| `manualintegrate.py` | Integration method emulating by-hand techniques with step-by-step rules (namedtuples); provides `integral_steps` and `manualintegrate`. |
| `meijerint.py` | Integration by rewriting integrands as Meijer G-functions; exposes `meijerint_indefinite`, `meijerint_definite`, and `meijerint_inversion`. Includes internal validity checking for combining hypergeometric functions (handling NaN edge cases in intermediate condition values for degenerate parameters) and lookup table management. |
| `meijerint_doc.py` | Auto-generates a Sphinx docstring listing all Meijer G-function lookup table entries for documentation purposes. |
| `transforms.py` | Integral transforms: Mellin, inverse Mellin, Laplace, inverse Laplace, Fourier, inverse Fourier, sine, cosine, and Hankel transforms with their unevaluated class representations. Helper functions `_rewrite_sin` and `_rewrite_gamma` decompose trigonometric and other expressions into products of gamma functions (generalized factorials) while ensuring the integration contour remains well-defined within the convergence strip. |
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
