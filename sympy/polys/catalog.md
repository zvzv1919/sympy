# polys — Module Catalog

## Glossary

- **dense representation**: Polynomials stored as nested coefficient lists indexed by degree (used by `dup_*`/`dmp_*` functions).
- **sparse representation**: Polynomials stored as `{monomial_tuple: coeff}` dicts (used by `PolyRing`/`PolyElement`).
- **dup_/dmp_ prefix**: Dense univariate / dense multivariate low-level functions operating on raw coefficient lists.
- **GF(p)**: Galois field of prime order; `gf_*` functions in `galoistools.py` operate on list-represented polynomials over GF(p).
- **kill flag**: In `DMP.per`/`DMF.per`, reduces the variable count by one; at level 0, returns the raw scalar instead of a wrapped object.
- **level (lev)**: Number of nested variable layers in dense multivariate representation; level 0 means univariate.

## Architecture Overview

- **High-level API** (`polytools.py`, `polyfuncs.py`): User-facing `Poly` class and free functions; delegates to internal representations.
- **OO representations** (`polyclasses.py`): `DMP` (dense multivariate poly), `DMF` (dense multivariate fraction), `ANP` (algebraic number poly).
- **Sparse ring layer** (`rings.py`, `fields.py`): `PolyRing`/`PolyElement` for dict-based sparse polynomials; `FracField`/`FracElement` for rational functions.
- **Dense algorithm layer** (`densearith.py`, `densebasic.py`, `densetools.py`): Low-level `dup_*`/`dmp_*` routines on coefficient lists.
- **GCD engines** (`euclidtools.py`, `heuristicgcd.py`, `modulargcd.py`): All GCD algorithms; `euclidtools` has dense-list heuristic/PRS/subresultant GCDs, `heuristicgcd` has Poly-object-level heuristic GCD, `modulargcd` has CRT-based modular GCD.
- **Galois field layer** (`galoistools.py`): Self-contained `gf_*` arithmetic, square-free, irreducibility, and factorization for GF(p) polynomials as lists.
- **Compatibility bridge** (`compatibility.py`): `IPolys` mixin adapting sparse ring objects to the dense function API.

---

## Core Representations

### [`polyclasses.py`](polyclasses.py)
OO wrappers for dense polynomial representations used internally by `Poly`.

- `DMP` — Dense Multivariate Polynomial over domain K.
  - `per(rep, dom, kill, ring)` — construct new DMP from internal rep; **if `kill=True` and `lev==0`, returns the raw coefficient instead of a DMP**.
  - `unify(g)` — reconcile two DMPs to a common domain; builds a local `per` closure with the same kill-at-zero-level behavior.
  - `__eq__` — catches `UnificationFailed` and returns `False` silently (never raises on incompatible domains).
  - `_strict_eq` — alternative that also checks domain and rep identity.
  - Arithmetic: `add`, `sub`, `mul`, `pow`, `div`, `quo`, `rem`, `exquo`.
  - Conversion: `to_dict`, `from_dict`, `from_list`, `to_ring`, `to_field`, `convert`, `slice`.
  - Enumeration: `all_monoms`, `all_coeffs`, `all_terms` — dense enumeration including zeros (univariate only); **for zero polynomial, returns single element `[(0,)]` / `[dom.zero]`** rather than empty list.
  - Content/primitive: `content`, `primitive`, `terms_gcd`.
- `DMF` — Dense Multivariate Fraction (numerator/denominator pair) over K.
  - `per(num, den, cancel, kill, ring)` — construct new DMF; **if `kill=True` and `lev==0`, returns scalar `num/den`**.
  - `frac_unify(g)` — unify two DMFs across different domains; creates a local `per` closure that captures the unified domain and has the same kill/level-zero scalar-return behavior.
  - `poly_unify(g)` — unify DMF with a DMP; same local `per` closure pattern.
  - `half_per(rep, kill)` — create DMP from rep; if `kill=True` and `lev==0`, returns the raw rep.
  - `numer`, `denom`, `cancel`, `neg`, `add`, `sub`, `mul`, `quo`, `exquo`.
- `ANP` — Algebraic Number Polynomial (univariate over algebraic extension).

### [`rings.py`](rings.py)
Sparse polynomial rings and their elements (dict-based representation).

- `ring()`, `xring()`, `vring()`, `sring()` — ring constructor functions.
- `PolyRing` — polynomial ring `K[x_1, ..., x_n]`.
  - `_gens_set` — cached set of canonical generator elements.
  - `free_module(rank)` — create free module over this ring.
  - `to_ground()` — strip coefficient domain to its base; checks `is_Composite` **or** `hasattr(domain, 'domain')` to also handle algebraic fields not formally marked as composite.
- `PolyElement` — element of a `PolyRing` (dict: monomial tuple → coefficient).
  - `evaluate(x, a)` — substitute scalar for one variable; **univariate case returns a plain domain scalar** (drops the ring).
  - `subs(x, a)` — substitute scalar; **univariate case wraps result via `ring.ground_new`, returning a constant polynomial still in the ring**.
  - `compose(x, a)` — substitute a polynomial expression for a variable.
  - `_iadd_monom(mc)` — in-place monomial addition; **copies self first if self is a canonical generator** to avoid mutating ring-cached generators.
  - `_iadd_poly_monom(p2, mc)` — in-place add product; same generator-copy safeguard.
  - `degree`, `degrees`, `tail_degree`, `leading_monom`, `leading_term`.
  - `diff`, `integrate`, `eval`, `content`, `primitive`, `strip_zero`.

### [`fields.py`](fields.py)
Sparse rational function fields and their elements.

- `FracField` — multivariate distributed rational function field K(x₁,…,xₙ).
- `FracElement` — element of a `FracField` (numerator/denominator pair).

---

## High-Level API

### [`polytools.py`](polytools.py)
User-facing `Poly` class and public free functions for polynomial manipulation.

- `Poly` — main symbolic polynomial class.
  - `_from_poly(rep, opt)` — construct from existing Poly; **if generators are the same set in different order, calls `reorder`; if generators differ, falls back to `_from_expr`**.
  - `_from_expr`, `_from_dict`, `_from_list` — alternative constructors.
  - `ltrim(gen)` — remove unused leading generators; **raises `PolynomialError` if two distinct terms collapse to the same monomial** after truncation.
  - `terms_gcd()` — extract GCD of monomial exponents from all terms; returns `(exponent_tuple, reduced_poly)`.
  - `cancel(g, include)` — cancel common factors of f/g; when `include=False`, converts domain to its associated Ring before returning the content ratio as a SymPy expression.
  - `count_roots(inf, sup)` — count roots in interval; **if one bound is real and the other complex, converts the real bound to `(value, QQ.zero)` tuple** before delegating to complex root counter.
  - `nth_power_roots_poly(n)` — polynomial whose roots are n-th powers of f's roots.
  - `real_roots`, `all_roots`, `root` — root enumeration via `CRootOf`.
  - `reorder`, `inject`, `eject` — generator manipulation; **`eject` only supports front or back generators**; raises `NotImplementedError` for middle generators.
  - `sturm(auto=True)` — Sturm sequence; **if `auto=True` and domain is a ring, auto-converts to field** (e.g. ZZ→QQ) before computing.
  - `to_ring`, `to_field`, `set_domain` — domain conversion.
  - Content/primitive: `content`, `primitive`, `monic`.
  - Arithmetic: `add`, `sub`, `mul`, `sqr`, `pow`, `div`, `rem`, `quo`, `exquo`, `pdiv`, `prem`, `pquo`, `pexquo`.
  - GCD/resultant: `gcd`, `lcm`, `cofactors`, `resultant`, `discriminant`, `subresultants`.
  - Factorization: `factor_list`, `sqf_list`, `sqf_part`.
- `to_rational_coeffs(f)` — transform polynomial with irrational (square-root) coefficients to rational coefficients.
  - **Tries rescaling `x → α·x` first, then translation `x → x + β`**; returns `(lc, alpha, None, g)` or `(None, None, beta, g)`.
- `terms_gcd(f)` (free function) — extract monomial GCD from expression; **returns the original expression unchanged if both the extracted coefficient and monomial factor are trivial (both equal 1)**.
- `cancel(f, g)`, `reduced`, `groebner`, `factor`, `sqf`, `decompose`, `sturm` — public free functions.
- `poly_from_expr` — expression-to-Poly conversion.
- `parallel_poly_from_expr(exprs)` — convert multiple expressions to Polys simultaneously; **collects all coefficients into one flat list to infer a single unified domain**, ensuring all resulting Polys share the same coefficient ring.
- `degree`, `degree_list`, `LC`, `LM`, `LT`, `content`, `primitive`, `monic` — query functions.
- `gcd`, `lcm`, `gcd_list`, `lcm_list`, `cofactors`, `resultant`, `discriminant` — algebraic operations.
- `count_roots`, `real_roots`, `nroots`, `intervals`, `refine_root` — root functions.
- `PurePoly` — Poly subclass with equality ignoring generator names.
- `GroebnerBasis` — Gröbner basis representation class.

### [`polyfuncs.py`](polyfuncs.py)
High-level polynomial utility functions (symbolic level).

- `symmetrize(poly)` — rewrite in terms of elementary symmetric polynomials.
- `horner(poly)` — convert polynomial to Horner form (symbolic rewriting, not evaluation).
- `interpolate(data, x)` — construct interpolating polynomial.
- `rational_interpolate(data, degnum, X)` — rational function interpolation.
- `viete(poly, roots)` — Viète's formulas relating roots to coefficients.

---

## Dense Polynomial Operations

### [`densearith.py`](densearith.py)
Low-level dense polynomial arithmetic on coefficient lists.

- `dup_add`, `dmp_add`, `dup_sub`, `dmp_sub`, `dup_mul`, `dmp_mul` — basic arithmetic.
- `dup_sqr`, `dmp_sqr`, `dup_pow`, `dmp_pow` — squaring and exponentiation.
- `dup_add_mul`, `dmp_add_mul`, `dup_sub_mul`, `dmp_sub_mul` — fused multiply-add/sub.
- `dup_mul_ground`, `dmp_mul_ground`, `dup_quo_ground`, `dmp_quo_ground` — ground element operations.
- `dup_div`, `dmp_div`, `dup_rem`, `dmp_rem`, `dup_quo`, `dmp_quo`, `dup_exquo`, `dmp_exquo` — division.

### [`densebasic.py`](densebasic.py)
Low-level dense polynomial basics: construction, conversion, queries.

- `dmp_validate`, `dmp_normal`, `dmp_convert` — validation and domain conversion.
- `dmp_from_dict`, `dmp_to_dict`, `dmp_from_sympy`, `dmp_to_tuple` — format conversions.
- `dmp_degree`, `dmp_LC`, `dmp_TC`, `dmp_ground_LC` — degree/coefficient queries.
- `dmp_zero`, `dmp_one`, `dmp_zero_p`, `dmp_one_p`, `dmp_ground` — constants and predicates.
- `dmp_strip`, `dmp_inject`, `dmp_eject`, `dmp_terms_gcd` — structural manipulation.

### [`densetools.py`](densetools.py)
Advanced dense polynomial operations: calculus, evaluation, composition, denominator clearing.

- `dup_eval(f, a, K)` — evaluate univariate polynomial at point using Horner scheme; **if `a` is zero (falsy), returns the trailing coefficient directly** instead of iterating.
- `dmp_eval`, `dmp_eval_in`, `dmp_eval_tail` — multivariate evaluation at points.
- `dup_diff`, `dmp_diff`, `dmp_diff_in` — differentiation.
- `dup_integrate`, `dmp_integrate`, `dmp_integrate_in` — integration.
- `dup_compose`, `dmp_compose` — polynomial composition.
- `dup_decompose` — functional decomposition of univariate polynomial.
- `dup_clear_denoms(f, K0, K1)` — clear fractional coefficients from univariate polynomial; computes LCM of denominators.
- `dmp_clear_denoms(f, u, K0, K1)` — clear fractional coefficients from multivariate polynomial; uses `_rec_clear_denoms` to **recursively traverse nested coefficient lists** computing LCM of all denominators across all nesting levels.
- `dup_trunc`, `dmp_trunc`, `dmp_ground_trunc` — coefficient truncation.
- `dup_monic`, `dmp_ground_monic` — make polynomial monic.
- `dup_content`, `dmp_ground_content`, `dup_primitive`, `dmp_ground_primitive` — content and primitive part.
- `dup_real_imag` — split into real/imaginary parts.
- `dup_mirror`, `dup_scale`, `dup_shift`, `dup_transform` — polynomial transformations.
- `dup_sign_variations` — count sign changes in coefficient sequence.
- `dup_revert`, `dmp_revert` — compute polynomial inverse modulo x^n.

---

## GCD & Euclidean Algorithms

### [`euclidtools.py`](euclidtools.py)
Euclidean algorithms, GCD/LCM, polynomial remainder sequences — all operating on dense coefficient lists.

- `dup_half_gcdex`, `dmp_half_gcdex` — half extended Euclidean algorithm.
- `dup_gcdex`, `dmp_gcdex` — extended Euclidean algorithm.
- `dup_invert(f, g, K)` — modular multiplicative inverse of f mod g; **raises `NotInvertible("zero divisor")` if gcd(f,g) ≠ 1**.
- `dmp_invert` — multivariate version.
- `dup_euclidean_prs`, `dmp_euclidean_prs` — Euclidean polynomial remainder sequence.
- `dup_primitive_prs`, `dmp_primitive_prs` — primitive PRS.
- `dup_inner_subresultants`, `dmp_inner_subresultants` — subresultant PRS.
- `dup_resultant`, `dmp_resultant` — resultant via multiple methods.
- `dup_discriminant`, `dmp_discriminant` — discriminant computation.
- `dup_rr_prs_gcd`, `dmp_rr_prs_gcd` — GCD via subresultant PRS over a ring.
- `dup_ff_prs_gcd`, `dmp_ff_prs_gcd` — GCD via Euclidean PRS over a field.
- `dup_zz_heu_gcd`, `dmp_zz_heu_gcd` — heuristic GCD over Z (dense list level).
- `dup_qq_heu_gcd`, `dmp_qq_heu_gcd` — heuristic GCD over Q (dense list level); **clears denominators first, then delegates to the Z version**; trivial-case handler returns early for zero polynomials.
- `_dmp_simplify_gcd(f, g, u, K)` — **tries to eliminate the outermost variable** from multivariate GCD when at least one input has degree 0 in that variable; extracts content/LC in fewer variables.
- `_dmp_rr_trivial_gcd`, `_dmp_ff_trivial_gcd` — trivial-case handlers for zero/unit inputs.
- `dup_inner_gcd`, `dmp_inner_gcd`, `dup_gcd`, `dmp_gcd` — main GCD entry points.
- `dup_lcm`, `dmp_lcm` — LCM computation.
- `dmp_content`, `dmp_primitive` — multivariate content and primitive part.
- `dup_cancel`, `dmp_cancel` — cancel common factors from numerator/denominator pair.

### [`heuristicgcd.py`](heuristicgcd.py)
Heuristic polynomial GCD at the **Poly-object level** (not dense lists).

- `heugcd(f, g)` — heuristic GCD for `PolyElement` objects in `ZZ[x₁,…,xₙ]`; evaluates at points, computes integer GCD, and interpolates back.
- `_gcd_interpolate` — helper for Lagrange interpolation step.

Caveat: Distinct from `dup_zz_heu_gcd`/`dmp_zz_heu_gcd` in `euclidtools.py`, which operate on raw dense coefficient lists.

### [`modulargcd.py`](modulargcd.py)
Modular GCD algorithms using Chinese Remainder Theorem and Lagrange interpolation.

- `modgcd_univariate`, `modgcd_bivariate`, `modgcd_multivariate` — modular GCD in Z[x], Z[x,y], Z[X].
- `func_field_modgcd` — modular GCD over algebraic function fields.
- `_to_ZZ_poly(f, ring)` — **converts polynomial from Q(α)[x₀,…,xₙ₋₁] to Z[…][x₀, z]** by clearing denominators and replacing the algebraic element α with a formal indeterminate z.
- `_to_ANP_poly(f, ring)` — inverse of `_to_ZZ_poly`.
- `_interpolate_multivariate(evalpoints, hpeval, ring, i, p, ground)` — Lagrange interpolation in Z_p; **when `ground=True`, the reconstructed variable comes from `ring.domain.gens[i]`** (coefficient ring) instead of `ring.gens[i]`.
- `_chinese_remainder_reconstruction_multivariate` — CRT for multivariate polynomials.
- `_rational_reconstruction_int_coeffs` — rational reconstruction of coefficients.
- `_trial_division` — verify candidate GCD by trial division.

---

## Factorization

### [`factortools.py`](factortools.py)
Polynomial factorization in characteristic zero (over Z, Q, algebraic extensions, and GF).

- `dup_zz_zassenhaus`, `dup_zz_factor_sqf`, `dup_zz_factor` — Zassenhaus factorization over Z.
- `dmp_zz_wang`, `dmp_zz_factor` — Wang's multivariate factorization over Z.
- `dup_zz_hensel_step`, `dup_zz_hensel_lift` — Hensel lifting.
- `dup_ext_factor`, `dmp_ext_factor` — factorization over algebraic extensions.
- `dup_gf_factor`, `dmp_gf_factor` — factorization in finite fields (wraps galoistools).
- `dup_factor_list`, `dmp_factor_list` — complete factorization with multiplicities.
- `dup_irreducible_p`, `dmp_irreducible_p` — irreducibility testing.
- `dup_trial_division`, `dmp_trial_division` — trial division.
- `dup_zz_mignotte_bound`, `dmp_zz_mignotte_bound` — coefficient bounds for factors.
- `dup_cyclotomic_p`, `dup_zz_cyclotomic_factor` — cyclotomic polynomial detection/factoring.

### [`sqfreetools.py`](sqfreetools.py)
Square-free decomposition for **characteristic-zero domains** (Z, Q, algebraic extensions).

- `dup_sqf_p`, `dmp_sqf_p` — square-free predicate (checks gcd(f, f') == 1).
- `dup_sqf_norm`, `dmp_sqf_norm` — square-free norm.
- `dup_sqf_part`, `dmp_sqf_part` — square-free part.
- `dup_sqf_list`, `dmp_sqf_list` — square-free decomposition with multiplicities.
- `dup_gf_sqf_part`, `dmp_gf_sqf_part`, `dup_gf_sqf_list`, `dmp_gf_sqf_list` — GF variants (thin wrappers around `galoistools`).
- `dup_gff_list`, `dmp_gff_list` — greatest factorial factorization.

Caveat: For native GF(p) polynomial square-free and factorization, see `galoistools.py`.

---

## Galois Field Operations

### [`galoistools.py`](galoistools.py)
Self-contained arithmetic, square-free, irreducibility, and factorization for **univariate polynomials over GF(p)**, represented as coefficient lists.

- Arithmetic: `gf_add`, `gf_sub`, `gf_mul`, `gf_sqr`, `gf_div`, `gf_rem`, `gf_quo`, `gf_exquo`, `gf_pow`, `gf_pow_mod`.
- Ground ops: `gf_add_ground(f, a, p, K)` — add scalar to GF(p) poly; **if f is the zero poly (empty list) and `a % p == 0`, returns `[]`** (empty list = zero polynomial representation).
- `gf_sub_ground`, `gf_mul_ground`, `gf_quo_ground`, `gf_neg` — ground field operations.
- `gf_monic`, `gf_diff`, `gf_eval`, `gf_multi_eval` — standard operations.
- `gf_gcd`, `gf_lcm`, `gf_cofactors`, `gf_gcdex` — GCD/LCM.
- `gf_sqf_p(f, p, K)` — **square-free test for GF(p)[x]**; after making monic, if result is empty (zero poly), returns True immediately.
- `gf_sqf_part`, `gf_sqf_list` — square-free decomposition in GF(p).
- `gf_irreducible_p`, `gf_irred_p_ben_or`, `gf_irred_p_rabin` — irreducibility testing.
- `gf_berlekamp`, `gf_zassenhaus`, `gf_shoup`, `gf_factor_sqf` — factorization of square-free polynomials.
- `gf_factor(f, p, K)` — **complete factorization of possibly non-square-free polynomial**; computes square-free decomposition first, then factors each component, preserving multiplicities.
- `gf_frobenius_monomial_base`, `gf_frobenius_map` — Frobenius automorphism.
- `gf_compose`, `gf_compose_mod`, `gf_trace_map` — composition and trace.
- `gf_random`, `gf_irreducible` — random/irreducible polynomial generation.
- `gf_value` — evaluate polynomial at integer point.
- `gf_crt`, `gf_crt1`, `gf_crt2` — Chinese Remainder Theorem.
- `linear_congruence`, `csolve_prime`, `gf_csolve` — congruence solving.
- Conversion: `gf_from_dict`, `gf_to_dict`, `gf_from_int_poly`, `gf_to_int_poly`.

Caveat: All operations here are list-based GF(p)-specific. For dense polynomial operations over general domains, see `densearith.py`/`densetools.py`. For square-free decomposition over Z/Q, see `sqfreetools.py`.

---

## Roots & Isolation

### [`rootoftools.py`](rootoftools.py)
Symbolic root representations and root-sum evaluation.

- `CRootOf` (alias `ComplexRootOf`) — indexed algebraic root of an irreducible polynomial.
  - `_real_roots`, `_all_roots`, `_roots_trivial`, `_roots_radical` — root enumeration.
  - `_get_interval`, `_refine_interval`, `_eval_evalf` — numerical evaluation.
  - `real_roots(poly)`, `all_roots(poly)` — class methods for root lists.
- `RootSum` — represents ∑ f(rᵢ) over all roots rᵢ of a polynomial.
  - `_rational_case(poly, func)` — **evaluates sum of a rational function over all roots using Viète's formulas and symmetric function decomposition**.
    Avoids computing roots explicitly by introducing formal root symbols, symmetrizing, then substituting Viète relations.
  - `_is_func_rational` — checks if the lambda is a rational function.
  - `doit` — attempts to evaluate the root sum.
- `rootof(poly, index)` — factory function creating `CRootOf` instances.
- `preprocess_roots` — preprocessing for root computation.

### [`polyroots.py`](polyroots.py)
Symbolic root-finding algorithms (closed-form solutions).

- `roots(f)` — compute symbolic roots using radical formulas (linear through quartic), plus special cases.
- `roots_cubic`, `roots_quartic`, `roots_binomial`, `roots_cyclotomic` — specialized solvers.

### [`rootisolation.py`](rootisolation.py)
Numerical root isolation and refinement for dense univariate polynomials.

- `dup_isolate_real_roots`, `dup_isolate_real_roots_sqf` — real root isolation via continued fractions / bisection.
- `dup_isolate_complex_roots_sqf` — complex root isolation.
- `dup_count_real_roots`, `dup_count_complex_roots` — count roots in intervals.
- `dup_refine_real_root` — refine root interval.
- `dup_sturm` — Sturm sequence for real root counting.
- `dup_root_upper_bound`, `dup_root_lower_bound` — root magnitude bounds.

---

## Polynomial Remainder Sequences

### [`subresultants_qq_zz.py`](subresultants_qq_zz.py)
Polynomial remainder sequences (Euclidean, Sturmian, subresultant) with **theoretical/reference implementations** using Sylvester/Bézout matrices.

- `sturm_q(p, q, x)` — **generalized Sturm sequence in Q[x]** using polynomial remainder; if LC(p) < 0, negates both inputs and flips the final sequence; removes trailing zero/NaN entry if GCD has degree > 0.
- `sturm_pg`, `sturm_amv` — Sturm sequences via alternative methods.
- `euclid_pg`, `euclid_q`, `euclid_amv` — Euclidean PRS.
- `subresultants_pg`, `subresultants_amv`, `subresultants_rem`, `subresultants_vv` — subresultant PRS (multiple methods).
- `modified_subresultants_pg`, `modified_subresultants_amv`, `modified_subresultants_bezout` — modified subresultant PRS.
- `sylvester(p, q, x)` — Sylvester matrix construction.
- `bezout(p, q, x)` — Bézout matrix construction.

Caveat: These are reference/theoretical implementations. For production PRS and Sturm sequences used in root isolation, see `euclidtools.py` and `rootisolation.py`.

---

## Domain & Ring Construction

### [`constructor.py`](constructor.py)
Automatic domain inference from coefficient lists.

- `construct_domain(coeffs, opt)` — determine minimal domain (ZZ, QQ, RR, algebraic, composite) for a set of coefficients.
- `_construct_simple`, `_construct_algebraic`, `_construct_composite`, `_construct_expression` — helpers for each domain type.

### [`compatibility.py`](compatibility.py)
Bridge between sparse polynomial ring interface and dense function API.

- `IPolys` — mixin class providing dense polynomial operations as methods on ring objects.
  - `wrap(element)` — coerce a `PolyElement` into this ring; **raises `NotImplementedError("domain conversions")` if the element belongs to a different ring**.
  - `ground_new`, `domain_new`, `from_dict`, `clone`, `drop` — ring interface methods.
- Re-exports all `dup_*`/`dmp_*`/`gf_*` functions from dense modules.

### [`polyoptions.py`](polyoptions.py)
Option processing and validation for `Poly` constructors and functions.

- `Options` — option container; manages `domain`, `field`, `gaussian`, `extension`, `modulus`, `order`, etc.
- `build_options`, `allowed_flags` — option construction helpers.

### [`polyconfig.py`](polyconfig.py)
Global configuration flags for polynomial algorithms.

- `query(key)`, `setup(key, value)` — get/set configuration flags (e.g., `USE_SIMPLIFY_GCD`, `GF_FACTOR_METHOD`).

---

## Utilities

### [`polyutils.py`](polyutils.py)
Expression-to-polynomial conversion utilities and generator management.

- `_parallel_dict_from_expr_if_gens`, `_parallel_dict_from_expr_no_gens` — convert expressions to monomial dictionaries.
- `dict_from_expr`, `parallel_dict_from_expr` — high-level conversion entry points.
- `expr_from_dict` — convert monomial dictionary back to expression.
- `_sort_gens`, `_unify_gens`, `_analyze_gens` — generator ordering and unification.
- `_sort_factors` — sort polynomial factors.
- `_dict_reorder` — reorder monomial dictionary for new generator order.
- `_nsort` — numerical sorting of roots.
- `PicklableWithSlots` — base class for picklable objects with `__slots__`.

### [`monomials.py`](monomials.py)
Monomial tuple arithmetic and generation.

- `itermonomials(variables, max_degrees)` — generate monomials up to given degrees.
- `monomial_count(n, d)` — count monomials of n variables and degree d.
- `monomial_mul`, `monomial_div`, `monomial_ldiv`, `monomial_pow` — tuple arithmetic.
- `monomial_gcd`, `monomial_lcm` — GCD/LCM of monomial tuples.
- `monomial_divides`, `monomial_max`, `monomial_min`, `monomial_deg` — predicates and queries.
- `Monomial` — symbolic monomial class.
- `MonomialOps` — optimized monomial operation dispatcher.

### [`orderings.py`](orderings.py)
Monomial orderings for polynomial rings.

- `LexOrder`, `GradedLexOrder`, `ReversedGradedLexOrder` — standard orderings (lex, grlex, grevlex).
- `InverseOrder`, `ProductOrder` — composite orderings.
- `monomial_key` — construct key function for a given ordering.

### [`rationaltools.py`](rationaltools.py)
Rational expression manipulation.

- `together(expr)` — combine fractions by finding a common denominator.

### [`polyerrors.py`](polyerrors.py)
Exception classes for polynomial operations.

- `PolynomialError`, `MultivariatePolynomialError`, `OperationNotSupported` — general errors.
- `UnificationFailed` — raised when domains/polynomials cannot be unified.
- `NotInvertible` — raised when polynomial modular inverse does not exist.
- `GeneratorsNeeded`, `GeneratorsError` — generator-related errors.
- `ComputationFailed`, `ExactQuotientFailed`, `RefinementFailed` — computation errors.

---

## Specialized Modules

### [`numberfields.py`](numberfields.py)
Computational algebraic number theory: minimal polynomials, field isomorphisms, primitive elements.

- `minimal_polynomial(expr, x)` — compute minimal polynomial of an algebraic expression.
- `primitive_element(*extensions)` — compute primitive element of algebraic extension.
- `field_isomorphism(a, b)` — find isomorphism between algebraic number fields.
- `to_number_field(expr)` — convert expression to algebraic number field element.
- `isolate(expr)` — numerically isolate an algebraic number.
- `_choose_factor` — select factor of a polynomial that has a specific root.

### [`partfrac.py`](partfrac.py)
Partial fraction decomposition.

- `apart(f, x, full)` — partial fraction decomposition of rational function.
  - Non-commutative fallback: if polynomial conversion fails, handles Mul (splits commutative/NC parts), Add (decomposes commutative terms).
  - For other non-commutative forms, **walks the expression tree in preorder**, decomposes each sub-expression, and replaces successes in-place.
  - `full=False` (default): uses undetermined coefficients method; `full=True`: uses Bronstein's algorithm.
- `apart_undetermined_coeffs(P, Q)` — partial fractions via undetermined coefficients; factors denominator, assigns symbolic unknowns per factor power, builds a linear system by matching polynomial powers, and solves for unknowns.
- `apart_full_decomposition(P, Q)` — Bronstein's full partial fraction decomposition.
- `apart_list` — structured partial fraction representation.
- `assemble_partfrac_list` — reassemble from structured representation.

### [`orthopolys.py`](orthopolys.py)
Classical orthogonal polynomial generation.

- `jacobi_poly`, `gegenbauer_poly`, `chebyshevt_poly`, `chebyshevu_poly`, `hermite_poly`, `legendre_poly`, `laguerre_poly` — generate orthogonal polynomials.

### [`specialpolys.py`](specialpolys.py)
Special polynomial constructors for testing and benchmarking.

- `swinnerton_dyer_poly`, `cyclotomic_poly`, `symmetric_poly`, `random_poly`, `interpolating_poly`.

### [`groebnertools.py`](groebnertools.py)
Gröbner basis computation algorithms.

- `groebner(seq, ring)` — compute Gröbner basis using Buchberger or F5B algorithm.
- `is_groebner`, `is_reduced` — basis validation.

### [`fglmtools.py`](fglmtools.py)
FGLM algorithm for Gröbner basis conversion between monomial orderings.

- `matrix_fglm(F, ring, O_to)` — convert Gröbner basis from one ordering to another.

### [`distributedmodules.py`](distributedmodules.py)
Sparse distributed module representations for submodule/syzygy computation.

- `sdm_nf_mora` — Mora normal form for module elements.
- `sdm_groebner` — Gröbner basis for submodules.
- `sdm_spoly` — S-polynomial computation.

### [`ring_series.py`](ring_series.py)
Power series arithmetic in sparse polynomial rings.

- `rs_add`, `rs_mul`, `rs_pow`, `rs_series_inversion` — ring series operations.
- `rs_exp`, `rs_log`, `rs_sin`, `rs_cos`, `rs_tan`, `rs_atan` — transcendental series.
- `rs_nth_root`, `rs_compose` — composition and roots.

### [`dispersion.py`](dispersion.py)
Dispersion of polynomials.

- `dispersion(p, q)` — compute the dispersion set of two polynomials.
- `dispersionset(p, q)` — compute all integer shifts j where gcd(p(x), q(x+j)) ≠ 1.

### [`solvers.py`](solvers.py)
Polynomial system solving.

- `solve_lin_sys` — solve linear systems over polynomial rings.
- `sympy_eqs_to_ring` — convert SymPy equations to ring form.

---

## Submodules

### [`agca/`](agca/catalog.md)
Algebraic geometry and commutative algebra: ideals, modules, homomorphisms over polynomial rings.

- `MatrixHomomorphism` (in `homomorphisms.py`) — base for homomorphisms expressed as generator-image lists; constructor uses codomain's **container** converter when codomain is a SubModule or SubQuotientModule.
- `FreeModuleHomomorphism._kernel` — kernel via syzygy module of image generators.
- `SubModuleHomomorphism._kernel` — kernel via syzygy, **translates relations back through domain generators** by forming linear combinations.
- `homomorphism(domain, codomain, matrix)` — public constructor for module homomorphisms.

### [`domains/`](domains/catalog.md)
Algebraic domain hierarchy: ZZ, QQ, RR, CC, GF(p), algebraic fields, polynomial rings, fraction fields, expression domain.

- `Domain` (in `domain.py`) — abstract base class for all domains; `__getitem__` supports bracket syntax `K[x]` / `K[x, y]` to construct polynomial rings; distinguishes single vs. multiple generators via iterable check.
- `FiniteField` (in `finitefield.py`) — GF(p) domain; `from_sympy` accepts Integer and whole-number Float (e.g. 3.0), raises `CoercionFailed` otherwise.
- `PolynomialRing` (in `polynomialring.py`) — `K[x₁,…,xₙ]` domain; `from_FractionField` converts a rational function to a ring element **only if the denominator is ground** (constant), else returns None.
