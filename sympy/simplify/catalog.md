# Simplify Module Catalog

## Glossary
- **TR rules** (`fu.py`): Individual named transformation rules (TR0–TR22, TRmorrie, TR111) each applying one trig identity.
- **Fu algorithm** (`fu.py`): Orchestrates TR rules via combination transforms (CTR1–4) and rule lists (RL1, RL2) to find simplest trig form.
- **Gröbner trig simplification** (`trigsimp.py`): Uses polynomial ideal / Gröbner basis over trig generators to simplify trig expressions.

## Notes
- Individual trig identity transforms (sin²↔cos², sum↔product, double-angle, factored-power identities) are in `fu.py`, not `trigsimp.py`.
- `trigsimp.py` is the high-level entry point that dispatches to Fu-based or Gröbner-based strategies, and also owns the product-of-powers rewrite engine (e.g. sin^a·cos^b → tan^c) with non-commutativity handling.
- Sign canonicalization of sub-expressions (`signsimp`) lives in `simplify.py`, not `fu.py`.

---

## Trigonometric Simplification

### [`fu.py`](fu.py)
Individual trig transformation rules and the Fu simplification algorithm. Each TR rule applies one specific trig identity.

- `fu(rv, measure)` — main Fu algorithm; applies TR rules via CTR and RL sequences, selects simplest result.
  - Uses JIT factoring: extracts common factors to attempt trig combination, discards factoring if it doesn't simplify.
- `TR0(rv)` — simplify rational trig subexpressions (combine like terms).
- `TR1(rv)` — replace sec/csc with 1/cos and 1/sin.
- `TR2(rv)` / `TR2i(rv)` — convert between tan/cot and sin/cos ratios.
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
- `TR14(rv)` — simplify factored sin/cos powers like (cos x−1)^a·(cos x+1)^b.
  - Handles unequal exponents by taking the minimum and reinserting the remainder.
- `TR15` / `TR16` — convert negative sin/cos powers to cot²/tan² forms.
- `TR22(rv)` — convert tan²→sec²−1 and cot²→csc²−1.
- `TRmorrie(rv)` — apply Morrie's law for cosine products.
- `TR111(rv)` — convert negative trig powers to reciprocal functions (sec, csc, cot).
- `L(rv)` — count trig functions in expression (used as complexity measure).
- `trig_split(a, b)` — decompose pair of trig terms for identity matching.
- `as_f_sign_1(e)` — decompose expression as g*(f ± 1) for factored-identity rules.
- `process_common_addends(rv, do)` — apply transform to groups sharing common factors in a sum.
- `hyper_as_trig(rv)` — convert hyperbolic expressions to trig (Osborne's rule) with reversal function.

### [`trigsimp.py`](trigsimp.py)
High-level trigonometric simplification entry points and Gröbner-basis trig solver.

- `trigsimp(expr, **opts)` — main entry point; dispatches to Gröbner, Fu-based, or old pattern-matching strategies.
- `trigsimp_groebner(expr, hints)` — simplifies trig expressions via polynomial Gröbner basis over trig generators.
  - `analyse_gens(gens, hints)` — groups generators by argument, computes GCD base frequency, ensures complementary functions (sin/cos/tan) are included.
  - `build_ideal(x, terms)` — generates polynomial relations (Pythagorean, multiple-angle) for the ideal.
  - `parse_hints(hints)` — interprets user hints for generator selection.
- `exptrigsimp(expr)` — simplifies mixed exponential/hyperbolic/trig expressions.
- `_trigsimp` / `__trigsimp` — recursive helper for trig simplification of sub-expressions.
  - For `Mul`: splits non-commutative products into commutative and non-commutative parts; simplifies only the commutative portion.
  - For commutative products: dispatches through division-pattern matchers to rewrite trig-power products (e.g. sin^a·cos^b → tan^c).
- `_replace_mul_fpowxgpow` — rewrites f(x)^a·g(x)^b into h(x)^c for matched trig pairs; only applies when base is positive or exponent is integer.
- `_match_div_rewrite` — dispatcher mapping pattern index to specific trig-pair rewrite (sin/cos→tan, tan/cos→sin, etc., plus hyperbolic variants).
- `trigsimp_old(expr)` — legacy pattern-matching trig simplifier.
- `futrig(expr)` — applies Fu-like transformation tree for trig simplification.

---

## General Simplification

### [`simplify.py`](simplify.py)
Main general-purpose simplification and miscellaneous simplification functions.

- `simplify(expr, ratio, measure, fu)` — primary heuristic simplifier; tries multiple strategies and picks the shortest.
- `signsimp(expr, evaluate)` — canonicalize sign of Add sub-expressions (e.g., y−x → −(x−y)).
  - Uses double-negation on Mul atoms to detect non-canonical signs; `evaluate` controls whether no-op transforms are kept.
- `separatevars(expr, symbols, dict, force)` — factor expression into product of single-variable terms.
- `posify(eq)` — replace symbols with positive dummies for assumption-sensitive simplification.
- `logcombine(expr, force)` — combine/split logarithms using log rules.
- `nsimplify(expr, constants, tolerance)` — find simple closed-form for numerical expressions.
- `hypersimp(f, k)` — compute consecutive-term ratio of hypergeometric sequences.
- `besselsimp(expr)` — simplify Bessel function expressions.
- `nthroot(expr, n)` — compute real nth root of sum of surds.
- `bottom_up(rv, F)` — apply function bottom-up through expression tree.
- `clear_coefficients(expr)` — strip rational leading coefficients.

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
  - For Add denominators with up to 4 radical terms, multiplies by algebraic conjugate to eliminate radicals.
- `collect(expr, syms)` — collect terms by powers of specified symbols.
- `rcollect(expr, *vars)` — recursive collect.
- `collect_sqrt(expr)` — collect terms sharing square-root factors.

### [`sqrtdenest.py`](sqrtdenest.py)
Denests nested square root expressions.

- `sqrtdenest(expr)` — main entry; denests expressions like √(2+√3).

---

## Combinatorial & Rational Simplification

### [`combsimp.py`](combsimp.py)
Simplifies combinatorial expressions (factorials, binomials, gamma, Pochhammer).

- `combsimp(expr)` — minimize number of combinatorial functions.

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
Path-based expression selection and manipulation.

- `EPath` class — XPath-like selector for expression subparts.
- `epath(path, expr, func)` — select or apply function to expression parts matching a path pattern.

### [`traversaltools.py`](traversaltools.py)
Tools for applying functions at specific expression-tree levels.

- `use(expr, func, level)` — apply a function at a given depth in the expression tree.
