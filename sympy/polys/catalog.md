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
  - `__init__(rep, dom, lev, ring)` — three-way input dispatch when `lev` is provided: **dict → `dmp_from_dict`; non-list scalar → `dmp_ground` (constant poly); list → used directly**. When `lev` is omitted, validates the nested list via `dmp_validate` and infers the nesting depth.
  - `per(rep, dom, kill, ring)` — construct new DMP from internal rep; **if `kill=True` and `lev==0`, returns the raw coefficient instead of a DMP**.
  - `unify(g)` — reconcile two DMPs to a common domain; builds a local `per` closure with the same kill-at-zero-level behavior.
  - `__eq__` — catches `UnificationFailed` and returns `False` silently (never raises on incompatible domains).
  - `_strict_eq` — alternative that also checks domain and rep identity.
  - Arithmetic: `add`, `sub`, `mul`, `pow`, `div`, `quo`, `rem`, `exquo`.
    - `__add__(g)` / `__sub__(g)` — when `g` is not a DMP, first tries `dom.convert(g)` to create a ground polynomial; **if that fails (`CoercionFailed`/`NotImplementedError`) and `f.ring` is set, retries via `f.ring.convert(g)`**; returns `NotImplemented` if both paths fail.
    - `__mul__(g)` — if `g` is a DMP, delegates to `mul(g)`; **otherwise tries `mul_ground(g)` first (not ground conversion like add/sub)**; on `CoercionFailed`/`NotImplementedError`, falls back to `f.ring.convert(g)` then full `mul`; returns `NotImplemented` if all paths fail.
    - `__div__(g)` — if `g` is a DMP, delegates to `exquo`; otherwise tries `mul_ground(g)`; **on `CoercionFailed`/`NotImplementedError`, falls back to `f.ring.convert(g)` then `exquo`** if ring is set; returns `NotImplemented` if all paths fail.
    - `pow(n)` — **raises `TypeError` if `n` is not an `int`** (rejects float, Rational, etc.).
    - `exquo(g)` — exact quotient; after computing via `dmp_exquo`, **validates ring membership if `f.ring` is set; raises `ExactQuotientFailed` if result is not in the ring** (secondary check beyond basic divisibility).
  - `subresultants(g)` — subresultant PRS of `f` and `g`.
  - `resultant(g, includePRS)` — resultant via PRS; **when `includePRS=True`, returns `(resultant, [PRS_polys])` tuple; when `False`, returns scalar only**; applies `kill=True` to reduce dimension.
  - `discriminant` — discriminant of `f`.
  - `eval(a, j=0)` — evaluate at point `a` in variable `x_j`; **coerces `a` to ground domain via `dom.convert(a)` before evaluation**; passes `kill=True` to `per`, reducing the variable count by one (eliminates the evaluated variable from the result).
  - Univariate-only operations (raise `ValueError` if `lev > 0`): `shift(a)` (Taylor shift `f(x+a)`), `decompose`, `sturm`, `gff_list`, `invert(g)` (modular inverse), `half_gcdex(g)`, `gcdex(g)`, `revert(n)`.
  - Conversion: `to_dict`, `from_dict`, `from_list`, `to_ring`, `to_field`, `convert`, `slice`.
  - Enumeration: `all_monoms`, `all_coeffs`, `all_terms` — dense enumeration including zeros (univariate only); **for zero polynomial, returns single element `[(0,)]` / `[dom.zero]`** rather than empty list.
  - Content/primitive: `content`, `primitive`, `terms_gcd`.
  - `homogenize(s)` — make all terms equal total degree by adjusting variable at index `s`; **if `s == number_of_variables` (new variable), appends a dimension and inserts padding exponents; if `s < number_of_variables` (existing variable), adds to that variable's exponent in-place**.
  - Structural: `exclude` (remove unused generators, returns removed indices + reduced DMP), `inject`, `eject`, `deflate`, `permute`.
  - `refine_root(s, t, eps, steps)` — refine an isolating interval; **if neither `eps` nor `steps` is given, defaults to `steps=1`** (single bisection step).
  - `is_cyclotomic` — **univariate only (`lev == 0`): delegates to `dup_cyclotomic_p`; multivariate: silently returns `False`** (no error raised). Contrast `PolyElement.is_cyclotomic` in `rings.py` which raises `MultivariatePolynomialError`.
  - Root isolation: `intervals(all, eps, sqf)` — isolate roots; **raises `PolynomialError` if multivariate (`lev > 0`)**; dispatches to 4 variants based on `all`/`sqf` flags. `refine_root`, `count_real_roots`, `count_complex_roots` — also univariate-only.
  - `cancel(g, include)` — cancel common factors in f/g; when `include=False`, returns `(cF, cG, F, G)` (content factors + reduced polys); when `include=True`, returns only `(F, G)`.
- `DMF` — Dense Multivariate Fraction (numerator/denominator pair) over K.
  - `new(rep, dom, lev, ring)` — classmethod that **skips `dmp_cancel`** (no GCD reduction); use when fraction is already known to be in reduced form. Contrast with `__init__` which always cancels.
  - `_parse(rep, dom, lev)` — input normalization; when rep is a (num, den) tuple, **negates both numerator and denominator if the denominator has a negative leading coefficient**, enforcing a canonical positive-denominator sign convention. Sets denominator to one if numerator is zero.
  - `per(num, den, cancel, kill, ring)` — construct new DMF; **if `kill=True` and `lev==0`, returns scalar `num/den`**.
  - `frac_unify(g)` — unify two DMFs across different domains; creates a local `per` closure that captures the unified domain and has the same kill/level-zero scalar-return behavior.
  - `poly_unify(g)` — unify DMF with a DMP; same local `per` closure pattern.
  - `half_per(rep, kill)` — create DMP from rep; if `kill=True` and `lev==0`, returns the raw rep.
  - `numer`, `denom`, `cancel`, `neg`, `add`, `sub`, `mul`, `pow`, `quo`, `exquo`.
    - `quo(f, g)` — computes fraction quotient; after computing result, **checks ring membership if a ring is set; raises `ExactQuotientFailed` if result is not in the ring**. `exquo` is an alias for `quo`.
  - `__rdiv__(g)` — reverse division (`g / self`); computes `invert()*g`, then **checks ring membership if a ring is set; raises `ExactQuotientFailed` if result is not in the ring**.
  - `__eq__(g)` — equality comparison; **if `g` is a DMP (whole polynomial, not fraction), unifies and checks that the denominator is the multiplicative identity (one) AND numerators match**; if `g` is another DMF, unifies fractions and compares directly; returns `False` on `UnificationFailed`.
- `ANP` — Algebraic Number Polynomial (univariate dense poly modulo a minimal polynomial over an algebraic extension).
  - Arithmetic: `neg`, `add`, `sub`, `mul`, `pow`, `div`, `rem`, `quo`, `exquo`.
  - `pow(n)` — for negative `n`, **computes modular inverse via `dup_invert` first**, then raises to `|n|`; result is always reduced modulo the defining relation.
  - `div(f, g)` — returns `(quotient, zero)`; `rem` always returns zero (field-like semantics via modular inverse).
  - `unify(g)` — reconcile two ANPs to a common domain/modulus; builds a local `per` closure. **When unified domain matches one original domain, reuses that original's modulus directly (no conversion); when it matches neither, converts the modulus from `f`'s domain**.
  - `LC`, `TC` — leading/trailing coefficient.
  - Conversion: `to_dict`, `to_sympy_dict`, `to_list`, `to_sympy_list`, `to_tuple`, `from_list`.

### [`rings.py`](rings.py)
Sparse polynomial rings and their elements (dict-based representation).

- `ring()`, `xring()`, `vring()` — ring constructor functions with explicit domain.
  - **`ring` returns `(ring,) + gens` (flat tuple); `xring` returns `(ring, gens)` (nested tuple for tuple-unpacking); `vring` injects gens into global namespace**.
- `sring(exprs, *symbols)` — construct ring from expressions; **auto-infers domain from coefficients via `construct_domain`** when no domain is specified.
- `PolyRing` — polynomial ring `K[x_1, ..., x_n]`.
  - `__new__` — caches ring objects; **when domain is composite, raises `GeneratorsError` if new ring symbols overlap with the domain's existing generators**.
  - `_gens_set` — cached set of canonical generator elements.
  - `free_module(rank)` — create free module over this ring.
  - `index(gen)` — resolve generator position; accepts `None` (→ 0), integer (**supports Python-style negative indexing**: `-1` → last gen), ring element, or string name; raises `ValueError` if out of bounds.
  - `add(*objs)` / `mul(*objs)` — aggregate a sequence of polynomials or nested containers (including generators) by sum/product; **recursively flattens nested lists and generator objects** via `is_sequence`; starts from `self.zero` / `self.one` respectively.
  - `to_ground()` — strip coefficient domain to its base; checks `is_Composite` **or** `hasattr(domain, 'domain')` to also handle algebraic fields not formally marked as composite.
  - `drop_to_ground(*gens)` — remove generators and inject them into the domain; **if no generators remain after removal, returns `self` unchanged** (does not reduce to the domain).
- `ring_new(element)` — convert Python object to ring element; **for list input, first tries `from_terms` (list of `(expv, coeff)` pairs); on `ValueError`, falls back to `from_list` (dense nested-list representation)**.
- `PolyElement` — element of a `PolyRing` (dict: monomial tuple → coefficient).
  - `_rebuild_expr(expr, mapping)` — recursively convert symbolic expression to ring element.
    - **Pow with non-negative integer exponents**: decomposed (base rebuilt and raised to the power).
    - **Pow with negative or non-integer exponents**: fall through to `domain.convert`, treated as ground domain elements.
  - `evaluate(x, a)` — substitute scalar for one variable; **univariate case returns a plain domain scalar** (drops the ring).
  - `subs(x, a)` — substitute scalar; **univariate case wraps result via `ring.ground_new`, returning a constant polynomial still in the ring**.
  - `compose(x, a)` — substitute a polynomial expression for a variable.
  - `set_ring(new_ring)` — transfer element to a different ring; **three-way dispatch**: same ring → return self; different symbols → reorder terms via `_dict_reorder` + `from_terms`; **same symbols, different domain → `from_dict` (no reorder)**.
  - `_sorted(seq, order)` — internal sorting helper for `coeffs`, `monoms`, `terms`; **when ordering is `lex`, sorts directly by monomial tuple** (Python tuple comparison is lexicographic); for other orderings, calls the order object as a key function.
  - `_iadd_monom(mc)` — in-place monomial addition; **copies self first if self is a canonical generator** to avoid mutating ring-cached generators.
  - `_iadd_poly_monom(p2, mc)` — in-place add product; same generator-copy safeguard.
  - `coeff(element)` — return scalar multiplier for a given monomial; accepts integer `1` for constant term or a monomial element; **raises `ValueError` for non-monomial arguments**.
  - `_term_div()` — returns a closure for term divisibility; **over non-field domains (e.g. ZZ), also checks that the coefficient divides evenly** before returning a quotient.
  - `__truediv__(p2)` — division operator; **if `p2` is a monomial (single-term), uses `p2**(-1) * p1` (inverse multiplication) instead of general `quo`** as a fast path; otherwise delegates to `quo(p2)`.
  - `div(fv)`, `rem(G)`, `quo(G)`, `exquo(G)` — multivariate polynomial division; `rem` manipulates the internal dict directly for efficiency, skipping quotient tracking; **`exquo` raises `ExactQuotientFailed` if remainder is nonzero**.
  - `degree`, `degrees`, `tail_degree`, `leading_monom`, `leading_term`.
  - `leading_expv()` — leading monomial exponent tuple per the ring's ordering; **returns `None` for the zero polynomial**, unlike `degree`/`degrees`/`tail_degree` which return `-oo`.
  - Arithmetic (`__add__`, `__sub__`, etc.): when subtracting/adding a scalar, **deletes the constant-term dict entry entirely if the result is zero** rather than storing a zero coefficient.
  - `__pow__(n)` — exponentiation; **raises `ValueError("0**0")` if self is zero and n is 0**; nonzero to zeroth power returns `ring.one`.
    - Single-term (monomial) fast path handles arbitrary exponents.
    - **≤5 terms → `_pow_multinomial` (multinomial coefficient expansion); >5 terms → `_pow_generic` (repeated squaring)**.
  - `is_squarefree`, `is_irreducible` — delegates to ring's dense algorithm wrappers (`dmp_sqf_p`, `dmp_irreducible_p`).
  - `is_cyclotomic` — **univariate only; raises `MultivariatePolynomialError` if ring has more than one generator**; delegates to `dup_cyclotomic_p`.
  - `cofactors(g)` — GCD with quotient factors; dispatches: both zero → triple zero; one zero → `_gcd_zero`.
    - `_gcd_zero(g)` — **if `g` is nonnegative, returns `g` as GCD; otherwise negates `g` (and cofactor sign) to ensure GCD is always nonnegative**.
    - **One is a single-term (monomial) → `_gcd_monom`** (componentwise monomial/coefficient GCD); general → deflates exponents, computes `_gcd`, inflates back.
  - `__eq__(p2)` — equality test; for non-ring-element `p2`: **if polynomial has >1 term, returns `False` immediately; if 0 or 1 term, compares the constant coefficient against `p2`** (single-term check via zero-monomial lookup).
  - `almosteq(p2, tolerance)` — approximate equality; for non-polynomial `p2`, **catches `CoercionFailed` and returns `False`** instead of raising.
  - `cancel(g)` — simplify fraction `f/g` by removing shared factors; **over non-field domains, divides by GCD and negates if denominator is negative; over field domains, clears denominators to ring, computes cofactors, then converts back with sign normalization** — if both numerator and denominator are negative, negates both; if only one is negative, shifts the sign to the content multiplier.
  - `clear_denoms()` — compute LCM of all coefficient denominators and multiply through; returns `(common_factor, integral_poly)`. **If domain is not a field or has no associated ring, returns `(domain.one, self)` unchanged**.
  - `drop(gen)` — remove a generator from the polynomial; **univariate case: returns ground domain scalar if polynomial is constant, raises `ValueError` if polynomial depends on the generator**; multivariate case: drops the variable position from each monomial, raising `ValueError` if any term has nonzero exponent in that generator.
  - `deflate(*G)` — compute GCD of all exponents per variable across `f` and `G`, divide all exponents by those GCDs; returns `(stride_tuple, [compressed_polys])`; used as preprocessing before GCD computation in `cofactors`.
  - `inflate(J)` — inverse of `deflate`; multiplies each exponent by the corresponding stride.
  - `trunc_ground(p)` / `rem_ground(p)` — reduce all coefficients modulo `p`; **over ZZ, uses symmetric representation** (if remainder > p//2, subtracts p to center around zero); over other domains, uses plain modular remainder. Strips zero-coefficient terms afterward.
  - `diff`, `integrate`, `eval`, `content`, `primitive`, `strip_zero`.
  - `_gcd(g)` — GCD dispatch: **QQ → `_gcd_QQ` (clears denoms, delegates to ZZ), ZZ → `_gcd_ZZ` (heuristic GCD via `heugcd`), other domains → fallback to `ring.dmp_inner_gcd`** (dense representation).
  - Cross-ring dispatch (`__add__`, `__sub__`, `__mul__`, `__divmod__`): when `p2` is a `PolyElement` from a different ring, checks nested domain relationships.
    - If `p2.ring.domain` is a `PolynomialRing` whose `.ring` matches `p1.ring`, **delegates to `p2.__radd__`/`__rsub__`/`__rmul__`** (outer ring handles).
    - `__divmod__`: if `p1.ring.domain` is a `PolynomialRing` whose `.ring` matches `p2.ring`, **falls through to coerce `p2` as a ground element**; if the reverse holds (`p2.ring.domain.ring == p1.ring`), **delegates to `p2.__rdivmod__` which returns `NotImplemented`**.
  - `quo_ground(x)` — divide all coefficients by scalar `x`; **over fields, uses exact division; over non-field domains (e.g. ZZ), silently drops terms whose coefficients are not evenly divisible** rather than raising an error.

### [`fields.py`](fields.py)
Sparse rational function fields and their elements.

- `field()`, `xfield()`, `vfield()` — construct rational function field with explicit domain.
- `sfield(exprs, *symbols)` — construct a **rational function field** from symbolic expressions; splits each expression into numerator/denominator, extracts dictionary representations, and **auto-infers domain from coefficients via `construct_domain`** when no domain is specified; pairs numerator/denominator reps (stepping by 2) to build `FracElement` objects.
- `FracField` — multivariate distributed rational function field K(x₁,…,xₙ).
  - `__new__` — caches field objects; assigns generator symbols as attributes on the field; **skips `setattr` if an attribute with that name already exists (`hasattr` guard)**, preventing generator names like `'domain'` or `'ring'` from overwriting internal attributes.
  - `ground_new(element)` — create element from ground coefficient; **if ring coercion fails and domain has an associated field (e.g. ZZ→QQ), splits element into numer/denom via the field and constructs a proper fraction**.
  - `from_expr(expr)` / `_rebuild_expr` — recursively decompose a symbolic expression tree into a **rational function** field element; maps Add→sum, Mul→product, integer Pow→power, and leaf nodes to domain coefficients.
    - **If ground domain fails to convert a leaf (`CoercionFailed`) and domain has an associated field, retries via `domain.get_field()`** (e.g. ZZ→QQ).
    - Distinct from `PolyElement._rebuild_expr` in `rings.py` which produces polynomial ring elements (not fractions).
- `FracElement` — element of a `FracField` (numerator/denominator pair).
  - `_extract_ground(element)` — coerce a scalar for arithmetic; tries `domain.convert` first.
    - **If that fails and domain has an associated field (e.g. ZZ→QQ), retries via the field and returns `(numer, denom)` split**; returns `(0, None, None)` on total failure.
  - Arithmetic (`__add__`, `__sub__`, `__mul__`, etc.): when the other operand is a `FracElement` from a different field, **checks nested domain relationships**: if `g.field` matches `self.field.domain.field`, treats `g` as a ground element; if `self.field` matches `g.field.domain.field`, **delegates to `g.__rsub__`/`g.__rmul__`** (the outer field handles the operation).
  - `__pow__(n)` — exponentiation; **non-negative `n`**: raises numer/denom to `n` via `raw_new` (no GCD reduction); **negative `n` with nonzero element**: swaps numer/denom and raises to `|n|`; **negative `n` with zero element**: raises `ZeroDivisionError`.
  - `diff(x)` — partial derivative of the rational function w.r.t. `x`; applies the **quotient rule**: `(numer' * denom - numer * denom') / denom²`.
  - `__eq__(g)` — if `g` is same dtype, compares both numer and denom; **if `g` is any other value, checks `numer == g` and `denom == ring.one`** (treats non-fraction values as having unit denominator).

---

## High-Level API

### [`polytools.py`](polytools.py)
User-facing `Poly` class and public free functions for polynomial manipulation.

- `Poly` — main symbolic polynomial class.
  - `__new__(cls, rep, *gens, **args)` — type-based dispatch: **dict → `_from_dict`; other iterable → `_from_list`; existing Poly → `_from_poly`; other expression → `_from_expr`**. Distinct from `DMP.__init__` which dispatches on dict/non-list-scalar/list for internal dense representations.
  - `free_symbols_in_domain` — symbols appearing only in the coefficient ring (not the indeterminates); **for composite domains (`is_Composite`), collects free symbols from the domain's sub-generators; for expression domain (`is_EX`), iterates over all coefficient values and collects their free symbols**; returns empty set for simple numeric domains.
  - `new(rep, *gens)` — construct Poly from raw `DMP` representation; **raises `PolynomialError` if `rep.lev != len(gens) - 1`** (nesting level must match generator count minus one).
  - `_from_poly(rep, opt)` — construct from existing Poly; **if generators are the same set in different order, calls `reorder`; if generators differ, falls back to `_from_expr`**.
  - `_from_expr`, `_from_dict`, `_from_list` — alternative constructors.
  - `ltrim(gen)` — remove unused leading generators; **raises `PolynomialError` if two distinct terms collapse to the same monomial** after truncation.
  - `terms_gcd()` — extract GCD of monomial exponents from all terms; returns `(exponent_tuple, reduced_poly)`.
  - `cancel(g, include)` — cancel common factors of f/g; when `include=False`, converts domain to its associated Ring before returning the content ratio as a SymPy expression.
  - `count_roots(inf, sup)` — count roots in interval; **if one bound is real and the other complex, converts the real bound to `(value, QQ.zero)` tuple** before delegating to complex root counter.
  - `nth_power_roots_poly(n)` — polynomial whose roots are n-th powers of f's roots; **raises `ValueError` if `n` is not a positive integer** (rejects zero, negative, non-integer).
  - `ground_roots()` — roots by factorization over the coefficient domain; **only returns roots from linear factors, silently omitting irreducible quadratic or higher-degree factors**.
  - `real_roots`, `all_roots`, `root` — root enumeration via `CRootOf`.
  - `slice(x, m, n)` — extract terms with degrees in `[m, n)` for generator `x`; **when called with only two numeric args (no generator), defaults to the first generator (`j=0`) and reinterprets args as `(m, n)`**.
  - `replace(x, y)` — replace generator `x` with symbol `y`; **if `y` is not already a generator and the domain is composite (`is_Composite`), raises `PolynomialError` when `y` appears in the coefficient domain's symbols** (prevents symbol appearing as both generator and coefficient variable); univariate shorthand: single arg replaces the sole generator.
  - `reorder`, `inject`, `eject` — generator manipulation.
    - `inject`: **returns `self` unchanged if the coefficient domain is purely numerical** (no ground generators to promote).
    - `eject`: **only supports front or back generators**; raises `NotImplementedError` for middle generators.
  - `half_gcdex(g, auto)`, `gcdex(g, auto)`, `invert(g, auto)` — extended Euclidean algorithm and modular inverse; **if `auto=True` and domain is a ring, auto-promotes to fraction field** (e.g. ZZ→QQ) before computing.
  - `sturm(auto=True)` — Sturm sequence; **if `auto=True` and domain is a ring, auto-converts to field** (e.g. ZZ→QQ) before computing.
  - `to_ring`, `to_field`, `set_domain` — domain conversion.
  - Content/primitive: `content`, `primitive`, `monic`.
    - `content()` — GCD of all coefficients; **only allows `polys` flag (not `auto`)**, unlike `monic` which accepts both.
    - `monic(auto=True)` — divides all coefficients by leading coefficient; **if `auto=True` and domain is a ring (e.g. ZZ), auto-converts to fraction field (e.g. QQ) before dividing**.
  - `clear_denoms(convert=False)` — eliminate fractional coefficients; **if domain is not a field (`not has_Field`), returns `(S.One, self)` immediately** (no-op for integer-coefficient polynomials); otherwise computes LCM of denominators and scales; when `convert=True`, retracts result to the associated ring (e.g. QQ→ZZ).
  - `per(rep, gens, remove)` — construct Poly from internal rep; **if `remove` index is given and removing that generator leaves no remaining generators, returns a plain SymPy scalar** (via `dom.to_sympy`) instead of a Poly.
  - `eval(a, j)` — evaluate polynomial at a point; **on `CoercionFailed`, auto-widens the coefficient domain** by constructing a domain for `a`, unifying with the current domain, converting, and retrying.
  - `_eval_subs(old, new)` — internal substitution: if `old` is a generator, evaluates at `new` when numeric.
    - **For non-numeric `new`, tries `replace(old, new)`; silently falls back to `as_expr().subs(old, new)` on `PolynomialError`**.
    - Also falls back to expression-level subs when `old` is not a generator.
  - `_gen_to_level(gen)` — resolve generator to internal nesting level; accepts integer index (supports **Python-style negative indexing**, e.g. -1 for last generator) or symbolic variable; **raises `PolynomialError` if integer index is out of range**.
  - `homogenize(s)` — make polynomial homogeneous using symbol `s`; **if `s` is already a generator, reuses its index; if new, appends it to generators**. Raises `TypeError` if `s` is not a `Symbol`.
  - `is_ground` — `True` if polynomial is an element of the ground (coefficient) domain; **symbols not among declared generators are part of the ground domain** (e.g. `Poly(y, x).is_ground` is `True`).
  - `is_univariate`, `is_multivariate` — determined purely by **number of declared generators** (`len(gens)`), not by which symbols actually appear in the expression; e.g. `Poly(x**2, x, y).is_multivariate` returns `True`.
  - `homogeneous_order()` — return the total degree if all terms share the same degree; `is_homogeneous` for a boolean check.
  - `unify(g)` / `_unify(g)` — reconcile two Polys to a common variable ordering and coefficient domain.
    - Merges generator sets via `_unify_gens`, reorders monomial dicts via `_dict_reorder`, converts coefficients to unified domain.
    - **If `g` is not a Poly (e.g. a plain scalar), attempts to interpret it as a constant in `f`'s coefficient domain**; raises `UnificationFailed` if conversion fails.
  - `__eq__(other)` — equality comparison; **if generators match but coefficient domains differ, attempts domain unification; returns `False` (not an error) if `UnificationFailed`**.
  - `__pow__(n)` — if `n` is a non-negative integer, delegates to `pow(n)`; **otherwise falls back to `as_expr()**n`** (converts to symbolic expression), enabling negative/fractional exponents at the expression level.
  - `coeff_monomial(monom)` — return coefficient of a specific monomial; delegates to `nth`.
    - **Unlike `Expr.coeff()`, does not collect terms across other variables** — returns the exact coefficient of the given power-product.
  - `nth(*N)` — return coefficient by generator exponents (e.g. `nth(1, 2)` for `x^1·y^2`); more efficient than `coeff_monomial` when exponents are already known.
  - Ground arithmetic: `add_ground`, `sub_ground`, `mul_ground`, `quo_ground` (truncating scalar division), `exquo_ground` (exact scalar division; **raises `ExactQuotientFailed` if any coefficient is not evenly divisible**).
  - `integrate(*specs, auto=True)` — antiderivative; **if `auto=True` and domain is a ring (e.g. ZZ), auto-converts to fraction field (e.g. QQ) before integrating** (since integration introduces fractional coefficients); `diff` does not perform this conversion.
  - `factor_list()` — returns list of irreducible factors; **catches `DomainError` from internal representation and falls back to `(S.One, [(f, 1)])`** (returns original polynomial as single factor) when the coefficient domain doesn't support factorization.
  - Arithmetic: `add`, `sub`, `mul`, `sqr`, `pow`, `div`, `rem`, `quo`, `exquo`, `pdiv`, `prem`, `pquo`, `pexquo`.
    - `prem(g)` — pseudo-remainder; **caveat: safe only for computing subresultant PRS in Z[x]**; for Euclidean/Sturmian PRS in Z[x], use functions in `subresultants_qq_zz` module instead (`rem_z` premultiplies by absolute value of LC, unlike `prem`).
    - `div(f, g, auto=True)` — when `auto=True` and domain is a ring (not a field), **promotes both operands to the fraction field before dividing**.
      Attempts to retract quotient/remainder back to the ring; keeps field-domain results silently if retraction fails.
    - `mul(g)` — if `g` is not a Poly, falls back to `mul_ground` (scalar multiplication); same pattern for `add`/`sub`.
    - `exquo(g)` — catches `ExactQuotientFailed` and **re-raises with `f.as_expr()`, `g.as_expr()`** so the error message contains human-readable symbolic expressions instead of internal representations.
    - `pexquo(g)` — catches `ExactQuotientFailed` and **re-raises via `exc.new(f.as_expr(), g.as_expr())`**, converting internal representations to symbolic expressions (same pattern as `exquo`).
    - `pquo` catches `ExactQuotientFailed` and **re-raises with original input expressions**.
  - GCD/resultant: `gcd`, `lcm`, `cofactors`, `resultant`, `discriminant`, `subresultants`.
    - `resultant(g, includePRS)` — when `includePRS=True`, returns `(resultant_value, [PRS_polys])` tuple instead of a single scalar.
  - Factorization: `factor_list`, `sqf_list`, `sqf_list_include`, `sqf_part`.
    - `_symbolic_factor_list` — internal helper for factoring symbolic expressions; when a base is raised to a non-integer exponent and its polynomial leading coefficient is negative (not positive), **appends the coefficient as a factor with exponent 1** (avoids computing negative numbers raised to fractional powers); when coefficient is positive, appends it normally with the non-integer exponent.
    - `factor_list` (via `_generic_factor_list`): **when input is a rational expression (nontrivial denominator) and `frac=True`, returns `(coeff, numer_factors, denom_factors)` 3-tuple; when `frac=False` (default), silently drops denominator factors** and returns only `(coeff, numer_factors)`.
    - `sqf_list` returns `(coeff, [(factor, mult), ...])` with leading coefficient separated; **`coeff` is converted from internal domain to SymPy via `dom.to_sympy`**, unlike similar list methods (e.g. `gff_list`) which return raw `Poly` wrappers only. `sqf_list_include` folds the coefficient into the factor tuples.
- `_sorted_factors(factors, method)` — sort `(poly, exp)` factor pairs; **for `method='sqf'` (square-free), primary sort key is the exponent; for other methods (regular factorization), primary key is representation length** (`len(rep)`); secondary keys are rep length, generator count, and raw rep.
- `to_rational_coeffs(f)` — transform polynomial with irrational coefficients to rational coefficients; **only applies to polynomials whose irrational coefficients involve exclusively square roots; returns `None` immediately if any coefficient contains a root of order > 2** (e.g. cube roots).
  - **Tries rescaling `x → α·x` first, then translation `x → x + β`**; returns `(lc, alpha, None, g)` or `(None, None, beta, g)`.
- `terms_gcd(f)` (free function) — extract monomial GCD from expression.
  - **If coefficient domain lacks a Ring (`not domain.has_Ring`), skips coefficient extraction and defaults coefficient to `S.One`**.
  - Returns the original expression unchanged if both the extracted coefficient and monomial factor are trivial (both equal 1).
- `reduced(f, G)` — divide polynomial `f` modulo a set of polynomials `G`, returning quotients and remainder.
  - **Auto-promotes ring domain to its fraction field** for division, then attempts to retract results back to the ring (keeps field results if retraction fails).
- `factor(f)` — compute irreducible factorization; **on `PolynomialError` for non-commutative expressions, falls back to `factor_nc` from `exprtools`**; re-raises for commutative expressions.
- `sqf_norm(f)` — compute square-free norm over algebraic extensions; returns `(Integer(s), shifted_poly, norm_poly)` where shift `s` is **always wrapped as `Integer` regardless of `polys` flag**.
- `div`, `rem`, `quo`, `exquo` — public free functions for polynomial division; each **catches `PolificationFailed` and re-raises as `ComputationFailed`**; `div`/`rem` accept `auto` flag for automatic ring→field promotion; `rem` over ZZ may return the dividend unchanged (LC not divisible), while over QQ produces a reduced remainder.
- `gff(f)` — **stub that unconditionally raises `NotImplementedError('symbolic falling factorial')`**; the list-of-factors variant `gff_list` is functional.
- `cancel(f, g)`, `groebner`, `sqf`, `decompose`, `sturm` — public free functions.
- `poly_from_expr` — expression-to-Poly conversion.
- `parallel_poly_from_expr(exprs)` — convert multiple expressions to Polys simultaneously; **raises `PolynomialError` if any auto-detected generator is a `Piecewise` expression**.
  - **When exactly 2 inputs are both already `Poly` objects, takes a fast path: unifies them directly and returns early** without dict-based construction.
  - For 3+ inputs or mixed Poly/expr inputs, **converts any already-constructed Poly objects back to symbolic expressions via `as_expr()`** before uniform dictionary extraction.
  - Then collects all coefficients into one flat list to infer a single unified domain.
- `degree`, `degree_list`, `content`, `monic` — query functions.
- `LM(f, order)` — leading monomial (dominant power-product **without** scalar coefficient); converts internal rep to symbolic expression via `as_expr()`; respects custom monomial ordering.
- `LT(f, order)` — leading term (monomial **with** scalar coefficient); same ordering dispatch as `LM`.
- `LC(f, order)` — leading coefficient; **when `order` is specified, delegates to `coeffs(order)` and returns first element** (re-sorts by custom ordering); when `order` is None, calls internal rep's `LC()` and converts via `dom.to_sympy`.
- `primitive(f)` — compute content and primitive form; **if `polys` option is set, returns primitive part as a `Poly`; otherwise converts to symbolic expression via `as_expr()`**.
- `gcd`, `lcm`, `gcd_list`, `lcm_list`, `resultant`, `discriminant` — algebraic operations.
  - `gcd_list(seq)` — GCD of a list of polynomials; returns `S.Zero` for empty input.
    - **Two-phase fallback**: first tries numerical GCD (via `construct_domain`); if polynomial conversion fails, **retries numerical GCD on the failed expressions**.
    - **Early-exits the iterative reduction when the running GCD becomes one** (cannot reduce further).
  - `gcd(f, g)` — **if `f` is iterable, delegates to `gcd_list`; when `g` is also provided, `g` is prepended to generators (not treated as a polynomial input)**.
    - On `PolificationFailed`, **falls back to `construct_domain` on raw expressions and delegates to `domain.gcd`**; raises `ComputationFailed` if domain doesn't support GCD.
- `half_gcdex`, `gcdex`, `invert` — extended Euclidean algorithm and modular inverse; **on `PolificationFailed`, fall back to `construct_domain` on raw expressions and delegate to `domain.gcdex`/`domain.invert`; raise `ComputationFailed` if domain doesn't support the operation**.
- `cofactors(f, g)` — GCD with quotient factors; **if polification fails, falls back to `construct_domain` on raw expressions and calls `domain.cofactors`; raises `ComputationFailed` if the fallback domain raises `NotImplementedError`**.
- `intervals(F, eps, inf, sup)` — compute isolating intervals for real roots; **accepts a single expression or an iterable of expressions**.
  - **Single expression**: wraps as `Poly` and delegates; **returns empty list `[]` for a constant/non-polynomial input** (catches `GeneratorsNeeded` silently).
  - **Iterable of expressions**: converts all via `parallel_poly_from_expr` into a common QQ domain; **raises `MultivariatePolynomialError` if more than one generator is detected**; delegates to `dup_isolate_real_roots_list`.
  - **Validates `eps > 0` (raises `ValueError` if not positive)**.
- `count_roots`, `real_roots`, `nroots`, `refine_root` — root functions; **each catches `GeneratorsNeeded` and re-raises as `PolynomialError`** when input has no generators (e.g. plain integer).
  - `Poly.nroots(n, maxsteps)` — compute numerical root approximations; **for QQ coefficients, multiplies through by LCM of all denominators to convert to ZZ** before passing to the iterative solver (for accuracy); for ZZ, casts to Python `int`; for other domains, evaluates coefficients numerically.
- `poly(expr)` — efficiently convert expression to `Poly` by recursively decomposing `Add`/`Mul`/`Pow` nodes; **non-sum factors in a product are collected separately: numeric factors are multiplied as scalars, while symbolic non-sum factors are converted to `Poly` via `_from_expr`** before multiplication.
- `PurePoly` — Poly subclass with equality ignoring generator names; compares by number of generators (not identity).
  - `__eq__` — checks `len(f.gens) == len(g.gens)` (not name equality); **attempts domain unification and returns `False` on `UnificationFailed`** (same pattern as `Poly.__eq__`).
- `GroebnerBasis` — Gröbner basis representation class.
  - `__new__(F, *gens, **args)` — converts input expressions to `Poly` objects via `parallel_poly_from_expr`, constructs a `PolyRing` from generators/domain/ordering, **converts each `Poly` to a ring element via `ring.from_dict(poly.rep.to_dict())`**, then calls the core Gröbner algorithm.
  - `__iter__`, `__getitem__` — **if `options.polys` is set, yields/returns `Poly` objects; otherwise yields/returns symbolic expressions** (via `.exprs`).
  - `__eq__(other)` — if `other` is a `GroebnerBasis`, compares internal basis and options; **if `other` is any iterable (e.g. plain list), compares against both `.polys` and `.exprs` representations** (equality succeeds if either matches).
  - `fglm(order)` — convert basis to a different monomial ordering via the FGLM algorithm; **promotes domain to its fraction field for computation, then clears denominators and resets domain** if the original was not a field (e.g. ZZ).
  - `is_zero_dimensional` — check if ideal is zero-dimensional.
  - `reduce(expr)` — reduce polynomial modulo the basis; **auto-promotes ring domain to its fraction field**, then attempts to retract results back; **silently keeps field-domain results if retraction fails** (`CoercionFailed` is caught).
  - `contains(poly)` — check ideal membership; **returns `True` iff `reduce(poly)` yields zero remainder**.

### [`polyfuncs.py`](polyfuncs.py)
High-level polynomial utility functions (symbolic level).

- `symmetrize(poly)` — rewrite in terms of elementary symmetric polynomials; returns `(symmetric_part, non_symmetric_remainder)` pair.
  - **If input cannot be converted to polynomial and is a plain number, returns `(number, 0)` without error**; in `formal` mode, appends an empty symbol-mapping list.
  - **Non-homogeneous inputs have their constant term extracted first**; iterative decomposition then operates on the homogeneous remainder.
- `horner(poly)` — convert polynomial to Horner form (symbolic rewriting, not evaluation).
- `interpolate(data, x)` — construct interpolating polynomial.
- `rational_interpolate(data, degnum, X)` — rational function interpolation.
- `viete(poly, roots)` — Viète's formulas relating roots to coefficients; **if `roots` is a `Basic` expression (not a list), reinterprets it as a generator symbol** and auto-generates numbered root symbols.

---

## Dense Polynomial Operations

### [`densearith.py`](densearith.py)
Low-level dense polynomial arithmetic on coefficient lists.

- `dup_add`, `dmp_add`, `dup_sub`, `dmp_sub` — basic arithmetic; `dup_add` **only strips leading zeros when both operands share the same degree** (possible cancellation); skips stripping when one operand has strictly higher degree.
  - `dmp_sub` degree mismatch: **when the subtrahend has higher degree, negates the excess higher-order prefix via `dmp_neg`** before element-wise recursive subtraction on the aligned tail.
- `dup_mul`, `dmp_mul` — multiplication; `dup_mul` **switches from naive O(n²) convolution to Karatsuba divide-and-conquer at `max(df,dg)+1 >= 100`**.
- `dup_sqr`, `dmp_sqr`, `dup_pow`, `dmp_pow` — squaring and exponentiation.
- `dup_add_term`, `dmp_add_term`, `dup_sub_term`, `dmp_sub_term` — add/subtract a monomial `c*x^i`.
  - **When exponent `i` exceeds the current degree, prepends the coefficient and zero-pads the gap** to extend the representation.
  - **`dmp_sub_term` delegates to `dup_add_term` with negated coefficient** (not `dup_sub_term`) when reducing to univariate.
- `dup_add_mul`, `dmp_add_mul`, `dup_sub_mul`, `dmp_sub_mul` — fused multiply-add/sub.
- `dup_mul_term`, `dmp_mul_term` — multiply polynomial by `c*x^i` (univariate) or `c(x₂..xₙ)*x₀^i` (multivariate); **`dmp_mul_term` returns `f` unchanged if `f` is zero, but returns a fresh canonical zero if `c` is zero**.
- `dup_add_ground`, `dmp_add_ground`, `dup_sub_ground`, `dmp_sub_ground` — add/subtract a ground domain scalar from a polynomial; **multivariate variants promote the scalar to nested representation via `dmp_ground`** before delegating to `*_add_term`/`*_sub_term` at degree 0.
- `dup_mul_ground`, `dmp_mul_ground` — multiply polynomial by ground constant; **`dup_mul_ground` short-circuits to `[]` (zero polynomial) if either `c` is zero or `f` is empty**, without iterating over coefficients.
- `dup_quo_ground`, `dmp_quo_ground` — divide all coefficients by a constant; **over fields (`K.has_Field`), uses `K.quo` (exact field division); over rings (e.g. ZZ), uses `//` (floor division)**.
- `dup_div`, `dmp_div` — polynomial division; **dispatches to `dup_ff_div`/`dup_rr_div` based on `K.has_Field`** (field domains get exact division, ring domains get truncated division).
  - `dup_ff_div`, `dup_rr_div`, `dmp_ff_div`, `dmp_rr_div` — **raise `PolynomialDivisionFailed` if remainder degree fails to strictly decrease** between loop iterations (stall detection to prevent infinite loops).
  - `dmp_ff_div`/`dmp_rr_div` (multivariate): **recursively divide leading coefficients** (which are polynomials in fewer variables); `dmp_ff_div` breaks the loop when the recursive leading coefficient division yields a nonzero remainder.
  - `dup_rr_div`/`dmp_rr_div` (ring division): **breaks the division loop early when the remainder's leading coefficient is not evenly divisible by the divisor's leading coefficient**, returning partial quotient and current remainder.
- `dup_rem`, `dmp_rem`, `dup_quo`, `dmp_quo` — remainder, quotient.
- `dup_exquo`, `dmp_exquo` — exact quotient; **raises `ExactQuotientFailed` if remainder is nonzero**.
- `dup_pdiv`, `dmp_pdiv`, `dup_prem`, `dmp_prem` — pseudo-division; **same `PolynomialDivisionFailed` stall detection as regular division**.
- `dup_pquo`, `dmp_pquo` — pseudo-quotient (discards remainder).
- `dup_pexquo`, `dmp_pexquo` — exact pseudo-quotient; **raises `ExactQuotientFailed` if pseudo-remainder is nonzero**.
- `dup_lshift(f, n, K)` — multiply by `x**n` (append n zeros); **returns empty list unchanged for zero polynomial** (no spurious zero-padding).
- `dup_rshift(f, n, K)` — divide by `x**n` (truncate trailing n coefficients).
- `dup_abs` — absolute values of coefficients.
- `dup_max_norm`, `dmp_max_norm` — maximum coefficient norm; **returns `K.zero` for zero polynomial (empty list)**.
- `dup_l1_norm`, `dmp_l1_norm` — L1 norm (sum of absolute coefficient values); **returns `K.zero` for zero polynomial (empty list)**.
- `dup_expand`, `dmp_expand` — multiply together several polynomials; **returns multiplicative identity (`[K.one]` / `dmp_one`) for empty input list**.

### [`densebasic.py`](densebasic.py)
Low-level dense polynomial basics: construction, conversion, queries.

- `dmp_validate`, `dmp_normal`, `dmp_convert` — validation and domain conversion.
- `dup_from_dict`, `dmp_from_dict`, `dmp_to_dict`, `dmp_from_sympy` — format conversions; `dup_from_dict` **accepts both integer keys and single-element tuple keys** `{(k,): c}`, dispatching by `type(max_key) is int`.
  - `dmp_to_dict` — **when the computed degree is negative infinity (zero polynomial not caught by earlier zero check), sets degree to −1** so the iteration loop produces an empty result.
- `dup_to_tuple`, `dmp_to_tuple` — convert coefficient lists to (nested) tuples for hashing; `dmp_to_tuple` **recursively converts each nesting level**, producing an immutable hashable form suitable for dict keys.
- `dmp_degree`, `dmp_LC`, `dmp_TC`, `dmp_ground_LC`, `dmp_ground_TC` — degree/coefficient queries; `dmp_ground_LC`/`dmp_ground_TC` drill through each nesting level to extract the innermost leading/trailing coefficient.
- `dmp_true_LT(f, u, K)` — leading term as `(monom_tuple, coeff)`; **if innermost univariate list is empty (zero poly), appends exponent 0** instead of computing `len-1` (which would give −1).
- `dmp_zero`, `dmp_one`, `dmp_zero_p`, `dmp_one_p`, `dmp_ground` — constants and predicates.
- `dmp_ground_p(f, c, u)` — test if polynomial is a constant; **if `c` is `None`, checks if `f` is any ground element** (not a specific value); if `c` is falsy (e.g. 0), delegates to `dmp_zero_p`.
- `dup_reverse(f)` — compute `x^n * f(1/x)` (reciprocal transformation) by reversing the coefficient list and stripping leading zeros.
- `dup_deflate`, `dmp_deflate` — map `x^m → y` by computing the GCD of all nonzero-coefficient exponents and slicing; **returns stride 1 unchanged for degree ≤ 0**.
- `dup_multi_deflate`, `dmp_multi_deflate` — simultaneously reduce exponent gaps across multiple polynomials; **`dmp_multi_deflate` delegates to `dup_multi_deflate` when `u==0`**.
- `dup_inflate`, `dmp_inflate` — inverse of deflation; maps `y` back to `x^m`; **raises `IndexError` if `m` ≤ 0; returns `f` unchanged if `m == 1` or `f` is empty**.
- `dup_apply_pairs(f, g, h, args, K)` — apply binary function `h` element-wise to paired coefficients of two univariate dense lists; **pads the shorter list with `K.zero` on the left (high-degree end)** to align by degree before zipping.
- `dup_strip` — remove leading zeros from univariate coefficient list; **short-circuits via `if not f or f[0]`** (returns immediately when list is empty or first coefficient is nonzero, avoiding iteration).
- `dmp_strip` — strip leading zero coefficients from nested lists; delegates to `dup_strip` at level 0.
- `dup_terms_gcd`, `dmp_terms_gcd` — remove GCD of term exponents; `dup_terms_gcd` returns `(0, f)` unchanged when trailing coefficient is nonzero or polynomial is empty.
- `dmp_inject(f, u, K, front)` — flatten `K[X][Y]` → `K[X,Y]`; when `front=True`, coefficient-ring generators precede outer generators in monomial tuples.
- `dmp_eject(f, u, K, front)` — reverse of inject: `K[X,Y]` → `K[X][Y]`; **splits monomial tuple using `K.ngens`: when `front=False` (default), trailing positions map to coefficient ring generators; when `front=True`, leading positions do**.
- `dmp_list_terms(f, u, K, order)` — list all non-zero terms as `(monom_tuple, coeff)` pairs; **for zero polynomial returns `[((0,)*(u+1), K.zero)]`** (single zero-monomial entry, not empty list).
- `dmp_nest(f, l, K)` — wrap a multivariate value in `l` additional nesting levels; **if `f` is not a list (plain scalar), delegates to `dmp_ground`** instead of wrapping in nested lists.
- `dmp_ground_nth(f, N, u, K)` — extract ground-level coefficient at multi-index N from nested lists; **if polynomial at some level has degree −∞ (zero polynomial), sets degree to −1** to avoid indexing errors; returns `K.zero` if index exceeds length.
- `dmp_swap(f, i, j, u, K)` — transpose two indeterminates by swapping positions `i` and `j` in each exponent tuple (via dict round-trip); **raises `IndexError` if either index is out of bounds**; returns `f` unchanged if `i == j`.
- `dmp_permute(f, P, u, K)` — reorder indeterminates by applying a permutation vector P to exponent tuples (via dict round-trip).
- `dmp_exclude(f, u, K)` — detect and remove variable dimensions unused by any term; returns `(removed_indices, reduced_poly, new_level)`.
- `dmp_include(f, J, u, K)` — re-insert previously excluded variable dimensions at specified positions.

### [`densetools.py`](densetools.py)
Advanced dense polynomial operations: calculus, evaluation, composition, denominator clearing.

- `dup_eval(f, a, K)` — evaluate univariate polynomial at point using Horner scheme; **if `a` is zero (falsy), returns the trailing coefficient directly** instead of iterating.
- `dmp_eval(f, a, u, K)` — evaluate multivariate polynomial at `x_0 = a` using Horner scheme; **if `a` is zero (falsy), returns the trailing coefficient `dmp_TC(f, K)` directly** (same shortcut as `dup_eval`).
- `dmp_eval_in(f, a, j, u, K)` — evaluate multivariate polynomial at `x_j = a` by index `j`; uses recursive descent through nesting levels to the target variable, then applies Horner evaluation; **raises `IndexError` if `j` is out of bounds**.
- `dmp_eval_tail(f, A, u, K)` — evaluate at trailing variables `x_{n-len(A)+1}, …, x_n`.
- `dup_diff`, `dmp_diff`, `dmp_diff_in` — differentiation; **when derivative order exceeds polynomial degree, univariate `dup_diff` returns `[]` (empty list) while multivariate `dmp_diff` returns `dmp_zero(u)`** (nested zero matching the variable depth).
- `dup_integrate`, `dmp_integrate`, `dmp_integrate_in` — integration.
- `dup_compose`, `dmp_compose` — polynomial composition; `dup_compose` **short-circuits when inner polynomial has length ≤ 1** (constant or zero): evaluates `f` at the leading coefficient of `g` and returns the scalar result wrapped as a single-element list.
- `dup_clear_denoms(f, K0, K1)` — clear fractional coefficients from univariate polynomial; computes LCM of denominators. **If `K1` is None and `K0` has no associated ring, falls back to using `K0` itself as the target domain**.
- `dmp_clear_denoms(f, u, K0, K1)` — clear fractional coefficients from multivariate polynomial; **if `K1` is None and `K0` has no associated ring, falls back to using `K0` itself as the target domain**; uses `_rec_clear_denoms` to recursively traverse nested coefficient lists computing LCM of all denominators.
- `dup_trunc(f, p, K)` — reduce coefficients modulo constant `p`; **over ZZ, uses symmetric representation** (if remainder > p//2, subtracts p to center around zero); over other domains, uses plain modular remainder.
- `dmp_trunc` — reduce multivariate polynomial modulo a polynomial in the inner variable.
- `dmp_ground_trunc` — reduce multivariate polynomial coefficients modulo a constant (delegates to `dup_trunc` at level 0).
- `dup_monic`, `dmp_ground_monic` — make polynomial monic.
- `dup_content`, `dmp_ground_content`, `dup_primitive`, `dmp_ground_primitive` — ground-level content and primitive part (GCD of scalar coefficients only; for multivariate coefficient GCD, see `dmp_content` in `euclidtools.py`).
  - `dup_content` **skips early termination (break when GCD reaches 1) for QQ domain**, iterating all coefficients; for other domains (e.g. ZZ), breaks early when running GCD becomes one.
- `dup_real_imag` — split into real/imaginary parts.
- `dup_mirror`, `dup_scale`, `dup_shift` — polynomial transformations (sign-flip, rescale, Taylor shift).
- `dup_transform(f, p, q, K)` — functional transformation `q^n * f(p/q)`; **returns `[]` immediately for zero polynomial**.
- `dup_decompose(f, K)` — functional decomposition into `[g, h]` where `f = g(h)` and `deg(g), deg(h) > 1`.
  - Iterates degree divisors, computes candidate inner part, validates outer part via repeated division (**rejects split if any remainder has positive degree**).
- `dup_sign_variations` — count sign changes in coefficient sequence.
- `dup_revert(f, n, K)` — compute `f⁻¹ mod x^n` (power series inversion) via **Newton iteration** (`g ← 2g − f·g²` mod increasing powers of x); distinct from `dup_invert` in `euclidtools.py` which computes modular inverse via extended GCD.
- `dmp_revert` — multivariate variant of `dup_revert`.
- `dmp_lift(f, u, K)` — convert algebraic coefficients to integers by multiplying conjugate permutations; **raises `DomainError` if `K` is not an algebraic domain** (e.g. plain QQ).

---

## GCD & Euclidean Algorithms

### [`euclidtools.py`](euclidtools.py)
Production Euclidean algorithms, GCD/LCM, polynomial remainder sequences — all operating on **dense coefficient lists** (`dup_*`/`dmp_*` API).

- `dup_half_gcdex`, `dup_gcdex`, `dmp_half_gcdex`, `dmp_gcdex` — extended Euclidean algorithms.
- `dup_invert(f, g, K)` / `dmp_invert` — modular inverse; **raises `NotInvertible("zero divisor")` if gcd(f,g) ≠ 1**.
- `dup_euclidean_prs(f, g, K)` — Euclidean polynomial remainder sequence in `K[x]`; **iteratively computes `dup_rem` until remainder is zero**, collecting all remainders into a list starting with `[f, g, …]`.
- `dmp_euclidean_prs` — multivariate wrapper; **raises `MultivariatePolynomialError` for `u > 0`** (univariate only).
- `dup_primitive_prs`, `dup_inner_subresultants` (and `dmp_` variants) — primitive and subresultant polynomial remainder sequences.
  - `dmp_primitive_prs` — **raises `MultivariatePolynomialError` for `u > 0`** (same pattern as `dmp_euclidean_prs`; delegates to `dup_primitive_prs` when univariate).
  - `dup_inner_subresultants` — computes subresultant PRS and scalar subresultants; **abnormal case (degree drop `d > 1`)**: updates scalar subdeterminant via `c = (-lc)^d / c^(d-1)` (quotient formula); **normal case (`d == 1`)**: simply `c = -lc`.
  - `dmp_inner_subresultants` — multivariate subresultant PRS; swaps inputs if `deg(f) < deg(g)`.
    - **Returns `([], [])` if both are zero; returns `([f], [K.one ground])` if only `g` is zero** (single-element PRS with unit cofactor).
    - Same abnormal-case formula as univariate: **degree drop `d > 1` → `c = (-lc)^d / c^(d-1)`; `d == 1` → `c = -lc`**.
- `dup_prs_resultant`, `dmp_prs_resultant` — resultant via subresultant PRS; **if the last PRS element has positive degree in the leading variable, returns zero (in n−1 variables) as the resultant** (non-trivial GCD implies zero resultant).
- `dup_resultant`, `dmp_resultant` — resultant via multiple methods; `dmp_resultant` **dispatches to Collins modular algorithm only for QQ (field) or ZZ (ring) when `USE_COLLINS_RESULTANT` config is set; for other field domains (e.g. algebraic extensions), always falls back to PRS subresultant**.
- `dmp_zz_modular_resultant(f, g, p, u, K)` — resultant mod prime via evaluation-interpolation; **raises `HomomorphismFailed` if evaluation points exhausted**.
- `dmp_zz_collins_resultant` / `dmp_qq_collins_resultant` — Collins's modular resultant in Z[X] / Q[X]; iterates over primes, **catches `HomomorphismFailed` from per-prime `dmp_zz_modular_resultant` and `continue`s to the next prime**; accumulates via CRT.
- `dup_discriminant`, `dmp_discriminant` — discriminant as `resultant(f, f') / (LC(f) * sign_factor)`; sign factor is `(-1)^(d(d-1)/2)` where `d` is degree; **returns zero (in n−1 variables) for degree ≤ 0**.
- GCD: `dup_rr_prs_gcd`/`dmp_rr_prs_gcd` (ring PRS), `dup_ff_prs_gcd`/`dmp_ff_prs_gcd` (field PRS).
  - **Ring PRS GCD** (`dup_rr_prs_gcd`): extracts primitive parts, computes content GCD, takes last element of subresultant PRS, strips content, then **negates the content multiplier if the leading coefficient is negative** to ensure canonical positive leading coefficient.
  - **Field PRS GCD** (`dup_ff_prs_gcd`): takes last element of subresultant PRS and **makes it monic** (no sign correction needed).
  - `dup_zz_heu_gcd`/`dmp_zz_heu_gcd` — heuristic over Z; same triple-fallback verification as `heugcd` in `heuristicgcd.py` but on dense coefficient lists.
  - `_dup_zz_gcd_interpolate` / `_dmp_zz_gcd_interpolate` — recover polynomial from a **single integer GCD value** using base-conversion-style symmetric remainder on dense coefficient lists (not `PolyElement` objects; contrast `_gcd_interpolate` in `heuristicgcd.py`).
    - **Negates result if leading ground coefficient is negative** to ensure positive leading coefficient.
- `dup_qq_heu_gcd`/`dmp_qq_heu_gcd` — heuristic GCD over Q; **clears denominators first, then delegates to the Z version**.
- `_dmp_simplify_gcd` — **eliminates outermost variable** from multivariate GCD when one input has degree 0 in it.
  - Asymmetric extraction: **uses leading coefficient for the degree-0 input but content (GCD of all coefficients in x₀) for the non-constant input**; then computes GCD in the remaining variables.
- `_dup_rr_trivial_gcd`, `_dmp_rr_trivial_gcd` (ring), `_dup_ff_trivial_gcd`, `_dmp_ff_trivial_gcd` (field) — short-circuit trivial GCD cases: both zero → zero; one zero → normalized non-zero; **`_dmp_rr_trivial_gcd` returns `(1, f, g)` immediately when either input is the constant 1** (unit polynomial).
- `dup_inner_gcd`, `dmp_inner_gcd`, `dup_gcd`, `dmp_gcd` — main GCD entry points; `dmp_inner_gcd` **deflates exponents via `dmp_multi_deflate` before computing, then inflates results back**; for inexact domains (e.g. floats), **converts to exact domain first; if no exact domain exists, returns `[K.one]` (trivial GCD) as fallback**.
- `dup_lcm`, `dmp_lcm` — LCM; `dmp_lcm` **dispatches to `dup_lcm` when `u==0`**; internally dispatches to `_rr_lcm` (ring: primitive-part based) vs `_ff_lcm` (field: normalizes to monic).
- `dmp_content`, `dmp_primitive` — multivariate content/primitive; content **negates if leading ground coeff is negative**.
- `dup_cancel`, `dmp_cancel` — cancel common factors; **when `K.has_Field` and `K.has_assoc_Ring`, converts to ring (clears denoms) before GCD, then converts back**.
  - Sign normalization: if both numerator and denominator have negative leading coefficients, negates both; **if only the denominator is negative, negates the content multiplier and the denominator** to enforce positive denominator convention.

### [`heuristicgcd.py`](heuristicgcd.py)
Heuristic polynomial GCD at the **Poly-object level** (not dense lists).

- `heugcd(f, g)` — heuristic GCD for `PolyElement` objects in `ZZ[x₁,…,xₙ]`; evaluates at points, computes integer GCD, and interpolates back.
  - **Triple-fallback verification**: after interpolating the candidate GCD, if trial division fails, tries interpolating the first cofactor and dividing, then the second cofactor and dividing — two alternative recovery paths before advancing to the next evaluation point.
- `_gcd_interpolate(h, x, ring)` — recover polynomial from integer GCD image via base-conversion-style interpolation with **symmetric modular representation**.
  - **Univariate**: integer modular arithmetic (`h % x`, adjusted to `[-x/2, x/2]`), stores coefficients by degree index.
  - **Multivariate**: uses `trunc_ground`/`quo_ground` on polynomial coefficients, lifting each layer with a prepended variable index.

Caveat: Distinct from `dup_zz_heu_gcd`/`dmp_zz_heu_gcd` in `euclidtools.py`, which implement the same triple-fallback algorithm but operate on raw dense coefficient lists (not `PolyElement` objects).

### [`modulargcd.py`](modulargcd.py)
Modular GCD algorithms using Chinese Remainder Theorem and Lagrange interpolation.

- `_primitive(f, p)` — compute content and primitive part of `f ∈ Z_p[x₀,…,x_{k-2}, y]` viewed as polynomial over `Z_p[y]`; groups terms by all variables except the last, iteratively GCDs the univariate coefficient sequences via `gf_gcd`, and returns `(content_in_y, quotient)`.
- `_deg(f)` — degree of `f ∈ K[x₀,…,x_{k-2}, y]` viewed as polynomial in `K[y][x₀,…,x_{k-2}]`; returns the **lexicographically largest** monomial prefix tuple (not total degree).
- `_LC(f)` — extract leading coefficient of `f ∈ K[x₀,…,x_{k-2}, y]` as a univariate polynomial in `K[y]`; collects all terms whose prefix exponents match `_deg(f)` and projects them into a single-variable ring in the last generator.
- `_trivial_gcd(f, g)` — handle zero-polynomial GCD cases; **negates the non-zero input if its leading coefficient is negative** to ensure the result has a positive leading coefficient; returns `(ring.zero, ring.zero, ring.zero)` if both are zero.
- `modgcd_univariate`, `modgcd_bivariate`, `modgcd_multivariate` — modular GCD in Z[x], Z[x,y], Z[X].
- `_primitive_in_x0(f)` — content and primitive part of `f ∈ Q(α)[x₀,…,xₙ₋₁]` viewed as univariate in x₀; iteratively GCDs coefficients via `func_field_modgcd`.
  - **Returns the original polynomial immediately (early exit) if running content becomes unit**.
- `func_field_modgcd` — modular GCD over algebraic function fields; **for multivariate inputs (n>1), extracts primitive parts w.r.t. leading variable via `_primitive_in_x0`** to prevent spurious content, then reattaches content GCDs after core computation.
- `_to_ZZ_poly(f, ring)` — **converts polynomial from Q(α)[x₀,…,xₙ₋₁] to Z[…][x₀, z]** by clearing denominators and replacing α with a formal indeterminate z.
  - **Branches on `isinstance(ring.domain, PolynomialRing)`**: if yes (has parameter vars), extracts inner domain for LCM and multiplies by `monom[1:]`; if no, uses `ring.domain` directly.
- `_func_field_modgcd_p(f, g, minpoly, p)` — per-prime subroutine for function-field modular GCD; **recursively reduces parametric variables** by evaluating at random points in Z_p and interpolating; base case (no parametric indeterminates, i.e. domain is not a `PolynomialRing`) **falls back to `_euclidean_algorithm`**; returns `None` when reconstruction fails.
- `_euclidean_algorithm(f, g, minpoly, p)` — monic GCD in Z_p[z]/(m(z))[x] via Euclidean algorithm; **returns `None` if a leading coefficient is not invertible mod m(z)** (detected via extended GCD when m(z) is not irreducible).
- `_degree_bound_bivariate(f, g)` — estimate upper degree bounds for bivariate GCD; reduces mod a prime, evaluates at points.
  - **Falls back to `min(deg(f), deg(g))` if no evaluation point avoids vanishing of the leading coefficient GCD**.
- `_to_ANP_poly(f, ring)` — convert from `Z[…][x₀, z]` back to `Q(α)[x₀,…,xₙ₋₁]`; reconstructs algebraic number field coefficients from the z-exponent in each monomial tuple.
- `_interpolate_multivariate(evalpoints, hpeval, ring, i, p, ground)` — Lagrange interpolation in Z_p; **when `ground=True`, the reconstructed variable comes from `ring.domain.gens[i]`** (coefficient ring) instead of `ring.gens[i]`.
- `_chinese_remainder_reconstruction_univariate` — CRT for univariate polynomials; combines two residue representations over coprime moduli into symmetric representation over their product.
- `_chinese_remainder_reconstruction_multivariate` — CRT for multivariate polynomials; **when coefficient domain is a `PolynomialRing` (nested structure), recurses on itself to CRT-combine the polynomial coefficients**; for plain integer coefficients, uses standard number-theoretic CRT.
- `_rational_function_reconstruction(c, p, m)` — recover rational function `a/b` in `Z_p(t)` from congruence residue `c mod m` via partial extended Euclidean algorithm with degree bounds.
  - **Returns `None` if denominator shares a common factor with modulus** (non-invertible).
- `_integer_rational_reconstruction(c, m, domain)` — reconstruct rational `a/b` from `c ≡ a/b mod m` via Euclidean algorithm; **returns `None` if denominator coefficient is zero (`s1 == 0`) or `|s1| ≥ bound`** (non-invertible); negates both `a, b` when `s1 < 0` to ensure positive denominator.
- `_rational_reconstruction_func_coeffs(hm, p, m, ring, k)` — reconstruct each coefficient as a rational function in parameter `t_k`.
  - **When `k == 0`, directly calls `_rational_function_reconstruction`; when `k > 0`, drops one variable layer and reconstructs each sub-coefficient**.
  - Returns `None` if any single coefficient reconstruction fails.
- `_rational_reconstruction_int_coeffs(hm, m, ring)` — reconstruct rational coefficients from integer image. Returns `None` if any coefficient fails.
  - **If `ring.domain` is a `PolynomialRing` (nested coefficients), recurses on itself; otherwise delegates to `_integer_rational_reconstruction`**.
- `_trial_division` — verify candidate GCD by **fraction-free pseudo-division in a quotient ring** `K[t₁,…,tₖ][z]/(m(z))`; two-level reduction: outer loop reduces by the candidate divisor in `x`, inner loop reduces modulo the minimal polynomial in `z`; optionally truncates coefficients mod a prime.

---

## Factorization

### [`factortools.py`](factortools.py)
Polynomial factorization in characteristic zero (over Z, Q, algebraic extensions, and GF).

- `dup_zz_zassenhaus`, `dup_zz_factor_sqf`, `dup_zz_factor` — Zassenhaus factorization over Z.
  - `dup_zz_zassenhaus` — factors primitive square-free polynomials in Z[x]; **prime selection heuristic**: iterates candidate primes, skipping those dividing LC; checks square-freeness mod each prime; collects up to 5 candidates and **selects the prime yielding the fewest irreducible factors**; early-exits if factor count < 15.
  - `_test_pl(fc, q, pl)` — helper for trial division phase; converts `q` to symmetric representation mod `pl`; **returns `True` unconditionally if `q` reduces to zero** (candidate passes); otherwise checks `fc % q == 0`.
  - `dup_zz_factor` extracts primitive part; **if primitive part has negative leading coefficient, negates both content and polynomial** before factoring.
- `dmp_zz_wang` — Wang's Enhanced Extended Zassenhaus multivariate factorization; **selects evaluation-point config with smallest univariate max-norm**; restarts with incremented modulus on `ExtraneousFactors` from Hensel lifting.
- `dmp_zz_wang_lead_coeffs` — correct leading coefficients during Wang/EEZ; distributes true LC divisors among trial factors, **raises `ExtraneousFactors` if any evaluated divisor is unassigned** (tracked via a J-array flag per evaluation value).
- `dmp_zz_factor` — top-level multivariate factorization over Z.
- `dup_zz_hensel_step`, `dup_zz_hensel_lift` — univariate Hensel lifting.
- `dmp_zz_wang_hensel_lifting` — **parallel Hensel lifting for multivariate factorization**; iteratively lifts univariate factor approximations to full multivariate factors; verifies final product matches original, raises `ExtraneousFactors` on mismatch.
- `dup_ext_factor`, `dmp_ext_factor` — factorization over algebraic extensions; computes square-free norm, factors norm over the base domain; **if norm yields a single irreducible factor, returns the square-free part with multiplicity `n // deg(sqf_part)`** (handles powers of irreducibles); otherwise recovers factors via GCD with shifted norm factors and trial division.
- `dup_gf_factor`, `dmp_gf_factor` — factorization in finite fields (wraps galoistools).
- `dup_factor_list`, `dmp_factor_list` — complete factorization with multiplicities; **if domain is not exact (e.g. RR), converts to exact domain, factors there, then converts results back**.
  - For exact fields, the cleared denominator is folded into the leading coefficient (`coeff/denom`); **for inexact fields, the denominator is instead divided out of each factor individually** via `dmp_quo_ground` before converting back to the inexact domain.
  - `dmp_factor_list` first extracts common variable powers via `dmp_terms_gcd`; **after core factorization, reinserts each extracted power as a separate monomial factor** (constructed as a single-term dict entry at the appropriate nesting level).
- `dup_zz_irreducible_p` — integer polynomial irreducibility test via **Eisenstein's criterion** (checks if a prime divides all non-leading coefficients but its square does not divide the constant term).
- `dup_irreducible_p`, `dmp_irreducible_p` — irreducibility testing (general).
- `dup_trial_division`, `dmp_trial_division` — determine factor multiplicities via repeated division; **includes factors with multiplicity 0** if candidate does not divide.
- `dmp_zz_wang_non_divisors(E, cs, ct, K)` — validate evaluation values for Wang's algorithm.
  - Iteratively extracts GCDs from evaluation results against accumulated divisors; **returns `None` if any value reduces to 1**.
- `dmp_zz_wang_test_points` — test evaluation points for suitability (leading coefficient non-vanishing, square-free, valid non-divisors).
- `dup_zz_diophantine`, `dmp_zz_diophantine` — Wang/EEZ Diophantine equation solvers; `dup_zz_diophantine` for >2 inputs **builds cumulative products and recursively reduces to the 2-input base case** (extended GCD).
  - `dmp_zz_diophantine` recursively peels evaluation points from the list, reducing dimension by one each step; **uses Taylor-like expansion with successive differentiation at evaluation points** to lift solutions back to full dimension.
- `dup_zz_mignotte_bound`, `dmp_zz_mignotte_bound` — coefficient bounds for factors.
- `dup_cyclotomic_p` — cyclotomic polynomial predicate.
- `dup_zz_cyclotomic_factor` — efficient factorization of `x^n ± 1` in Z[x]; **validates input form: leading coeff must be 1, trailing coeff must be ±1, all interior coefficients must be zero**; returns `None` if input doesn't match either binomial form.
  - **`x^n - 1`**: returns cyclotomic decomposition of `n` directly; **`x^n + 1`**: computes decomposition of `2n` and **filters out factors present in the decomposition of `n`**, yielding only the cyclotomic polynomials unique to `2n`.

### [`sqfreetools.py`](sqfreetools.py)
Square-free decomposition for **characteristic-zero domains** (Z, Q, algebraic extensions).

- `dup_sqf_p`, `dmp_sqf_p` — square-free predicate (checks gcd(f, f') == 1); **returns `True` for the zero polynomial** (special-cased before GCD computation).
- `dup_sqf_norm`, `dmp_sqf_norm` — square-free norm over algebraic extensions; iteratively shifts input by the algebraic generator until the resultant is square-free.
  - Returns `(shift_count, shifted_poly, resultant_in_ground_domain)`.
- `dup_sqf_part`, `dmp_sqf_part` — square-free part; **over fields, normalizes result to monic; over rings, extracts primitive part** (content-free form).
- `dup_sqf_list`, `dmp_sqf_list` — square-free decomposition with multiplicities; **over fields, makes `f` monic; over rings (e.g. ZZ), extracts primitive part and negates `f` (adjusting content sign) if leading coefficient is negative** after content extraction.
- `dup_sqf_list_include`, `dmp_sqf_list_include` — same as `sqf_list` but folds the leading coefficient into the factor list; **if no factor has multiplicity 1, prepends a ground constant `(coeff, 1)` entry**.
- `dup_gf_sqf_part`, `dmp_gf_sqf_part`, `dup_gf_sqf_list`, `dmp_gf_sqf_list` — GF variants (thin wrappers around `galoistools`).
- `dup_gff_list` — greatest factorial factorization (univariate only).
- `dmp_gff_list` — **raises `MultivariatePolynomialError` for polynomials with more than one variable**; delegates to `dup_gff_list` when univariate.

Caveat: For native GF(p) polynomial square-free and factorization, see `galoistools.py`.

---

## Galois Field Operations

### [`galoistools.py`](galoistools.py)
Self-contained arithmetic, square-free, irreducibility, and factorization for **univariate polynomials over GF(p)**, represented as coefficient lists.

- `gf_int(a, p)` — coerce `a mod p` to symmetric range `[-p/2, p/2]`; values above `p//2` become negative.
- `gf_strip` — strip leading zeros; **short-circuits via `if not f or f[0]`** (returns immediately when list is empty or first element is nonzero, same optimization as `dup_strip`). `gf_trunc` — reduce coefficients mod p.
- Arithmetic: `gf_add`, `gf_sub`, `gf_mul`, `gf_sqr`, `gf_div`, `gf_rem`, `gf_quo`, `gf_exquo`, `gf_pow`, `gf_pow_mod`.
  - `gf_add`/`gf_sub` — **only strips leading zeros (via `gf_strip`) when both operands share the same degree** (possible cancellation); skips stripping when degrees differ.
  - `gf_sub` degree mismatch: **when the subtrahend has higher degree, negates its excess leading coefficients via `gf_neg`** before combining with element-wise subtraction on the aligned tail (asymmetric with `gf_add` which simply copies excess coefficients).
  - `gf_div` — **when dividend degree < divisor degree, returns `([], f)` immediately** (empty quotient, dividend as remainder) without entering the division loop.
  - `gf_exquo` — exact quotient; **raises `ExactQuotientFailed` if the divisor does not evenly divide the dividend** (nonzero remainder).
- Ground ops: `gf_add_ground(f, a, p, K)` — add scalar to GF(p) poly; **if f is zero poly and `a % p == 0`, returns `[]`**. Also `gf_sub_ground`, `gf_mul_ground`, `gf_quo_ground`, `gf_neg`.
- `gf_monic`, `gf_diff`, `gf_eval`, `gf_multi_eval`, `gf_gcd`, `gf_lcm`, `gf_cofactors`, `gf_gcdex` — standard operations and GCD/LCM.
  - `gf_lcm` — **returns empty list `[]` if either input is the zero polynomial**; otherwise computes via `f*g / gcd(f,g)` and makes result monic.
- `gf_sqf_p`, `gf_sqf_part`, `gf_sqf_list` — square-free testing and decomposition in GF(p).
- `gf_irreducible_p` — irreducibility dispatch; queries `GF_IRRED_METHOD` config and **defaults to `gf_irred_p_rabin` when no preference is set**. `gf_irred_p_ben_or`, `gf_irred_p_rabin` — the two concrete algorithms.
- `gf_ddf_zassenhaus` — distinct degree factorization (DDF); **appends non-trivial remainder as a factor of its own degree**.
- `gf_edf_zassenhaus` — probabilistic equal degree factorization (EDF).
- `gf_ddf_shoup` — improved distinct-degree factorization using **baby-step/giant-step** Frobenius powers; precomputes small steps `x^(p^i)` for `i ≤ k=ceil(sqrt(n/2))` and giant steps `x^(p^(k*j))`, then combines via products and GCDs. Faster than `gf_ddf_zassenhaus` for large degree or modulus.
- `gf_edf_shoup` — improved equal-degree factorization (Shoup variant).
- `gf_Qmatrix` — compute Berlekamp's Q matrix (rows are `x^(ip) mod f` for each i).
- `gf_Qbasis` — find kernel (null space) of `Q - I` via Gaussian elimination over GF(p); returns basis vectors for Berlekamp factorization.
- `gf_berlekamp`, `gf_zassenhaus`, `gf_shoup` — square-free factorization algorithms (small/medium/large `p`).
- `gf_factor_sqf(f, p, K, method)` — dispatch for square-free factorization; uses `method` arg or `query('GF_FACTOR_METHOD')` config; **if both are None, defaults to `gf_zassenhaus`**.
- `gf_factor(f, p, K)` — **complete factorization of possibly non-square-free polynomial**; square-free decomposition first, then factors each component.
- `gf_frobenius_monomial_base`, `gf_frobenius_map` — Frobenius automorphism.
- `gf_compose`, `gf_compose_mod` — polynomial composition and modular composition.
- `gf_trace_map` / `_gf_trace_map` — trace map computation using **binary doubling**; initializes differently for even vs odd `n`.
- `gf_expand(F, p, K)` — reconstruct polynomial from factored form; **accepts `(lc, factors)` tuple or plain list of `(factor, mult)` pairs**.
- `gf_random`, `gf_irreducible`, `gf_value` — random/irreducible polynomial generation and evaluation.
- `gf_crt`, `gf_crt1`, `gf_crt2` — Chinese Remainder Theorem. Also `linear_congruence`, `csolve_prime`, `gf_csolve`.
- Conversion: `gf_from_dict`, `gf_to_dict`, `gf_from_int_poly`, `gf_to_int_poly`.
  - `gf_to_int_poly(f, p, symmetric)` — convert GF(p) coefficient list to integer list; **when `symmetric=True` (default), centers each coefficient via `gf_int` into `[-p/2, p/2]`; when `False`, returns the non-negative residues unchanged**.
  - `gf_to_dict` — same `symmetric` flag behavior as `gf_to_int_poly` but returns a sparse `{degree: coeff}` dict.
  - `gf_from_dict` — **accepts both plain integer keys and single-element tuple keys** (e.g. `{10: c}` or `{(10,): c}`), dispatching by `isinstance(max_key, int)`.

Caveat: All operations here are list-based GF(p)-specific. For dense polynomial operations over general domains, see `densearith.py`/`densetools.py`. For square-free decomposition over Z/Q, see `sqfreetools.py`.

---

## Roots & Isolation

### [`rootoftools.py`](rootoftools.py)
Symbolic root representations and root-sum evaluation.

- `CRootOf` (alias `ComplexRootOf`) — indexed algebraic root of an irreducible polynomial.
  - `free_symbols` — **always returns empty set**, even when internal `poly` attribute is a `Poly` (not `PurePoly`), since `CRootOf` only represents univariate roots.
  - `__new__(f, x, index)` — constructor; **negative index is normalized by adding the polynomial degree**; raises `IndexError` if out of range.
  - `_new(poly, index)` — raw classmethod constructor; converts poly to `PurePoly` and **attempts to copy real/complex root isolation caches from original poly key to PurePoly key; silently catches `KeyError` if caches are missing**, returning the object without cached intervals.
    - **If coefficient domain is not exact (e.g. RR), converts to exact domain before proceeding**; after preprocessing, **raises `NotImplementedError` if domain is not ZZ** (sorted roots only supported over integers).
    - When second positional arg is an integer and no explicit `index` kwarg, **reinterprets it as root index** (not generator).
  - `_real_roots`, `_all_roots`, `_roots_radical` — root enumeration.
  - `_roots_trivial(poly, radicals)` — closed-form roots for linear/quadratic/binomial; **if `radicals=False`, returns `None` for all degree > 1** (only linear is always solved).
  - `_reals_index`, `_complexes_index` — map global root index to per-factor local index; `_complexes_index` **offsets the local index by the number of real roots** of the same factor (via `_reals_cache`).
  - `_get_interval`, `_refine_interval`, `_eval_evalf` — numerical evaluation; `_eval_evalf` **creates a Dummy variable and substitutes when the polynomial generator is a compound expression** (not a plain Symbol).
    - **For complex roots, pre-refines the isolation interval until both imaginary bounds have changed** from their initial values (ensures disjointness with neighboring roots before numerical root-finding).
    - When the complex isolation interval converges to a single point (`ax==bx` and `ay==by`), **assigns the sign of the imaginary part using the polynomial degree and root index parity** (roots sorted with negative-imaginary before positive-imaginary), rather than trusting the interval's sign directly.
  - `_eval_Eq(other)` — symbolic equality check; **returns `S.false` if `other` has no imaginary part but the root is non-real (complex), or vice versa**; refines bounding interval and checks containment for compatible real/imaginary types.
  - `_separate_imaginary_from_complex` — classify non-real roots into imaginary vs complex.
    - For two-term polynomials of power-of-2 degree with opposite-sign LC·TC, marks 2 roots as imaginary (mixed case).
    - **Refines bounding rectangles until non-imaginary roots have boxes fully to one side of the y-axis**.
  - `_complexes_sorted(complexes)` — make complex isolating intervals disjoint and sort roots; **separates purely imaginary roots first** (whose x-intervals can never become disjoint from each other through refinement), sorts them by `±root(|TC/LC|, degree)` keyed by imaginary-part sign, then merges with the refined/sorted non-imaginary complex roots at the correct insertion point.
  - `_reals_sorted(reals)` — makes real root isolating intervals from different irreducible factors disjoint by pairwise refinement, then sorts by left endpoint; updates `_reals_cache` with the refined intervals.
  - `real_roots(poly)`, `all_roots(poly)` — class methods for root lists.
- `bisect(f, a, b, tol)` — standalone interval-halving root-finder used by `CRootOf.eval_rational()`; **returns `c` immediately if `f(c) == 0` at the midpoint** (exact root found); raises `ValueError` if `f(a)` and `f(b)` have the same sign.
- `RootSum` — represents ∑ f(rᵢ) over all roots rᵢ of a polynomial.
  - `__new__(expr, func, x)` — constructor; separates additive and multiplicative constants from the mapping that are independent of the summation variable before dispatching.
    - **If mapping expression has no dependence on the variable, returns `degree * expression`** without constructing a RootSum.
    - Factors the polynomial; dispatches linear factors to direct evaluation, optional quadratic to `roots_quadratic`, and higher-degree to `_rational_case` (if rational) or raw `_new`.
  - `_rational_case(poly, func)` — **evaluates sum of a rational function over all roots using Viète's formulas and symmetric function decomposition**.
    Avoids computing roots explicitly by introducing formal root symbols, symmetrizing, then substituting Viète relations.
  - `_is_func_rational` — checks if the lambda is a rational function.
  - `doit` — attempts to evaluate the root sum; **if `roots()` finds fewer roots than the polynomial degree, returns `self` unevaluated** instead of summing partial results.
  - `_eval_evalf(prec)` — numerical evaluation; computes `nroots` and sums `fun(r)` for each; **on `DomainError` or `PolynomialError`, returns `self` unevaluated** (silent fallback, no error raised).
- `rootof(poly, index)` — factory function creating `CRootOf` instances.

### [`polyroots.py`](polyroots.py)
Symbolic root-finding algorithms (closed-form solutions).

- `roots(f, filter, predicate)` — compute symbolic roots using radical formulas (linear through quartic), plus special cases; **also accepts a plain list of numerical coefficients** (builds a dummy variable internally).
  - `filter` parameter restricts root domain: `'Z'` (integer), `'Q'` (rational), `'R'` (real), `'I'` (imaginary), `'C'` (no-op); **raises `ValueError("Invalid filter: ...")` for unrecognized strings** (catches `KeyError` from handler lookup).
  - **Two-term (binomial) optimization**: for polynomials with exactly 2 terms and degree > 1, extracts nth-power factors from the constant term, substitutes a dummy base, solves the simpler form, then back-substitutes.
  - **Expression domain (EX) branch**: when polynomial has a single irreducible factor with irrational/symbolic coefficients, attempts `to_rational_coeffs` to transform via rescaling (`x→αx`) or translation (`x→x+β`) to rational coefficients, recursively solves the transformed polynomial, then adjusts solutions back.
  - Internal `_try_heuristics`: **tests -1 then 1 as roots and divides out only one trivial linear factor** (breaks after first success) before dispatching to degree-specific solvers (linear, quadratic, cubic, quartic, quintic, cyclotomic).
- `roots_cubic`, `roots_quartic`, `roots_binomial`, `roots_cyclotomic` — specialized solvers.
  - `roots_quartic` handles a **quasisymmetric case** when `(C/A)^2 == D`: factors the quartic into two quadratics via an intermediate quadratic `g`, then solves each factor with `roots_quadratic`.
- `roots_quintic` — solvable quintic solver using Lagrange resolvents; determines solvability before attempting radical computation.
  - **Returns empty list early if**: (1) x⁴ term is present, (2) leading coefficient ≠ 1 and dividing through produces any irrational coefficient, (3) polynomial is reducible, or (4) the associated degree-20 resolvent `f20` (from `PolyQuintic`) is irreducible over Z (no linear factor).
  - Swaps resolvent parameters when numerical check against discriminant fails.
- `root_factors(f)` — decompose univariate polynomial into linear factors from discovered roots; **if fewer roots are found than the degree, appends the quotient remainder as a non-linear factor**.
- `preprocess_roots(poly)` — simplify symbolic coefficients before root-finding; injects generators and checks for consistent exponent ratios.
  - **When one exponent in a base/generator pair is zero but the other is not, breaks** (no consistent ratio), preventing elimination of that generator.
- `_integer_basis(poly)` — find integer scaling factor `div` such that substitution `x = div*y` minimizes coefficient magnitudes.
  - **Returns `None` immediately if `|LC| >= |constant term|`** (only attempts reduction when the constant term dominates).
  - **Reverses the coefficient list when the leading coefficient is 1** before searching for the scaling constant.

### [`rootisolation.py`](rootisolation.py)
Numerical root isolation and refinement for dense univariate polynomials. Also defines `RealInterval` and `ComplexInterval` classes for bounding root locations.

- `RealInterval` — bounding interval for a real root; stores Möbius transform for refinement.
- `ComplexInterval` — bounding rectangle for a complex root; stores southwest/northeast corners. When `conj=True` (root in lower half-plane), **y-coordinates are reflected**: `ay` returns `-b[1]` and `by` returns `-a[1]`.
- `dup_isolate_real_roots`, `dup_isolate_real_roots_sqf` — real root isolation via continued fractions / bisection.
- `dup_isolate_complex_roots_sqf` — complex root isolation.
- `dup_isolate_all_roots(f, K)` — isolate both real and complex roots of a non-square-free polynomial; performs square-free factorization.
  - **Raises `NotImplementedError` if more than one distinct irreducible factor exists** (only handles the single-factor case).
- `dup_isolate_all_roots_sqf` — isolate real and complex roots of a square-free polynomial (delegates to the individual real/complex isolators).
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
- `sturm_pg(p, q, x, method)`, `sturm_amv` — Sturm sequences via alternative methods (Pell-Gordon / AMV theorems).
  - `sturm_pg` **negates both inputs when LC(p) < 0** and flips the output sequence.
  - `method=0` scales remainders by `LC(p)^(deg_diff)` for modified subresultant coefficients; `method=1` produces plain (unscaled) coefficients.
- `euclid_pg`, `euclid_q`, `euclid_amv` — Euclidean PRS via sign-flipping of Sturm sequences.
  - `euclid_amv` — Euclidean PRS using **Collins-Brown-Traub coefficient reduction**: initializes reduction variable `c = -1`, iteratively updates via `c = (-LC)^(δ-1) / c^(δ-2)`, and normalizes each remainder by dividing by `|c^(δ-1) · σ|`; produces subresultant coefficients without determinant evaluation.
  - `euclid_q` — Euclidean sequence in Q[x]; **normalizes LC(p) to positive before computing remainders (negating both inputs); after completion, negates entire output sequence if original LC was negative**; removes trailing zero/NaN entry.
- `subresultants_pg`, `subresultants_amv`, `subresultants_rem`, `subresultants_vv`, `subresultants_vv_2` — subresultant PRS (multiple methods); `subresultants_pg` **converts modified subresultant PRS to standard PRS** by dividing each remainder (after the first two) by `LC(p)^(deg(p)-deg(q))` and applying sign corrections `(-1)^(j(j-1)/2)` based on degree gaps; `subresultants_rem` swaps inputs if deg(p) < deg(q); `subresultants_vv` uses **Van Vleck's triangularization of Sylvester's 1853 matrix**, explicitly maintaining and optionally printing the triangularized matrix (`method=1`); `subresultants_vv_2` is the **implicit-matrix variant** (Sylvester matrix not stored explicitly) for large-dimension cases; **returns `[f, g]` early if `deg(f) > 0` and `deg(g) == 0`** (constant second input).
- `modified_subresultants_pg`, `modified_subresultants_amv`, `modified_subresultants_bezout` — modified subresultant PRS; `modified_subresultants_pg` uses Pell-Gordon 1917 theorem with degree-gap-aware denominator calculation.
- `sylvester(p, q, x, method)` — Sylvester matrix construction (1840 variant `(m+n)×(m+n)` or 1853 variant `(2·max(m,n))×(2·max(m,n))`).
  - **Returns empty `Matrix([])` when both polys are zero, both are constants, or one is constant and the other is zero**.
  - **Returns `Matrix([0])` when one poly has degree ≥ 1 and the other is zero** (not an empty matrix).
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
- `_construct_simple` — handle simple numeric domains (ZZ, QQ, RR, algebraic); **returns `False` (signal to use EX domain) if coefficients mix floats and algebraic numbers**; returns `None` (signal to try composite) for non-numeric coefficients.
- `_construct_algebraic` — handle algebraic coefficients: decomposes each into (irrational_part, multiplicative_factor, additive_constant).
  - Collects distinct irrational parts, **computes a single primitive element to unify all extensions into one algebraic field**.
  - Reconstructs each coefficient in the unified field using the primitive element representation.
- `_construct_composite` — handle composite domains (ZZ[X], QQ[X], ZZ(X), QQ(X)); **returns `None` (fallback to EX) if any generator is number-like or if two generators share free symbols** (potential algebraic relations).
  - **Ring-vs-field decision**: if `opt.field` is set, always uses a fraction field; otherwise inspects all denominators.
  - If every denominator is a single-term constant, uses `ground.poly_ring(*gens)` (ring); if any is non-trivial (multi-term or non-constant), uses `ground.frac_field(*gens)`.
  - **Ground domain selection**: scans all coefficients — any `Float` → RR; any non-integer `Rational` → QQ; otherwise ZZ.
- `_construct_expression` — fallback to the expression domain EX.

### [`compatibility.py`](compatibility.py)
Bridge between sparse polynomial ring interface and dense function API.

- `IPolys` — mixin class providing dense polynomial operations as methods on ring objects.
  - `wrap(element)` — coerce a `PolyElement` into this ring; **raises `NotImplementedError("domain conversions")` if the element belongs to a different ring**; for non-`PolyElement` inputs, delegates to `ground_new`.
  - `ground_new`, `domain_new`, `from_dict`, `clone`, `drop` — ring interface methods.
  - Multivariate term operations (`dmp_add_term`, `dmp_sub_term`, `dmp_mul_term`): **coefficient is preprocessed via `wrap(c).drop(0).to_dense()`**.
    - Wraps into the ring, drops the leading generator, converts to dense (treats coefficient as polynomial in remaining variables).
    - Univariate variants (`dup_add_term`, etc.) pass the coefficient directly without preprocessing.
  - Multivariate result methods (`dmp_LC`, `dmp_TC`, `dmp_eval_tail`, `dmp_resultant`, `dmp_discriminant`, `dmp_content`, `dmp_primitive`): **check `isinstance(result, list)` to decide output form**.
    - If list → reconstruct via `self[1:].from_dense()` (ring with one fewer generator); if scalar → return raw value.
  - Wang factorization bridge methods (`dmp_zz_wang_hensel_lifting`, `dmp_zz_wang_lead_coeffs`, etc.): **slice the ring into univariate `self[:1]` and multivariate `self[1:]` sub-rings** to convert lifted factors (univariate) and leading coefficient expressions (multivariate) separately before delegating to the dense implementation.
  - `dup_sqf_norm`, `dmp_sqf_norm` — bridge methods; the resultant (third return value) is converted via `self.to_ground().from_dense()` (ground domain ring), not `self.from_dense()`.
  - `to_gf_dense(element)` — convert sparse element to dense coefficient list for GF(p) arithmetic; **converts each coefficient through `domain.dom`** (the base integer domain of the finite field).
  - `from_gf_dense(element)` — convert dense GF(p) list back to sparse representation via `dmp_to_dict`.
  - `dup_clear_denoms(f, convert)` / `dmp_clear_denoms(f, convert)` — clear fractional coefficients; **when `convert=True`, clones the ring with `domain.get_ring()` (e.g. QQ→ZZ) and reconstructs the result in that new ring**; otherwise reconstructs in the original ring.
  - `gf_*` wrapper methods (e.g. `gf_trunc`, `gf_normal`, `gf_neg`, `gf_add`, …) — convert sparse ↔ dense via `to_gf_dense`/`from_gf_dense` and pass through domain modulus/base to the corresponding `galoistools` functions.
- Re-exports all `dup_*`/`dmp_*`/`gf_*` functions from dense modules.

### [`polyoptions.py`](polyoptions.py)
Option processing and validation for `Poly` constructors and functions.

- `Options` — option container (dict subclass); manages `domain`, `field`, `gaussian`, `extension`, `modulus`, `order`, etc.
  - `__init__` — preprocesses explicit args first, then **prunes defaults that conflict with already-set options via `cls.excludes` lists** before applying defaults; enforces mutual-exclusion and dependency constraints after all options are set.
- `Domain.preprocess(domain)` — parses domain specification from string, `Domain` instance, or object with `to_domain()`.
  - **Regex-based string parsing**: `'Z'`/`'ZZ'` → ZZ, `'Q'`/`'QQ'` → QQ, `'EX'` → EX, `'RR_<prec>'`/`'CC_<prec>'` → `RealField(prec)`/`ComplexField(prec)` with extracted precision, `'GF(<p>)'` → FF(p), `'ZZ[x,y]'`/`'QQ(x,y)'` → polynomial ring/fraction field, `'QQ<a,b>'` → algebraic extension.
- `Domain.postprocess` — **raises `GeneratorsError` if EX domain is requested without providing generators**, or if composite domain symbols overlap with polynomial generators.
- `Symbols.default()` — returns a **lazy generator of numbered placeholder names** `s1, s2, s3, …` (via `numbered_symbols('s', start=1)`); used as default symbol names for algebraic decomposition when no explicit names are provided.
- `Gen.preprocess(arg)` — validates generator index; accepts only `Basic` or `int`; **raises `OptionError` for other types** (e.g. strings).
- `Auto` — boolean flag; defaults to `True`; **`postprocess` automatically sets `auto=False` when `domain` or `field` is explicitly provided** (disables automatic domain inference).
- `Frac` — boolean flag for fraction field mode; defaults to `False`.
- `Extension.preprocess(extension)` — validates extension parameter; `1` → `True`, `0` → raises `OptionError`; **empty iterable (e.g. `[]`) → `None` (silently disables extension)** rather than raising an error; non-empty iterable → set of extensions.
- `build_options(gens, args)` — if `args` has exactly one key `'opt'` and no generators, **returns the existing `Options` object directly** (reuse); otherwise constructs a new `Options`.

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
- `_sort_gens` — sort generators by configurable priority (algebraic ordering, then user-defined `_gens_order`, then fallback).
- `_unify_gens(f_gens, g_gens)` — merge two ordered generator sequences into one, preserving relative ordering from both; common elements anchor the merge, non-common elements are interleaved around them.
- `_analyze_gens` — normalize `*gens` / `[gens]` calling conventions to a flat tuple.
- `_sort_factors` — sort polynomial factors.
- `_dict_reorder(rep, gens, new_gens)` — reorder monomial exponent tuples to match a new generator ordering; appends zero for new generators not in the original set.
  - **Raises `GeneratorsError` if an original generator with non-zero exponent is absent from the new ordering**.
- `_nsort(roots, separated)` — numerical sorting of roots; **raises `NotImplementedError` if any evaluated real/imaginary part has `_prec == 1`** (insufficient precision).
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
  - `__mul__`, `__div__`, `__floordiv__`, `__truediv__` — monomial multiplication/division via `monomial_mul`/`monomial_div`; accept `Monomial`, `tuple`, or `Tuple`; **for unrecognized types, `return NotImplementedError` (returns the class, does not raise)** — known bug.
  - `__pow__(n)` — exponentiation via repeated `monomial_mul`; **raises `ValueError` for negative `n`**.
- `MonomialOps` — code-generation engine for fast exponent-vector arithmetic; generates specialized Python functions (via `exec`) for `mul`, `div`, `ldiv`, `pow`, `mulpow`, `lcm`, `gcd` on fixed-arity monomial tuples.
  - Generated `div` — component-wise subtraction with **early `return None` if any result component is negative** (non-divisibility); contrast `ldiv` which allows negative exponents.

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
- `PolynomialDivisionFailed` — raised when polynomial division cannot reduce degree; `__str__` **branches on domain type**: EX domain → suggests different simplification; inexact domain → suggests adjusting precision; **exact domain → warns of possible SymPy bug** or user domain with improper zero detection.
- `ExactQuotientFailed` — raised when exact division has nonzero remainder.
- `UnificationFailed` — raised when domains/polynomials cannot be unified.
- `NotInvertible` — raised when polynomial modular inverse does not exist.
- `GeneratorsNeeded`, `GeneratorsError` — generator-related errors.
- `PolificationFailed` — raised when expression-to-Poly conversion fails; `__init__` **wraps single expressions in a list** so `.exprs` and `.origs` are always iterables (uniform interface for single and multi-expression cases).
- `ComputationFailed`, `RefinementFailed` — computation errors.

---

## Specialized Modules

### [`numberfields.py`](numberfields.py)
Computational algebraic number theory: minimal polynomials, field isomorphisms, primitive elements.

- `minimal_polynomial(expr, x)` — compute minimal polynomial of an algebraic expression; **auto-switches from compositional (resultant) to Gröbner strategy when any `AlgebraicNumber` subexpression is detected** in the expression tree.
- `_minpoly_pow(ex, pw, x, dom)` — minimal polynomial of `ex**pw`; for negative exponents, **inverts the polynomial and raises `ZeroDivisionError` if `mp == x`** (element is zero); short-circuits for `pw == -1`.
- `_minpoly_groebner(ex, x, dom)` — Gröbner-basis strategy for minimal polynomial.
  - Includes `simpler_inverse` heuristic: **inverts the expression first when it is a product of powers or a negative-exponent power with Add base**, then transforms back via `_invertx`.
- `_minpoly_op_algebraic_element(op, ex1, ex2, x, dom)` — minimal polynomial for sum or product of two algebraic elements via resultant; **when `dom` is QQ and `op` is Add, uses fast `rs_compose_add` instead of general resultant**.
  - **When one input minimal polynomial is linear (degree 1), skips expensive factorization** and returns the resultant directly (already irreducible).
- `_minimal_polynomial_sq(p, n, x)` — minimal polynomial for `p^(1/n)` where `p` is a sum of surds; eliminates square roots via repeated `_separate_sq`.
  - **When `n==1`, skips factorization and directly normalizes** (sign correction + primitive part), since elimination already yields a constant multiple of the minimal polynomial.
- `_minpoly_exp(ex, x)` — minimal polynomial of `exp(ex)`; for `e^(i·p·π/q)`, **uses hardcoded results for small primes q; general case generates cyclotomic polynomials for divisors of 2q and picks the correct factor**.
- `_minpoly_compose` — compositional minimal polynomial; dispatches by expression type (Add, Mul, Pow, sin, cos, exp, CRootOf).
  - **Mul over QQ**: separates rational-base/rational-exponent factors from others; computes LCM of exponent denominators across rational factors.
  - Constructs a separate annihilating polynomial for the rational product, then combines with the non-rational part via `_minpoly_op_algebraic_element`.
- `_minpoly_add`, `_minpoly_mul`, `_minpoly_sin`, `_minpoly_cos` — compositional minimal polynomial helpers for arithmetic and trigonometric subexpressions.
- `primitive_element(*extensions)` — compute primitive element of algebraic extension; **in explicit mode (`ex=True`), validates each extension: raises `ValueError` if a `Poly` extension is multivariate** (only univariate minimal polynomials accepted).
- `field_isomorphism(a, b)` — find isomorphism between algebraic number fields; **returns `None` early if `deg(b.minpoly) % deg(a.minpoly) != 0`** (degree-divisibility check); otherwise tries PSLQ (fast path, default) then factorization.
- `to_number_field(extension, theta)` — express algebraic extensions in a generated field; if `theta` is given, uses `field_isomorphism` to map into theta's field, **raises `IsomorphismFailed` if the extension is not in a subfield of theta**.
- `isolate(expr)` — give a rational isolating interval for an algebraic number (accepts symbolic expressions); **if input is rational, returns degenerate interval `(alg, alg)` immediately** without computing minimal polynomial; otherwise computes minimal polynomial, gets candidate intervals, and **doubles mpmath precision repeatedly until interval-arithmetic evaluation fits within one candidate**.
- `_choose_factor(factors, x, v, prec, bound)` — select factor of a polynomial that has a specific root via increasing-precision numerical evaluation; **accepts factor-multiplicity tuple pairs (e.g. from `factor_list`), stripping to plain polynomials first**; **raises `NotImplementedError` if the precision loop exhausts `prec` without narrowing to a single candidate**.

### [`partfrac.py`](partfrac.py)
Partial fraction decomposition.

- `apart(f, x, full)` — partial fraction decomposition of rational function.
  - Non-commutative fallback: if polynomial conversion fails, handles Mul (splits commutative/NC parts), Add (decomposes commutative terms).
  - For other non-commutative forms, **walks the expression tree in preorder**, decomposes each sub-expression, and replaces successes in-place.
  - `full=False` (default): uses undetermined coefficients method; `full=True`: uses Bronstein's algorithm.
- `apart_undetermined_coeffs(P, Q)` — partial fractions via undetermined coefficients; factors denominator, assigns symbolic unknowns per factor power, builds a linear system by matching polynomial powers, and solves for unknowns.
- `apart_full_decomposition(P, Q)` — Bronstein's full partial fraction decomposition.
- `apart_list(f, x)` — structured partial fraction as `(common, poly_part, fraction_list)` tuple; **if no custom dummy generator is supplied, creates a default generator that yields the same `Dummy` symbol indefinitely** (used as placeholder variables in the decomposition); **if input is atomic (plain number or symbol), returns the expression directly** (not a tuple).
- `assemble_partfrac_list` — reassemble from structured representation; **if roots are given as a `Poly`, constructs a `RootSum`; if roots are an explicit list of algebraic numbers, directly evaluates numerator/denominator at each root**.

### [`orthopolys.py`](orthopolys.py)
Classical orthogonal polynomial generation.

- `jacobi_poly`, `gegenbauer_poly`, `chebyshevt_poly`, `chebyshevu_poly`, `hermite_poly`, `legendre_poly`, `laguerre_poly` — generate orthogonal polynomials of given degree.
  - When `x` is provided, returns a `Poly` in that variable; **when `x` is omitted, returns a `PurePoly` with a `Dummy('x')` symbol** (generator-name-independent).
  - When `polys=True`, returns the polynomial object; otherwise (default) converts to a symbolic expression via `as_expr()`.
  - `laguerre_poly(n, x, alpha)` — generalized Laguerre polynomials; **when `alpha` is provided, infers coefficient domain via `construct_domain(alpha, field=True)`; when omitted, defaults to `QQ` with `alpha=0`**. Other families (Hermite, Legendre, etc.) use fixed domains (ZZ or QQ).

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
- `cyclotomic_poly`, `symmetric_poly`, `random_poly`.
- `interpolating_poly(n, x, X, Y)` — construct Lagrange interpolating polynomial; builds basis functions as ratios of products of differences `∏(x−Xⱼ)/∏(Xᵢ−Xⱼ)`.
  - Returns a symbolic `Add` of weighted basis terms. Distinct from `polyfuncs.interpolate` which is a higher-level wrapper.
- `fateman_poly_F_1/F_2/F_3`, `dmp_fateman_poly_F_1/F_2/F_3` — Fateman GCD benchmarks (symbolic and dense multivariate).
  - F_1 = trivial GCD (shared factor is 1), F_2 = linearly dense quartic inputs (shared factor is squared sum of variables), F_3 = sparse (degree ~ vars).
  - **In `dmp_fateman_poly_F_3`, the GCD's constant term is added at nesting level `n-1`** (not `n` as in F_1), reflecting the sparse structure.

### [`groebnertools.py`](groebnertools.py)
Gröbner basis computation algorithms.

- `groebner(seq, ring)` — compute Gröbner basis using Buchberger or F5B algorithm; **if domain is not a field, clones ring with `domain.get_field()`, computes in the field, then clears denominators and resets ring** on each result.
- `spoly(p1, p2, ring)` — compute S-polynomial (cancellation polynomial): scales each input by LCM(LM(p1),LM(p2))/LM and subtracts; core primitive for critical pair reduction in Buchberger's algorithm.
- `red_groebner(G, ring)` — compute reduced Gröbner basis; selects a generating subset, then reduces each polynomial by taking its remainder w.r.t. all others — **silently drops any polynomial that reduces to zero**.
- `groebner_lcm(f, g)` — LCM via ideal intersection: introduces variable `t`, computes basis of `(t*f, (1-t)*g)` in lex order, filters out elements free of `t`.
  - **When both inputs are single-term (monomial) polynomials**, bypasses Gröbner computation and directly returns componentwise monomial/coefficient LCM.
- `groebner_gcd(f, g)` — GCD via `f*g / lcm(f, g)`.
- `is_groebner`, `is_reduced` — basis validation.
- `sig_cmp(u, v, order)` — compare two signatures `(monomial, index)` by extending the term order to K[X]^n: **u < v iff v's module index is greater, or indices are equal and u's monomial is smaller under `order`**.
- `lbp`, `lbp_cmp`, `lbp_key` — labeled polynomial constructors and comparators for the F5B signature-based algorithm.
- `lbp_sub(f, g)` — subtract labeled polynomials; **propagates signature and number from whichever operand has the larger signature** (via `sig_cmp`), not necessarily from the minuend.
- `lbp_mul_term(f, cx)` — multiply labeled polynomial by a term; scales both signature and polynomial.
- `critical_pair(f, g, ring)` — construct critical pair tuple for S-polynomial; **builds lightweight proxy labeled polynomials containing only leading terms** (not full polynomials) for comparison, then uses `lbp_cmp` on these proxies to determine which element appears first in the returned 6-tuple (larger signature first ensures correct S-polynomial subtraction order).
- `cp_cmp`, `cp_key` — critical pair ordering; `cp_cmp` uses **two-level comparison: first the dominant (signature) component, then the subordinate component as tiebreaker** when dominants are equal.

### [`fglmtools.py`](fglmtools.py)
FGLM algorithm for Gröbner basis conversion between monomial orderings.

- `matrix_fglm(F, ring, O_to)` — convert Gröbner basis from one ordering to another; **silently drops candidate basis polynomials that become zero after conversion to the target ring**.
- `_basis(G, ring)` — enumerate standard monomials (not divisible by any leading monomial of G); forms the vector-space basis of the quotient ring `K[X]/(G)`.
- `_update(s, _lambda, P)` — row-reduce projection matrix P so that `P' v = e_s` (s-th unit vector); pivots on the first non-zero entry at index ≥ s in `_lambda`, eliminates other rows, then swaps pivot row into position s.

### [`distributedmodules.py`](distributedmodules.py)
Sparse distributed module representations for submodule/syzygy computation.

- Basic element operations: `sdm_add` (add two module elements with cancellation), `sdm_LC` (**returns `K.zero` for empty/zero module elements**), `sdm_from_dict`, `sdm_sort`, `sdm_strip` — element arithmetic and construction.
- `sdm_mul_term(f, term, O, K)` — multiply module element by a polynomial term `(monomial, coeff)`; **if `K.is_one(c)`, skips coefficient multiplication** (returns original coefficients unchanged); returns `[]` if `f` is empty or `c` is zero.
- Module monomial operations: `sdm_monomial_mul`, `sdm_monomial_deg`, `sdm_monomial_lcm`, `sdm_monomial_divides`.
  - `sdm_monomial_lcm(A, B)` — computes LCM by **preserving the generator index (first tuple element) and delegating `monomial_lcm` on the remaining exponent entries**; result is undefined if A and B belong to different generators.
  - `sdm_monomial_divides(A, B)` — checks if polynomial monomial X exists such that XA = B; **returns False if A and B belong to different free module generators** (different first tuple element), even if polynomial exponents satisfy divisibility.
- `sdm_nf_buchberger(f, G, O, K, phantom)` — weak normal form using standard Buchberger algorithm (global orderings); optional `phantom` pair tracks companion coefficient vectors in parallel; **when phantom is None, uses `itertools.repeat([])` as dummy** to avoid branching in the divisor-search loop.
- `sdm_nf_buchberger_reduced` — reduced normal form (unique but more expensive); does NOT support phantom tracking.
- `sdm_nf_mora` — generalized Mora algorithm for weak normal forms with non-global orderings; **dynamically appends current element to the reducer set when the chosen reducer's ecart exceeds the element's ecart**.
- `sdm_ecart(f)` — difference between total degree and leading monomial degree.
- `sdm_groebner` — Gröbner basis (minimal standard basis) for submodules; uses "sugar" strategy for pair selection.
  - **If all input generators are zero, returns `[]` (no basis); when `extended=True`, returns `([], [])` (empty basis + empty transition matrix)**.
  - Sugar priority: `max(degree_i − deg(LM_i), degree_j − deg(LM_j)) + deg(lcm(LM_i, LM_j))`, combining element degrees and leading monomial LCM degree.
  - Inner `update` applies **chain criterion to prune critical pairs** whose LCM is divisible by the new element's LCM with both pair members.
- `sdm_spoly(f, g, O, K, phantom)` — S-polynomial of two module elements; returns zero if leading terms involve different basis generators; **optional `phantom` pair tracks auxiliary coefficient vectors through the same linear combination operations** (used for extended standard basis computation).

### [`ring_series.py`](ring_series.py)
Power series arithmetic in sparse polynomial rings.

- `_invert_monoms(p1)` — compute `x^n * p1(1/x)` for a sparse univariate polynomial, reversing the coefficient ordering by mapping degree k to degree (n−k).
- `rs_trunc` — truncate series to given precision.
- `rs_add`, `rs_mul`, `rs_pow`, `rs_series_inversion` — ring series operations.
  - `rs_mul` — truncated series multiplication; **single-generator rings use a fast path that manually constructs exponent tuples via integer addition; multi-generator rings use `monomial_mul`** for general exponent combination.
- `rs_exp`, `rs_log`, `rs_sin`, `rs_cos`, `rs_tan`, `rs_atan` — transcendental series.
- `rs_nth_root`, `rs_compose` — composition and roots.
- `rs_compose_add(p1, p2)` — composed sum `prod(p2(x - β) for β root of p1)` via Newton sums and Hadamard exponential transforms; **if result degree < deg(p1)*deg(p2), multiplies by x^dp to account for shared roots**.
- `rs_hadamard_exp(p1, inverse)` — coefficient-wise factorial division (`f_i/i!`) or multiplication (`f_i*i!`).

### [`dispersion.py`](dispersion.py)
Dispersion of polynomials — integer shift relationships between polynomial factor sets.

- `dispersionset(p, q)` — compute all non-negative integer shifts j where gcd(p(x), q(x+j)) ≠ 1.
  - **Constant polynomials (degree < 1) return `{0}` immediately** without factoring.
  - Factors both inputs over the rationals, then iterates over all factor pairs of equal degree and matching leading coefficient.
  - Candidate shift α is derived from sub-leading coefficients; **for linear factor pairs (degree 1), accepts α without further verification; for higher-degree pairs, performs full shifted-polynomial equality check** (`s == t.shift(α)`).
- `dispersion(p, q)` — maximum of the dispersion set (returns `-oo` if set is empty).

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
  - `__mul__(e)` — when `e` is not an `Ideal`, coerces via `self.ring.ideal(e)`; **returns `NotImplemented` (not an error) if coercion raises `CoercionFailed`**.
  - `__pow__(exp)` — exponentiation; **zeroth power returns unit ideal `ring.ideal(1)`** via `reduce` with empty list.
  - `subset(other)` — check if `other` is a subset of this ideal; **if `other` is an Ideal, delegates to `_contains_ideal`; if `other` is a plain iterable (e.g. list of ring elements), checks each element individually** via `_contains_elem`.
- `ModuleImplementedIdeal` (in `ideals.py`) — ideal implementation that delegates all operations to an underlying `SubModule`.
  - `_product(J)` — ideal product; assembles **all pairwise products of generators** from both ideals and creates a new submodule from them. Only method that depends on the underlying structure being a `SubModule` (not just a generic module).
- `Module.__div__(e)` (in `modules.py`) — quotient module; **if `e` is not a `Module` instance, unpacks it as generators to `self.submodule(*e)` before calling `quotient_module`**.
- `Module.__eq__(other)` (in `modules.py`) — equality via **mutual submodule inclusion**: returns True iff `self.is_submodule(other)` and `other.is_submodule(self)`.
- `Module.__mul__(e)` (in `modules.py`) — if `e` is not an `Ideal`, **coerces it to an ideal via `self.ring.ideal(e)` before delegating to `multiply_ideal`**; returns `NotImplemented` if coercion fails.
- `ModuleElement` (in `modules.py`) — base class for module element wrappers; stores reference to containing module.
  - `__add__`, `__sub__` — if operand is from a different module/class, **attempts `self.module.convert(om)`; returns `NotImplemented` on `CoercionFailed`** (no error raised).
  - `__mul__` — coerces scalar to ring element via `self.module.ring.convert(o)`; returns `NotImplemented` on failure.
  - `__eq__` — attempts coercion; **returns `False` (not `NotImplemented`) on `CoercionFailed`**, unlike arithmetic operators which return `NotImplemented`.
- `FreeModuleElement` (in `modules.py`) — element of a free module; data stored as a **tuple of ring entries**; arithmetic (`add`, `mul`, `div`) is component-wise over the tuple.
- `FreeModulePolyRing` (in `modules.py`) — free module over a generalized polynomial ring; **constructor requires the ring's ground domain to be a Field** (raises `NotImplementedError` for e.g. ZZ[x]).
- `FreeModuleQuotientRing` (in `modules.py`) — free module over a quotient ring `R/I`; internally holds a `.quot` attribute representing the same set as an R-module (modulo `I·R^n`).
  - `lift(elem)` — promote element from `R/I`-module to the `.quot` R-module by **extracting underlying `.data` from each component**; enables computation in the larger ring setting.
  - `unlift(elem)` — reverse of `lift`; push element of `.quot` back down to the quotient module.
- `SubModule.in_terms_of_generators(e)` (in `modules.py`) — express element as linear combination of generators; **catches `CoercionFailed` and raises `ValueError`** if `e` is not a member of the submodule.
- `SubModule.convert(elem, M)` (in `modules.py`) — if element is already the correct dtype and belongs to `self`, **returns immediately without membership check**.
  - Otherwise converts via container and checks `_contains`, raising `CoercionFailed` if not a member.
- `SubModule.submodule(*gens)` (in `modules.py`) — create a nested submodule from given generators; **raises `ValueError` if any generator is not contained in the parent module** (validates via `subset` check before construction).
- `SubModule.union(other)` (in `modules.py`) — combine generators of `self` and `other` into a single submodule; **raises `ValueError` if `other` belongs to a different free module container** (same check in `intersect` and `module_quotient`).
- `SubModule.is_submodule(other)` (in `modules.py`) — if `other` is a SubModule, checks all generators are contained; **if `other` is a FreeModule, returns True only when `self` is the full module** (via `is_full_module`); otherwise returns False.
- `SubModulePolyRing` (in `modules.py`) — submodule of a free module over a polynomial ring with Gröbner basis support.
  - Uses `ModuleOrder` for monomial comparison — extends `ProductOrder` (from `orderings.py`) with a zeroth component for the module generator index.
  - **`TOP=True` (default): term ordering takes priority over component index; `TOP=False`: component index takes priority**.
- `SubModulePolyRing._module_quotient(other)` (in `modules.py`) — compute the ideal quotient (colon ideal) `(self : other)`; **returns unit ideal `ring.ideal(1)` if `other` has no generators** (zero submodule); raises `NotImplementedError` if `relations=True` and `other` has more than one generator.
  - **Single generator**: embeds into a higher-rank free module with an elimination ordering (`ilex`) and extracts quotient from Gröbner basis elements whose non-last components are all zero.
  - **Multiple generators**: reduces to intersection of pairwise single-generator colon ideals.
- `SubModule.syzygy_module()` (in `modules.py`) — compute kernel of the map from a free module to `self`; **filters out zero relations** from the result for convenience.
- `FreeModule.convert(elem)` (in `modules.py`) — coerces lists (checks length matches rank), `FreeModuleElement` from other modules (checks rank compatibility), or literal `0` (creates zero vector); raises `CoercionFailed` otherwise.
- `SubQuotientModule` (in `modules.py`) — submodule of a quotient module; `__init__` builds a `base` submodule by placing original generators first, then killed-module generators (ordering is critical).
  - `_syzygies()` — computes kernel of the surjection onto the subquotient; **truncates full syzygy vectors to only the first `len(self.gens)` components**, projecting away the killed-module entries; relies on the generator ordering established in `__init__`.
- `QuotientModule.is_submodule(other)` (in `modules.py`) — for two QuotientModules, **requires killed submodules to be equal AND base modules to have containment**; for SubQuotientModule, checks container identity.
- `QuotientModule.convert(elem)` (in `modules.py`) — when source is another QuotientModule, succeeds **only if `self.killed_module` is a submodule of `elem.module.killed_module`**; raises `CoercionFailed` otherwise.
- `ModuleHomomorphism.__mul__` (in `homomorphisms.py`) — **if other is a `ModuleHomomorphism` with compatible domain/codomain, composes the two maps; otherwise attempts `ring.convert(other)` for scalar multiplication**; returns `NotImplemented` on `CoercionFailed`. `__rmul__` is aliased to `__mul__`.
- `ModuleHomomorphism.__init__` (in `homomorphisms.py`) — validates source/target are Module instances and **raises `ValueError` if they are defined over different base rings**.
- `MatrixHomomorphism` (in `homomorphisms.py`) — base for homomorphisms expressed as generator-image lists; constructor uses codomain's **container** converter when codomain is a SubModule or SubQuotientModule.
  - `_quotient_codomain(sm)` — quotient the codomain by `sm`; uses `Q.container.convert` for matrix entries **when codomain is a SubModule**, else uses `Q.convert`.
- `FreeModuleHomomorphism._kernel` — kernel via syzygy module of image generators.
- `SubModuleHomomorphism._kernel` — kernel via syzygy, **translates relations back through domain generators** by forming linear combinations.
- `ModuleHomomorphism.restrict_codomain(sm)` — narrow target module to submodule `sm`; **raises `ValueError` if `sm` does not contain the image**; returns `self` if `sm` equals the full codomain.
- `ModuleHomomorphism.quotient_domain(sm)` — replace domain with `domain/sm`; **raises `ValueError` if `sm` is not contained in the kernel**; returns `self` unchanged if `sm` is zero.
- `ModuleHomomorphism.quotient_codomain(sm)` — replace codomain with `codomain/sm`; **raises `ValueError` if `sm` is not a submodule of codomain**; returns `self` unchanged if `sm` is zero.
- `ModuleHomomorphism.restrict_domain(sm)` — restrict source to submodule `sm`.
- `homomorphism(domain, codomain, matrix)` — public constructor for module homomorphisms.
  - Internally decomposes domain/codomain into free presentations `(free_module, submodule, killed_module, convert_fn)` via type dispatch:
    - **FreeModule** → `(self, self, empty_submodule, convert)`; **QuotientModule** → `(base, base, killed_module, extract_data)`.
    - **SubQuotientModule** → `(container_of_base, base, killed_module, extract_data)`; **plain SubModule** → `(container, self, empty_submodule, convert_via_container)`.
  - Builds a `FreeModuleHomomorphism`, then restricts/quotients domain and codomain to match the original module types.

### [`domains/`](domains/catalog.md)
Algebraic domain hierarchy: ZZ, QQ, RR, CC, GF(p), algebraic fields, polynomial rings, fraction fields, expression domain.

- `Domain` (in `domain.py`) — abstract base class for all domains; `__getitem__` supports bracket syntax `K[x]` / `K[x, y]` to construct polynomial rings.
  - `map(seq)` — recursively convert all elements of a nested list to this domain; **recurses on sublists, calls `self(elt)` on leaf elements**; only handles `list` (not tuples or other iterables).
  - `convert(element, base=None)` — coerce element to this domain; **when `base` is None, dispatches by Python type** (int → ZZ, float → RR, complex → CC, GMPY types, `DomainElement` → parent, `Basic` → `from_sympy`).
    - **For numerical domains (`is_Numerical`), if element has `is_ground` (e.g. a constant-valued ring element), extracts its leading coefficient via `element.LC()` and recursively converts that scalar**.
    - **For unknown non-Basic, non-sequence types, attempts `sympify(element)` then retries via `from_sympy`**; raises `CoercionFailed` if all strategies fail.
  - `convert_from(element, base)` — dispatch conversion by looking up `from_<alias>` if the source domain has an alias, else `from_<ClassName>`.
  - Base arithmetic: `half_gcdex(a, b)` **delegates to `gcdex` and discards second Bézout coefficient**; `gcdex`, `gcd`, `lcm` raise `NotImplementedError` at base level — subclasses must override. `cofactors(a, b)` computes GCD then derives cofactors via `quo`.
  - `unify_with_symbols(K1, symbols)` — unify two domains given explicit generators; **raises `UnificationFailed` if either domain is composite (e.g. polynomial ring) and its generators overlap with the provided symbols**.
  - `unify(K0, K1)` — construct minimal domain containing both K0 and K1.
    - **Inexact domains (RR, CC)**: uses `max(precision)` and `max(tolerance)` from both operands; CC absorbs RR (complex wins over real).
    - When one is a FractionField and the other a PolynomialRing, **demotes merged ground back to ring** if neither original ground was a field but the unified ground is.
    - When both are `FiniteField` (GF(p)), **selects the one with the larger modulus** (via `default_sort_key`); if no known pairing matches, falls back to the expression domain `EX`.
- `CompositeDomain` (in `compositedomain.py`) — base class for composite domains (e.g. `ZZ[x]`, `ZZ(x)`).
  - `inject(*symbols)` — **raises `GeneratorsError` if new generators overlap with existing ones** (set intersection check); otherwise constructs a new ring/field with merged generators.
- `SimpleDomain` (in `simpledomain.py`) — base class for simple domains (ZZ, QQ); `inject(*gens)` **directly creates a polynomial ring over itself** (`self.poly_ring(*gens)`) without checking for generator overlaps (no existing generators to conflict with).
- `CharacteristicZero` (in `characteristiczero.py`) — mixin for domains with infinitely many elements; `characteristic()` returns 0. Inherited by ZZ, QQ, RR, CC, algebraic fields.
- `Ring` (in `ring.py`) — abstract base for ring domains.
  - `is_unit(a)` — test invertibility by attempting `revert`; `revert(a)` **only succeeds for the multiplicative identity** (raises `NotReversible` otherwise).
- `RationalField` (in `rationalfield.py`) — the field of rationals QQ.
  - `from_AlgebraicField(a, K0)` — convert algebraic number field element back to QQ; **succeeds only if element is ground (constant)**, extracting its leading coefficient.
    - Implicitly returns `None` (conversion failure) for non-ground algebraic elements.
- `AlgebraicField` (in `algebraicfield.py`) — algebraic number field `Q(α)`; ground domain must be QQ.
  - `is_positive(a)` / `is_negative(a)` — sign determination; **inspects only the leading coefficient** of `a`'s internal polynomial representation via `a.LC()`, delegating to the ground domain's sign check.
  - `from_sympy(a)` — two-stage conversion: first tries ground rational field (`dom.from_sympy`); **on `CoercionFailed`, falls back to `to_number_field` to interpret `a` as an algebraic element** of the extension; raises `CoercionFailed` if both fail.
- `Field` (in `field.py`) — abstract base for field domains; inherits from `Ring`.
  - `gcd(a, b)` — tries to delegate to associated ring's GCD on numerators/denominators; **if no associated ring exists (raises `DomainError`), falls back to returning `self.one`** (trivial GCD).
  - `lcm(a, b)` — same fallback pattern; **returns `a*b` if no associated ring exists**.
  - `exquo`, `quo`, `div` — all equivalent to exact division (`a / b`); `rem` always returns `self.zero`.
- `RealField` (in `realfield.py`) — real numbers up to given precision (mpmath `mpf`).
  - `from_ComplexField(element, base)` — converts complex domain element to real; **silently returns `None` (no error) if element has nonzero imaginary part**, signaling conversion failure to the domain machinery.
- `ComplexField` (in `complexfield.py`) — complex numbers up to given precision (mpmath `mpc`).
  - `get_ring()` — **always raises `DomainError`** ("no ring associated with CC"); `has_assoc_Ring = False`.
  - `from_sympy(expr)` — convert SymPy expression to complex number; evaluates numerically, splits into real/imaginary parts, **raises `CoercionFailed` if either part fails `is_Number` check** (e.g. unevaluated symbols remain).
  - `from_ComplexField(element, base)` — converts between complex domains; **if source and target are the same domain (same precision/tolerance), returns element unchanged**; otherwise re-constructs via `self.dtype(element)`.
- `ModularInteger` (in `modularinteger.py`) — element class for finite residue rings (GF(p) elements); created by `ModularIntegerFactory` which caches per-modulus classes.
  - `to_int()` — convert back to plain integer; **when symmetric mode (`sym=True`), values exceeding `mod // 2` are mapped to negative by subtracting the modulus**.
  - `__pow__(exp)` — exponentiation; **for negative `exp`, computes multiplicative inverse first** (via `invert`), then raises to `|exp|`.
  - `invert()` — compute modular inverse via `dom.invert(val, mod)`.
  - `ModularIntegerFactory(_mod, _dom, _sym, parent)` — creates and caches a `ModularInteger` subclass for a given modulus; **raises `ValueError` if modulus < 1**; names class `SymmetricModularIntegerMod<n>` or `ModularIntegerMod<n>` depending on `_sym` flag.
- `FiniteField` (in `finitefield.py`) — GF(p) domain; `from_sympy` accepts Integer and whole-number Float (e.g. 3.0), raises `CoercionFailed` otherwise.
  - `from_QQ_python` / `from_QQ_gmpy` — convert rational (Fraction/mpq) to GF(p); **returns `None` (silent failure) if denominator ≠ 1**; only whole-number rationals are accepted.
  - `from_RealField(K1, a, K0)` — convert mpmath `mpf` to GF(p) element; **contains a bug: references `self` instead of `K1`**, causing `NameError` at runtime.
- `IntegerRing` (in `integerring.py`) — abstract ZZ domain; `from_AlgebraicField(a, K0)` — **succeeds only if element is ground (constant)**, converting its leading coefficient; **implicitly returns `None` for non-ground elements** (same pattern as `RationalField.from_AlgebraicField`).
- `PythonIntegerRing` (in `pythonintegerring.py`) — ZZ domain backed by Python `int`; `from_sympy` accepts Integer directly and **also accepts Float if it represents a whole number** (e.g. 3.0 → 3).
- `PolynomialRing` (in `polynomialring.py`) — `K[x₁,…,xₙ]` domain wrapper.
  - `__init__(domain_or_ring, symbols, order)` — **dual-path initialization**: if first arg is already a `PolyRing` and no other args given, reuses it directly; otherwise constructs a new `PolyRing` from the provided symbols, domain, and ordering.
  - `from_FractionField` converts a rational function to a ring element **only if the denominator is ground** (constant), else returns None.
  - `from_AlgebraicField(a, K0)` — **succeeds only if `K1.domain == K0`**; **implicitly returns `None` otherwise** (silent failure, no conversion attempted).
- `PolynomialRing(dom, *gens, **opts)` factory (in `old_polynomialring.py`) — creates a generalized multivariate polynomial ring.
  - **If monomial order is global → `GlobalPolynomialRing` (DMP-based); otherwise → `GeneralizedPolynomialRing` (DMF-based, localization)**.
  - `GeneralizedPolynomialRing.new(a)` — construct element; **validates that the denominator's leading term under the ring's ordering has all-zero exponents** (i.e., denominator is a unit in the localization); raises `CoercionFailed` otherwise.
  - `PolynomialRingBase.__init__(dom, *gens, **opts)` — initializes the ring; **if `order` key is absent from opts (e.g. when constructed via `inject`), falls back to `default_order` ("grevlex")** via `monomial_key`.
  - `PolynomialRingBase.revert(a)` — multiplicative inverse; attempts `1/a` and **catches both `ExactQuotientFailed` and `ZeroDivisionError`**, raising `NotReversible` if the element is not a unit.
  - `PolynomialRingBase.from_AlgebraicField(a, K0)` — convert algebraic number field element; **returns `None` (silent failure) if `K1.dom != K0`** (ground domain mismatch), unlike other `from_*` methods which always attempt conversion.
  - `GeneralizedPolynomialRing._vector_to_sdm` — converts a vector of rational function elements to sparse distributed module form.
    **Clears all denominators first** by computing the product of all entry denominators, making entries integral before delegation.
  - For product/mixed orders given as tuples, builds the product order first, then checks `order.is_global`.
- `GlobalPolynomialRing` (in `old_polynomialring.py`) — legacy generalized polynomial ring using `DMP` dtype; `from_FractionField` converts only if **denominator is trivial (one)**, else returns None (silent failure). `from_GlobalPolynomialRing` handles cross-ring conversion: same gens → direct rep copy; different gens → reorders monomials via `_dict_reorder` and converts coefficients if domains differ.
- `FractionField` (in `fractionfield.py`) — multivariate rational function field domain `K(x₁,…,xₙ)` using `FracField` dtype.
  - `is_positive(a)` / `is_negative(a)` — sign determination; **inspects only the leading coefficient of the numerator** (`a.numer.LC`), delegating to the base domain's sign check; denominator sign is ignored.
  - `from_AlgebraicField(a, K0)` — converts algebraic number to fraction field element **only if `K1.domain == K0`** (target's ground domain equals source field).
  - **Silently returns `None` otherwise** (no error, no attempt), unlike other `from_*` methods which always delegate to `K1.domain.convert`.
  - `from_PolynomialRing(a, K0)` — converts polynomial ring element; requires same or subset generators; **returns `None` if generators are incompatible**.
- `FractionField` (in `old_fractionfield.py`) — legacy rational function field domain using `DMF` dtype.
  - `from_sympy` — splits expression into numerator/denominator, converts coefficients, then **calls `.cancel()` to ensure reduced form**.
  - `from_FractionField(a, K0)` — convert between fraction fields: same gens → direct copy or domain conversion; source gens ⊂ target gens → reorders monomials; **incompatible gens → implicitly returns `None`** (silent conversion failure).
  - `from_GlobalPolynomialRing` — cross-ring conversion: **same gens + same domain → direct rep copy; same gens + different domain → converts element to target domain first**; different gens → reorders monomials via `_dict_reorder` and converts coefficients.
- `QuotientRing` (in `quotientring.py`) — commutative quotient ring `R/I`; `QuotientRingElement.__eq__` **returns `False` immediately if the other operand is not the same class or belongs to a different ring** (no conversion attempt), unlike `__mul__`/`__add__` which try `ring.convert`; for same-ring elements, tests whether their difference belongs to the ideal.
  - `revert(a)` — compute multiplicative inverse of `a` in `R/I`; **forms the sum of the principal ideal `(a)` and the base ideal, then tests if 1 is expressible in their generators**; raises `NotReversible` if not a unit.
