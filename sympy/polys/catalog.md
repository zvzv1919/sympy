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
  - Univariate-only operations (raise `ValueError` if `lev > 0`): `invert(g)` (modular inverse), `half_gcdex(g)`, `gcdex(g)`, `revert(n)`.
  - Conversion: `to_dict`, `from_dict`, `from_list`, `to_ring`, `to_field`, `convert`, `slice`.
  - Enumeration: `all_monoms`, `all_coeffs`, `all_terms` — dense enumeration including zeros (univariate only); **for zero polynomial, returns single element `[(0,)]` / `[dom.zero]`** rather than empty list.
  - Content/primitive: `content`, `primitive`, `terms_gcd`.
  - `cancel(g, include)` — cancel common factors in f/g; when `include=False`, returns `(cF, cG, F, G)` (content factors + reduced polys); when `include=True`, returns only `(F, G)`.
- `DMF` — Dense Multivariate Fraction (numerator/denominator pair) over K.
  - `per(num, den, cancel, kill, ring)` — construct new DMF; **if `kill=True` and `lev==0`, returns scalar `num/den`**.
  - `frac_unify(g)` — unify two DMFs across different domains; creates a local `per` closure that captures the unified domain and has the same kill/level-zero scalar-return behavior.
  - `poly_unify(g)` — unify DMF with a DMP; same local `per` closure pattern.
  - `half_per(rep, kill)` — create DMP from rep; if `kill=True` and `lev==0`, returns the raw rep.
  - `numer`, `denom`, `cancel`, `neg`, `add`, `sub`, `mul`, `quo`, `exquo`.
  - `__rdiv__(g)` — reverse division (`g / self`); computes `invert()*g`, then **checks ring membership if a ring is set; raises `ExactQuotientFailed` if result is not in the ring**.
- `ANP` — Algebraic Number Polynomial (univariate dense poly modulo a minimal polynomial over an algebraic extension).
  - Arithmetic: `neg`, `add`, `sub`, `mul`, `pow`, `div`, `rem`, `quo`, `exquo`.
  - `div(f, g)` — returns `(quotient, zero)`; `rem` always returns zero (field-like semantics via modular inverse).
  - `unify(g)` — reconcile two ANPs to a common domain/modulus; builds a local `per` closure.
  - `LC`, `TC` — leading/trailing coefficient.
  - Conversion: `to_dict`, `to_sympy_dict`, `to_list`, `to_sympy_list`, `to_tuple`, `from_list`.

### [`rings.py`](rings.py)
Sparse polynomial rings and their elements (dict-based representation).

- `ring()`, `xring()`, `vring()`, `sring()` — ring constructor functions.
- `PolyRing` — polynomial ring `K[x_1, ..., x_n]`.
  - `_gens_set` — cached set of canonical generator elements.
  - `free_module(rank)` — create free module over this ring.
  - `to_ground()` — strip coefficient domain to its base; checks `is_Composite` **or** `hasattr(domain, 'domain')` to also handle algebraic fields not formally marked as composite.
  - `drop_to_ground(*gens)` — remove generators and inject them into the domain; **if no generators remain after removal, returns `self` unchanged** (does not reduce to the domain).
- `PolyElement` — element of a `PolyRing` (dict: monomial tuple → coefficient).
  - `evaluate(x, a)` — substitute scalar for one variable; **univariate case returns a plain domain scalar** (drops the ring).
  - `subs(x, a)` — substitute scalar; **univariate case wraps result via `ring.ground_new`, returning a constant polynomial still in the ring**.
  - `compose(x, a)` — substitute a polynomial expression for a variable.
  - `_iadd_monom(mc)` — in-place monomial addition; **copies self first if self is a canonical generator** to avoid mutating ring-cached generators.
  - `_iadd_poly_monom(p2, mc)` — in-place add product; same generator-copy safeguard.
  - `coeff(element)` — return scalar multiplier for a given monomial; accepts integer `1` for constant term or a monomial element; **raises `ValueError` for non-monomial arguments**.
  - `_term_div()` — returns a closure for term divisibility; **over non-field domains (e.g. ZZ), also checks that the coefficient divides evenly** before returning a quotient.
  - `div(fv)`, `rem(G)` — multivariate polynomial division; `rem` manipulates the internal dict directly for efficiency, skipping quotient tracking.
  - `degree`, `degrees`, `tail_degree`, `leading_monom`, `leading_term`.
  - Arithmetic (`__add__`, `__sub__`, etc.): when subtracting/adding a scalar, **deletes the constant-term dict entry entirely if the result is zero** rather than storing a zero coefficient.
  - `diff`, `integrate`, `eval`, `content`, `primitive`, `strip_zero`.
  - `_gcd(g)` — GCD dispatch: **QQ → `_gcd_QQ` (clears denoms, delegates to ZZ), ZZ → `_gcd_ZZ` (heuristic GCD via `heugcd`), other domains → fallback to `ring.dmp_inner_gcd`** (dense representation).
  - `__mul__` cross-ring dispatch: when `p2` is a `PolyElement` from a different ring, checks if `p2.ring.domain` is a `PolynomialRing` whose `.ring` matches `p1.ring`; if so, **delegates to `p2.__rmul__(p1)`**.

### [`fields.py`](fields.py)
Sparse rational function fields and their elements.

- `FracField` — multivariate distributed rational function field K(x₁,…,xₙ).
  - `from_expr(expr)` / `_rebuild_expr` — reconstruct a symbolic expression into a field element; **if ground domain fails to convert a leaf (CoercionFailed) and the domain is a ring with an associated field, retries conversion via `domain.get_field()`** (e.g. ZZ falls back to QQ).
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
  - `reorder`, `inject`, `eject` — generator manipulation.
    - `inject`: **returns `self` unchanged if the coefficient domain is purely numerical** (no ground generators to promote).
    - `eject`: **only supports front or back generators**; raises `NotImplementedError` for middle generators.
  - `sturm(auto=True)` — Sturm sequence; **if `auto=True` and domain is a ring, auto-converts to field** (e.g. ZZ→QQ) before computing.
  - `to_ring`, `to_field`, `set_domain` — domain conversion.
  - Content/primitive: `content`, `primitive`, `monic`.
  - Arithmetic: `add`, `sub`, `mul`, `sqr`, `pow`, `div`, `rem`, `quo`, `exquo`, `pdiv`, `prem`, `pquo`, `pexquo`.
    - `mul(g)` — if `g` is not a Poly, falls back to `mul_ground` (scalar multiplication); same pattern for `add`/`sub`.
    - `exquo(g)` — catches `ExactQuotientFailed` and **re-raises with `f.as_expr()`, `g.as_expr()`** so the error message contains human-readable symbolic expressions instead of internal representations.
    - `pquo` catches `ExactQuotientFailed` and **re-raises with original input expressions**; `pexquo` lets the exception propagate directly from the internal method.
  - GCD/resultant: `gcd`, `lcm`, `cofactors`, `resultant`, `discriminant`, `subresultants`.
    - `resultant(g, includePRS)` — when `includePRS=True`, returns `(resultant_value, [PRS_polys])` tuple instead of a single scalar.
  - Factorization: `factor_list`, `sqf_list`, `sqf_list_include`, `sqf_part`.
    - `sqf_list` returns `(coeff, [(factor, mult), ...])` with leading coefficient separated; `sqf_list_include` folds the coefficient into the factor tuples.
- `to_rational_coeffs(f)` — transform polynomial with irrational (square-root) coefficients to rational coefficients.
  - **Tries rescaling `x → α·x` first, then translation `x → x + β`**; returns `(lc, alpha, None, g)` or `(None, None, beta, g)`.
- `terms_gcd(f)` (free function) — extract monomial GCD from expression; **returns the original expression unchanged if both the extracted coefficient and monomial factor are trivial (both equal 1)**.
- `reduced(f, G)` — divide polynomial `f` modulo a set of polynomials `G`, returning quotients and remainder.
  - **Auto-promotes ring domain to its fraction field** for division, then attempts to retract results back to the ring (keeps field results if retraction fails).
- `cancel(f, g)`, `groebner`, `factor`, `sqf`, `decompose`, `sturm` — public free functions.
- `poly_from_expr` — expression-to-Poly conversion.
- `parallel_poly_from_expr(exprs)` — convert multiple expressions to Polys simultaneously; **collects all coefficients into one flat list to infer a single unified domain**, ensuring all resulting Polys share the same coefficient ring.
- `degree`, `degree_list`, `LC`, `LM`, `LT`, `content`, `primitive`, `monic` — query functions.
- `gcd`, `lcm`, `gcd_list`, `lcm_list`, `cofactors`, `resultant`, `discriminant` — algebraic operations.
- `count_roots`, `real_roots`, `nroots`, `intervals`, `refine_root` — root functions.
- `PurePoly` — Poly subclass with equality ignoring generator names.
- `GroebnerBasis` — Gröbner basis representation class.
  - `fglm(order)` — convert basis to a different monomial ordering via the FGLM algorithm; **promotes domain to its fraction field for computation, then clears denominators and resets domain** if the original was not a field (e.g. ZZ).
  - `is_zero_dimensional` — check if ideal is zero-dimensional.
  - `reduce(expr)` — reduce polynomial modulo the basis.

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
- `dup_div`, `dmp_div` — polynomial division; **dispatches to `dup_ff_div`/`dup_rr_div` based on `K.has_Field`** (field domains get exact division, ring domains get truncated division).
- `dup_rem`, `dmp_rem`, `dup_quo`, `dmp_quo`, `dup_exquo`, `dmp_exquo` — remainder, quotient, exact quotient.
- `dup_pdiv`, `dmp_pdiv`, `dup_prem`, `dmp_prem`, `dup_pquo`, `dmp_pquo`, `dup_pexquo`, `dmp_pexquo` — pseudo-division variants; **raise `PolynomialDivisionFailed` if remainder degree fails to decrease between iterations**.
- `dup_abs` — absolute values of coefficients.
- `dup_max_norm`, `dmp_max_norm` — maximum coefficient norm; **returns `K.zero` for zero polynomial (empty list)**.
- `dup_l1_norm`, `dmp_l1_norm` — L1 norm (sum of absolute coefficient values); **returns `K.zero` for zero polynomial (empty list)**.
- `dup_expand`, `dmp_expand` — multiply together several polynomials.

### [`densebasic.py`](densebasic.py)
Low-level dense polynomial basics: construction, conversion, queries.

- `dmp_validate`, `dmp_normal`, `dmp_convert` — validation and domain conversion.
- `dmp_from_dict`, `dmp_to_dict`, `dmp_from_sympy`, `dmp_to_tuple` — format conversions.
- `dmp_degree`, `dmp_LC`, `dmp_TC`, `dmp_ground_LC` — degree/coefficient queries.
- `dmp_zero`, `dmp_one`, `dmp_zero_p`, `dmp_one_p`, `dmp_ground` — constants and predicates.
- `dmp_ground_p(f, c, u)` — test if polynomial is a constant; **if `c` is `None`, checks if `f` is any ground element** (not a specific value); if `c` is falsy (e.g. 0), delegates to `dmp_zero_p`.
- `dmp_strip`, `dmp_inject`, `dmp_eject`, `dmp_terms_gcd` — structural manipulation.
- `dmp_list_terms(f, u, K, order)` — list all non-zero terms as `(monom_tuple, coeff)` pairs; **for zero polynomial returns `[((0,)*(u+1), K.zero)]`** (single zero-monomial entry, not empty list).
- `dmp_permute(f, P, u, K)` — reorder indeterminates by applying a permutation vector P to exponent tuples (via dict round-trip).
- `dmp_exclude(f, u, K)` — detect and remove variable dimensions unused by any term; returns `(removed_indices, reduced_poly, new_level)`.
- `dmp_include(f, J, u, K)` — re-insert previously excluded variable dimensions at specified positions.

### [`densetools.py`](densetools.py)
Advanced dense polynomial operations: calculus, evaluation, composition, denominator clearing.

- `dup_eval(f, a, K)` — evaluate univariate polynomial at point using Horner scheme; **if `a` is zero (falsy), returns the trailing coefficient directly** instead of iterating.
- `dmp_eval`, `dmp_eval_in`, `dmp_eval_tail` — multivariate evaluation at points.
- `dup_diff`, `dmp_diff`, `dmp_diff_in` — differentiation.
- `dup_integrate`, `dmp_integrate`, `dmp_integrate_in` — integration.
- `dup_compose`, `dmp_compose` — polynomial composition.
- `dup_decompose` — functional decomposition of univariate polynomial.
- `dup_clear_denoms(f, K0, K1)` — clear fractional coefficients from univariate polynomial; computes LCM of denominators. **If `K1` is None and `K0` has no associated ring, falls back to using `K0` itself as the target domain**.
- `dmp_clear_denoms(f, u, K0, K1)` — clear fractional coefficients from multivariate polynomial; uses `_rec_clear_denoms` to **recursively traverse nested coefficient lists** computing LCM of all denominators across all nesting levels.
- `dup_trunc(f, p, K)` — reduce coefficients modulo constant `p`; **over ZZ, uses symmetric representation** (if remainder > p//2, subtracts p to center around zero); over other domains, uses plain modular remainder.
- `dmp_trunc` — reduce multivariate polynomial modulo a polynomial in the inner variable.
- `dmp_ground_trunc` — reduce multivariate polynomial coefficients modulo a constant (delegates to `dup_trunc` at level 0).
- `dup_monic`, `dmp_ground_monic` — make polynomial monic.
- `dup_content`, `dmp_ground_content`, `dup_primitive`, `dmp_ground_primitive` — ground-level content and primitive part (GCD of scalar coefficients only; for multivariate coefficient GCD, see `dmp_content` in `euclidtools.py`).
- `dup_real_imag` — split into real/imaginary parts.
- `dup_mirror`, `dup_scale`, `dup_shift`, `dup_transform` — polynomial transformations.
- `dup_sign_variations` — count sign changes in coefficient sequence.
- `dup_revert`, `dmp_revert` — compute polynomial inverse modulo x^n.

---

## GCD & Euclidean Algorithms

### [`euclidtools.py`](euclidtools.py)
Production Euclidean algorithms, GCD/LCM, polynomial remainder sequences — all operating on **dense coefficient lists** (`dup_*`/`dmp_*` API).

- `dup_half_gcdex`, `dup_gcdex`, `dmp_half_gcdex`, `dmp_gcdex` — extended Euclidean algorithms.
- `dup_invert(f, g, K)` / `dmp_invert` — modular inverse; **raises `NotInvertible("zero divisor")` if gcd(f,g) ≠ 1**.
- `dup_euclidean_prs`, `dup_primitive_prs`, `dup_inner_subresultants` (and `dmp_` variants) — polynomial remainder sequences.
- `dup_resultant`, `dmp_resultant` — resultant via multiple methods.
- `dmp_zz_modular_resultant(f, g, p, u, K)` — resultant mod prime via evaluation-interpolation; **raises `HomomorphismFailed` if evaluation points exhausted**.
- `dup_discriminant`, `dmp_discriminant` — discriminant computation.
- GCD: `dup_rr_prs_gcd`/`dmp_rr_prs_gcd` (ring PRS), `dup_ff_prs_gcd`/`dmp_ff_prs_gcd` (field PRS), `dup_zz_heu_gcd`/`dmp_zz_heu_gcd` (heuristic over Z).
- `dup_qq_heu_gcd`/`dmp_qq_heu_gcd` — heuristic GCD over Q; **clears denominators first, then delegates to the Z version**.
- `_dmp_simplify_gcd` — **eliminates outermost variable** from multivariate GCD when one input has degree 0 in it.
- `dup_inner_gcd`, `dmp_inner_gcd`, `dup_gcd`, `dmp_gcd` — main GCD entry points.
- `dup_lcm`, `dmp_lcm` — LCM; `dmp_lcm` **dispatches to `dup_lcm` when `u==0`**.
- `dmp_content`, `dmp_primitive` — multivariate content/primitive; content **negates if leading ground coeff is negative**.
- `dup_cancel`, `dmp_cancel` — cancel common factors; **when `K.has_Field` and `K.has_assoc_Ring`, converts to ring (clears denoms) before GCD, then converts back**.

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
- `_chinese_remainder_reconstruction_univariate` — CRT for univariate polynomials; combines two residue representations over coprime moduli into symmetric representation over their product.
- `_chinese_remainder_reconstruction_multivariate` — CRT for multivariate polynomials.
- `_rational_reconstruction_int_coeffs` — rational reconstruction of coefficients.
- `_trial_division` — verify candidate GCD by trial division.

---

## Factorization

### [`factortools.py`](factortools.py)
Polynomial factorization in characteristic zero (over Z, Q, algebraic extensions, and GF).

- `dup_zz_zassenhaus`, `dup_zz_factor_sqf`, `dup_zz_factor` — Zassenhaus factorization over Z.
- `dmp_zz_wang` — Wang's Enhanced Extended Zassenhaus multivariate factorization; **selects evaluation-point config with smallest univariate max-norm**; restarts with incremented modulus on `ExtraneousFactors` from Hensel lifting.
- `dmp_zz_factor` — top-level multivariate factorization over Z.
- `dup_zz_hensel_step`, `dup_zz_hensel_lift` — Hensel lifting.
- `dup_ext_factor`, `dmp_ext_factor` — factorization over algebraic extensions.
- `dup_gf_factor`, `dmp_gf_factor` — factorization in finite fields (wraps galoistools).
- `dup_factor_list`, `dmp_factor_list` — complete factorization with multiplicities.
- `dup_zz_irreducible_p` — integer polynomial irreducibility test via **Eisenstein's criterion** (checks if a prime divides all non-leading coefficients but its square does not divide the constant term).
- `dup_irreducible_p`, `dmp_irreducible_p` — irreducibility testing (general).
- `dup_trial_division`, `dmp_trial_division` — determine factor multiplicities via repeated division; **includes factors with multiplicity 0** if candidate does not divide.
- `dup_zz_diophantine`, `dmp_zz_diophantine` — Wang/EEZ Diophantine equation solvers; `dup_zz_diophantine` for >2 inputs **builds cumulative products and recursively reduces to the 2-input base case** (extended GCD).
- `dup_zz_mignotte_bound`, `dmp_zz_mignotte_bound` — coefficient bounds for factors.
- `dup_cyclotomic_p`, `dup_zz_cyclotomic_factor` — cyclotomic polynomial detection/factoring.

### [`sqfreetools.py`](sqfreetools.py)
Square-free decomposition for **characteristic-zero domains** (Z, Q, algebraic extensions).

- `dup_sqf_p`, `dmp_sqf_p` — square-free predicate (checks gcd(f, f') == 1).
- `dup_sqf_norm`, `dmp_sqf_norm` — square-free norm over algebraic extensions; iteratively shifts input by the algebraic generator until the resultant is square-free.
  - Returns `(shift_count, shifted_poly, resultant_in_ground_domain)`.
- `dup_sqf_part`, `dmp_sqf_part` — square-free part.
- `dup_sqf_list`, `dmp_sqf_list` — square-free decomposition with multiplicities.
- `dup_sqf_list_include`, `dmp_sqf_list_include` — same as `sqf_list` but folds the leading coefficient into the factor list; **if no factor has multiplicity 1, prepends a ground constant `(coeff, 1)` entry**.
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
- `gf_compose`, `gf_compose_mod` — polynomial composition and modular composition.
- `gf_trace_map(a, b, c, n, f, p, K)` — compute trace map `a + a^t + a^t^2 + … + a^t^n` in `GF(p)[x]/(f)` using **binary doubling**; initializes accumulators differently for even vs odd `n`.
- `_gf_trace_map` — simpler iterative trace map (utility for `gf_edf_shoup`).
- `gf_expand(F, p, K)` — reconstruct polynomial from factored form; **accepts either a `(lc, factors)` tuple or a plain list of `(factor, multiplicity)` pairs** (defaults leading coefficient to `K.one` for the list form).
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
  - `__new__(f, x, index)` — constructor; **negative index is normalized by adding the polynomial degree**; raises `IndexError` if out of range.
  - `_real_roots`, `_all_roots`, `_roots_radical` — root enumeration.
  - `_roots_trivial(poly, radicals)` — closed-form roots for linear/quadratic/binomial; **if `radicals=False`, returns `None` for all degree > 1** (only linear is always solved).
  - `_reals_index`, `_complexes_index` — map global root index to per-factor local index; `_complexes_index` **offsets the local index by the number of real roots** of the same factor (via `_reals_cache`).
  - `_get_interval`, `_refine_interval`, `_eval_evalf` — numerical evaluation; `_eval_evalf` **creates a Dummy variable and substitutes when the polynomial generator is a compound expression** (not a plain Symbol).
  - `_separate_imaginary_from_complex` — classify non-real roots into imaginary vs complex.
    - For two-term polynomials of power-of-2 degree with opposite-sign LC·TC, marks 2 roots as imaginary (mixed case).
    - **Refines bounding rectangles until non-imaginary roots have boxes fully to one side of the y-axis**.
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
- `roots_quintic` — solvable quintic solver using Lagrange resolvents; swaps resolvent parameters when numerical check against discriminant fails.
- `_integer_basis(poly)` — find integer scaling factor `div` such that substitution `x = div*y` minimizes coefficient magnitudes; **reverses the coefficient list when the leading coefficient is 1** before searching for the scaling constant.

### [`rootisolation.py`](rootisolation.py)
Numerical root isolation and refinement for dense univariate polynomials. Also defines `RealInterval` and `ComplexInterval` classes for bounding root locations.

- `RealInterval` — bounding interval for a real root; stores Möbius transform for refinement.
- `ComplexInterval` — bounding rectangle for a complex root; stores southwest/northeast corners. When `conj=True` (root in lower half-plane), **y-coordinates are reflected**: `ay` returns `-b[1]` and `by` returns `-a[1]`.
- `dup_isolate_real_roots`, `dup_isolate_real_roots_sqf` — real root isolation via continued fractions / bisection.
- `dup_isolate_complex_roots_sqf` — complex root isolation.
- `dup_count_real_roots`, `dup_count_complex_roots` — count roots in intervals.
- `dup_refine_real_root` — refine root interval.
- `dup_sturm` — Sturm sequence for real root counting; **raises `DomainError` if the coefficient domain is not a field** (e.g. ZZ).
- `_discard_if_outside_interval` — filter isolation intervals against user bounds; **if interval partially overlaps, repeatedly refines** until it is fully inside or fully outside.
- `dup_root_upper_bound`, `dup_root_lower_bound` — root magnitude bounds.

---

## Polynomial Remainder Sequences

### [`subresultants_qq_zz.py`](subresultants_qq_zz.py)
Polynomial remainder sequences (Euclidean, Sturmian, subresultant) with **theoretical/reference implementations** using Sylvester/Bézout matrices.

- `sign_seq(poly_seq, x)` — extract the sequence of signs of leading coefficients from a polynomial remainder sequence.
- `sturm_q(p, q, x)` — **generalized Sturm sequence in Q[x]** using polynomial remainder; if LC(p) < 0, negates both inputs and flips the final sequence; removes trailing zero/NaN entry if GCD has degree > 0.
- `sturm_pg`, `sturm_amv` — Sturm sequences via alternative methods (Pell-Gordon / AMV theorems); `sturm_pg` **negates both inputs when LC(p) < 0** and flips the output sequence to ensure correctness.
- `euclid_pg`, `euclid_q`, `euclid_amv` — Euclidean PRS via sign-flipping of Sturm sequences.
- `subresultants_pg`, `subresultants_amv`, `subresultants_rem`, `subresultants_vv` — subresultant PRS (multiple methods); `subresultants_rem` swaps inputs if deg(p) < deg(q).
- `modified_subresultants_pg`, `modified_subresultants_amv`, `modified_subresultants_bezout` — modified subresultant PRS; `modified_subresultants_pg` uses Pell-Gordon 1917 theorem with degree-gap-aware denominator calculation.
- `sylvester(p, q, x)` — Sylvester matrix construction.
- `bezout(p, q, x, method)` — Bézout matrix construction; `method='prs'` reverses index ordering; `method='bz'` uses natural ordering.
  - **Identity: `bezout(..., 'prs') = backward_eye(n) * bezout(..., 'bz') * backward_eye(n)`**, connecting to Sylvester's 1853 matrix.
- `rem_z(p, q, x)` — integer polynomial remainder using **absolute value** of LC(q) for premultiplication (unlike `prem` which uses LC directly), ensuring correct signs in Euclidean/Sturmian PRS.
- `quo_z(p, q, x)` — integer polynomial quotient, same absolute-value premultiplication as `rem_z`.

- `find_degree(M, deg_f)` — find degree of the polynomial from the last row of a triangularized small matrix; **returns 0 (clamped) if leading zeros exceed `deg_f`**; returns `None` if the row is all zeros.
- `create_ma`, `rotate_r`, `rotate_l`, `row2poly`, `final_touches` — helper functions for the Van Vleck triangularization method.

Caveat: These are reference/theoretical implementations operating on symbolic expressions (not dense coefficient lists). For production PRS/subresultant/Sturm computation on dense lists, see `euclidtools.py` and `rootisolation.py`.

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
  - Multivariate result methods (`dmp_resultant`, `dmp_discriminant`, `dmp_content`, `dmp_primitive`): **check `isinstance(result, list)` to decide output form**.
    - If list → reconstruct via `self[1:].from_dense()` (ring with one fewer generator); if scalar → return raw value.
  - `dup_sqf_norm`, `dmp_sqf_norm` — bridge methods; the resultant (third return value) is converted via `self.to_ground().from_dense()` (ground domain ring), not `self.from_dense()`.
  - `gf_*` wrapper methods (e.g. `gf_trunc`, `gf_normal`, `gf_neg`, `gf_add`, …) — convert sparse ↔ dense and pass through domain modulus/base to the corresponding `galoistools` functions.
- Re-exports all `dup_*`/`dmp_*`/`gf_*` functions from dense modules.

### [`polyoptions.py`](polyoptions.py)
Option processing and validation for `Poly` constructors and functions.

- `Options` — option container; manages `domain`, `field`, `gaussian`, `extension`, `modulus`, `order`, etc.
- `Domain.postprocess` — **raises `GeneratorsError` if EX domain is requested without providing generators**, or if composite domain symbols overlap with polynomial generators.
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
- `_dict_reorder(rep, gens, new_gens)` — reorder monomial exponent tuples to match a new generator ordering; appends zero for new generators not in the original set.
  - **Raises `GeneratorsError` if an original generator with non-zero exponent is absent from the new ordering**.
- `_nsort` — numerical sorting of roots.
- `PicklableWithSlots` — base class for picklable objects with `__slots__`.

### [`monomials.py`](monomials.py)
Monomial tuple arithmetic and generation.

- `itermonomials(variables, max_degrees)` — generate monomials up to given degrees.
- `monomial_count(n, d)` — count monomials of n variables and degree d.
- `monomial_mul`, `monomial_div`, `monomial_ldiv`, `monomial_pow` — tuple arithmetic.
  - `monomial_div(A, B)` — returns `None` if any resulting exponent would be negative (exact division only); `monomial_ldiv` allows negative exponents.
- `monomial_gcd`, `monomial_lcm` — GCD/LCM of monomial tuples.
- `monomial_divides`, `monomial_max`, `monomial_min`, `monomial_deg` — predicates and queries.
- `term_div(a, b, domain)` — divide two `(monomial, coefficient)` terms; **over a field, only checks monomial divisibility; over a ring, additionally requires coefficient divides evenly**; returns `None` on failure.
- `Monomial` — symbolic monomial class (pure power-product, coefficient must be 1).
  - `__init__(monom, gens)` — accepts exponent tuple or symbolic expression; **raises `ValueError` if expression has non-unit coefficient or multiple terms**.
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

- `minimal_polynomial(expr, x)` — compute minimal polynomial of an algebraic expression; **auto-switches from compositional (resultant) to Gröbner strategy when any `AlgebraicNumber` subexpression is detected** in the expression tree.
- `_minpoly_pow(ex, pw, x, dom)` — minimal polynomial of `ex**pw`; for negative exponents, **inverts the polynomial and raises `ZeroDivisionError` if `mp == x`** (element is zero); short-circuits for `pw == -1`.
- `_minpoly_groebner(ex, x, dom)` — Gröbner-basis strategy for minimal polynomial.
  - Includes `simpler_inverse` heuristic: **inverts the expression first when it is a product of powers or a negative-exponent power with Add base**, then transforms back via `_invertx`.
- `_minpoly_compose`, `_minpoly_add`, `_minpoly_mul`, `_minpoly_sin`, `_minpoly_cos` — compositional minimal polynomial helpers for arithmetic and trigonometric subexpressions.
- `primitive_element(*extensions)` — compute primitive element of algebraic extension.
- `field_isomorphism(a, b)` — find isomorphism between algebraic number fields.
- `to_number_field(extension, theta)` — express algebraic extensions in a generated field; if `theta` is given, uses `field_isomorphism` to map into theta's field, **raises `IsomorphismFailed` if the extension is not in a subfield of theta**.
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
- `assemble_partfrac_list` — reassemble from structured representation; **if roots are given as a `Poly`, constructs a `RootSum`; if roots are an explicit list of algebraic numbers, directly evaluates numerator/denominator at each root**.

### [`orthopolys.py`](orthopolys.py)
Classical orthogonal polynomial generation.

- `jacobi_poly`, `gegenbauer_poly`, `chebyshevt_poly`, `chebyshevu_poly`, `hermite_poly`, `legendre_poly`, `laguerre_poly` — generate orthogonal polynomials.

### [`polyquinticconst.py`](polyquinticconst.py)
Precomputed coefficient arrays and resolvent parameters for solving solvable quintic equations (Dummit's algorithm).

- `PolyQuintic` — encapsulates the resolvent computation for a monic solvable quintic `x^5 + px^3 + qx^2 + rx + s`.
  - Properties `b`, `o`, `a`, `c` — large coefficient arrays (polynomials in p, q, r, s) used in resolvent construction.
  - `F` — discriminant-related invariant (sextic resolvent discriminant factor).
  - `T(theta, d)` — compute resolvent T-values by evaluating `b` arrays at a root `theta` and dividing by `F`.
  - `l0(theta)` — evaluate the `a` array at `theta` divided by `F`.
  - `order(theta, d)` — determine ordering of Lagrange resolvents using `o` array.
  - `uv(theta, d)` — compute u, v parameters for the radical solution.

### [`specialpolys.py`](specialpolys.py)
Special polynomial constructors for testing and benchmarking.

- `swinnerton_dyer_poly(n, x)` — Swinnerton-Dyer polynomial; **n ≤ 3 returns hardcoded expressions; n > 3 computes via `minimal_polynomial` of sum of square roots of primes**.
- `cyclotomic_poly`, `symmetric_poly`, `random_poly`, `interpolating_poly`.

### [`groebnertools.py`](groebnertools.py)
Gröbner basis computation algorithms.

- `groebner(seq, ring)` — compute Gröbner basis using Buchberger or F5B algorithm.
- `is_groebner`, `is_reduced` — basis validation.
- `lbp`, `lbp_cmp`, `lbp_key` — labeled polynomial constructors and comparators for the F5B signature-based algorithm.
- `lbp_sub(f, g)` — subtract labeled polynomials; **propagates signature and number from whichever operand has the larger signature** (via `sig_cmp`), not necessarily from the minuend.
- `lbp_mul_term(f, cx)` — multiply labeled polynomial by a term; scales both signature and polynomial.
- `critical_pair`, `cp_cmp`, `cp_key` — critical pair construction and ordering; `cp_cmp` uses **two-level comparison: first the dominant (signature) component, then the subordinate component as tiebreaker** when dominants are equal.

### [`fglmtools.py`](fglmtools.py)
FGLM algorithm for Gröbner basis conversion between monomial orderings.

- `matrix_fglm(F, ring, O_to)` — convert Gröbner basis from one ordering to another.
- `_basis(G, ring)` — enumerate standard monomials (not divisible by any leading monomial of G); forms the vector-space basis of the quotient ring `K[X]/(G)`.

### [`distributedmodules.py`](distributedmodules.py)
Sparse distributed module representations for submodule/syzygy computation.

- Module monomial operations: `sdm_monomial_mul`, `sdm_monomial_deg`, `sdm_monomial_lcm`, `sdm_monomial_divides`.
  - `sdm_monomial_divides(A, B)` — checks if polynomial monomial X exists such that XA = B; **returns False if A and B belong to different free module generators** (different first tuple element), even if polynomial exponents satisfy divisibility.
- `sdm_nf_buchberger(f, G, O, K, phantom)` — weak normal form using standard Buchberger algorithm (global orderings); optional `phantom` pair tracks companion coefficient vectors in parallel; **when phantom is None, uses `itertools.repeat([])` as dummy** to avoid branching in the divisor-search loop.
- `sdm_nf_buchberger_reduced` — reduced normal form (unique but more expensive); does NOT support phantom tracking.
- `sdm_nf_mora` — generalized Mora algorithm for weak normal forms with non-global orderings; **dynamically appends current element to the reducer set when the chosen reducer's ecart exceeds the element's ecart**.
- `sdm_ecart(f)` — difference between total degree and leading monomial degree.
- `sdm_groebner` — Gröbner basis (minimal standard basis) for submodules; uses "sugar" strategy for pair selection.
  - Inner `update` applies **chain criterion to prune critical pairs** whose LCM is divisible by the new element's LCM with both pair members.
- `sdm_spoly` — S-polynomial of two module elements; returns zero if leading terms involve different basis generators.

### [`ring_series.py`](ring_series.py)
Power series arithmetic in sparse polynomial rings.

- `_invert_monoms(p1)` — compute `x^n * p1(1/x)` for a sparse univariate polynomial, reversing the coefficient ordering by mapping degree k to degree (n−k).
- `rs_trunc` — truncate series to given precision.
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

- `Ideal` (in `ideals.py`) — abstract base class for ideals of polynomial rings.
  - `_equals(J)` — equality via **mutual containment**: returns True iff `self` contains `J` and `J` contains `self`.
  - `__add__(e)` — when `e` is another Ideal, computes the union (join); **when `e` is a plain ring element, constructs the quotient ring `R/self` and coerces `e` into it** instead.
- `FreeModule.convert(elem)` (in `modules.py`) — coerces lists (checks length matches rank), `FreeModuleElement` from other modules (checks rank compatibility), or literal `0` (creates zero vector); raises `CoercionFailed` otherwise.
- `QuotientModule.convert(elem)` (in `modules.py`) — when source is another QuotientModule, succeeds **only if `self.killed_module` is a submodule of `elem.module.killed_module`**; raises `CoercionFailed` otherwise.
- `ModuleHomomorphism.__init__` (in `homomorphisms.py`) — validates source/target are Module instances and **raises `ValueError` if they are defined over different base rings**.
- `MatrixHomomorphism` (in `homomorphisms.py`) — base for homomorphisms expressed as generator-image lists; constructor uses codomain's **container** converter when codomain is a SubModule or SubQuotientModule.
  - `_quotient_codomain(sm)` — quotient the codomain by `sm`; uses `Q.container.convert` for matrix entries **when codomain is a SubModule**, else uses `Q.convert`.
- `FreeModuleHomomorphism._kernel` — kernel via syzygy module of image generators.
- `SubModuleHomomorphism._kernel` — kernel via syzygy, **translates relations back through domain generators** by forming linear combinations.
- `ModuleHomomorphism.restrict_codomain(sm)` — narrow target module to submodule `sm`; **raises `ValueError` if `sm` does not contain the image**; returns `self` if `sm` equals the full codomain.
- `ModuleHomomorphism.restrict_domain(sm)` — restrict source to submodule `sm`.
- `homomorphism(domain, codomain, matrix)` — public constructor for module homomorphisms.

### [`domains/`](domains/catalog.md)
Algebraic domain hierarchy: ZZ, QQ, RR, CC, GF(p), algebraic fields, polynomial rings, fraction fields, expression domain.

- `Domain` (in `domain.py`) — abstract base class for all domains; `__getitem__` supports bracket syntax `K[x]` / `K[x, y]` to construct polynomial rings.
  - `convert_from(element, base)` — dispatch conversion by looking up `from_<alias>` if the source domain has an alias, else `from_<ClassName>`.
  - `unify(K0, K1)` — construct minimal domain containing both K0 and K1.
    - When one is a FractionField and the other a PolynomialRing, **demotes merged ground back to ring** if neither original ground was a field but the unified ground is.
- `Ring` (in `ring.py`) — abstract base for ring domains.
  - `is_unit(a)` — test invertibility by attempting `revert`; `revert(a)` **only succeeds for the multiplicative identity** (raises `NotReversible` otherwise).
- `FiniteField` (in `finitefield.py`) — GF(p) domain; `from_sympy` accepts Integer and whole-number Float (e.g. 3.0), raises `CoercionFailed` otherwise.
- `PythonIntegerRing` (in `pythonintegerring.py`) — ZZ domain backed by Python `int`; `from_sympy` accepts Integer directly and **also accepts Float if it represents a whole number** (e.g. 3.0 → 3).
- `PolynomialRing` (in `polynomialring.py`) — `K[x₁,…,xₙ]` domain; `from_FractionField` converts a rational function to a ring element **only if the denominator is ground** (constant), else returns None.
- `GlobalPolynomialRing` (in `old_polynomialring.py`) — legacy generalized polynomial ring using `DMP` dtype; `from_FractionField` converts only if **denominator is trivial (one)**, else returns None (silent failure). `from_GlobalPolynomialRing` handles cross-ring conversion: same gens → direct rep copy; different gens → reorders monomials via `_dict_reorder` and converts coefficients if domains differ.
- `QuotientRing` (in `quotientring.py`) — commutative quotient ring `R/I`; `QuotientRingElement.__eq__` checks equality of coset representatives by testing whether their difference belongs to the ideal.
