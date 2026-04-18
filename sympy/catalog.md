# SymPy Root Catalog

## Architecture Overview

SymPy is organized into submodules by mathematical domain. Key routing rules:
- **Expression tree internals** (Pow, Add, Mul, Expr, Basic, Symbol, Number, _eval_subs, _eval_power) → `core/`
- **Polynomial algebra** (Gröbner bases, factorization, domains, modules, number fields) → `polys/`
- **Mathematical function classes** (trig, hyperbolic, piecewise, elliptic, combinatorial number sequences) → `functions/`
- **Expression transformation/simplification** (fu, trigsimp, hyperexpand, combsimp) → `simplify/`
- **Pretty/code/string printing** (LaTeX, Fortran, C, Python, MathML, lambda repr) → `printing/`
- **Equation solving** (algebraic, ODE, recurrence, diophantine, bivariate) → `solvers/`
- **Numeric code generation from expressions** (lambdify, autowrap, codegen, ufuncify) → `utilities/`
- **AST node definitions for code generation** (not expression printing) → `codegen/`
- **Property inference on symbols** (handlers, sathandlers, fact rules) → `assumptions/`
- **Propositional/boolean logic** (And, Or, satisfiability, DPLL) → `logic/`
- **Probability and statistics** (random variables, distributions, PDF/CDF) → `stats/`
- **New-style 3D vector algebra** (CoordSysCartesian, Point, divergence, curl) → `vector/`
- **Classical mechanics vectors** (ReferenceFrame, dynamicsymbols, mechanics printing) → `physics/vector/`

## Glossary

- **_eval_subs / _eval_power / _eval_is_***: internal hooks on expression classes. `Pow._eval_power` is in `core/power.py`.
  - Function classes (e.g. `Abs`, `exp`) define their own `_eval_power` AND `_eval_is_*` (e.g. `_eval_is_rational`) in `functions/`. Core types define `_eval_is_*` in `core/`, NOT `assumptions/handlers/`.
- **handlers**: assumption inference callbacks in `assumptions/handlers/`, NOT in `logic/`.
- **lambdify / lambdastr / ufuncify / autowrap**: numeric callable generation in `utilities/`, NOT `codegen/` or `printing/`.
- **codegen (utilities/)**: generates Fortran/C/Julia **source files** with `Routine`, `FCodeGen`, `CCodeGen`.
- **codegen/ (submodule)**: defines AST node classes (`ast.py`). Does NOT generate source or callable code.
- **fcode / FCodePrinter**: Fortran expression printer in `printing/fcode.py`, NOT in `codegen/`.
- **ring_series**: formal power series via polynomial ring operations, in `polys/`, NOT `series/`.

## Top-Level Files

### [`__init__.py`](__init__.py)
Package entry point: version check, mpmath dependency guard, submodule re-exports.
- `__sympy_debug`: reads `SYMPY_DEBUG` env var; only accepts `'True'`/`'False'`, raises RuntimeError on other values.
- `SYMPY_DEBUG` global flag set at import time.

### [`abc.py`](abc.py)
Pre-defined single-letter Symbol instances (latin and greek) for interactive convenience.
- `clashing` — dict of multi-letter clashing names that conflict with SymPy objects.
- NOT where Symbol or Dummy classes are defined (those are in `core/symbol.py`).

### [`galgebra.py`](galgebra.py)
Deprecated stub redirecting to the external `galgebra` package.

## Submodules

### [`core/`](core/catalog.md)
Foundational expression tree: base classes, arithmetic operations, and evaluation hooks.
- `basic.py` — `Basic` base class: `subs` (simultaneous mode uses product of two Dummy placeholders to trigger `Subs` on bound variables), `_has`, `matches`, `atoms`, `xreplace`, `replace` (pattern-driven substitution; `exact` flag gates multi-Wild matches to reject zero-valued placeholder bindings), structural traversal.
  - `rewrite`/`_eval_rewrite`: recursive rule-dispatch expression transformation; `deep` hint controls descent into sub-expressions; checks node type against pattern before applying.
  - `as_poly`: convert expression to `Poly` or return None; catches `PolynomialError` and checks `is_Poly` flag.
- `expr.py` — `Expr` class: `coeff` (extract prefactor of x**n from a sum; n=0 returns x-independent terms), `as_coefficient`, `leadterm`, `extract_multiplicatively`, `as_leading_term`; `is_rational_function` (checks if expression is ratio of polynomials; returns False for NaN/infinities), `is_algebraic_expr`; `series` (iterative retry with linear prediction when internal nseries returns fewer terms; raises ValueError after 8 failed attempts); ordering operators `__ge__`, `__le__`, `__gt__`, `__lt__` (validate operands: reject non-real complex and NaN with TypeError; real operands → compute difference and check sign properties; otherwise return unevaluated relational).
  - `_eval_interval`: definite evaluation self.subs(x,b)−self.subs(x,a) with limit fallback on NaN/∞; if lower-bound limit yields NaN, returns NaN immediately (short-circuits before evaluating upper bound).
  - `extract_branch_factor`: decompose polar exponential into winding number and remainder; `allow_half` also extracts half-turn (pi*I) components.
  - `is_constant`: invariance check with assertion guard for is_number/free_symbols inconsistency; numerical evaluation + differentiation fallback.
  - `getn`: extract truncation order from O(...) term; handles product expressions (x^n*log^n) by substituting positive Dummy.
  - `as_independent`: separate expression into parts independent/dependent on given symbols; mismatched type (Add vs Mul) returns (identity, self) immediately.
  - `as_ordered_terms`: sort terms with O(...) handling; when Order terms present, always uses reverse=True regardless of order spec.
  - `expand`: applies rewriting hints in sorted order; special key ensures multinomial runs before mul; `denom`/`numer`/`frac` flags split expression via `fraction()` and selectively expand only denominator, numerator, or both.
- `power.py` — `Pow` class: `_eval_power`, `_eval_subs` (exponent splitting for substitution), `_eval_expand_power_base` (distribute exponent over product; imaginary unit count mod 4), `_eval_transpose` (checks `is_complex` first — differs from adjoint/conjugate which check `is_positive`; complex base → return unchanged, integer exp → wrap base in transpose), `_eval_adjoint`, `_eval_conjugate`.
  - `integer_nthroot`: exact floor n-th root of nonneg integer; overflow-safe initial estimate via log₂ + bit-shifting when float exponentiation raises OverflowError; Newton iteration for large values, linear compensate for small.
  - `as_content_primitive`: extract positive Rational from Pow; rational base + fractional exponent → divmod separates integer/remainder exponent parts; Mul base → recursive content extraction.
- `add.py` — `Add` class: `flatten` (infinity filtering, order processing; reconstruction phase: Mul-typed symbolic parts use fast rawargs slot-0 insertion, Add-typed parts use unevaluated Mul to avoid flattening, other types use full Mul machinery), `_eval_as_leading_term`, `primitive`, `as_content_primitive` (radical factoring: extracts common nth-root factors from integer bases raised to fractional exponents across additive terms).
  - `_combine_inverse`: static method computing lhs−rhs for pattern matching; treats oo−oo and oo*I−oo*I as zero instead of NaN.
  - `_unevaluated_Add`: construct well-formed unevaluated sum; flattens nested Adds, collects numeric constants into slot 0, sorts remaining args. Used when args changed but evaluation is unwanted.
- `mul.py` — `Mul` class: `flatten`, `as_content_primitive`, `as_two_terms`.
- `numbers.py` — `Integer`, `Rational` (`__new__`: string input with `/` splits via rsplit and converts each side through `fractions.Fraction`, e.g. `'1e-2/3.2'` → 1/320; rejects >1 slash), `Float` (tuple+hex-string reconstruction from pickle), `ImaginaryUnit`, `AlgebraicNumber`, `Exp1`, `_eval_power`, `mod_inverse`; `igcd`, `igcdex` (extended Euclidean returning Bézout coefficients x,y and gcd; handles zero inputs), `ilcm`.
  - `NumberSymbol` constants (`Pi`, `EulerGamma`, `Catalan`, `GoldenRatio`, `NegativeInfinity`, `Infinity`): `approximation_interval` returns bounding rational/integer intervals for efficient comparison without full numerical evaluation.
  - `NegativeInfinity`/`Infinity`: ordering operators (`__lt__`, `__le__`, `__gt__`, `__ge__`) with explicit subcases for finite, nonneg, and same-sign infinite operands.
- `function.py` — `FunctionClass` (metaclass for Function; `__init__` validates nargs: empty sequence → ValueError suggesting 0 or None), `Function`, `Lambda` (anonymous callable: `__eq__` implements alpha-equivalence by renaming bound variables before body comparison), `Derivative.__new__`, `expand()`, `_mexpand` (combined multinomial+mul expansion, optionally iterating to fixed point), `AppliedUndef`, `WildFunction`, `nfloat`; `_coeff_isneg`.
  - `expand_power_base` (split (a*b)**n → a**n * b**n; refuses unless base is non-negative or exponent is integer; `force=True` overrides), `expand_trig`, `expand_func`, `expand_complex`: selective expansion wrappers.
- `containers.py` — `Tuple` (`tuple_count`), `Dict`; `cache.py` — `cacheit` memoization decorator, `__cacheit_debug`.
- `relational.py` — `Relational`, `Equality`, `GreaterThan`, `StrictLessThan`; `as_set` (univariate inequality → real set; raises NotImplementedError for multivariate).
- `exprtools.py` — `gcd_terms`, `_gcd_terms` (extracts shared divisor from additive components; returns zero/zero/one for empty input), `factor_terms`: GCD extraction with non-commutative masking.
  - `Factors`: power-factor dictionary wrapper; `__init__` (from Rational: stores numerator with exponent 1, denominator with exponent −1; negative values prepend NegativeOne factor); `normal` (remove GCD from two factor dicts; symbolic exponent fallback via `extract_additively` then numeric coefficient comparison), `mul`, `div`.
  - `Term`: efficient `coeff*(numer/denom)` representation; `__init__` decomposes factors via `decompose_power` — when a factor's base is a sum (is_Add), extracts primitive content and folds it into coefficient; positive exponents → numer, negative → denom.
  - `_mask_nc`: replace non-commutative entities with Dummy placeholders before polynomial factoring; single nc-entity → commutative Dummy (standard factor works), multiple → non-commutative Dummies preserving order.
  - `factor_nc`: non-commutative factoring; delegates to standard `factor` when `_mask_nc` returns substitutions, else manual common-prefix/suffix extraction.
- `evalf.py` — `hypsum` (infinite hypergeometric series summation; polynomial convergence with exponent p==1 only converges if alternating), `evalf_sum`, `evalf_prod` (numerical product evaluation; non-integer-width bounds → rewrite as Sum): numerical evaluation, precision.
  - `pure_complex`: decompose symbolic expression into (real, imag) numeric pair via as_coeff_Add + as_coeff_Mul; returns None for irrational/non-numeric terms like √2; `or_real` flag returns (h, 0) for purely real inputs.
  - `evalf_trig`: numerical sin/cos of complex args; zero-argument dispatch returns (1, None) for cos, (None, None) for sin; iterative precision refinement loop near multiples of π.
  - `do_integral`: numerical definite integration; optimizes constant-integrand case by simplifying bounds difference.
  - `get_integer_part`: floor/ceiling with near-integer handling.
- `mod.py` — `Mod`: symbolic modulo operation; `eval` handles float coefficient extraction (e.g. `Mod(.6*x, .3*y)` → `0.3*Mod(2*x, y)`), GCD simplification, denesting, and Add-term reduction.
- `logic.py` — fuzzy logic (`fuzzy_and`, `fuzzy_or`, `fuzzy_not`) AND internal propositional `Logic` class with `fromstring` (space-delimited boolean formula parser), `And`/`Or`/`AndOr_Base` (`__new__`: auto-simplifies contradictions — if both a proposition and its negation are present, returns annihilating value (False for And, True for Or); skips identity elements, deduplicates, flattens nested same-type ops). NOT `logic/boolalg.py` (which defines the user-facing boolean algebra classes).
  - `_fuzzy_group`: like `fuzzy_and` but conservative; `quick_exit=True` returns None on second False (one exception tolerable, two makes result indeterminate).
  - `And.expand`: distributes conjunction over disjunction to convert to sum-of-products (disjunctive normal) form. NOT `logic/boolalg.py` `to_dnf`.
- `symbol.py` — `Symbol`, `Dummy`, `Wild` class definitions; `Symbol._sanitize` (in-place assumption cleanup: translates deprecated property names — `bounded`→`finite`, `unbounded`→`infinite`, `infinitesimal`→`zero` — with deprecation warnings; converts values to bool, removes None entries); `Wild.matches` (checks `exclude` list via `has`, checks `properties` predicate callables — any failing predicate returns None/no-match); `symbols` (batch creation from comma/space-separated names; colon range notation with letter-inclusive/digit-exclusive endpoints); `operations.py` — `AssocOp` (`_matches_commutative`: Add/Mul pattern matching; filters exact-part free symbols against target), `LatticeOp` (join/meet for algebraic lattices; `__new__`: `_new_args_filter` raises `ShortCircuit` on absorbing element → returns `zero` immediately; empty args → `identity`; single → unwrap).
- `sympify.py` — `sympify()`: convert arbitrary objects to SymPy types; `kernS` (string-to-expression with placeholder insertion to prevent autosimplification of products with parenthesized sums; falls back to original string on parse error).
- `compatibility.py` — `ordered`: conservative tie-breaking sort with successive keys; fallback/warn on unresolved ties.
- `assumptions.py` — `_assume_rules`: `FactRules` definition encoding logical implications between number-theoretic properties (e.g. integer→rational→real→complex, positive→real).
  - `StdFactKB`: specialized `FactKB` for built-in rules; `__init__` three-way dispatch: falsy/empty→empty `_generator` (no deduction), `FactKB` instance→copies its `.generator`, plain dict→stores copy and runs full deduction.
  - `_ask`: recursive property resolution with anti-recursion guard and shuffled prerequisites.
  - `make_property`: copy-on-write property factory (copies shared class-level assumptions dict before mutation).
  - `ManagedProperties` metaclass: protects subclasses from inheriting parent's cached static assumption values by replacing with dynamic descriptors.
- `singleton.py` — `SingletonRegistry` (`S`): lazy-instantiation registry; `__getattr__` creates instance on first access, installs as attribute to bypass future lookups; `Singleton` metaclass auto-registers classes.
- `facts.py` — `deduce_alpha_implications`: transitive closure of inference rules with contrapositive generation; `apply_beta_to_alpha_route` (extend single-premise forward-chaining tables with conjunctive AND rules via fixed-point iteration).
  - `Prover`: logic rule prover; `_process_rule` decomposes compound implications (And on RHS → split; Or on RHS → contrapositive + conditional elimination); detects tautologies via `TautologyDetected` exception.
  - `FactKB`: propositional knowledge base dict; `_tell` (add fact: overwrites None, returns False on duplicate, raises InconsistentAssumptions on conflict); `deduce_all_facts` (forward-chain propagation).
  - `rules_2prereq`: builds reverse dependency (prerequisite) table mapping each conclusion back to its source propositions, stripping negation wrappers from both sides.
- `multidimensional.py` — `apply_on_element` (recursively apply scalar function to every leaf of a nested iterable; distinguishes positional int vs named str parameter substitution), `vectorize`; `trace.py` — `Tr`: generic trace; `__new__` distributes over Add, separates commutative/non-commutative factors in Mul (scalars pulled outside), delegates to `.trace()` for matrix-like objects, returns Pow expr directly when both base and exponent are scalar; `permute` (cyclic argument reorder).

### [`polys/`](polys/catalog.md)
Polynomial algebra, domains, Gröbner bases, factorization, root isolation, and module theory.
- `polytools.py` — `Poly` class, `degree`, `primitive`, `factor`, `gcd`, `groebner` (high-level API); `div`, `rem`, `quo`, `exquo` (top-level expression-in/expression-out polynomial division; convert to Poly, operate, convert back); `gcd_list`, `quo_ground`/`exquo_ground`, `cancel`; `LT` (leading term: extracts monom+coeff, returns `coeff*monom.as_expr()`), `LM` (leading monomial as expr), `LC` (leading coefficient).
  - `Poly.coeff_monomial` (coefficient of a monomial; converts monomial expression to exponent tuple via `Monomial(...).exponents`, delegates to `nth`), `Poly.nth` (coefficient by exponent tuple; validates exponent count matches number of generators).
  - `Poly.eval` (evaluate at point: dict→per-generator substitution, tuple/list→zipped with generators raising ValueError if too many values), `Poly.termwise` (apply transformation function to each term; duplicate monomial detection raises PolynomialError), `Poly.as_expr` (convert to Expr; dict arg substitutes generators by index, raises GeneratorsError on unrecognized key).
  - `to_rational_coeffs`: convert univariate polynomial with square-root coefficients to rational-coefficient form via rescaling x→αx or translation x→x+β; only activates for second-order radicals.
  - `cancel`: simplify rational functions; noncommutative fallback sifts Add/Mul args; other expr types: preorder tree walk with recursive cancel.
- `polyclasses.py` — `GenericPoly` (base class for low-level polynomial representations): `ground_to_ring`, `ground_to_field`, `ground_to_exact` (convert approximate coefficient domain to exact); `DMP`, `DMF`, `ANP` (algebraic number field element; `__eq__`/`__ne__` catch `UnificationFailed` and return False/True gracefully; `__lt__`/`__le__`/`__gt__`/`__ge__` do NOT catch — propagate exception on incompatible fields).
- `polyroots.py` — `roots`: main polynomial root finder; `_try_decompose` (functional decomposition: solves innermost layer, iterates outward substituting previous roots), `_try_heuristics` (formula/trick-based fallback).
  - `roots_quadratic` (inner `_sqrt` extracts perfect-square factors from discriminant), `roots_cubic`, `roots_quartic`, `roots_quintic` (solvable degree-5 via resolvent radicals; deduplication check returns [] if any two roots coincide numerically): algebraic root formulas; `preprocess_roots` strips symbolic coefficients.
- `rootoftools.py` — `RootOf`, `ComplexRootOf` (implicit algebraic roots of univariate polynomials), `RootSum`; `_separate_imaginary_from_complex` (classify bounding intervals as purely imaginary vs general complex; two-term polynomials with power-of-2 degree get special handling), `_refine_complexes` (ensure non-conjugate root rectangles are disjoint when slid).
- `rootisolation.py` — `dup_isolate_real_roots_list` (VAS continued fractions); `_classify_point`, `dup_isolate_complex_roots_sqf`.
  - Collins-Krandick complex root isolation: `_vertical_bisection`/`_horizontal_bisection` (split rectangles, refine straddling boundary intervals).
  - `_traverse_quadrants`: convert quadrant sequences to winding-number rules with selective edge/corner exclusion via compass directions.
- `polyconfig.py` — `configure`: initializes polynomial algorithm settings from `SYMPY_`-prefixed OS environment variables at import time (uses `eval`, falls back to raw string on NameError); `setup`, `query`, `using` context manager.
- `polyoptions.py` — `Options._init_dependencies_order`: resolves processing order of polynomial options via topological sort of `before`/`after` declarations; raises RuntimeError on cycles.
  - `Wrt.preprocess`: parse 'with-respect-to' variable specification; comma/whitespace-separated string splitting via regex; trailing comma raises OptionError; Basic→stringified, iterable→mapped to str list.
- `polyfuncs.py` — `interpolate` (construct interpolating polynomial; plain list of y-values defaults to x=1..n), `horner`, `rational_interpolate`.
- `orthopolys.py` — `hermite_poly`, `laguerre_poly`, `legendre_poly`: computational polynomial generators.
- `factortools.py` — low-level factorization: Hensel lifting, Zassenhaus algorithm; `dmp_zz_diophantine` (multivariate Diophantine equation solver for Wang/EEZ factorization; reduces to univariate via evaluation points, lifts solutions via Taylor-like expansion with differentiation and factorial division).
- `numberfields.py` — `minimal_polynomial`, algebraic field extensions.
- `distributedmodules.py` — sparse distributed module elements (free modules over multivariate rings).
  - `sdm_add` (merge two sparse representations, deletes entries when coefficients cancel to zero), `sdm_strip`, `sdm_monomial_divides`, `sdm_nf_mora`.
- `densetools.py` — `dup_integrate`, `dmp_integrate` (indefinite integration of dense polynomial in flat-list representation; repeated antidifferentiation via factorial-product divisor), `dmp_diff_in` (m-th order partial derivative w.r.t. j-th variable; recursive descent through nested coefficient levels via `_rec_diff_in` to reach target variable depth), `dup_real_imag` (split univariate poly into real/imag bivariate parts via x→x+iy; uses k%4 cycle for i powers), `dup_mirror`, `dup_extract`.
  - `dup_decompose`: functional decomposition f=f₁(f₂(...fₙ)); iteratively splits via `_dup_right_decompose`/`_dup_left_decompose` until no further decomposition is possible.
- `specialpolys.py` — `swinnerton_dyer_poly` (benchmark poly from sums of √primes; orders 1–3 use hardcoded expressions, n>3 computes via `minimal_polynomial`), `cyclotomic_poly`, `symmetric_poly`, `random_poly`, `interpolating_poly`.
- `solvers.py` — low-level linear system solver over polynomial rings; `solve_lin_sys` (row-reduces to RREF; underdetermined systems express pivot variables in terms of free generators), `eqs_to_matrix`.
- `subresultants_qq_zz.py` — subresultant PRS algorithms: `sturm_pg`/`sturm_q`/`sturm_amv` (generalized Sturm sequences via Pell-Gordon/quotient/AMV methods), `euclid_pg`/`euclid_q`/`euclid_amv`, `subresultants_bezout`, `subresultants_rem`.
- `ring_series.py` — formal power series via polynomial rings: `rs_sin`, `rs_cos`, `rs_tan`, `rs_log` (truncated log via integral of p'/p; no constant term → raises error), `rs_tanh`, `rs_exp`, `rs_asin`, `rs_atan`, `rs_atanh`, `rs_LambertW`, `rs_series_inversion`.
  - `rs_sin`/`rs_cos`: nonzero constant term triggers trig addition identity; if trig values can't be represented in the current ring, extends via `add_gens` to accommodate sin/cos of constant; >20 terms switches to half-angle tangent identity for performance.
  - `rs_series_from_list`: evaluate sum c[n]*p^n mod O(x^prec); baby-step/giant-step √n block decomposition to reduce multiplications.
  - `rs_puiseux`: adapter for fractional-exponent (Puiseux) series; computes LCM of exponent denominators, rescales to integer exponents, delegates, rescales back.
- `galoistools.py` — finite-field polynomial arithmetic: factorization (Berlekamp, Cantor-Zassenhaus), GCD; `gf_crt`/`gf_crt1`/`gf_crt2` (two-phase CRT split: precompute modular inverses for reuse across multiple residue sets). NOT `ntheory/modular.py` (higher-level CRT).
- `polytools.py` also has `terms_gcd` (extract shared monomial factor from addends, with `deep` flag for recursive traversal into function args).
- `agca/` — abstract algebra: `modules.py` (free modules, `SubModule`: `is_submodule` checks containment including FreeModule/QuotientModule via `is_full_module`; `identity_hom` restricts container's identity map to self in both domain and codomain).
  - `homomorphisms.py` — `ModuleHomomorphism`: `__eq__`, `quotient_domain`, `restrict_codomain`, `quotient_codomain`, `_compose`.
    - `__mul__`: composition vs scaling dispatch; compatible homomorphism → `oth._compose(self)` (reversed order); else scalar multiplication via ring coercion.
    - `homomorphism` factory: decomposes source/target into free presentations via `freepres` (FreeModule/QuotientModule/SubQuotientModule/submodule), then chains restrict/quotient ops. NOT `categories/` morphisms.
- `polyutils.py` — expression-to-multinomial conversion: `dict_from_expr`, `_dict_from_expr` (iterative expand loop: repeatedly applies `expand_multinomial`/`expand_mul` until no integer powers of sums or products-of-sums remain), `parallel_dict_from_expr`, `expr_from_dict`.
- `rings.py` — `PolyElement`: multivariate polynomial ring elements; `__pow__`, `__mul__`, `square`.
  - `__add__`: same-ring poly addition with zero-entry cleanup; scalar addition coerces to domain and deletes zero-monom key on exact cancellation.
  - `str`: text representation of sparse multivariate poly; handles leading sign (strips " + " prefix, converts " - " to "-" for first term); constant terms strip redundant negative sign from coefficient string.
- `densebasic.py` — dense polynomial representation utilities; `dup_deflate`, `dmp_deflate`, `dup_degree`, `dmp_to_dict`, `dmp_raise`, `dmp_nest`.
  - `dmp_ground_p`: check if nested-list poly is constant; c=None → any constant, c falsy → delegates to zero check, c truthy → exact match. Also `dmp_ground`, `dmp_zero_p`.
- `fields.py` — `FracElement` arithmetic: dispatch for nested quotient-of-quotient domain operations.
- `euclidtools.py` — `dmp_cancel`, `dup_cancel` (cancel common factors in rational functions; clears denominators for field domains before GCD), `dmp_content`, `dmp_primitive`, `dmp_gcd`, `dmp_inner_gcd`.
- `rationaltools.py` — `together`: combine fractional subexpressions into a single quotient (always recurses into Pow base, but only recurses into exponent when `deep=True`).
- `partfrac.py` — `apart`: partial fraction decomposition; handles non-commutative expressions by splitting commutative/non-commutative factors.
- `modulargcd.py` — modular GCD algorithms; `_integer_rational_reconstruction` (recover a/b from residue mod composite via extended Euclidean).
- `orderings.py` — monomial orderings: `LexOrder`, `GradedLexOrder`, `ReversedGradedLexOrder`, `ProductOrder`, `InverseOrder` (reverses any ordering for local rings; recursively negates nested tuple output), `ilex`, `igrlex`, `igrevlex`.
- `monomials.py` — `Monomial`: product-of-powers representation with `__mul__`, `__div__`, `gcd`, `lcm`; `MonomialOps` (code-generated fast operations).
- `domains/` — coefficient domains: `Domain` base class (`map`: recursively convert nested lists, distinguishing sublists from leaf values).
  - `ExpressionDomain` (auto-cancels via `.cancel()` after every arithmetic op), `IntegerRing`, `RationalField`.
  - `complexfield.py` — `ComplexField`: approximate complex number domain; `from_sympy` (evalf + `as_real_imag` decomposition; raises `CoercionFailed` if parts are non-numeric).
  - `realfield.py` — `RealField`: `from_ComplexField` (returns None silently if imaginary part is nonzero, rather than raising an error).
  - `old_fractionfield.py` — `FractionField`: field of rational functions over indeterminates; sign predicates (`is_positive`, etc.) delegate to leading coefficient of numerator.
  - `mpelements.py` — `MPContext.__init__` (tolerance three-way: None→computed default, False→zero tolerance with max_denom=1000000, else converted; falsy tolerance defaults max_denom to 1000000 rather than computing from tolerance), `MPContext.to_rational`: continued-fraction rational approximation with bounded denominator.
  - `groundtypes.py` — fallback `python_sqrt`/`python_factorial` via mpmath when no gmpy available.
  - `modularinteger.py` — `ModularIntegerFactory`: cached type generation for ℤ/nℤ.
  - `pythonrational.py` — `PythonRational`: pure-Python rational number; `__pow__` (negative exp swaps p,q; uses `new` which skips GCD reduction unlike `__class__` used by other arithmetic ops); `_cmp` (comparison helper: catches TypeError from subtraction, returns NotImplemented for incompatible types).

### [`functions/`](functions/catalog.md)
Mathematical function classes (symbolic, unevaluated). Defines the functions, does NOT simplify them.
- `elementary/trigonometric.py` — `sin`, `cos`, `tan`, `sec`, `csc`, `cot`, `sinc` (unnormalized sin(x)/x; eval handles half-integer pi multiples), `acot`, `atan`, `asin`, `acos` and other inverse trig; `ReciprocalTrigonometricFunction` (base for sec/csc/cot; `fdiff` delegates to reciprocal's fdiff and returns −f'/f², `_calculate_reciprocal`/`_rewrite_reciprocal` pattern for all operations; `taylor_term`); `_peeloff_pi` (split additive arg into residual + largest rational multiple of π/2), `_pi_coeff`, `_eval_aseries`.
  - `cos._eval_rewrite_as_sqrt`: rewrite cosine as nested radicals; internal `ipartfrac` (partial fraction decomposition of rational coefficients with single-prime-factor early return), `_fermatCoords`, `_cospi257`.
- `elementary/piecewise.py` — `Piecewise`, `piecewise_fold` (distribute operations over branches; Boolean outer expr→Or/And/Not instead of Piecewise), `_sort_expr_cond`, `_eval_integral`.
  - `_eval_interval`: definite integral evaluation for piecewise; raises NotImplementedError when both limits are symbolic and incomparable to condition boundaries.
- `elementary/hyperbolic.py` — `sinh`, `cosh`, `tanh` and inverses; `_eval_expand_trig` (decomposes integer-multiple arguments n*x into x+(n-1)*x for recursive addition-formula expansion).
- `elementary/complexes.py` — `conjugate`, `Abs` (`_eval_power`: odd integer exponent reduces by 1; skips -1 to avoid circularity), `arg`, `re`, `im`, `sign`, `transpose`, `adjoint` (Hermitian conjugate: eval first tries `_eval_adjoint`, falls back to `conjugate(_eval_transpose())` if adjoint not directly available).
  - `polar_lift`: lift to logarithmic Riemann surface; factors positive/polar/unknown — positive-only products get `exp_polar(0)` wrapper.
- `elementary/miscellaneous.py` — `IdentityFunction` (singleton identity map; `__new__` bypasses Lambda constructor to avoid infinite recursion, constructs via `Expr.__new__` directly), `Id`; `Min`, `Max` (`MinMaxBase._is_connected`: two-pass ordering check with factor_terms fallback; `_find_localzeros`), `root`, `real_root`.
- `elementary/integers.py` — `floor`, `ceiling`, `frac`: rounding functions; `RoundFunction.eval` (splits arg into integral/numerical/symbolic parts; evaluates numerical part independently only when it and symbolic part occupy orthogonal domains — e.g. one real, one imaginary).
- `special/tensor_functions.py` — `KroneckerDelta` (fermi-level index logic: `_get_preferred_index` selects index with more fermi-level info; equal info defaults to index 0; `preferred_index`, `killable_index`), `LeviCivita`.
- `special/bessel.py` — Bessel functions (`besselj`, `bessely`, `besseli`, `besselk`), spherical Bessel (`jn`, `yn`), `jn_zeros` (spherical Bessel zeros; initial estimate n+π, spaced by π), Airy functions (`airyai`, `airybi`, `airyaiprime`, `airybiprime`; eval returns closed-form gamma-function values at origin).
- `special/polynomials.py` — symbolic orthogonal polynomial classes: `laguerre` (symbolic n at ∞ → `(-1)**n * ∞`), `hermite`, `chebyshev`, `legendre`, `assoc_legendre` (negative order → factorial-ratio conversion to positive order), `chebyshevt_root`/`chebyshevu_root` (root finders: raise ValueError if index k ≥ degree n), etc.
- `special/delta_functions.py` — `DiracDelta` (Dirac delta distribution; `_eval_expand_diracdelta` expands delta of polynomial into sum over roots weighted by |derivative|; repeated roots (multiplicity>1) abort expansion to avoid division by zero), `Heaviside`.
- `special/beta_functions.py` — `beta` (Euler's first integral B(x,y)=Γ(x)Γ(y)/Γ(x+y)); `fdiff` (digamma-based derivatives; raises ArgumentIndexError for argindex > 2).
- `special/elliptic_integrals.py` — `elliptic_f`, `elliptic_e`, `elliptic_k`, `elliptic_pi`.
- `combinatorial/numbers.py` — `fibonacci`, `lucas` (eval delegates to fibonacci(n+1)+fibonacci(n-1) rather than own recurrence), `bernoulli`, `genocchi` (2t/(e^t+1) generating function; `_eval_is_prime` returns True only for n=8 since SymPy does not consider negatives as prime), `catalan`, `harmonic` (`eval`: at n=∞ dispatches on numeric order m — negative→NaN, m≤1→∞, m>1→zeta(m); symbolic m unsupported; `_eval_expand_func`: decomposes H(n±k) into partial terms), `euler`, `nP`, `nC` (multiset support).
  - `nT`: partition count (integer→identical items, sequence→multiset; detects n-th roots via repeated self-convolution).
  - `stirling`: Stirling numbers of first/second kind; `_stirling1`/`_stirling2` (recursive with closed-form shortcuts for k near n); `d` parameter computes reduced Stirling S^d(n,k)=S(n-d+1,k-d+1) for partitions with minimum pairwise distance constraint.
- `combinatorial/factorials.py` — `factorial` (`_eval_is_composite`: returns True when arg ≥ 3, since n! for n≥3 is always composite), `subfactorial` (derangement count; `_eval_is_even` returns True for odd nonneg args), `binomial`, `RisingFactorial` (symbolic unevaluated Pochhammer; does NOT expand for integer args — see `simplify/combsimp.py` `_rf` for that), `FallingFactorial`, `factorial2`.
- `elementary/exponential.py` — `log` (eval handles `AccumBounds`, imaginary args, rationals; base=1 → NaN if arg=1, else ComplexInfinity), `exp`, `exp_polar`, `LambertW` (inverse of w·e^w; `_eval_is_real`: k=0 real when x>−1/e, k=−1 real when −1/e<x<0, other k always non-real for real x).
- `special/hyper.py` — `hyper` (generalized hypergeometric function), `meijerg` (Meijer G-function class: `fdiff`, `_diff_wrt_parameter`, `integrand` (Mellin-Barnes kernel from four parameter groups via gamma products), `_eval_evalf` (numerical evaluation with polar branch handling; bails out returning None if argument has ≠1 polar exponential factors)).
  - `meijerg` parameter accessors: `an`, `aother`, `bm`, `bother`, `ap`, `bq`.
  - `HyperRep`: base for branched pFq representations; `_eval_rewrite_as_nonrep` converts to Piecewise (|x|<1 vs |x|>1) for analytic continuation.
  - `hyper.radius_of_convergence`: checks non-positive integer parameter cancellation between numerator/denominator lists; returns 0 if denominator entries can't be cancelled.
  - `hyper.convergence_statement`: returns symbolic condition on argument variable under which series converges; finite radius → three-case Or based on real part of eta (sum of upper minus lower params) vs |z|≤1/|z|<1.
- `special/gamma_functions.py` — `gamma`, `loggamma`, `digamma`, `trigamma`, `polygamma` (iterated log-derivative of Γ), `uppergamma`, `lowergamma`.
  - `polygamma._eval_aseries`: asymptotic expansion at ∞ via Bernoulli numbers; intentionally returns extra terms for higher orders.
- `special/mathieu_functions.py` — `mathieus`, `mathieuc`, `mathieusprime`, `mathieucprime`: Mathieu function classes for the periodic ODE y''+(a−2q·cos(2x))y=0; `eval` methods handle q=0 closed forms and sign extraction from argument (even/odd parity behavior under negation).
- `special/error_functions.py` — `TrigonometricIntegral` (base for `Si`, `Ci`, `Shi`, `Chi`), `FresnelIntegral` (base for `fresnels`/`fresnelc`), `Ei`, `expint` (generalized exponential integral E_ν(z); branch cut handling: integer order uses factorial correction, non-integer uses gamma function phase factor), `li`, `Li`; Fresnel integrals with `taylor_term` recurrence using previous terms.
  - `erf`, `erfc`, `erfi` (imaginary error function), `erf2`, `erfinv`, `erfcinv`: Gaussian error function family; `erfi.eval` simplifies composed inverse forms.
- `special/zeta_functions.py` — `lerchphi` (Lerch transcendent Φ(z,s,a) with `_eval_expand_func`: reduces to polylog/zeta sums when a is rational), `polylog`, `zeta`, `dirichlet_eta`, `stieltjes`.
- Caveats: `special/polynomials.py` defines symbolic classes; `polys/orthopolys.py` has computational generators.

### [`simplify/`](simplify/catalog.md)
Expression transformation and simplification algorithms. Operates ON functions, does not define them.
- `fu.py` — trigonometric simplification rules: `TR3` (induced formula: sign simplification + co-function swap for angles between π/4 and π/2 via complementary-angle identity), `TR5`/`TR6` (Pythagorean substitution sin²↔1−cos², cos²↔1−sin²; `_TR56` helper with `max`/`pow` controls for exponent threshold and power-of-2 restriction).
  - `TR10` (expand sin/cos of sums into products; recursive peeling for 3+ term args), `TR10i` (product-to-sum with sqrt(3):1 ratio handling), `TR11` (double-angle to half-angle reduction); `hyper_as_trig`.
  - `TR12` (separate tan of sums into nested addition-formula ratios; first call orders args, recursive calls preserve original order), `TR12i` (inverse: combine tan quotients back).
  - `TR111`: convert negative powers of sin/cos/tan to positive powers of reciprocal co-functions (csc/sec/cot); leaves cot/csc/sec bases unchanged.
- `hyperexpand_doc.py` — auto-generated documentation module; builds `__doc__` at import by iterating `FormulaCollection` and rendering each generalized series closed-form identity as a `.. math::` LaTeX equation.
- `hyperexpand.py` — `hyperexpand`, `hyperexpand_special` (closed-form evaluation of 2F1 at z=±1 via Gauss/Kummer summation; Kummer branch dispatches on negative-integer upper parameter for alternate cosine formula), `ReduceOrder`, `add_formulae` (lookup table of hypergeometric identities), `add_meijerg_formulae` (Meijer G-function formula table with matcher functions), `_meijergexpand` (Slater expansion; convergence: upper params < lower params → unconditional, equal → |z|<1, else fails).
  - `devise_plan_meijer`: determine shift-operator ordering to convert between Meijer G-functions with integer-differing parameters.
  - `_mod1`: parameter remainder mod 1 for bucket classification; numeric values use Mod directly, symbolic expressions split into numeric coefficient + symbolic remainder to preserve parameter information.
  - `try_polynomial`: detect terminating hypergeometric series (non-positive integer upper params); returns ∞ if lower params also contain non-positive integers (divergence).
  - `Hyper_Function.build_invariants`: computes invariant vector by bucketing parameters mod 1; skips sorting when bucket keys are symbolic (`Mod` instances).
  - `Hyper_Function.difficulty`: estimates transformation steps between hypergeometric functions via residue-class bucket comparison; returns -1 if impossible.
  - Shift operators (`ShiftA`, `ShiftB`); unshift operators (`MeijerUnShiftA`–`MeijerUnShiftD`: raise ValueError when auxiliary polynomial constant term cancels).
- `trigsimp.py` — `trigsimp_groebner`: Gröbner-basis trig/hyperbolic simplification (substitutes I with Dummy, adds I²+1 to ideal); `exptrigsimp` (convert exponentials to hyperbolic/trig: detects reciprocal exp pairs for sinh/cosh, substitutes (e^a−1)/(e^a+1) ratios for tanh/tan).
- `ratsimp.py` — `ratsimpmodprime`: simplify rational expressions modulo a Gröbner basis ideal.
- `radsimp.py` — `fraction` (extract numerator/denominator; `exact` flag: symbolic negative exponents stay in numerator), `rad_rationalize`.
  - `collect`: group additive terms by pattern (supports derivatives, exact-match flag); nested `parse_term` decomposes bases/exponents including `exp()` (rational arg → E base, product arg → split coeff from tail). `collect_sqrt` (groups by second-order radicals AND imaginary unit; unevaluated mode returns term tuple + radical count; no radicals found → collapses to single sum), `collect_const`.
- `combsimp.py` — `combsimp` (combinatorial simplification including gamma products), `_rf` (internal rising factorial evaluator: expands Pochhammer symbol for integer second arg — positive→product, negative→reciprocal of product; non-integer splits via as_coeff_Add recurrence. NOT `functions/combinatorial/factorials.py` `RisingFactorial` which is the symbolic unevaluated class).
  - `rule_gamma`: reflection identity γ(x)γ(1−x)=π/sin(πx) — skips integer gamma args to avoid sin(nπ)=0 in denominator; duplication theorem for γ(2s)/γ(s).
- `powsimp.py` — `powsimp`, `powdenest`: power/exponent simplification.
  - `_denest_pow`: denest nested powers; exp with log-containing product exponent → separates log from non-log factors, combines logs, restructures as base^remaining.
- `simplify.py` — `simplify()`: general-purpose dispatch; `logcombine`; `separatevars`; `bottom_up`; `nthroot`; `sum_add`; `product_mul` (helper for Product simplification: method 0 merges same-limits products, method 1 merges adjacent-range products only if same index variable); `_real_to_rational` (replace Float atoms with exact Rational; non-Rational nsimplify result for negative floats → negate, compute power-of-10 scale, convert via string to avoid precision loss).
  - `signsimp`: canonicalize Add signs via sub_pre/sub_post; `evaluate=True` restores original if no net change to avoid hollow transformations.
  - `clear_coefficients`: strip rational additive/multiplicative prefactors from expression, applying inverse ops to RHS; iterates content_primitive + coeff_Add until stable.
  - `besselsimp` (Bessel simplification: half-integer order→trig via spherical rewrite, imaginary arg rewrites between J/I types).
- `cse_main.py` — `cse` (common subexpression elimination), `opt_cse` (pre-optimization: extracts shared args between Add/Mul pairs; asymmetric handling when first expr becomes empty).
- `traversaltools.py` — `use`: apply a function at a specified depth in the expression tree; returns atoms unchanged if target depth not reached.
- `sqrtdenest.py` — `sqrtdenest` (denest nested square roots), `is_algebraic` (check if expression is built from rationals, integer powers, and root extraction only), `sqrt_depth`.
- `epathtools.py` — `EPath`: XPath-like tool for selecting and transforming expression sub-trees.

### [`printing/`](printing/catalog.md)
String/code representation of SymPy expressions. Outputs text, NOT callable code or AST nodes.
- `preview.py` — `preview`: render expression to PNG/DVI/PS/PDF/SVG via external LaTeX; viewer dispatch: `pyglet` only supports PNG (raises SystemError for other formats), `file`/`BytesIO` return raw output.
- `ccode.py` — `CCodePrinter`, `ccode`: C expression printing; `known_functions` maps Abs→fabs with conditional guard (not x.is_integer), other math functions mapped as simple strings; `_print_For` (only supports Range iterables, raises NotImplementedError for lists/tuples).
- `fcode.py` — `FCodePrinter`, `fcode`, `indent_code`: Fortran expression printing.
- `lambdarepr.py` — `LambdaPrinter`, `NumExprPrinter` (numexpr string backend; blacklists matrices/collections), `_print_Piecewise`.
- `python.py` — `python()`: generate executable Python code string for an expression.
- `latex.py` — `LatexPrinter`, `latex()`: LaTeX representation.
- `jscode.py` — `JavascriptCodePrinter`, `jscode`: JavaScript expression printing; `_print_MatrixElement` (row-major 2D→1D index flattening).
- `codeprinter.py` — `CodePrinter`: base class for all code printers; `_doprint_loops` (loop code for indexed expressions; initializes to zero when all terms have contractions).
  - `doprint`: accepts `assign_to` as Symbol/MatrixSymbol/string; `human=True` → formatted string, `human=False` → (number_symbols, not_supported, code) tuple.
  - `_get_expression_indices`: extract/validate lhs/rhs index sets; scalar broadcast: copies lhs indices to rhs when rhs has no free indices but lhs does.
  - `_sort_optimized` (score-based loop nesting for indexed access; innermost = highest score via `_rate_index_position`).
  - `_print_Mul` (splits numerator/denominator; `evaluate=False` for non-(-1) negative rational exponents to prevent simplification).
- `mathml.py` — `MathMLPrinter`: XML content MathML output; `_print_Integral` (recursive nesting for multi-variable integrals; 2-element limit tuple → upper bound only, 3-element → both bounds).
- `llvmjitcode.py` — `llvm_callable`: JIT-compile expressions to machine code via LLVM.
- `precedence.py` — bracket-necessity system; `precedence_PolyElement` (4-way dispatch: generator→Atom, ground→delegate, term→Mul, multi-term→Add), `precedence_FracElement`.
- `pretty/pretty.py` — `PrettyPrinter`: 2D human-readable output; `_print_meijerg` (4-parameter 2×2 grid with annotated G symbol), `_print_hyper`, `_print_Integral`, `_print_Matrix`.
- `defaults.py` — `DefaultPrinting` mixin: aliases `__repr__` to `__str__` so elements in Python lists/dicts display in human-readable form; forces default (lex) ordering regardless of global setting.
- `str.py` / `repr.py` — default `str()` / `repr()` printers; `_print_Pow` (uses identity `is` checks, not `==`, to avoid matching -0.5 as -S.Half); `_print_FiniteSet` truncates sets >10 elements; `_print_DMP` (dense polynomial: tries ring.to_sympy conversion, falls back to raw `cls(rep, dom, ring)` format on SympifyError).
- `octave.py` — `OctaveCodePrinter`: Octave/MATLAB code; rewrites spherical Bessel via cylindrical Bessel.
- `julia.py` — `JuliaCodePrinter`: Julia code; restructures `Piecewise` assignments in non-inline mode.
- `theanocode.py` — `TheanoPrinter`: Theano graph builder; `_print_Piecewise` uses `np.nan` fallback for single-branch.

### [`solvers/`](solvers/catalog.md)
Equation solving: algebraic, ODE, PDE, recurrence, diophantine, systems.
- `solvers.py` — `solve`, `_solve`, `unrad`, `solve_linear_system`, `solve_undetermined_coeffs`, `check_assumptions`, `sub_func_doit`.
  - `_invert`: recursively invert algebraic ops to isolate symbol-dependent parts; returns (indep, dep) tuple; no matching free symbols → (expr, 0). NOT `Expr.as_independent` (which splits terms without inversion).
  - `det_quick`: determinant dispatch — all-symbolic + <8 rows → permutation-based `det_perm`, mixed symbolic → cofactor `det_minor`, pure numeric → `Matrix.det`. Also `inv_quick`.
- `solveset.py` — `solveset`, `linsolve`, `linear_eq_to_matrix`: new-style set-based solver API; `_invert_real` (invert power/trig/exp expressions; even-numerator rational exponents yield both ± roots).
  - `_solve_trig`: rewrite trig equation as exponentials, substitute exp(I*x)→dummy; falls back to ConditionSet if substitution doesn't eliminate the original symbol.
- `recurr.py` — `rsolve`, `rsolve_poly`, `rsolve_hyper`: linear recurrence equation solvers.
- `ode.py` — ODE classification and solution: `classify_ode`, `dsolve`, `_frobenius`, `ode_2nd_power_series_ordinary`, Lie group methods, `odesimp`; `_linear_neq_order1_type1` (system of n first-order constant-coefficient linear ODEs via eigenvalue/eigenvector; constructs generalized eigenvectors for deficient eigenspaces); `homogeneous_order` (determine degree of homogeneity; nested Function assumed order 0); `constantsimp` (simplify arbitrary integration constants in ODE solutions; `_conditional_term_factoring` reverts GCD factoring when exponential appears both as standalone factor and inside additive sub-expression), `constant_renumber`.
  - `infinitesimals`: compute Lie symmetry group tangent vectors ξ,η for first-order ODEs; cycles through multiple heuristic strategies until one succeeds.
- `diophantine.py` — `diophantine`, `sum_of_four_squares`, `power_representation` (enumerate k-tuples whose p-th powers sum to n; Fermat's Last Theorem guard: k=2, p>2, n is perfect power → immediate empty return), `PQa` (periodic expansion of (P+√D)/Q for Pell equations), `transformation_to_DN`/`_transformation_to_DN` (convert general ax²+bxy+cy²+dx+ey+f=0 to X²−DY²=N via recursive affine transforms; each recursion composes transformation matrices): integer/diophantine solving.
  - `prime_as_sum_of_two_squares`: Cornacchia-style decomposition; p%8==5 uses b=2 directly, p%8==1 searches for quadratic non-residue.
- `bivariate.py` — `bivariate_type`: solve equations with two variables via back-substitution; `_filtered_gens` (extract symbol-dependent generators from a polynomial, deduplicating multiplicative inverses by preferring denominator-free form).
- `inequalities.py` — `solve_poly_inequality`, `solve_rational_inequalities`, `reduce_abs_inequality` (nested absolute-value decomposition into piecewise cases via Cartesian product of branches).
- `pde.py` — `pde_separate` (variable separation for PDEs; `strategy` param selects additive/multiplicative; bug: unknown strategy silently passes due to `assert ValueError` instead of `raise`), `pde_separate_add`, `pde_separate_mul`; `_separate` (two-pass internal routine: first pass extracts derivative coefficients as divisors and divides equation — FIXME: sums divisors instead of computing LCM; second pass splits normalized equation into lhs/rhs by variable dependence).
- `decompogen.py` — `decompogen`: general functional decomposition of expressions; detects algebraic structure in transcendental generators (e.g. sin(x)²+sin(x)+1 → [x²+x+1, sin(x)]) by converting to Poly and filtering symbol-dependent generators.
- `polysys.py` — `solve_generic` (Gröbner-based polynomial system solver), `solve_biquadratic`, `solve_triangulated` (Gianni-Kalkbrenner: iterative back-substitution with algebraic field extension for non-rational roots).
- `deutils.py` — `ode_order`: utility to determine the order of a differential equation.

### [`utilities/`](utilities/catalog.md)
Utility functions: numeric code generation, iterables, source inspection, multiset enumeration.
- `lambdify.py` — `lambdify`, `lambdastr`: convert expressions to callable Python/NumPy functions; `_import` (module namespace builder: `Abs` maps to `mpmath.fabs` for mpf return type, but builtin `abs` for math/numpy).
- `autowrap.py` — `autowrap`, `ufuncify`, `CythonCodeWrapper._partition_args`: compile to binary.
- `codegen.py` — `Routine`, `CCodeGen`, `FCodeGen`, `OctaveCodeGen`: generate Fortran/C/Octave source files; `FCodeGen.dump_f95` (raises CodeGenError when variable names collide under case-insensitive comparison — Fortran ignores case); `Variable` (typed variable with `get_datatype` for language-specific type lookup; raises error listing supported languages on unknown language).
  - `OutputArgument` (write-back parameter; multiple inheritance from `Argument` + `ResultBase`, explicitly calls both parent `__init__`s for name/type and expr/result_var), `InputArgument`, `InOutArgument`.
- `iterables.py` — `flatten` (recursively denest nested containers; checks `args` attribute on reducible elements to extract children of tree-structured symbolic objects before recursing; `cls` filter restricts to specific types; `levels` controls depth), `ibin` (integer-to-binary-list; non-integer `bits` arg switches to enumerating all binary sequences of given length via `variations`), `multiset_combinations` (unique combinations from multiset; unhashable elements → TypeError fallback to sort+group instead of dict-based frequency counting), `partitions`, `generate_bell`, `generate_involutions`, `generate_derangements`, `generate_oriented_forest`, `_set_partitions`, `topological_sort`, `numbered_symbols`, `minlex`.
  - `variations`: n-sized ordered selections from a sequence; without repetition: empty generator when n > len(seq); with repetition: Cartesian product. Also `subsets`.
  - `runs`: group sequence into monotonic sublists by comparison operator (default `gt`); returns `[]` for empty input.
  - `ordered_partitions`: integer partitions in lexicographic order; when `m` given, yields lists in-place (caller must copy to avoid duplicates).
  - `interactive_traversal`: user-guided step-by-step navigation through expression tree with re-prompt on invalid input.
- `randtest.py` — `verify_numerically` (test symbolic equivalence by substituting random complex values for all free symbols), `test_derivative_numerically`; `_randint`, `_randrange`: deterministic pseudo-random generators.
- `misc.py` — `replace` (simultaneous multi-pattern text substitution, longer keys matched first), `translate` (character-level replacement/deletion), `_replace` (regex-compiled helper).
  - `rawlines`: convert multiline string to pasteable repr; chooses parenthesized per-line format vs triple-quoted dedent based on trailing whitespace/backslash/quote presence.
- `decorator.py` — `threaded_factory` (decorator: maps function over iterables/matrices; silently returns input unchanged if container constructor rejects list), `threaded`, `xthreaded`.
- `enumerative.py` — `MultisetPartitionTraverser`: multiset partition enumeration (Knuth's algorithm); `enum_all`, `enum_small`, `enum_large`, `enum_range` (bounded part-count enumeration combining upper+lower constraints; upper-bound exceeded during spread → sets lpart=ub−2 to trigger backtrack); `factoring_visitor` (interpret partition state + prime bases to enumerate integer factorizations), `list_visitor`.
- `benchmarking.py` — `BenchSession`: py.test-based performance measurement; `print_bench_results` formats timing output with decimal-point alignment across time-unit columns (s/ms/μs/ns).
- `runtests.py` — `_doctest` (internal doctest runner; conditionally extends file blacklist based on missing optional libraries like numpy/matplotlib/pyglet/theano), `SymPyOutputChecker`: test runner with float comparison, matplotlib backend management.
  - `SymPyDocTestFinder._find`: recursive doctest discovery in modules/classes; property accessors checked via `val.fget.__module__` for module membership (unlike functions which use `val.__module__` directly).

### [`codegen/`](codegen/catalog.md)
AST node definitions for code generation (abstract syntax tree). NOT source file generation (that's `utilities/codegen.py`).
- `ast.py` — `Assignment`, `AugmentedAssignment` (+=, -= etc.), `Variable`, `Declaration`, `FunctionPrototype`.
  - `CodeBlock`: `topological_sort` (reorder assignments so variables are defined before use; raises ValueError if RHS references undefined variable); `.cse` (CSE on code blocks; rejects AugmentedAssignment and duplicate LHS).

### [`assumptions/`](assumptions/catalog.md)
Property inference system for symbolic objects (is_positive, is_integer, etc.).
- `ask.py` — `Q` predicate definitions (`Q.transcendental`, `Q.algebraic`, `Q.prime`, `Q.composite`, `Q.hermitian`, `Q.commutative`, `Q.triangular` (true if upper OR lower triangular), `Q.orthogonal`, `Q.unitary`, `Q.singular`, `Q.normal`, `Q.unit_triangular`, etc.), `ask()` query function; `get_known_facts` (complete lattice of logical implications/equivalences between mathematical properties, e.g. integer→rational→algebraic→complex). Predicate property definitions live HERE, not in `ntheory/` or `core/`.
- `assume.py` — `AppliedPredicate` (result of `Q.prop(expr)`; `__eq__` uses strict `type(other) is AppliedPredicate` — subclass instances won't match), `Predicate.eval`: handler dispatch with contradiction detection; `assuming` context manager.
- `refine.py` — `refine`, `refine_atan2`, `refine_Pow`: simplify expressions under assumptions.
- `handlers/order.py` — `AskNegativeHandler`, `AskPositiveHandler`, `AskNonNegativeHandler`: sign/ordering inference for `Add`, `Mul`, `Pow`; `Add` handler counts nonpositive terms to determine strict negativity.
- `handlers/calculus.py` — `AskFiniteHandler`: boundedness inference for `Add`, `Mul`, `Pow` (|base|≥1 + unbounded exp → unbounded), `log`, `exp`.
- `handlers/sets.py` — `AskImaginaryHandler`, `AskRealHandler`, `AskIntegerHandler`: set-membership queries.
  - `AskHermitianHandler` (Mul: tracks nc-factor count, >1 nc → None; XOR toggles result for antihermitian args).
  - `AskAntiHermitianHandler` (Mul: same nc-factor limit and XOR parity as Hermitian but starts False; Pow: even exp→False, odd→True for antihermitian base).
- `handlers/common.py` — `TautologicalHandler` (evaluate boolean expressions under assumptions: `Implies` rewrites p→q as ¬p∨q before evaluation; `Not`/`Or`/`And` use three-valued logic).
  - `AskCommutativeHandler` (defaults to True unless explicitly non-commutative), `CommonHandler` (`AlwaysTrue`/`AlwaysFalse`).
- `handlers/ntheory.py` — `AskEvenHandler` (Mul: sum-parity deduction; tracks irrational factors), `AskOddHandler`, `AskPrimeHandler`: number-theoretic property inference.
  - `AskCompositeHandler` (positive integer that is not prime; explicitly returns False for 1 since it is neither prime nor composite).
- `handlers/matrices.py` — `AskSquareHandler`, `AskSymmetricHandler` (MatMul: recursive outer-factor transpose matching to determine self-transpose of products), `AskInvertibleHandler`, `AskOrthogonalHandler`, `AskUnitaryHandler`, `AskDiagonalHandler`, `AskUpperTriangularHandler`, `AskLowerTriangularHandler` (Transpose handler checks dual triangularity of inner arg): matrix property inference.
- `sathandlers.py` — `register_fact`: register assumption rules for SAT-based inference; `AllArgs`, `AnyArgs`, `ExactlyOneArg` (vectorize predicates over expression args; ExactlyOneArg builds Or-of-And-of-Not clauses using ordinary disjunction, not XOR).
- NOT propositional logic (that's `logic/`). This module infers numeric properties of expressions.

### [`logic/`](logic/catalog.md)
Propositional and boolean logic: representation, inference, satisfiability.
- `boolalg.py` — `And`, `Or`, `Not`, `Implies`, `Equivalent`: boolean algebra classes.
- `algorithms/dpll2.py` — `SATSolver`, `_vsids_calculate`: DPLL-based SAT solver.
- `inference.py` — `satisfiable`, `valid`, `entails`: logical inference functions; `PropKB` (propositional knowledge base: `tell`/`ask`/`retract` manage clause sets via CNF decomposition; retract discards individual conjuncts).
- `utilities/dimacs.py` — `load` (DIMACS CNF format parser: negative literals → negated symbols, zero as clause terminator, lines as disjunctions conjoined into And), `load_file`.
- NOT assumption handlers (those are in `assumptions/`).

### [`physics/`](physics/catalog.md)
Physics subpackages: quantum mechanics, classical mechanics, optics, units, second quantization.
- `secondquant.py` — `Dagger` (Hermitian conjugate: reverses factor order over products; NOT matrix adjoint in `matrices/expressions/adjoint.py`).
  - `AntiSymmetricTensor`: antisymmetric two-electron integral; `_sortkey` (canonical index ordering: anonymous/dummy indices get higher sort priority than named indices).
  - Creation/annihilation operators, `Commutator`, Wick's theorem.
  - `evaluate_deltas`: simplify KroneckerDelta in products under Einstein summation; respects fermi-level index priority and equal-information checks.
  - `substitute_dummies`: canonicalize summation (dummy) indices across additive terms; handles cyclic swaps via temporary placeholder symbols to avoid clobbering.
  - `simplify_index_permutations`, `PermutationOperator`.
- `mechanics/functions.py` — `inertia` (construct inertia dyadic from frame + 6 components; TypeError if frame arg invalid), `msubs`, `angular_momentum`, `kinetic_energy`.
  - `_smart_subs`: intelligent substitution; selectively simplifies only subexpressions whose denominators become zero.
- `vector/frame.py` — `CoordinateSym` (coordinate base scalar symbol associated with a ReferenceFrame; `__new__` validates index in range 0–2, raises ValueError for out-of-range dimension number), `ReferenceFrame`: `__getitem__` (dual dispatch: numeric→coord var, string→basis vector), `orient` (Body/Space/Quaternion/Axis).
  - `set_ang_vel`/`set_ang_acc`: store bidirectionally with negated reverse; wraps scalar 0 to Vector(0).
  - `_w_diff_dcm`, `variable_map`.
- `vector/vector.py` — `Vector`: spatial directional quantity; `diff` (partial derivative w.r.t. variable in a frame; optimizes by differentiating scalar coefficients only when the DCM between component frame and target frame is independent of the variable, otherwise re-expresses first), `separate` (decompose into per-ReferenceFrame constituent dict), `doit` (applyfunc + lambda to propagate **hints to each scalar coefficient), `subs` (substitution iterates over all (coefficient, frame) pairs, applying subs to each coefficient while preserving frames), `to_matrix`, `dot`, `cross`, `magnitude`, `normalize`.
- `vector/dyadic.py` — `Dyadic`: second-rank tensor (outer product of vectors); `__and__`/`dot` (dispatches: Dyadic×Dyadic → Dyadic via pairwise inner products, Dyadic×Vector → Vector via right-contraction); `__eq__` (set-based comparison of internal component tuples; documented as weak/limited), `__str__`, `_latex`.
- `vector/printing.py` — `VectorStrPrinter` (_print_Derivative: dot-notation only for UndefinedFunction; defined functions like sin/cos fall back to standard StrPrinter), `VectorLatexPrinter`, `VectorPrettyPrinter`: Newton dot notation for time-derivatives.
  - `init_vprinting`: globally enables compact temporal derivative display (e.g. f') across all output formats.
- `quantum/gate.py` — qubit gate operators: `HadamardGate`, `XGate`, `YGate`, `ZGate`, `PhaseGate`, `TGate`, `SwapGate`, `CNotGate`, `CGate`, `UGate`.
  - `normalized`: global toggle for Hadamard 1/√2 scaling factor in matrix representation. Also `gate_sort`, `gate_simp`, `random_circuit`.
- `quantum/cg.py` — `Wigner3j` (3j coupling coefficients with own `_pretty`/`_latex` methods for 2D grid layout), `CG` (Clebsch-Gordan coefficients, subclass of Wigner3j), `Wigner6j` (6j recoupling coefficients; `_pretty` renders 2×3 grid with curly brace delimiters, column-aligned padding), `Wigner9j`.
- `quantum/hilbert.py` — Hilbert space algebra: `DirectSumHilbertSpace`, `TensorProductHilbertSpace`, `TensorPowerHilbertSpace`, `ComplexSpace`, `L2` (square-integrable function space; validates Interval input type), `FockSpace`.
- `quantum/shor.py` — Shor's quantum factoring algorithm: `shor`, `period_find` (quantum order-finding via QFT).
- `quantum/qasm.py` — QASM circuit description parser: `Qasm`, `get_index` (reverses register declaration order to qubit indices via `flip_index`).
- `quantum/constants.py` — `HBar` (reduced Planck constant singleton; `_pretty` dispatches Unicode ℏ glyph vs ASCII 'hbar' fallback).
- `quantum/qexpr.py` — `QExpr`: base class for quantum expressions; `_qsympify_sequence` (argument normalizer: strings → Symbol instead of sympify, so names like 'pi' stay as abstract placeholders); `_eval_adjoint` (fallback creates Dagger wrapper, preserves Hilbert space attribute).
- `quantum/represent.py` — `represent`: convert symbolic quantum expressions to matrix form; fallback dispatch (state vectors → inner product, operators → expectation value) when direct `_represent` is not implemented.
- `quantum/state.py` — states (`Ket`, `Bra`, `Wavefunction`: `__call__` returns 0 when evaluated outside coordinate bounds (skips comparison when arg contains bound symbols); `limits` property returns coordinate→bounds dict, defaulting bare symbols to (-∞,∞); `normalize`, `norm`), spin eigenstates, operator sets.
- `quantum/cartesian.py` — Cartesian coordinate operators (`XOp`, `YOp`, `ZOp`) and momentum operators (`PxOp`); position/momentum eigenkets (`XKet`, `PositionKet3D`). YOp/ZOp only dispatch `_apply_operator` to 3D kets, not 1D `XKet`.
- `quantum/matrixcache.py` — `MatrixCache`: caches small matrices in sympy/numpy/scipy.sparse formats; `cache_matrix` silently catches `ImportError` per format, `get_matrix` raises `NotImplementedError` if format unavailable. Pre-populates common gate matrices (Pauli, Hadamard, SWAP, etc.) at module level.
- `quantum/anticommutator.py` — `AntiCommutator`: symmetric bracket {A,B}=AB+BA; eval separates commutative prefactors via args_cnc, enforces canonical argument ordering.
- `quantum/commutator.py` — `Commutator`: Lie bracket [A,B]=AB-BA with scalar prefix extraction and canonical ordering.
- `quantum/density.py` — `Density`: statistical mixture / density operator; `doit` expands to weighted outer products (handles superposition cross-terms).
- `quantum/operator.py` — `Operator`, `HermitianOperator` (`_eval_inverse`: returns self if also unitary, else falls back to Operator inverse), `UnitaryOperator`, `IdentityOperator` (identity simplification in multiplication), `OuterProduct`.
- `quantum/matrixutils.py` — type-dispatch conversions between sympy Matrix, numpy ndarray, scipy sparse (stub classes when libraries unavailable); `flatten_scalar` (reduce 1×1 matrix to plain scalar value), `sympy_to_numpy` (Expr that isn't Number/NumberSymbol/I raises TypeError), `_scipy_sparse_tensor_product` (Kronecker product; converts final result to CSR format for downstream multiplication), `matrix_tensor_product`.
- `quantum/tensorproduct.py` — quantum Kronecker products and simplification (NOT `tensor/`).
- `quantum/qapply.py` — `qapply` (symbolic operator application: distributes over Add, recurses into TensorProduct/Pow/Density).
  - `qapply_Mul`: pairwise dispatch — tries lhs._apply_operator(rhs), then rhs._apply_operator(lhs), then Bra+Ket→InnerProduct; `dagger` retries via Dagger.
- `quantum/identitysearch.py` — gate identity BFS discovery; `GateIdentity` (precomputes equivalent permutations), `is_degenerate` (checks permutation membership in known identity sets); `lr_op`, `ll_op` rule operations.
- `quantum/circuitutils.py` — `convert_to_symbolic_indices` (replace integer position labels with symbolic placeholders; maintains bijective map), `kmp_table`, `find_subcircuit` (KMP pattern matching for gate sequences), `replace_subcircuit` (locate and swap/remove gate subsequences; `replace=None` defaults to empty tuple, effectively deleting the matched pattern), `random_reduce`.
- `quantum/circuitplot.py` — circuit diagram visualization (NOT `plotting/`); `Mz`, `Mx` (mock measurement gates for drawing only).
  - `CreateOneQubitGate` (metaclass factory for dynamic gate classes), `CreateCGate` (closure-based controlled gate factory wrapping a dynamically-generated gate class).
- `vector/functions.py` — `get_motion_params` (compute acceleration/velocity/position tuple from any one input via integration/differentiation with boundary conditions; zero-vector input short-circuits to (0, 0, condition)).
  - `partial_velocity` (compute partial derivatives of velocity/angular-velocity vectors w.r.t. generalized speeds in a frame; `var_in_dcm=False` ignores direction cosine matrix dependence on speed variables).
  - `time_derivative` (transport theorem: cross-frame components add ω×v term), `express`, `outer`, `kinematic_equations`, `dynamicsymbols`.
- `vector/point.py` — `Point`: kinematic point; `_pdict_list` (BFS graph search for shortest chain of position/velocity/acceleration relationships between two points; raises ValueError if no connecting path), `locatenew`, `pos_from`, `vel`, `acc`, `a1pt_theory`, `v1pt_theory`.
- `vector/fieldfunctions.py` — `divergence` (zero vector → scalar S(0)), `curl` (zero vector → Vector(0)), `gradient`; `scalar_potential`, `scalar_potential_difference` (potential difference between points; uses scalar value directly if input is not a vector), `is_conservative` (auto-extracts frame via `separate()`; zero vector→True), `is_solenoidal`.
- `optics/waves.py` — `TWave`: transverse sine wave; `__init__` validates frequency/time_period consistency.
  - `__add__`: superposition — same-frequency → resultant amplitude via law of cosines + phase via atan2; different frequencies → NotImplementedError.
- `optics/utils.py` — `lens_formula` (thin-lens equation solver; handles infinite distances via symbolic Limit to avoid division-by-zero), `mirror_formula`, `hyperfocal_distance`.
- `sho.py` — 3D isotropic harmonic oscillator: `R_nl` (radial wavefunction; increments user's n≥0 to formula's n≥1), `E_nl` (energy levels).
- `mechanics/linearize.py` — `Linearizer`: state-space linearization of mechanical systems; `linearize` returns A, B matrices (empty B if no forcing variables); `permutation_matrix` (construct reordering matrix mapping one symbol arrangement to another; validates both vectors contain same symbol set).
- `hep/gamma_matrices.py` — Dirac gamma matrix traces and simplification for high-energy physics.
- `wigner.py` — Wigner 3j/6j/9j, Clebsch-Gordan, Gaunt coefficients (spherical harmonic integrals).
- `units.py` — legacy physical units; `Unit` class with `is_positive=True`.
- `unitsystems/units.py` — `UnitSystem`: `print_unit_base` (express derived unit in basis; sorts by decreasing power, normalizes scale factors), `get_unit`, `extend`.
- `unitsystems/dimensions.py` — `Dimension` (physical quantity characteristic; filters zero-valued exponent entries during construction for equality invariance), `DimensionSystem`: `get_dim` (lookup by string name/symbol or Dimension object; returns None if not found), `extend`, `sort_dims`.
- `unitsystems/quantities.py` — `Quantity`: physical quantity as (factor, unit) pair; `add` (converts other to same unit before combining; raises TypeError for non-Quantity operands), `mul`, `div`, `convert_to`.
- `unitsystems/simplifiers.py` — `dim_simplify`: dimensional analysis simplification.

### [`matrices/`](matrices/catalog.md)
Matrix classes and matrix expression algebra.
- `matrices.py` — `DeferredVector` (symbolic vector for `lambdify`; `__getitem__` raises IndexError on negative index), `MatrixBase`: core matrix operations.
  - `dual`: covariant second-rank tensor from contravariant via Levi-Civita contraction; returns zero matrix if input is symmetric.
  - `D` property: Dirac conjugate (relativistic spinor dual); only defined for 4-row matrices — raises AttributeError (not ShapeError) for non-4-row inputs due to Python property constraints.
  - Linear system solvers: `gauss_jordan_solve` (RREF row-reduction on augmented matrix with parametric tau-symbol free variables for underdetermined systems; raises ValueError on inconsistent systems; NOT `polys/solvers.py` which operates over polynomial rings), `LUsolve`, `cholesky_solve`, `QRsolve`, `pinv_solve`.
- `expressions/blockmatrix.py` — `BlockMatrix`, `block_collapse`; `bc_matmul` (pairwise block multiply: wraps non-block neighbor in 1×1 BlockMatrix when only one factor is block).
- `dense.py` — `MutableDenseMatrix`, `wronskian`, `hessian`, `casoratian` (Casoratian determinant for recurrence solutions; `zero` flag: True→evaluate at 0,1,2…, False→symbolic offset n,n+1,n+2…); `rot_axis1`/`rot_axis2`/`rot_axis3` (3D rotation matrices; axis-2 has negative sine in transposed position vs axes 1,3); `matrix_multiply_elementwise` (Hadamard/entry-by-entry product; validates matching shapes before computing); `symarray` (numpy ndarray of Symbols with index-derived names like `prefix_i_j`; forwards kwargs to Symbol for assumptions like real=True).
- `sparse.py` — `SparseMatrix`: dictionary-backed sparse matrix; `add` (zero-entry cleanup on cancellation), `_LDL_solve` (L·D·L^T forward/diagonal/back-solve), `_cholesky_solve`.
- `densetools.py` — list-of-lists dense matrix operations: `trace`, `transpose`, `conjugate`/`conjugate_row` (element-wise; catches `AttributeError` for entries lacking `.conjugate()`, falls back to raw value), `conjugate_transpose`.
- `immutable.py` — `ImmutableMatrix`, `_eval_Eq`: immutable matrix with equality support.
- `expressions/determinant.py` — `Determinant`, `det`; `refine_Determinant` (orthogonal→1, singular→0, unit_triangular→1).
- `expressions/inverse.py` — `Inverse` (symbolic matrix inverse, subclass of MatPow); `refine_Inverse` (orthogonal→transpose, unitary→conjugate, singular→error).
- `expressions/transpose.py` — `Transpose`, `refine_Transpose`: symbolic transpose operations.
- `expressions/matmul.py` — `MatMul`, `_eval_trace`: symbolic matrix multiplication and trace; `merge_explicit` (combine adjacent concrete MatrixBase operands; non-adjacent concrete matrices separated by symbolic operand are NOT merged), `xxinv`, `remove_ids`.

### [`vector/`](vector/catalog.md)
New-style 3D vector algebra module (sympy.vector). NOT physics.vector.
- `point.py` — `Point.position_wrt`: position vector between points in coordinate systems.
- `coordsysrect.py` — `CoordSysCartesian`: coordinate system definition; `scalar_map` (express one frame's coordinate variables in another's, combining rotation and translation).
- `functions.py` — `express`, `matrix_to_vector`, `scalar_potential`, `scalar_potential_difference` (potential difference between points; scalar input used directly without computing potential from vector field); `_path` (find traversal route between two nodes in parent-child hierarchy via lowest common ancestor; raises ValueError if no shared root).
- `deloperator.py` — `Del`: vector differential operator (nabla); `gradient` (always creates Derivative instances), `dot` (divergence; uses `_diff_conditional` which skips differentiation when component is independent of coordinate variable, returning zero directly), `cross` (curl).
- `orienters.py` — `QuaternionOrienter`, `BodyOrienter`, `SpaceOrienter`, `AxisOrienter`: coordinate system orientation transformers.
- Caveats: Separate from `physics/vector/` which uses `ReferenceFrame`-based classical mechanics vectors. `scalar_map` is here, NOT in `physics/vector/frame.py`.

### [`stats/`](stats/catalog.md)
Probability and statistics: random variables, distributions, expected values.
- `crv_types.py` — continuous distribution classes: `BetaDistribution`, `GammaDistribution`, `GammaInverseDistribution` (inverse gamma: x^(-α-1)·exp(-β/x) density), `KumaraswamyDistribution`, `WeibullDistribution` (power-law × stretched exponential; `sample` delegates to `random.weibullvariate`), `VonMisesDistribution`, etc.
- `crv.py` — `ContinuousPSpace` (`probability`: univariate via `where`; multivariate fallback computes density of lhs-rhs and reduces to univariate), `SingleContinuousDistribution`, `_inverse_cdf_expression`.
- `frv.py` — `FiniteDomain` (`as_boolean`: convert finite sample space to Or-of-And-of-Eq propositional logic), `SingleFiniteDomain`, `FinitePSpace`.
  - `ConditionalFiniteDomain`: restricted discrete event space; `_test` evaluates condition on outcome, falls back to equality LHS==RHS when substitution doesn't yield bool.
- `rv.py` — `RandomSymbol._eval_is_real`, `pspace`, `where`, `sample_iter_lambdify`/`sample_iter_subs` (Monte Carlo sampling).
- NOT special mathematical functions (those are `functions/special/`).

### [`integrals/`](integrals/catalog.md)
Symbolic integration: indefinite, definite, transforms, Risch algorithm, Meijer-G methods.
- `integrals.py` — `Integral` class, `Integral.transform`: variable substitution in definite integrals; `line_integrate` (scalar path integral along a Curve).
- `manualintegrate.py` — step-by-step antidifferentiation: `integral_steps`, `manualintegrate`; trig rules (`trig_sincos_rule`, `trig_tansec_rule`, `trig_cotcsc_rule`) with reciprocal-form normalization before pattern matching.
- `transforms.py` — `HankelTypeTransform`, Fourier/Laplace/Mellin/Hankel/sine/cosine integral transforms; `cosine_transform`, `sine_transform` (unitary half-line spectral decomposition with sqrt(2/π) scaling).
  - `_rewrite_gamma`: convert gamma/trig factors to Meijer G-function parameters; gamma multiplication theorem step requires integer coefficient on integration variable (raises TypeError for non-integer like √2).
- `deltafunctions.py` — `change_mul` (rearrange product to bring simple DiracDelta to front; non-simple deltas expanded via `diracdelta=True`; returns `(None, None)` when expansion produces no change), `deltaintegrate`.
- `meijerint.py` — Meijer-G integration helpers: `_get_coeff_exp`, `_check_antecedents_inversion`.
  - `_create_lookup_table`: builds the G-function lookup table mapping standard functions (Ei, Si, Ci, erf, Bessel, etc.) to Meijer G representations; Ei uses `addi` mechanism with constant offset term (−iπ) plus G-function, unlike most entries which use simpler `add`.
  - `_rewrite_single`: rewrite function as sum of C*x^s*G(a*x^b) forms using cached formula lookup tables; short-circuits for G-function input.
- `heurisch.py` — `heurisch` (heuristic Risch indefinite integration), `heurisch_wrapper`; `components` (extract functional building blocks: fractional p/q exponents normalized to base**(1/q), symbolic/irrational exponents kept as-is with full power expression included).
- `rationaltools.py` — `log_to_atan` (complex logarithms → real arctangents via extended GCD), `log_to_real`.
- `quadrature.py` — `gauss_hermite`, `gauss_legendre`, `gauss_laguerre`, `gauss_chebyshev_t`, `gauss_jacobi`: numerical integration nodes/weights; handles `RootOf` (implicit algebraic root) by converting to rational approximation.
- `risch.py` — Risch algorithm core: `risch_integrate`; `DifferentialExtension` (container for transcendental extension tower; manual construction via `extension` dict requires at least key `D`; auto-sets `T`, `t`, `d`, `level`); `derivation` (compute Dp in differential extension tower; `coefficientD` mode differentiates only lower-level extensions treating top variable as constant; returns zero at base level), `gcdex_diophantine` (extended Euclidean for polynomial ideal linear combinations), `frac_in`, `as_poly_1t`, `NonElementaryIntegralException`.
- `trigonometry.py` — `trigintegrate`: antidifferentiation of sin^n*cos^m products; odd exponent→smallest-exponent substitution, even exponents→largest-exponent Pythagorean expansion with recursive reduction.
- `rde.py` — `rischDE` (top-level solver for Dy + f*y = g in differential field extensions; catches `NotImplementedError` from `bound_degree` and falls back to n=∞, which may cause non-termination), `solve_poly_rde` (main dispatcher: degree-of-b vs degree-of-derivation branching selects no-cancel large/small sub-algorithms; small-branch triple return triggers recursive level descent), `order_at` (p-adic valuation; shortcut for p=t via lowest-degree term), `cancel_primitive`, `cancel_exp`, `no_cancel_equal`, `bound_degree`: Risch differential equation solvers.
- `prde.py` — `constant_system` (enforce constant-field solutions by differentiating non-constant RREF entries to generate constraint rows), `is_log_deriv_k_t_radical`: structure theorem test; `real_imag` (separate rational function into real/imag parts at √-1 without substitution, using degree mod 4 sign assignment on a rational expression's numerator and denominator; NOT polynomial-level splitting — see `polys/densetools.py` for that).

### [`series/`](series/catalog.md)
Series expansions, limits, sequences, formal power series, Fourier series, and asymptotic analysis.
- `order.py` — `Order` (big-O notation): `__new__` (handles nested Order expressions; raises NotImplementedError if same variable has different limit point), `_eval_power` (nonneg exponent → raise inner expr; O(1) exponent → identity).
- `limits.py` — `Limit.doit`: compute limits using the Gruntz algorithm; rewrites factorial→gamma only when approach point is positive (factorial is zero for negative inputs unlike gamma); Mul at ∞ uses leading-term substitution trick; falls back to heuristics, then `limit_seq` for sequences.
- `fourier.py` — `FourierSeries`, `fourier_series` (even symmetry → cosine-only, odd → sine-only, skipping unnecessary integrals): precomputed trigonometric series with `scale`, `shift`, `shiftx`, `scalex` (fast coefficient transforms without recomputing integrals).
- `formal.py` — `fps`, `solve_de`, `hyper_re`, `exp_re`: formal power series via DE-to-recurrence conversion.
  - `compute_fps`: public API for coefficient formula; validates direction parameter ('+'/'-'/1/-1, raises ValueError on invalid); returns None if expression is independent of variable.
  - `rsolve_hypergeometric`: solve Q(k)*a(k+m)−P(k)*a(k) recurrences; fractional index offsets → extracts x^frac(j) factor and floors the integer part.
  - `_transform_explike_DE` (convert DE with free parameters to constant coefficients; collects terms and filters out x-dependent ones via for/else before solving linear system); `_transform_DE_RE` (normalize index so lowest term is g(k)); `FormalPowerSeries.integrate` (iterable arg delegates to standard `integrate`).
- `approximants.py` — `approximants`: generator for consecutive Padé approximants from a coefficient list; terminates when all remaining coefficients are zero; normalizes output by LCM of coefficient denominators to clear fractions.
- `limitseq.py` — `difference_delta`: sequence limits and difference operators.
- `sequences.py` — `SeqAdd`, `SeqMul`, `SeqFormula`, `SeqPer`: symbolic sequence algebra with pairwise reduction; `SeqBase.find_linear_recurrence` (discovers shortest recurrence from initial terms via matrix determinant; verifies against remaining terms).
- `kauers.py` — `finite_diff` (polynomial forward difference), `finite_diff_kauers` (forward difference of Sum: substitutes each index variable with its upper bound + 1 by iterating limit tuples).
- Caveats: `formal.py` derives series from DEs; `polys/ring_series.py` does series arithmetic on polynomial rings.

### [`combinatorics/`](combinatorics/catalog.md)
Combinatorics: permutation groups, polyhedra, partitions, free groups, tensor canonicalization.
- `perm_groups.py` — `PermutationGroup`, `derived_series`, `sylow_subgroup`, orbit/stabilizer, `max_div` (largest proper divisor of degree via smallest prime factor from sieve; used by `minimal_block` and `_union_find_merge`).
- `named_groups.py` — `DihedralGroup`, `SymmetricGroup`, `CyclicGroup`, `AlternatingGroup` constructors.
- `free_group.py` — `FreeGroup`, `FreeGroupElement`: word reduction, cyclic reduction in free groups.
- `fp_groups.py` — finitely presented groups, coset enumeration (Todd-Coxeter, Felsch strategies).
- `permutations.py` — `Permutation`: finite bijective mappings; `inversions` (count out-of-order pairs; brute-force for n<130, merge-sort for larger); `__pow__` (rejects non-integer exponent with conjugate hint).
  - `from_sequence`: derives rearrangement mapping from arbitrary comparable items relative to sorted order.
- `subsets.py` — `Subset`: `subset_indices` (locate elements of one list within another; returns [] for empty input or missing elements via for/else), `subset_from_bitlist`, `bitlist_from_subset` (inverse: convert subset to binary string; unwraps Subset object input via `args[0]`), `unrank_binary`, `unrank_gray`; `ksubsets`.
- `tensor_can.py` — `double_coset_can_rep`, `dummy_sgs` (symmetry generators for contracted index pairs): Butler-Portugal canonicalization; `canonical_free` (lexicographic minimization of free indices via slot symmetries; empty base → immediate copy return); `tensor_gens` (BSGS for multiple tensors of same type; early-returns identity when base is empty and fewer than 2 tensors lack fixed indices); `gens_products` (combined BSGS for multiple distinct tensor types via iterative direct product of per-type symmetry groups); `get_minimal_bsgs`, `_is_minimal_bsgs`.
- `util.py` — `_check_cycles_alt_sym` (detect prime-length cycles with n/2 < p < n-2 for alt/sym group recognition), `_base_ordering`, `_distribute_gens_by_base`.
- `testutil.py` — `graph_certificate` (isomorphism-invariant identifier for undirected graphs; encodes vertices as symmetric tensors with edge-paired indices and computes canonical form via `canonicalize`), `_cmp_perm_lists`, `_verify_bsgs`.
- `partitions.py` — `Partition`, `IntegerPartition`: set and integer partition classes; `random_integer_partition` (random partition summing to n; raises ValueError for n < 1).
- `polyhedron.py` — `Polyhedron`: labeled-vertex solid; `__new__` normalizes face tuples by rotation and reversal to lowest lexical ordering (cw and ccw windings map to same face); `rotate` (apply permutation in-place; integer → pgroup lookup with validation, Permutation object → size check only, no physical-validity check), `edges`, `reset`.

### [`concrete/`](concrete/catalog.md)
Concrete mathematics: sums, products, Kronecker delta summation, Gosper's algorithm, sequence guessing.
- `delta.py` — `deltasummation` (sum expressions with KroneckerDelta; multiple solutions for index → returns unevaluated Sum), `deltaproduct`.
- `expr_with_limits.py` — `ExprWithLimits`: base class for Sum/Product/Integral; `as_dummy` (replace bound variables with explicit Dummy placeholders; skips single-element limits as "evaluate at a point"), `_eval_subs`, `free_symbols`, `is_number`.
- `expr_with_intlimits.py` — `ExprWithIntLimits`: discrete (integer-limits) base for Sum/Product; `change_index` (linear variable substitution; only ±1 numeric scaling allowed, symbolic scaling swaps bounds), `index`, `reorder`, `reverse_order`.
- `summations.py` — `Sum`, `eval_sum_hyper`: symbolic summation, hypergeometric closed forms; `is_convergent` (convergence tests including Dirichlet), `is_absolutely_convergent`; `euler_maclaurin` (integral-based asymptotic approximation; reversed bounds a>b → swaps to b+1..a-1 with negated function; a−b==1 → returns zero); `reverse_order` (flip iteration bounds ±1 and negate body).
- `products.py` — `Product`: symbolic products; `is_convergent` (tests infinite product convergence via log→Sum transformation, falls back to |term-1| absolute convergence); `_eval_product` (closed-form evaluation); `product` (lowercase convenience function: creates Product and calls doit; returns non-Product results directly if constructor simplifies).
- `gosper.py` — `gosper_normal` (rational factorization for hypergeometric summation), `gosper_term`, `gosper_sum`.
- `guess.py` — `guess_generating_function`: guess a closed-form generating function from terms.
  - `find_simple_recurrence` (detect linear recurrence from integer/rational sequence; returns symbolic equation with smallest index at n, never n-1), `rationalize`.

### [`ntheory/`](ntheory/catalog.md)
Number theory: primes, residues, continued fractions, factorization, partitions, multinomial coefficients.
- `bbp_pi.py` — `pi_hex_digits` (extract hex digits of π at arbitrary position via BBP formula); `_series` (left sum + right tail that stops when bit-shifted division yields zero).
- `egyptian_fraction.py` — `egyptian_fraction` (unit fraction decomposition: Greedy/Graham-Jewett/Takenouchi/Golomb algorithms; harmonic prefix extraction with early return when remainder is zero).
- `generate.py` — `prime`, `primerange`, `primorial`, `cycle_length`: prime generation and cycle detection.
- `factor_.py` — `factorint` (integer factorization; trial division, Pollard rho, Pollard p-1), `divisors`, `primefactors`, `smoothness`.
  - `perfect_power`: test if n=b^e; finds small divisor, checks exact root, recursively strips factors via GCD of exponents.
- `partitions_.py` — `npartitions`: exact partition count via Hardy-Ramanujan-Rademacher series; `_a` (inner exponential sum with special-case branching for primes 2, 3, and general primes).
- `continued_fraction.py` — `continued_fraction_periodic` (periodic CF expansion of quadratic irrationals (p+√d)/q; normalizes by scaling when (d−p²) % q ≠ 0), `continued_fraction_reduce`, `continued_fraction_iterator`, `continued_fraction_convergents` (successive best rational approximations via Wallis recurrence p=a·p₁+p₂, q=a·q₁+q₂ with seeds 0/1 and 1/0).
- `residue_ntheory.py` — `is_nthpow_residue`, `nthroot_mod`, `primitive_root`, `is_quad_residue`, `discrete_log`; power residue checks for prime powers (odd prime with odd exponent mod 2^k returns True immediately).
- `primetest.py` — `isprime` (deterministic primality test: small-n lookup, trial division, Miller-Rabin, Lucas), `_lucas_sequence` (modular second-order linear recurrence U_k, V_k, Q^k mod n; three code paths: Q==1 skips Q^k tracking, P==1/Q==-1 uses sign toggle, general case tracks full Q^k), `_lucas_selfridge_params`, `mr` (Miller-Rabin witness test).
- `multinomial.py` — `multinomial_coefficients` (main entry: co-lex enumeration; switches to iterator-based strategy when m≥2n and n>1), `multinomial_coefficients0` (Miller Pure Recurrence; m=0 edge case: returns empty dict if n>0, {():1} if n=0; m=2 delegates to binomial), `binomial_coefficients`, `multinomial_coefficients_iterator`.

### [`plotting/`](plotting/catalog.md)
Plotting backends for 2D/3D mathematical visualization.
- `plot.py` — `plot`, `plot3d_parametric_line`, `Plot` class: matplotlib-based plotting; `Parametric2DLineSeries.get_segments` (adaptive refinement: recursive collinearity check; both-endpoints-complex branch samples 10 intermediate points to recover real-valued curve portions).
- `plot_implicit.py` — `plot_implicit`, `ImplicitSeries`: implicit equation/inequality rendering (adaptive interval and uniform grid with inequality sign handling).
- `intervalmath/` — `interval`: bounded numeric range with three-valued validity flag; `__mul__` (invalid/uncertain operand → full (-∞,∞) range unlike `__add__`/`__sub__` which still compute endpoint bounds), `__eq__` delegates to `__lt__` to distinguish overlap (indeterminate) from separation (false). NOT symbolic interval arithmetic (that's `calculus/util.py`).
  - `lib_interval.py` — `sin`, `cos`, `cosh` (lower bound=1 when range crosses zero), `sinh`, `tanh`, `asin`, `acos`, `exp`, `log`, `atan`.
- `experimental_lambdify.py` — `vectorized_lambdify` (callable class: three-tier fallback — numpy array eval → python cmath vectorized → evalf wrapper; catches TypeError/ValueError from unhashable/invalid-limits errors to trigger fallback), `Lambdifier.translate_func`: expression-to-string with float/complex wrapping for plotting.
- `pygletplot/plot.py` — `PygletPlot`: alternative pyglet-based 3D plotting backend; `show`, `append`, `firstavailableindex`.
  - `__setitem__`: index-based function assignment; wraps GeometryEntity in list to prevent sequence unpacking during parse; validates non-negative integer index.
- `pygletplot/plot_axes.py` — `PlotAxes`: 3D axis display; `__init__` processes alias kwargs (`none`, `frame`, `box`, `ordinate`) sequentially — last truthy alias wins style; `flexible_boolean` parses mixed string/bool inputs.
- `pygletplot/plot_mode.py` — `PlotMode._interpret_args` (argument parser: when first arg is a GeometryEntity, extracts coordinate functions via `arbitrary_point()` and range via `plot_interval()`), `_extract_options`.
- `pygletplot/color_scheme.py` — `ColorScheme` (surface appearance mapping); `_interpret_args` (3 per-channel range tuples (r1,r2),(g1,g2),(b1,b2) transposed to endpoint RGB triples); `ColorGradient` (multi-stop color interpolation).

### [`geometry/`](geometry/catalog.md)
Euclidean geometry: points, lines, polygons, circles, ellipses, planes.
- `entity.py` — `GeometryEntity` base class; `_repr_svg_` (inline SVG for notebook display; zero-area bounding box → adds ±0.5 buffer; returns None if no SVG representation), `bounds`.
  - `GeometrySet` (hybrid GeometryEntity+Set): `_union` (FiniteSet merge: filters contained points, returns None if no points overlap — signals no simplification possible), `_intersect`, `_contains`.
- `ellipse.py` — `Ellipse` (`minor`/`major`: shorter/longer axis; falls back to vradius/hradius when symbolic radii ordering is indeterminate), `Circle`: tangent lines, intersection, `encloses_point`; `normal_lines` (perpendicular lines from point; fast Poly.real_roots with solve fallback on DomainError/PolynomialError).
- `line3d.py` — `Line3D`, `Ray3D`, `Segment3D`: 3D linear entities; `Segment3D.distance` (point-to-segment via projection parameter t).
- `plane.py` — `Plane`: `projection_line` (project line onto plane; returns Point3D if line is along normal), `angle_between`, `distance`, `intersection`, `are_concurrent`.
- `point.py` — `Point` (2D/3D Euclidean): `Point3D.__new__` (auto-pads 2-coord input with zero for third component), `intersection` (delegates to other entity if not a Point). NOT `vector/point.py` (coordinate system points).
- `curve.py` — `Curve`: parametric 2D path; `arbitrary_point` (parameterized sample point with name-collision detection against existing free symbols; raises ValueError on conflict), `plot_interval`, `translate`.
- `polygon.py` — `Polygon`, `Triangle` (medial, nine-point circle, Euler line), `RegularPolygon`; `_sss` (side-side-side construction: returns None if side lengths violate triangle inequality), `_sas`, `_asa`.
- `util.py` — `idiff` (implicit differentiation dy/dx from eq==0; accepts list of dependent variables — first is primary, rest are additional x-dependent symbols replaced by Functions), `intersection`, `convex_hull`, `closest_points`, `farthest_points`, `are_coplanar`, `are_similar`, `centroid`.

### [`holonomic/`](holonomic/catalog.md)
Holonomic function representation via differential equations.
- `holonomic.py` — `DifferentialOperators` (factory returning algebra + Dx operator), `DifferentialOperatorAlgebra` (Weyl algebra for differential operators; None generator defaults to Symbol('Dx', commutative=False)); `HolonomicFunction`: `__add__` (finds annihilator via linear system; reconciles mismatched initial points by checking which x0 is 0 and singularity conditions to decide which operand to shift via `change_ics`), `diff` (differentiation; shifts ODE when zeroth coefficient is zero), `integrate` (definite: tries `to_expr` then `evalf` fallback).
  - `to_sequence`: convert ODE to recurrence relation for power series coefficients; computes minimum valid index from degree offset and leading-coefficient roots; non-zero expansion point handled via `shift_x`.
  - `series`: compute truncated power series from recurrence; shifts variable at the end when expansion center is non-zero.
  - `_indicial`, `_extend_y0`, `evalf` (numerical evaluation via RK4/Euler).
  - `_convert_meijerint`: convert expression to holonomic form via Meijer G-function decomposition; absorbs x-power prefactors into G-function parameters before converting each term.
  - `from_meijerg`/`from_hyper`: convert Meijer G / hypergeometric functions to holonomic form; constructs ODE from parameters, finds initial conditions by incrementing evaluation point when function/derivatives diverge.
- `recurrence.py` — `RecurrenceOperator` (shift operator algebra with `__eq__`), `HolonomicSequence`: recurrence-based holonomic function algebra.
- NOT recurrence solvers (those are `solvers/recurr.py`). NOT general ODE solving (that's `solvers/ode.py`).

### [`tensor/`](tensor/catalog.md)
Abstract index notation for tensors: `TensorHead`, `TensorIndex`, `TIDS`, Einstein summation.
- `tensor.py` — `TensMul.canon_bp` (canonicalization), `TensAdd`, `TensorIndexType`, `TIDS.from_components_and_indices`; `TensorSymmetry` (monoterm slot symmetry via BSGS; `__new__` accepts 1 tuple or 2 args; raises TypeError on >2 args), `tensorsymmetry` (factory from Young tableaux shapes).
  - `tensor_indices`: create named indices from comma-separated string; returns bare object for single name, list for multiple.
- `index_methods.py` — `get_contraction_structure` (recursive analysis of repeated/dummy indices in expressions; returns nested dict mapping index tuples to term sets; Pow/exp recurse base and exponent independently), `get_indices`.
- `indexed.py` — `Idx` (integer subscript for array access; single dimension arg→lower=0, upper=dim-1; numeric label short-circuits to number), `Indexed`, `IndexedBase`.
- `array/` — N-dimensional arrays (dense/sparse, mutable/immutable): `Array`, `tensorproduct`, `tensorcontraction` (index summation); `NDimArray._parse_index` (convert multi-axis coordinate tuple to flat storage index via row-major linearization; validates rank and per-axis bounds).
  - `DenseNDimArray.__getitem__`: tuple-of-slices → new typed array with computed shape; plain (non-tuple) slice → raw `self._array[index]` elements directly (NOT a new array object).
  - `NDimArray.__mul__`: scalar-only; raises ValueError("scalar expected, use tensorproduct(...)") for iterables, other arrays, or Matrix inputs.
  - `SparseNDimArray.tomatrix`: convert to SparseMatrix; raises ValueError if rank ≠ 2.
  - `derive_by_array`: element-wise symbolic derivatives w.r.t. basis variables; combine with contraction for divergence-like operations.
- NOT quantum tensor products (those are `physics/quantum/tensorproduct.py`).
- NOT second quantization index ordering (that's `physics/secondquant.py`).

### [`sets/`](sets/catalog.md)
Set theory: intervals, finite sets, unions, complements, images.
- `sets.py` — `Set` base class (`_infimum_key`: sorting key via infimum; returns ∞ on failure), `Interval`, `FiniteSet`, `Union` (with `reduce`: merge FiniteSets then iterative pairwise simplification), `Complement`, `ProductSet` (Cartesian product of sets; `_union`: simplifies only when first or last factor matches — middle-only match returns None; `_intersect`: pairwise intersection of corresponding factors).
  - `Interval._eval_Eq`: returns None (unevaluated) for compound set types (Union/Complement/Intersection/ProductSet), returns false for all other non-Interval types.
  - `Interval._eval_imageset`: piecewise function image via progressive domain removal; continuous function image via critical points.
- `conditionset.py` — `ConditionSet`: set defined by a condition over a base set; FiniteSet base → sifts elements via fuzzy_bool into true/false/indeterminate; indeterminate elements remain wrapped in a new ConditionSet.
- `contains.py` — `Contains`: element-membership predicate (delegates to set's `contains`; returns None to break mutual recursion).
- `fancysets.py` — `ImageSet` (image of a set under a Lambda function; `__iter__` deduplicates via seen-set; constructor returns base set for identity map, FiniteSet for constant expression), `Range`, `Naturals`, `Integers`, `Reals`; `ComplexRegion` (polar/rectangular complex sets); `normalize_theta_set` (angular interval normalization to [0,2π), splits wrap-around).

### [`calculus/`](calculus/catalog.md)
Calculus utilities: finite differences, Euler equations, singularities, accumulation bounds.
- `util.py` — `AccumBounds`: symbolic interval arithmetic for limit computation; `__pow__`, `__add__`, `__sub__` (∞−∞ returns full real line), `__contains__` with ±∞ pairing semantics.
  - `__lt__`/`__le__`/`__gt__`/`__ge__`: compare ranges; overlapping → None; unknown finiteness/sign falls through to Expr base class.
  - `function_range` (output range over a domain; limits at open endpoints, substitution at closed); `continuous_domain`; `not_empty_in`. NOT plotting interval math.
- `singularities.py` — `singularities`: find singularities of a function.
  - `is_increasing`, `is_decreasing`, `is_strictly_increasing`, `is_strictly_decreasing`, `is_monotonic`: monotonicity tests via derivative sign analysis.
  - Constants (no free symbols): non-strict variants return True, strict variants return False.
- `euler.py` — `euler_equations`: derive Euler-Lagrange stationary-condition equations from a Lagrangian.
- NOT ODE solving (that's `solvers/ode.py`). NOT limits (that's `series/limits.py`).

### [`diffgeom/`](diffgeom/catalog.md)
Differential geometry: manifolds, forms, connections, tensor products.
- `diffgeom.py` — `CoordSystem` (coordinate charts with auto-generated labels), `Manifold`, `Patch`, `BaseScalarField`; `contravariant_order`/`covariant_order` (compute tensor degree of expr; Mul rejects products of two nonzero-order fields); `twoform_to_matrix` (convert rank-2 covariant form to matrix by evaluating on basis vector pairs; validates covariant order==2 and single coord system); `Commutator` (Lie bracket of vector fields; evaluates explicitly when both fields use single coord system, returns unevaluated symbolic object when multiple coord systems involved); `TensorProduct` (differential form tensor product; filters scalars from forms, returns scalar*form if only one form), `WedgeProduct`, `LieDerivative`, `Differential`.
- `rn.py` — predefined Euclidean spaces `R2`, `R3` with coordinate systems (rectangular, polar/cylindrical/spherical); all pairwise transition maps explicitly specified with `inverse=False, fill_in_gaps=False`.

### [`liealgebras/`](liealgebras/catalog.md)
Lie algebra representations, root systems, and Weyl groups.
- `type_a.py`, `type_b.py`, `type_c.py`, `type_d.py`, `type_f.py` — classical Lie algebra families: `simple_root`, `positive_roots`, `cartan_matrix`, `dimension`.
- `type_e.py` — exceptional E₆/E₇/E₈: `positive_roots` generates half-integer coordinate vectors via nested binary-variable loops with even-parity filter. Known bug: root vector mutated in-place without reset between iterations.
- `type_g.py` — exceptional G₂ (rank-2 only): `positive_roots` (NOTE: docstring/example erroneously references A_n series — copy-paste error), `simple_root`, `cartan_matrix`.
- `weyl_group.py` — `WeylGroup`, `element_order` (matrix exponentiation for most types; string-reduction for G2), `delete_doubles`.
- `cartan_type.py` — `CartanType` factory, `Standard_Cartan` base class.
- NOT permutation groups (those are `combinatorics/perm_groups.py`).

### [`categories/`](categories/catalog.md)
Category theory: objects, morphisms, diagrams.
- `baseclasses.py` — `Morphism` (base arrow: `compose`/`__mul__` for sequential composition in mathematical order g∘f), `IdentityMorphism`, `NamedMorphism`, `CompositeMorphism`, `Object`, `Class`.
- `diagram_drawing.py` — `DiagramGrid` (2D lattice layout with disconnected-component handling), `XypicDiagramDrawer`, `ArrowStringDescription`.
  - `_juxtapose_edges`: construct new edge from pair sharing one endpoint; identical edges → None. `_build_skeleton`/`_list_triangles` decompose morphism graphs into triangles.

### [`parsing/`](parsing/catalog.md)
Expression parsing: string-to-SymPy conversion, Mathematica/Maxima translators.
- `sympy_parser.py` — `parse_expr`, `implicit_multiplication`, `implicit_application`.
  - `lambda_notation`: converts `lambda` keyword to `Lambda()` call; raises `TokenError` on starred arguments (`*`/`**`) in lambda parameters.
  - `split_symbols_custom`: break multi-char names into chars for implicit multiplication; known names emitted as direct refs, unknown wrapped in Symbol().
  - `convert_equals_signs`: nested `=` to `Eq()` via recursive parenthesis grouping.
- `mathematica.py` — `mathematica`, `parse`: Mathematica-to-SymPy parser; `translateFunction` (Arc-prefix → 'a' + lowercase for inverse trig, else lowercase), `translateOperator`.

### [`interactive/`](interactive/catalog.md)
Interactive session setup: `init_session`, `init_printing`.
- `printing.py` — `_init_ipython_printing`: configures IPython display hooks.
  - `_print_latex_png`: renders expression as PNG via external LaTeX; on RuntimeError falls back to matplotlib, re-rendering in inline mode if current mode differs.
  - `_can_print_latex`: type-gate for LaTeX rendering; explicitly excludes `bool` (even though bool subclasses int); recurses into containers.
  - NOT `printing/latex.py` (which formats LaTeX strings).

### [`unify/`](unify/catalog.md)
Unification algorithms for expression pattern matching.
- `core.py` — generic tree unification (AIMA-based): `Variable` (unconstrained wildcard), `CondVariable` (wildcard with predicate filter: `valid(x)` must return True for match), `Compound`, `unify`/`unify_var`.
- `usympy.py` — `unify` (structural unification with commutative/associative matching), `is_commutative` (for Mul: checks all args, not hardcoded), `deconstruct`/`construct` (SymPy↔Compound; construct uses three-way dispatch: AssocOp/Pow/FiniteSet→evaluate=False, MatrixExpr→Basic.__new__, else normal constructor).
- NOT `core/basic.py` `_has`/`matches` (those are tree-structural matching).

### [`crypto/`](crypto/catalog.md)
Classical cryptographic ciphers and key exchange protocols (educational).
- `crypto.py` — ciphers, `encode_morse`/`decode_morse` (Morse code encoding/decoding; `encode_morse` preserves trailing whitespace by appending double-separator), `lfsr_sequence`, `lfsr_autocorrelation` (raises TypeError if input is not a list), `lfsr_connection_polynomial` (Berlekamp-Massey).
  - `dh_private_key`/`dh_public_key` (Diffie-Hellman), `elgamal_private_key` (ElGamal).

### [`external/`](external/catalog.md)
Utilities for importing optional third-party packages.
- `importtools.py` — `import_module`: safe optional dependency loader with version checking; supports callable version attributes via `module_version_attr_call_args`.
  - `__sympy_debug`: reads `SYMPY_DEBUG` env var; raises RuntimeError on values other than 'True'/'False'.

### [`strategies/`](strategies/catalog.md)
Rule-based expression transformation strategies.
- `tree.py` — `treeapply` (recursive dispatch: maps container types to combining functions via a dict; non-matching leaf elements get identity/leaf function), `greedy` (execute strategic tree greedily: tuples=chained sequence, lists=select alternative minimizing objective), `allresults`.
- `core.py` — `exhaust` (apply rule to fixed point), `chain`, `do_one`, `tryit`, `condition`, `minimize`, `memoize`, `switch`.
- `rl.py` — `rebuild` (recursively re-invoke constructors for canonicalization; silently returns original node if constructor raises), `flatten`, `unpack`, `distribute`, `subs`, `sort`, `glom`.
- `branch/core.py` — nondeterministic (branching) strategies: `exhaust` (repeated application with cycle detection via seen-set), `multiplex`, `condition`, `sfilter`, `notempty`, `do_one`, `chain`.
- `traverse.py` — `top_down`, `bottom_up`, `sall`: tree traversal combinators for applying rules.

### [`sandbox/`](sandbox/catalog.md)
Experimental/sandbox code.
- `indexed_integrals.py` — `IndexedIntegral`: extends `Integral` to support integration over `Indexed` (subscripted array-element) variables; replaces Indexed limits with Dummy placeholders before evaluation, restores after via reverse mapping.
