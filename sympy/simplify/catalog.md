# Simplify Module Catalog

## Glossary
- **TR rules** (`fu.py`): Individual named transformation rules (TR0–TR22, TRmorrie, TR111) each applying one trig identity.
- **Fu algorithm** (`fu.py`): Orchestrates TR rules via combination transforms (CTR1–4) and rule lists (RL1, RL2); uses single-metric measure (trig count) to pick simplest form.
- **Gröbner trig simplification** (`trigsimp.py`): Uses polynomial ideal / Gröbner basis over trig generators to simplify trig expressions.

## Notes
- Individual trig identity transforms (sin²↔cos², sum↔product, double-angle rewrite, factored-power identities) are in `fu.py` as named TR rules; `trigsimp_groebner` in `trigsimp.py` can also produce double-angle forms via degree-minimizing ideal reduction.
- `trigsimp.py` is the high-level entry point that dispatches to Fu-based or Gröbner-based strategies, and also owns the product-of-powers rewrite engine (e.g. sin^a·cos^b → tan^c) with non-commutativity handling.
- Sign canonicalization of sub-expressions (`signsimp`) lives in `simplify.py`, not `fu.py`.

---

## Trigonometric Simplification

### [`fu.py`](fu.py)
Individual trig transformation rules and the Fu simplification algorithm. Each TR rule applies one specific trig identity.

- `fu(rv, measure)` — main Fu algorithm; applies TR rules via CTR and RL sequences, selects simplest by caller-supplied `measure` (defaults to `L`, i.e. trig-function count only).
  - Uses JIT factoring: extracts common factors to attempt trig combination, discards factoring if it doesn't simplify.
- `TR0(rv)` — rational polynomial normalization (combine like terms); uses `.normal().factor().expand()` instead of `cancel` to support noncommutative expressions.
- `TR1(rv)` — replace sec/csc with 1/cos and 1/sin.
- `TR2(rv)` — replace tan/cot with sin/cos and cos/sin ratios.
- `TR2i(rv, half)` — single-pass rule: converts sin/cos ratio terms within Add sums back to tan; with `half=True` also rewrites sin/(cos+1) → tan(x/2).
  - Does not handle Mul-level powered-product rewriting (that is `_match_div_rewrite` in trigsimp.py).
  - Exponent/base guard: only rewrites when exponent is integer or base is positive; otherwise leaves expression unchanged.
- `TR3(rv)` — canonicalize angles via induced formulas.
- `TR4(rv)` — evaluate trig at special angles (0, π/6, π/4, π/3, π/2).
- `_TR56(rv, f, g, h, max, pow)` — helper for TR5/TR6; replaces even powers of sin/cos via Pythagorean identity.
  - `max` limits eligible exponent size; `pow` restricts rewrites to exponents that are perfect powers of 2.
- `TR5` / `TR6` — replace sin²→1−cos² and cos²→1−sin² (use `_TR56`).
- `TR7(rv)` — lower cos² degree via double-angle formula.
- `TR8(rv)` — convert products of sin/cos into sums (product-to-sum).
- `TR9(rv)` — convert sums of sin/cos into products (sum-to-product).
  - For >2 terms: tries all pairs, combines successful ones, recursively re-applies on the reduced sum.
- `TR10` / `TR10i` — expand/contract angle sums in sin/cos arguments.
- `TR11(rv)` — rewrite double angles as single-angle products.
- `TR12` / `TR12i` — expand/contract angle sums in tan arguments.
- `TR13(rv)` — simplify products of tan/cot via addition formulas.
- `TR14(rv, first)` — simplify factored sin/cos difference-of-squares like (cos x−1)·(cos x+1) → −sin²x.
  - Exponent/base guard: skips Pow factors where exponent is non-integer and base is not positive (avoids invalid transforms on e.g. (cos x−1)^(1/2)).
  - Handles unequal exponents by taking the minimum and reinserting the remainder.
  - Splits numer/denom and recurses with `first=False` to prevent infinite re-splitting on fractional expressions.
- `TR15` / `TR16` — convert negative sin/cos powers to cot²/tan² forms.
- `TR22(rv)` — convert tan²→sec²−1 and cot²→csc²−1.
- `TRmorrie(rv)` — apply Morrie's law for cosine products.
- `TR111(rv)` — convert negative trig powers to reciprocal functions (sec, csc, cot).
- `L(rv)` — count trig functions in expression (used as complexity measure).
- `trig_split(a, b)` — decompose pair of trig terms for identity matching.
- `as_f_sign_1(e)` — decompose expression as g*(f ± 1) for factored-identity rules.
- `process_common_addends(rv, do, key2, key1)` — group addends by absolute value of their coefficient (and optional `key2`), then apply `do` to each group with >1 member.
  - Negative coefficients: negates both coefficient and argument so sign is transferred to the argument before grouping.
- `hyper_as_trig(rv)` — unify mixed trig/hyperbolic expressions: masks pre-existing trig functions with placeholders, converts hyperbolics to trig via Osborne's rule, returns transformed expr + reversal callable that restores original trig functions and converts back.

### [`trigsimp.py`](trigsimp.py)
High-level trigonometric simplification entry points and Gröbner-basis trig solver.

- `trigsimp(expr, **opts)` — main entry point; dispatches to Gröbner, Fu-based, or old pattern-matching strategies.
  - `recursive` option: uses CSE to extract common subexpressions, simplifies the reduced expression, then back-substitutes in reverse order, re-simplifying after each substitution.
- `trigsimp_groebner(expr, hints)` — simplifies trig expressions via polynomial Gröbner basis over trig generators; minimizes total degree of the result.
  - Numeric hints (e.g. `2`) expand search space to find double-angle/multiple-angle forms like sin(x)·cos(x) → sin(2x)/2.
  - `analyse_gens(gens, hints)` — groups generators by argument, computes GCD base frequency, ensures complementary functions (sin/cos/tan) are included.
  - `build_ideal(x, terms)` — generates polynomial relations (Pythagorean, multiple-angle) for the ideal.
  - `parse_hints(hints)` — interprets user hints for generator selection.
- `exptrigsimp(expr)` — simplifies mixed exponential/hyperbolic/trig expressions.
- `_trigsimp` / `__trigsimp` — cached recursive helper for trig/hyperbolic simplification of sub-expressions.
  - For `Mul`: splits non-commutative products into commutative and non-commutative parts; simplifies only the commutative portion.
  - For commutative products: tries fast rewrite via `_match_div_rewrite` first; if it returns None (e.g. skipped indices 6,7), falls back to SymPy `.match()` pattern matching with positivity guards.
  - For `Add`: applies identity matchers per term, then addition matchers with `TR10i` contraction; skips if matched residual contains trig/hyper of matched args.
  - Artifact reduction phase: reverses Pythagorean substitutions (e.g. 1−cos²→sin²) that made the expression more complex; uses restricted wildcards (excluding certain functions) to influence better matches.
  - Loop guard: iterates artifact reversal with `was != expr` check to prevent infinite re-matching; breaks early when matched coefficient is zero or cancels with other terms.
- `_dotrig(a, b)` — guard that checks whether expression `a` and pattern `b` share the same function category (both TrigonometricFunction or both HyperbolicFunction) and outer type (`func`); skips pattern matching when categories don't match.
- `_replace_mul_fpowxgpow` — rewrites f(x)^a·g(x)^b into h(x)^c for matched trig pairs; only applies when base is positive or exponent is integer.
- `_trigpats()` — initializes global wildcard-based pattern tables (`matchers_division`, `matchers_add`, `matchers_identity`, `artifacts`) for rewriting ratios/products of trig and hyperbolic functions.
  - `artifacts` table: reverses Pythagorean identity substitutions that made an expression more complex (e.g. 1−cos²→sin², 1−1/cos²→−tan²), restoring the simpler original form.
  - First 14 division patterns must stay in fixed order — `_match_div_rewrite` indexes them by position.
- `_match_div_rewrite` — dispatcher mapping pattern index to specific trig-pair rewrite (sin/cos→tan, tan/cos→sin, etc., plus hyperbolic variants); explicitly skips indices 6,7 (sum-and-difference-of-one factors like (cos±1)(cos∓1)) which can't be expressed as f^a·g^b.
- `trigsimp_old(expr)` — legacy pattern-matching trig simplifier.
  - Multi-symbol handling: uses `separatevars` to factor into per-variable groups (returned as dict); each factor is expanded via `expand_mul` then simplified.
  - Hollow-factoring removal: if simplifying an expanded factor yields no improvement (result == expanded form), reverts to the original unexpanded form to prevent expression degradation.
  - If unfactorable sum, iterates per-symbol `as_independent` splits, stopping early when result is no longer Add.
  - `recursive` option: extracts common subexpressions via CSE, simplifies the reduced expression, then re-substitutes in reverse order, re-simplifying after each substitution.
- `futrig(expr)` — applies Fu-like transformation tree for trig simplification; uses `_futrig` helper internally.
  - Post-processing: if result differs from input and is a Mul whose first arg is Rational, redistributes the leading coefficient via `as_coeff_Mul()`.
  - Hyperbolic handling: when `hyper=True` (default), converts hyperbolic sub-expressions to trig via `hyper_as_trig`, simplifies, then converts back.
  - `_futrig` builds a nested rule tree of TR transforms and applies them via `greedy` search strategy; defines its own composite objective function `Lops` ranking candidates by tuple (trig count via L, op count, node count, arg count, is_Add) to determine simplest form.

---

## General Simplification

### [`simplify.py`](simplify.py)
Main general-purpose simplification and miscellaneous simplification functions.

- `simplify(expr, ratio, measure, fu)` — primary heuristic simplifier; tries multiple strategies and picks the shortest.
  - For non-arithmetic functions (not Add/Mul/Pow/Exp) with an `inverse` attribute: detects and unwraps inverse-function compositions (e.g. f(f⁻¹(x)) → x) before recursing into args.
- `signsimp(expr, evaluate)` — canonicalize sign of Add sub-expressions (e.g., y−x → −(x−y)).
  - Uses double-negation on Mul atoms to detect non-canonical signs; `evaluate` controls whether no-op transforms are kept.
- `separatevars(expr, symbols, dict, force)` — factor expression into product of single-variable terms.
  - `dict=True` delegates to `_separatevars_dict`: returns a dict mapping each symbol to its factor plus a `'coeff'` key; returns `{'coeff': expr}` if symbols is None, None if unseparable.
- `posify(eq)` — replace symbols with positive dummies for assumption-sensitive simplification.
- `logcombine(expr, force)` — merge additive log terms: log(x)+log(y)→log(x·y) when args positive; a·log(x)→log(x^a) when a is real and x positive.
  - `force=True`: assumes positivity/realness when no conflicting assumption exists; does not override explicit assumptions (e.g. imaginary coefficient).
  - Negative rational coefficients are decomposed into −1 and positive remainder before log merging; the −1 is explicitly excluded from exponent absorption, while the positive part is absorbed normally.
  - Complex coefficients (e.g. 2+3i) block combination unless first expanded into real and imaginary parts.
- `nsimplify(expr, constants, tolerance)` — find simple closed-form for numerical expressions.
- `hypersimp(f, k)` — compute consecutive-term ratio f(k+1)/f(k) for combinatorial/hypergeometric sequences; rewrites via gamma functions, returns simplified rational function or None if not hypergeometric.
- `besselsimp(expr)` — simplify Bessel function expressions.
- `nthroot(expr, n)` — compute real nth root of sum of surds; for negative `expr` with odd `n`, negates before root extraction and returns negated result.
  - `_nthroot_solve(p, n, prec)` — helper; denests `p**(1/n)` using minimal polynomial. For power-of-2 `n`, repeatedly sqrtdenests and halves `n`, returning early without polynomial solving.
- `bottom_up(rv, F, atoms, nonbasic)` — apply function bottom-up through expression tree; recurses into `rv.args`.
  - If `rv` lacks `args` (non-Basic object): catches `AttributeError`; applies `F` only when `nonbasic=True` (catching `TypeError` if `F` rejects it).
  - `atoms=True`: applies `F` even to leaf nodes (args is empty).
- `clear_coefficients(expr)` — strip rational leading coefficients.
- `sum_simplify(s)` — simplify sums of Sum objects; absorbs constants into Sum bodies and pairwise merges terms.
- `sum_add(s1, s2, method)` — helper merging two Sums: method 0 combines matching limits; method 1 merges adjacent index ranges when bodies match and index variable is the same.
- `product_simplify(s)` — analogous simplification for Product objects.

---

## Power & Radical Simplification

### [`powsimp.py`](powsimp.py)
Simplifies expressions by combining powers with similar bases or exponents.

- `powsimp(expr, deep, combine)` — combine powers: x^a·x^b → x^(a+b), x^a·y^a → (xy)^a.
- `powdenest(eq, force, polar)` — denest powers: (x^a)^b → x^(a·b).

### [`radsimp.py`](radsimp.py)
Radical simplification, term collection, and rationalization.

- `radsimp(expr)` — rationalize denominators containing radicals.
  - Recursively reduces `1/d` forms: splits Mul denominators, denests sqrt powers via `sqrtdenest`, decomposes `1/d**i → (1/d)**i` for integer/positive-base powers before recursing on the base.
  - For Add denominators with up to 4 radical terms, multiplies by algebraic conjugate to eliminate radicals; for >4 terms whose squares are rational, delegates to `rad_rationalize`.
- `collect(expr, syms)` — collect terms by powers of specified symbols; also collects by derivative orders, traversing nested derivative towers.
  - Mixed partial derivatives (differentiation w.r.t. multiple different variables) raise `NotImplementedError`; only single-variable derivative chains are supported.
- `rcollect(expr, *vars)` — recursive collect.
- `collect_sqrt(expr)` — collect terms sharing square-root factors.
- `fraction(expr, exact)` — decompose expression into (numerator, denominator) pair by splitting powers with negative exponents.
  - `exact=True`: only moves constant negative exponents to denominator; non-constant negative exponents stay in numerator; returns unevaluated Muls.
- `numer(expr)` / `denom(expr)` — shorthand for `fraction(expr)[0]` / `[1]`.
- `rad_rationalize(num, den)` — recursively rationalize a fraction whose denominator is a sum of square-root terms with rational squares.
  - Each step uses `split_surds` to decompose the denominator into conjugate-like halves (a, b), multiplies by (a−b), and recurses until the denominator is no longer an Add.
- `split_surds(expr)` — split a sum of square-root terms into groups by GCD of squared radicands.
  - If all radicands share a common factor (no coprime group), divides out that factor and re-partitions.
- `_split_gcd(*a)` — partition integers into a GCD-sharing group and a coprime remainder group.

### [`sqrtdenest.py`](sqrtdenest.py)
Denests nested square root expressions (simplifies radical form, not denominator rationalization).

- `sqrtdenest(expr)` — main entry; denests expressions like √(2+√3).
- `sqrt_depth(p)` — max nesting depth of square roots; only counts `is_sqrt` powers (exponent ±½), returns 0 for non-sqrt Pow nodes without recursing into their base.
- `is_sqrt(expr)` — True if expr is a Pow with rational exponent of absolute value ½.

---

## Combinatorial & Rational Simplification

### [`combsimp.py`](combsimp.py)
Simplifies combinatorial function expressions (factorials, binomials, gamma, Pochhammer) — rewrites and reduces function counts, not term-ratio analysis.

- `combsimp(expr)` — minimize number of combinatorial functions (factorials, binomials, gamma, Pochhammer).
  - For multiplicative expressions: splits non-commutative factors out, simplifies only the commutative part; returns expr unchanged if no commutative args.
  - Gamma simplification: applies reflection formula, recursive absorption of rational offsets, duplication theorem, and Gauss multiplication theorem.
  - Duplication theorem: cancels gamma(2s)/gamma(s) pairs, replacing with half-integer-shifted gamma plus power-of-two and √π factors.
  - Multiplication theorem: detects n gamma args forming an arithmetic progression with common difference 1/n, collapses the product into a single gamma with scaled argument (n·x).

### [`ratsimp.py`](ratsimp.py)
Simplifies rational expressions by computing common denominators.

- `ratsimp(expr)` — put expression over common denominator and cancel.
- `ratsimpmodprime(expr, G, gens)` — simplify rational expression modulo a prime ideal.

---

## Common Subexpression Elimination

### [`cse_main.py`](cse_main.py)
Common subexpression elimination (CSE) for expression DAG compression.

- `cse(exprs, symbols, optimizations)` — identify and extract repeated subexpressions.
- `tree_cse(exprs, symbols)` — CSE via expression-tree traversal.
- `opt_cse(exprs, order)` — optimize expression list before CSE.

### [`cse_opts.py`](cse_opts.py)
Preprocessing/postprocessing optimizations for CSE.

- `sub_pre(e)` — pre-CSE: rewrite Add terms for better sharing.
- `sub_post(e)` — post-CSE: undo pre-processing rewrites.

---

## Hypergeometric Expansion

### [`hyperexpand.py`](hyperexpand.py)
Expands hypergeometric and Meijer G-functions into named special functions using lookup tables and algorithmic reduction.

- `hyperexpand(f, allow_hyper, rewrite)` — main entry; expands hyper/Meijer G to closed form.

### [`hyperexpand_doc.py`](hyperexpand_doc.py)
Sphinx documentation generator for hypergeometric expansion formulae.

---

## Utilities

### [`epathtools.py`](epathtools.py)
XPath-like path language for navigating and transforming nested expression trees using slash-delimited selectors (type filters, attribute checks, slice indexing).

- `EPath` class — compiles a path string into a reusable selector; provides `.select(expr)` to retrieve matches and `.apply(expr, func)` to transform them.
- `epath(path, expr, func)` — convenience wrapper with three-way dispatch: returns compiled path if no expr, retrieves matched sub-expressions if no func, applies func to matches otherwise.

### [`traversaltools.py`](traversaltools.py)
Depth-level function application on expression args (no path language or pattern matching).

- `use(expr, func, level)` — apply a function to all args at a fixed depth level in the expression tree.
