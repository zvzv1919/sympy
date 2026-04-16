# sympy/polys — Polynomial Algebra

## Glossary

- **DMP** — Dense Multivariate Polynomial. The primary internal polynomial representation, stored as nested lists of coefficients.
- **DMF** — Dense Multivariate Fraction. Rational function represented as a pair of DMPs.
- **ANP** — Algebraic Number Polynomial. Element of an algebraic number field, stored as a polynomial modulo a minimal polynomial.
- **dup\_/dmp\_** — Function prefixes for univariate (`dup`) and multivariate (`dmp`) dense polynomial operations.
- **gf\_** — Function prefix for Galois field (finite field) polynomial operations.
- **K** — Conventional name for the coefficient domain (ring or field) in low-level routines.
- **PRS** — Polynomial Remainder Sequence.
- **SDM** — Sparse Distributed Module element.

---

## Core

### `__init__.py`
Package entry point; re-exports the public API from `polytools`, `polyfuncs`, `rationaltools`, `polyerrors`, `numberfields`, `monomials`, `orderings`, `rootoftools`, `polyroots`, `domains`, `constructor`, `specialpolys`, `orthopolys`, `partfrac`, `polyoptions`, `rings`, and `fields`.

### `polytools.py`
User-facing symbolic polynomial API — the primary entry point for most polynomial operations.

- **`Poly`** — The main polynomial class wrapping a `DMP` representation with symbolic generators.
  - Construction from expressions, dicts, or lists
  - Arithmetic: add, sub, mul, pow, div, rem, quo, exquo
  - Calculus: `diff`, `integrate`, `eval`
  - Factorization: `factor_list`, `sqf_list`, `sqf_part`
  - GCD/LCM, resultants, discriminants, subresultants
  - Root isolation: `intervals`, `refine_root`, `count_roots`, `real_roots`, `nroots`
  - Composition & decomposition
  - Conversion between representations and domains
- **`PurePoly`** — `Poly` subclass that compares equal regardless of generators (used for canonical comparison).
- **`GroebnerBasis`** — Container for a computed Groebner basis with reduction and membership testing.
- Module-level functions mirror `Poly` methods for direct use on expressions:
  - `degree`, `LC`, `LM`, `LT`
  - `div`, `rem`, `quo`, `exquo`, `pdiv`, `prem`, `pquo`, `pexquo`
  - `half_gcdex`, `gcdex`, `invert`
  - `subresultants`, `resultant`, `discriminant`
  - `cofactors`, `gcd`, `gcd_list`, `lcm`, `lcm_list`
  - `terms_gcd`, `trunc`, `monic`, `content`, `primitive`
  - `compose`, `decompose`, `sturm`
  - `sqf_norm`, `sqf_part`, `sqf_list`, `sqf`, `factor_list`, `factor`
  - `intervals`, `refine_root`, `count_roots`, `real_roots`, `nroots`
  - `cancel`, `reduced`, `groebner`, `is_zero_dimensional`
  - `poly` — convenience wrapper to create a `Poly` from an expression
- `to_rational_coeffs` — rescale a polynomial to have rational coefficients when possible.

### `polyclasses.py`
Low-level OO wrappers around the dense polynomial list representation.

- **`GenericPoly`** — Base class with `ground_to_ring/field/exact`.
- **`DMP`** — Dense multivariate polynomial over a domain. Delegates to `densebasic`/`densearith`/`densetools`/`euclidtools`/`sqfreetools`/`factortools` functions. Supports full arithmetic, GCD, factoring, evaluation, etc.
- **`DMF`** — Dense multivariate fraction (numerator/denominator pair of DMPs). Arithmetic with automatic cancellation.
- **`ANP`** — Algebraic number represented as a polynomial modulo a minimal polynomial. Supports field arithmetic in algebraic extensions.

### `polyoptions.py`
Options manager for `Poly` and public API functions.

- **`Options`** — dict subclass collecting and validating keyword arguments (`domain`, `field`, `gaussian`, `extension`, `modulus`, `symmetric`, `order`, `gens`, etc.).
- **`Option`**, **`Flag`**, **`BooleanOption`** — base classes for individual option types.
- `build_options`, `allowed_flags`, `set_defaults` — helper functions.

### `polyerrors.py`
Exception hierarchy for the polynomial module.

- **`BasePolynomialError`** — root exception.
- Key exceptions: `ExactQuotientFailed`, `PolynomialDivisionFailed`, `OperationNotSupported`, `HeuristicGCDFailed`, `ModularGCDFailed`, `HomomorphismFailed`, `IsomorphismFailed`, `CoercionFailed`, `NotInvertible`, `NotReversible`, `NotAlgebraic`, `DomainError`, `PolynomialError`, `UnificationFailed`, `GeneratorsNeeded`, `ComputationFailed`, `PolificationFailed`, `OptionError`, `FlagError`.

### `polyconfig.py`
Runtime configuration for polynomial algorithms (e.g. GCD method, factoring method, Groebner algorithm).

- `setup(key, value)` / `query(key)` / `using(**kwargs)` context manager
- Reads env vars prefixed with `SYMPY_` on import.

### `polyutils.py`
Shared utility functions used throughout the polynomial subsystem.

- `_nsort(roots)` — numerically sort roots (real first, then by real/imaginary parts).
- `_sort_gens`, `_unify_gens`, `_analyze_gens` — generator ordering/unification.
- `_dict_from_expr`, `_parallel_dict_from_expr` — convert SymPy expressions to polynomial coefficient dicts.
- `_dict_reorder` — reorder a polynomial dict to a new generator ordering.
- `_sort_factors` — sort factorization results.
- `PicklableWithSlots` — mixin for pickling `__slots__`-based classes.

### `constructor.py`
Automatic domain construction from expression coefficients.

- `construct_domain(coeffs, **opts)` — infer the best coefficient domain (ZZ, QQ, RR, algebraic, EX, or composite) for a set of coefficients.
- Internal: `_construct_simple`, `_construct_algebraic`, `_construct_composite`, `_construct_expression`.

---

## Monomials & Orderings

### `monomials.py`
Tools and arithmetic for monomials (exponent tuples).

- `itermonomials(variables, degree)` — generate all monomials up to a given total degree.
- `monomial_count(V, N)` — count of monomials of degree ≤ N in V variables.
- Tuple arithmetic: `monomial_mul`, `monomial_div`, `monomial_ldiv`, `monomial_pow`, `monomial_gcd`, `monomial_lcm`, `monomial_divides`, `monomial_max`, `monomial_min`, `monomial_deg`.
- `term_div(a, b, domain)` — divide terms (coefficient × monomial pairs).
- **`MonomialOps`** — JIT-compiled monomial operation cache for a given number of variables.
- **`Monomial`** — Symbolic monomial class with SymPy integration.

### `orderings.py`
Monomial ordering definitions.

- **`MonomialOrder`** — abstract base.
- **`LexOrder`** (`lex`), **`GradedLexOrder`** (`grlex`), **`ReversedGradedLexOrder`** (`grevlex`).
- **`ProductOrder`** — composite order built from component orders on variable subsets.
- **`InverseOrder`** — reverses any ordering (produces `ilex`, `igrlex`, `igrevlex`).
- `monomial_key(order, gens)` — build a sort key function for a given ordering.
- `build_product_order` — construct product orderings from a specification.

---

## Dense Polynomial Internals

### `densebasic.py`
Basic operations on dense recursive polynomial representations (nested coefficient lists).

- Constructors/converters: `dmp_normal`, `dmp_convert`, `dmp_from_sympy`, `dup_from_dict`, `dmp_from_dict`, `dmp_to_dict`
- Degree: `dup_degree`, `dmp_degree`, `dmp_degree_in`, `dmp_degree_list`
- Coefficients: `dup_LC`/`dmp_LC`, `dup_TC`/`dmp_TC`, `dmp_ground_LC`, `dmp_ground_TC`, `dmp_ground_nth`
- Predicates: `dmp_zero_p`, `dmp_one_p`, `dmp_ground_p`, `dmp_negative_p`
- Zeros/ones: `dmp_zero`, `dmp_one`, `dmp_ground`, `dmp_zeros`
- Structural: `dup_strip`, `dmp_strip`, `dmp_validate`, `dmp_raise`, `dmp_nest`, `dmp_inject`, `dmp_eject`, `dmp_permute`, `dmp_exclude`, `dmp_include`, `dmp_deflate`, `dmp_inflate`, `dmp_multi_deflate`, `dmp_slice_in`
- Listing: `dmp_list_terms`, `dmp_terms_gcd`
- `dup_random` — random polynomial generation.

### `densearith.py`
Arithmetic on dense recursive polynomials.

- Term operations: `dup_add_term`, `dmp_add_term`, `dup_sub_term`, `dmp_sub_term`, `dup_mul_term`, `dmp_mul_term`
- Ground operations: `dmp_add_ground`, `dmp_sub_ground`, `dmp_mul_ground`, `dmp_quo_ground`, `dmp_exquo_ground`
- Unary: `dmp_abs`, `dup_neg`, `dmp_neg`
- Binary arithmetic: `dup_add`/`dmp_add`, `dup_sub`/`dmp_sub`, `dup_mul`/`dmp_mul`, `dmp_sqr`, `dup_pow`/`dmp_pow`
- Combined: `dup_add_mul`, `dmp_add_mul`, `dup_sub_mul`, `dmp_sub_mul`
- Division: `dup_pdiv`/`dmp_pdiv` (pseudo-division), `dup_prem`/`dmp_prem`, `dup_pquo`/`dmp_pquo`, `dup_div`/`dmp_div`, `dup_rem`/`dmp_rem`, `dup_quo`/`dmp_quo`, `dup_exquo`/`dmp_exquo`, `dup_rr_div`/`dmp_rr_div` (ring div), `dup_ff_div`/`dmp_ff_div` (field div)
- Norms: `dup_max_norm`, `dmp_max_norm`, `dup_l1_norm`, `dmp_l1_norm`
- Shifts: `dup_lshift`, `dup_rshift`
- `dmp_expand` — multiply a list of polynomials.

### `densetools.py`
Advanced operations on dense recursive polynomials.

- Calculus: `dup_integrate`/`dmp_integrate`, `dup_diff`/`dmp_diff`, `dmp_integrate_in`, `dmp_diff_in`
- Evaluation: `dup_eval`/`dmp_eval`, `dmp_eval_in`, `dmp_eval_tail`, `dmp_diff_eval_in`
- Normalization: `dup_trunc`/`dmp_trunc`/`dmp_ground_trunc`, `dup_monic`/`dmp_ground_monic`, `dup_content`/`dmp_ground_content`, `dup_primitive`/`dmp_ground_primitive`, `dup_extract`/`dmp_ground_extract`
- Transformations: `dup_mirror`, `dup_scale`, `dup_shift`, `dup_transform`, `dup_compose`/`dmp_compose`
- Decomposition: `dup_decompose` — functional decomposition f = g ∘ h.
- `dup_real_imag` — split polynomial into real/imaginary parts.
- `dup_sign_variations` — count sign changes (Descartes' rule).
- `dup_clear_denoms`/`dmp_clear_denoms` — clear denominators over QQ.
- `dup_revert` — compute the inverse power series truncated to order n.
- `dmp_lift` — lift algebraic extension coefficients.

---

## Algorithms — GCD & Euclidean

### `euclidtools.py`
Euclidean algorithms, GCDs, LCMs, and polynomial remainder sequences.

- Extended GCD: `dup_half_gcdex`, `dup_gcdex`, `dmp_half_gcdex`, `dmp_gcdex`
- Inversion: `dup_invert`, `dmp_invert`
- PRS algorithms: `dup_euclidean_prs`, `dup_primitive_prs`, `dmp_euclidean_prs`, `dmp_primitive_prs`
- Subresultants: `dup_inner_subresultants`, `dup_subresultants`, `dmp_inner_subresultants`, `dmp_subresultants`
- Resultants: `dup_resultant`, `dmp_resultant`, `dmp_prs_resultant`, `dmp_zz_modular_resultant`, `dmp_zz_collins_resultant`, `dmp_qq_collins_resultant`
- Discriminants: `dup_discriminant`, `dmp_discriminant`
- GCD (multiple strategies): `dup_rr_prs_gcd`, `dup_ff_prs_gcd`, `dmp_rr_prs_gcd`, `dmp_ff_prs_gcd`, `dup_zz_heu_gcd`, `dmp_zz_heu_gcd`, `dup_qq_heu_gcd`, `dmp_qq_heu_gcd`
- Unified GCD entry points: `dup_inner_gcd`, `dmp_inner_gcd`, `dup_gcd`, `dmp_gcd`
- LCM: `dup_lcm`, `dmp_lcm`, `dup_rr_lcm`, `dup_ff_lcm`, `dmp_rr_lcm`, `dmp_ff_lcm`
- Content & primitive part: `dmp_content`, `dmp_primitive`
- Cancellation: `dup_cancel`, `dmp_cancel`

### `heuristicgcd.py`
Heuristic polynomial GCD (HEUGCD) in Z[X].

- `heugcd(f, g)` — compute GCD by evaluating at large points, taking integer GCD, and interpolating back. Falls back with `HeuristicGCDFailed` if the heuristic fails.

### `modulargcd.py`
Modular GCD algorithms using CRT-based reconstruction.

- `modgcd_univariate(f, g)` — modular GCD for univariate integer polynomials.
- `modgcd_bivariate(f, g)` — modular GCD for bivariate integer polynomials.
- `modgcd_multivariate(f, g)` — modular GCD for general multivariate integer polynomials.
- `func_field_modgcd(f, g)` — modular GCD over algebraic function fields (polynomials over algebraic number fields).
- Internal helpers: `_trivial_gcd`, `_gf_gcd`, `_degree_bound_univariate`, `_chinese_remainder_reconstruction_univariate`, `_rational_function_reconstruction`, `_func_field_modgcd_p`, `_func_field_modgcd_m`, etc.

---

## Algorithms — Factoring & Square-Free

### `factortools.py`
Polynomial factorization routines in characteristic zero.

- **Hensel lifting**: `dup_zz_hensel_step`, `dup_zz_hensel_lift`
- **Zassenhaus factoring**: `dup_zz_zassenhaus` — factor over ZZ using modular approach + subset recombination.
- **Wang's algorithm**: `dmp_zz_wang` — multivariate factoring over ZZ via evaluation, univariate factoring, and Hensel lifting. Supporting functions: `dmp_zz_wang_non_divisors`, `dmp_zz_wang_test_points`, `dmp_zz_wang_lead_coeffs`, `dmp_zz_wang_hensel_lifting`.
- **Diophantine solvers** for Hensel lifting: `dup_zz_diophantine`, `dmp_zz_diophantine`
- Cyclotomic: `dup_cyclotomic_p`, `dup_zz_cyclotomic_poly`, `dup_zz_cyclotomic_factor`
- Irreducibility: `dup_zz_irreducible_p`, `dup_irreducible_p`, `dmp_irreducible_p`
- Extension field factoring: `dup_ext_factor`, `dmp_ext_factor`
- Galois field factoring: `dup_gf_factor`
- Unified entry points: `dup_factor_list`, `dmp_factor_list`, `dup_factor_list_include`, `dmp_factor_list_include`
- Bounds: `dup_zz_mignotte_bound`, `dmp_zz_mignotte_bound`
- Trial division: `dup_trial_division`, `dmp_trial_division`

### `sqfreetools.py`
Square-free decomposition algorithms.

- `dup_sqf_p`/`dmp_sqf_p` — test if polynomial is square-free.
- `dup_sqf_norm`/`dmp_sqf_norm` — compute square-free norm (for algebraic extensions).
- `dup_sqf_part`/`dmp_sqf_part` — extract the square-free part.
- `dup_sqf_list`/`dmp_sqf_list` — full square-free decomposition returning list of (factor, multiplicity) pairs.
- `dup_sqf_list_include`/`dmp_sqf_list_include` — include content in the result.
- `dup_gff_list`/`dmp_gff_list` — greatest factorial factorization.

---

## Algorithms — Groebner Bases

### `groebnertools.py`
Groebner basis computation algorithms.

- `groebner(seq, ring, method)` — main entry point; dispatches to `_buchberger` or `_f5b`.
- `_buchberger(f, ring)` — improved Buchberger algorithm with selection strategies and critical pair criteria (lcm, Gebauer-Moller).
- `_f5b(f, ring)` — F5B algorithm (signature-based, more efficient for many inputs).
- Internal: `lbp` (labeled polynomial), `sig` (signature), `Polyn`, `Num`, `cp` (critical pair), `critical_pair`, `s_poly`, `f5_reduce`, `is_rewritable_or_comparable`, `is_connected`.

### `fglmtools.py`
FGLM algorithm for Groebner basis conversion between monomial orderings.

- `matrix_fglm(F, ring, O_to)` — convert a reduced Groebner basis of a zero-dimensional ideal from one ordering to another using the matrix-based FGLM method.
- Internal: `_basis`, `_representing_matrices`, `_update`, `_identity_matrix`, `_matrix_mul`, `_incr_k`.

---

## Algorithms — Galois Fields

### `galoistools.py`
Dense univariate polynomials over Galois fields (GF(p)).

- **CRT**: `gf_crt`, `gf_crt1`, `gf_crt2` — Chinese Remainder Theorem.
- **Basic**: `gf_int`, `gf_degree`, `gf_LC`, `gf_TC`, `gf_strip`, `gf_trunc`, `gf_normal`
- **Conversion**: `gf_from_dict`, `gf_to_dict`, `gf_from_int_poly`, `gf_to_int_poly`
- **Arithmetic**: `gf_neg`, `gf_add`, `gf_sub`, `gf_mul`, `gf_sqr`, `gf_pow`, `gf_div`, `gf_rem`, `gf_quo`, `gf_exquo`, `gf_expand`, `gf_add_ground`, `gf_sub_ground`, `gf_mul_ground`, `gf_quo_ground`, `gf_lshift`, `gf_rshift`
- **GCD/LCM**: `gf_gcd`, `gf_lcm`, `gf_cofactors`, `gf_gcdex`, `gf_monic`
- **Calculus**: `gf_diff`, `gf_eval`, `gf_multi_eval`
- **Composition**: `gf_compose`, `gf_compose_mod`
- **Frobenius**: `gf_frobenius_monomial_base`, `gf_frobenius_map`, `gf_trace_map`
- **Irreducibility**: `gf_irred_p_ben_or`, `gf_irred_p_rabin`, `gf_irreducible_p`, `gf_irreducible`, `gf_random`
- **Square-free**: `gf_sqf_p`, `gf_sqf_part`, `gf_sqf_list`
- **Factoring (Berlekamp)**: `gf_Qmatrix`, `gf_Qbasis`, `gf_berlekamp`
- **Factoring (Zassenhaus)**: `gf_ddf_zassenhaus`, `gf_edf_zassenhaus`, `gf_zassenhaus`
- **Factoring (Shoup)**: `gf_ddf_shoup`, `gf_edf_shoup`, `gf_shoup`
- **Unified**: `gf_factor_sqf`, `gf_factor`
- **Congruences**: `gf_value`, `linear_congruence`, `csolve_prime`, `gf_csolve`
- `gf_pow_mod` — modular exponentiation of polynomials.

---

## Roots & Isolation

### `polyroots.py`
Symbolic root-finding algorithms for polynomials.

- `roots_linear`, `roots_quadratic`, `roots_cubic(f, trig=False)`, `roots_quartic`, `roots_quintic` — closed-form solutions by degree.
- `roots_binomial` — roots of binomial polynomials x^n - a.
- `roots_cyclotomic` — roots of cyclotomic polynomials.
- `roots(f, *gens, **flags)` — main entry point; attempts exact roots using all available methods, returns a dict of root → multiplicity.
- `preprocess_roots` — rescale/shift polynomial for better root finding.
- `root_factors` — factor a polynomial into linear factors over the splitting field.
- `_integer_basis` — find an integer rescaling to integralize roots.

### `rootoftools.py`
Indexed algebraic roots and root sum expressions.

- `rootof(f, x, index, radicals, expand)` — factory that returns `CRootOf` or an explicit radical.
- **`RootOf`** — base class for indexed roots.
- **`CRootOf`** (ComplexRootOf) — represents the i-th complex root of an irreducible polynomial, ordered by real part then imaginary part.
  - Caches isolated real and complex root intervals.
  - Numerical evaluation via interval refinement or Newton's method.
  - `_get_roots` dispatches between `_real_roots` and `_all_complex_roots`.
- **`RootSum`** — represents a sum over roots: Σ f(α_i). Supports partial evaluation and `doit()` for explicit expansion.

### `rootisolation.py`
Real and complex root isolation and refinement algorithms.

- `dup_sturm` — Sturm sequence computation.
- `dup_root_upper_bound`, `dup_root_lower_bound` — Cauchy-type root bounds.
- Real root isolation:
  - `dup_isolate_real_roots_sqf` — isolate real roots of a square-free polynomial using continued fractions / bisection.
  - `dup_isolate_real_roots` — handles non-square-free input via square-free decomposition.
  - `dup_isolate_real_roots_list` — isolate roots of multiple polynomials simultaneously (disjoint intervals).
  - `dup_refine_real_root`, `dup_inner_refine_real_root` — refine a root interval to a given precision.
- Complex root isolation:
  - `dup_isolate_complex_roots_sqf` — isolate complex roots in rectangular regions using winding numbers.
  - `dup_count_complex_roots` — count roots in a rectangular region.
- `dup_count_real_roots` — count real roots in an interval (Sturm's theorem).
- **`RealInterval`**, **`ComplexInterval`** — interval objects for root refinement.

---

## Algebraic Number Fields

### `numberfields.py`
Computational algebraic number field theory.

- `minimal_polynomial(ex, x, **args)` — compute the minimal polynomial of an algebraic expression.
  - Handles radicals, trig values (sin/cos of rational multiples of π), exponentials.
  - `_minpoly_compose` — recursive decomposition into sub-problems.
  - `_minpoly_groebner` — fallback via Groebner basis computation.
- `primitive_element(extension, x)` — find a primitive element for a list of algebraic extensions.
- `field_isomorphism(a, b)` — find an isomorphism between algebraic number fields.
  - Two methods: `field_isomorphism_pslq` (numerical, via PSLQ), `field_isomorphism_factor` (symbolic, via factoring).
- `to_number_field(extension, theta)` — express elements in terms of a given primitive element.
- `isolate(alg, eps, fast)` — isolate the real/complex interval for an algebraic number.
- `is_isomorphism_possible(a, b)` — quick check via norm/degree conditions.

---

## Partial Fractions & Rational Functions

### `partfrac.py`
Partial fraction decomposition of rational functions.

- `apart(f, x, full=False)` — main entry point.
  - Default: undetermined coefficients method (requires rational roots).
  - `full=True`: Bronstein's algorithm (handles irrational roots, returns `RootSum`).
- `apart_undetermined_coeffs`, `apart_full_decomposition` — internal implementations.
- `apart_list(f, x, dummies)` — structured partial fraction list output.
- `assemble_partfrac_list(partial_list)` — reassemble from structured list.

### `rationaltools.py`
Rational expression manipulation.

- `together(expr, deep=False)` — combine rational subexpressions over a common denominator. Complement of `apart`.

---

## Polynomial Generators

### `specialpolys.py`
Functions for generating special and benchmark polynomials.

- `swinnerton_dyer_poly(n, x)` — n-th Swinnerton-Dyer polynomial.
- `cyclotomic_poly(n, x)` — n-th cyclotomic polynomial.
- `symmetric_poly(n, *gens)` — elementary symmetric polynomial of degree n.
- `random_poly(x, n, inf, sup, domain)` — random polynomial with integer coefficients.
- `interpolating_poly(n, x, X, Y)` — Lagrange interpolating polynomial.
- `fateman_poly_F_1/F_2/F_3` — Fateman benchmark polynomials (both expression and DMP forms).
- `f_polys()`, `w_polys()` — predefined test polynomial collections.

### `orthopolys.py`
Efficient generation of classical orthogonal polynomials.

- `jacobi_poly(n, a, b, x)` / `dup_jacobi`
- `gegenbauer_poly(n, a, x)` / `dup_gegenbauer`
- `chebyshevt_poly(n, x)` / `dup_chebyshevt` — Chebyshev of the first kind.
- `chebyshevu_poly(n, x)` / `dup_chebyshevu` — Chebyshev of the second kind.
- `hermite_poly(n, x)` / `dup_hermite`
- `legendre_poly(n, x)` / `dup_legendre`
- `laguerre_poly(n, x, alpha)` / `dup_laguerre`
- `spherical_bessel_fn(n, x)` / `dup_spherical_bessel_fn`

### `polyfuncs.py`
High-level polynomial manipulation functions.

- `symmetrize(F, *gens)` — rewrite a polynomial in terms of elementary symmetric polynomials.
- `horner(f, *gens)` — convert polynomial to Horner form for efficient evaluation.
- `interpolate(data, x)` — interpolate through given data points.
- `viete(f, *gens)` — Vieta's formulas relating roots and coefficients.

---

## Ring Series

### `ring_series.py`
Power series manipulation using sparse polynomial rings (truncated formal power series).

- **Arithmetic**: `rs_mul`, `rs_square`, `rs_pow`, `rs_trunc`
- **Substitution**: `rs_subs`
- **Inversion**: `rs_series_inversion` — multiplicative inverse of a series.
- **Reversion**: `rs_series_reversion` — compositional inverse.
- **Calculus**: `rs_diff`, `rs_integrate`
- **Elementary functions**: `rs_exp`, `rs_log`, `rs_sin`, `rs_cos`, `rs_cos_sin`, `rs_tan`, `rs_cot`, `rs_atan`, `rs_asin`, `rs_sinh`, `rs_cosh`, `rs_tanh`, `rs_atanh`, `rs_LambertW`, `rs_nth_root`
- **Composition**: `rs_compose_add` — resultant-based addition of series.
- **Newton's method**: `rs_newton` — solve functional equations via Newton iteration.
- **Hadamard product**: `rs_hadamard_exp`
- **Puiseux series**: `rs_is_puiseux`, `rs_puiseux`, `rs_puiseux2` — handle fractional exponents.
- `rs_series(expr, a, prec)` — high-level: expand a SymPy expression as a ring series.
- `rs_fun(p, f, *args)` — apply a ring series function generically.
- `mul_xin`, `pow_xin` — multiply/raise x_i^n in a multivariate polynomial.

**Caveats**: Algorithms are designed for Taylor series; Puiseux series require the `rs_puiseux` wrappers.

---

## Subresultants

### `subresultants_qq_zz.py`
Comprehensive implementations of polynomial remainder sequences (PRS) and subresultant algorithms over QQ and ZZ.

- **Sylvester matrix**: `sylvester(f, g, x, method)` — constructs Sylvester matrices (two variants).
- **Bezout matrix**: `bezout(p, q, x, method)` — constructs Bezout matrices (three methods: `bz`, `prs`, `modified`).
- **Euclidean PRS**: `euclid_pg`, `euclid_q`, `euclid_amv`
- **Sturmian PRS**: `sturm_pg`, `sturm_q`, `sturm_amv`
- **Subresultant PRS**: `subresultants_pg`, `subresultants_amv`, `subresultants_amv_q`, `subresultants_rem`, `subresultants_vv`, `subresultants_vv_2`, `subresultants_bezout`
- **Modified subresultant PRS**: `modified_subresultants_pg`, `modified_subresultants_amv`, `modified_subresultants_bezout`
- Utility: `sign_seq`, `rem_z`, `quo_z`, `process_bezout_output`

**Caveats**: This module does *not* use SymPy's `prem()`; it uses its own `rem_z()` for pseudo-remainder.

---

## Distributed Modules

### `distributedmodules.py`
Sparse distributed elements of free modules over polynomial rings.

Implements monomial-module operations and Groebner basis computation for modules (generalizing Groebner bases from ideals to submodules). Used internally by the AGCA sub-package.

- Monomial operations: `sdm_monomial_mul`, `sdm_monomial_deg`, `sdm_monomial_lcm`, `sdm_monomial_divides`
- Element operations: `sdm_LC`, `sdm_LM`, `sdm_LT`, `sdm_add`, `sdm_mul_term`, `sdm_sort`, `sdm_strip`, `sdm_deg`, `sdm_ecart`
- Conversion: `sdm_to_dict`, `sdm_from_dict`, `sdm_from_vector`, `sdm_to_vector`
- S-polynomials: `sdm_spoly`
- Normal forms: `sdm_nf_mora` (Mora's tangent cone algorithm), `sdm_nf_buchberger`, `sdm_nf_buchberger_reduced`
- `sdm_groebner(G, NF, O, K, extended)` — compute a Groebner basis for a submodule.

---

## Miscellaneous

### `dispersion.py`
Dispersion set and dispersion of polynomials.

- `dispersionset(p, q, *gens)` — compute the set {a ∈ N₀ | gcd(f(x), g(x+a)) ≠ 1}.
- `dispersion(p, q, *gens)` — max of the dispersion set (or -∞ if empty).

### `solvers.py`
Low-level linear system solver over polynomial rings.

- `eqs_to_matrix(eqs, ring)` — convert polynomial equations to an augmented matrix.
- `solve_lin_sys(eqs, ring, _raw)` — solve linear systems via row-reduction (RREF).
- **`RawMatrix`** — `Matrix` subclass that skips sympification.

### `compatibility.py`
Compatibility layer between dense and sparse polynomial representations.

Provides the **`IPolys`** mixin class that wraps all `dup_`/`dmp_` functions from `densebasic`, `densearith`, `densetools`, `euclidtools`, `sqfreetools`, and `factortools` as methods delegating to the sparse ring's internal representation. This allows `PolyRing` to expose the dense polynomial API.

### `polyquinticconst.py`
Constants and formulas for solvable quintic equations (Dummit's algorithm).

- **`PolyQuintic`** — computes resolvent polynomials (`f20`), discriminant-related quantities (`b`, `c`, `d`, `e`, `f`, `l`), and theta values needed for the radical solution of a solvable quintic.

---

## AGCA — Algebraic Geometry & Commutative Algebra

### `agca/__init__.py`
Package entry point; exports `homomorphism`.

### `agca/ideals.py`
Computations with ideals of polynomial rings.

- **`Ideal`** — abstract base class for ideals. Defines containment, quotient, intersection, union, product, and ideal-theoretic predicates (`is_zero`, `is_whole_ring`, `is_prime`, `is_maximal`, `is_primary`, `is_radical`, `is_principal`).
- **`ModuleImplementedIdeal`** — concrete implementation using the module machinery; an ideal is a rank-1 submodule. All operations reduce to module operations (Groebner bases under the hood).

### `agca/modules.py`
Computations with modules over polynomial rings.

- **`Module`** — abstract base (ring, dtype, submodule, quotient_module, is_zero, is_submodule, multiply_ideal).
- **`ModuleElement`** — abstract element base.
- **`FreeModule`** — abstract free module. **`FreeModulePolyRing`** (over `PolynomialRing`) and **`FreeModuleQuotientRing`** (over `QuotientRing`) are concrete implementations.
- **`SubModule`** — abstract submodule. **`SubModulePolyRing`** (Groebner basis–based, supports syzygy computation, containment, inclusion, intersection, module quotient) and **`SubModuleQuotientRing`** are concrete.
- **`SubQuotientModule`** — submodule of a quotient module.
- **`QuotientModule`** / **`QuotientModuleElement`** — M/N construction.
- **`ModuleOrder`** — product order for module monomials.

### `agca/homomorphisms.py`
Module and ring homomorphisms.

- **`ModuleHomomorphism`** — abstract base with kernel, image, restrict, quotient, compose, add, multiply, equality, surjectivity, injectivity, isomorphism checks.
- **`MatrixHomomorphism`** — homomorphism specified by a matrix (images of generators).
- **`FreeModuleHomomorphism`** — from a free module.
- **`SubModuleHomomorphism`** — from a submodule.
- `homomorphism(domain, codomain, matrix)` — public factory function.

---

## Domains — Mathematical Coefficient Domains

### `domains/__init__.py`
Package entry point; assembles the domain hierarchy and instantiates global singletons.

- Singletons: `ZZ` (integers), `QQ` (rationals), `RR` (reals), `CC` (complex), `EX` (expression domain).
- Factory: `FF`/`GF` (finite fields).
- Backend selection: Python-native vs GMPY-based implementations chosen at import time via `GROUND_TYPES`.

### `domains/domain.py`
Abstract base class for all coefficient domains.

- **`Domain`** — defines the domain interface: `convert`, `convert_from`, `unify`, algebraic properties (`has_Ring`, `has_Field`, `is_Exact`, `is_Numerical`, etc.), factory methods (`poly_ring`, `frac_field`, `old_poly_ring`, `old_frac_field`, `algebraic_field`), arithmetic stubs, and domain navigation (`get_ring`, `get_field`, `get_exact`).

### `domains/ring.py`
- **`Ring`** — domain with `has_Ring = True`. Adds `exquo`, `quo`, `rem`, `div`, `invert`, `revert`, `is_unit`, `ideal`, `quotient_ring`, `free_module`.

### `domains/field.py`
- **`Field`** — domain with `has_Field = True` (extends Ring). Exact division, GCD via associated ring, `free_module`.

### `domains/simpledomain.py`
- **`SimpleDomain`** — base for ground domains (ZZ, QQ, etc.). `inject(*gens)` creates a polynomial ring.

### `domains/compositedomain.py`
- **`CompositeDomain`** — base for polynomial rings and fraction fields. Tracks generators and base domain.

### `domains/characteristiczero.py`
- **`CharacteristicZero`** — mixin setting `has_CharacteristicZero = True`.

### `domains/domainelement.py`
- **`DomainElement`** — mixin trait; `parent()` returns the containing domain.

### `domains/finitefield.py`
- **`FiniteField`** — GF(p) using modular integers. Wraps `ModularIntegerFactory`.

### `domains/integerring.py`
- **`IntegerRing`** — abstract ZZ. `get_field()` returns QQ. `algebraic_field()` creates QQ(α).

### `domains/rationalfield.py`
- **`RationalField`** — abstract QQ. `algebraic_field(*ext)` creates QQ(α).

### `domains/realfield.py`
- **`RealField`** — multiprecision real numbers (backed by mpmath). Not exact.

### `domains/complexfield.py`
- **`ComplexField`** — multiprecision complex numbers (backed by mpmath). Not exact.

### `domains/algebraicfield.py`
- **`AlgebraicField`** — QQ(α), representing an algebraic number field. Elements are `ANP` objects.

### `domains/polynomialring.py`
- **`PolynomialRing`** — K[x₁,…,xₙ] as a domain, backed by `PolyRing`.

### `domains/fractionfield.py`
- **`FractionField`** — K(x₁,…,xₙ) as a domain, backed by `FracField`.

### `domains/expressiondomain.py`
- **`ExpressionDomain`** (EX) — fallback domain wrapping arbitrary SymPy expressions. Not exact, not efficient.

### `domains/quotientring.py`
- **`QuotientRing`** — R/I construction from a ring and ideal.
- **`QuotientRingElement`** — element class with coerced arithmetic.

### `domains/groundtypes.py`
Ground type selection: imports `PythonInteger`/`PythonRational` or `GMPYInteger`/`GMPYRational` depending on availability. Also provides `python_sqrt`, `python_factorial`.

### `domains/modularinteger.py`
- **`ModularInteger`** — element of Z/nZ with full arithmetic.
- `ModularIntegerFactory(mod, dom, symmetric)` — dynamically creates a `ModularInteger` subclass for a given modulus.

### `domains/mpelements.py`
Multiprecision real and complex number elements backed by mpmath.

- **`RealElement`**, **`ComplexElement`** — element types for `RealField` and `ComplexField`.
- **`MPContext`** — custom mpmath context with domain-aware element types.

### `domains/pythonrational.py`
- **`PythonRational`** — pure-Python rational number type (faster than `fractions.Fraction`).

### `domains/pythonintegerring.py`
- **`PythonIntegerRing`** — ZZ backed by Python `int`.

### `domains/gmpyintegerring.py`
- **`GMPYIntegerRing`** — ZZ backed by GMPY `mpz`.

### `domains/pythonrationalfield.py`
- **`PythonRationalField`** — QQ backed by `PythonRational`.

### `domains/gmpyrationalfield.py`
- **`GMPYRationalField`** — QQ backed by GMPY `mpq`.

### `domains/pythonfinitefield.py`
- **`PythonFiniteField`** — GF(p) using Python integers.

### `domains/gmpyfinitefield.py`
- **`GMPYFiniteField`** — GF(p) using GMPY integers.

### `domains/old_polynomialring.py`
Legacy polynomial ring domain using dense `DMP` representation.

- **`PolynomialRingBase`** — base class with `free_module`, `ideal`, `quotient_ring`.
- **`GlobalPolynomialRing`** — global ordering (standard poly ring).
- **`GeneralizedPolynomialRing`** — supports non-global orderings (e.g. local/mixed orderings for localization).

### `domains/old_fractionfield.py`
Legacy fraction field domain using dense `DMF` representation.

- **`FractionField`** — K(x₁,…,xₙ) backed by `DMF`.

---

## Sparse Polynomial Rings & Fields

### `rings.py`
Sparse polynomial rings using dict-based representation.

- `ring(symbols, domain, order)` — construct `(ring, x1, ..., xn)`.
- `xring(symbols, domain, order)` — returns `(ring, (x1, ..., xn))`.
- `vring(symbols, domain, order)` — injects generators into global namespace.
- `sring(exprs, *symbols, **options)` — construct ring from SymPy expressions.
- **`PolyRing`** — the ring object. Manages generators, domain, ordering, and caches `MonomialOps`. Provides `ring_new`, `ground_new`, `from_dict`, `from_expr`, and all `dup_`/`dmp_` methods via `IPolys` compatibility mixin.
- **`PolyElement`** — ring element (subclass of `dict`, mapping monomials to coefficients). Full arithmetic, GCD, content/primitive, evaluation, composition, diff/integrate, and Groebner-based operations.

### `fields.py`
Sparse rational function fields.

- `field(symbols, domain, order)` — construct `(field, x1, ..., xn)`.
- `xfield`, `vfield` — variants like `xring`/`vring`.
- `sfield(exprs, *symbols, **options)` — construct field from SymPy expressions.
- **`FracField`** — the field object. Manages numerator ring (`PolyRing`) and domain.
- **`FracElement`** — field element (numerator/denominator pair of `PolyElement`s). Full arithmetic with automatic GCD-based cancellation.

---

## Benchmarks

### `benchmarks/bench_groebnertools.py`
Benchmark for Groebner basis computation — vertex coloring problem on a 12-vertex graph.

### `benchmarks/bench_galoispolys.py`
Benchmarks for factoring polynomials over Galois fields — Gathen and Shoup test polynomials with Zassenhaus vs Shoup algorithms.

### `benchmarks/bench_solvers.py`
Benchmark for the low-level linear system solver on a large sparse system.

---

## Appendix

### Architecture Notes
The polynomial subsystem is organized in layers:
1. **Low-level dense operations** (`densebasic`, `densearith`, `densetools`) operate on plain Python lists.
2. **Algorithm modules** (`euclidtools`, `factortools`, `sqfreetools`, `groebnertools`, `galoistools`, `rootisolation`, `modulargcd`) implement specific mathematical algorithms using the dense layer.
3. **OO wrappers** (`polyclasses`) wrap dense lists into `DMP`/`DMF`/`ANP` objects.
4. **Sparse representations** (`rings`, `fields`) provide a dict-based alternative with `PolyRing`/`FracField`.
5. **User API** (`polytools`, `polyfuncs`, `partfrac`, `numberfields`, `polyroots`, `rootoftools`) provides the symbolic, expression-level interface.

### Domain Hierarchy
```
Domain
├── Ring
│   ├── IntegerRing (ZZ)
│   ├── PolynomialRing (K[x])
│   └── QuotientRing (R/I)
└── Field (extends Ring)
    ├── RationalField (QQ)
    ├── RealField (RR)
    ├── ComplexField (CC)
    ├── FiniteField (GF(p))
    ├── AlgebraicField (QQ(α))
    ├── FractionField (K(x))
    └── ExpressionDomain (EX)
```

### Dual Representation Systems
The module maintains two parallel polynomial representation systems:
- **Dense/recursive** (`DMP` in `polyclasses`, backed by `densebasic`/`densearith`) — original system, used by `Poly` and the old domain rings.
- **Sparse/dict-based** (`PolyElement` in `rings`) — newer system, used by the new domain rings and `GroebnerBasis`.

The `compatibility.py` module bridges these by wrapping dense functions as methods on the sparse ring.
