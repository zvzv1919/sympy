# sympy/simplify — Expression Simplification

## Glossary

- **CSE**: Common Subexpression Elimination — replacing repeated subexpressions with temporaries.
- **Fu algorithm**: A heuristic rule-based trigonometric simplification method (Fu, Zhong & Zeng, 2006).
- **Meijer G-function**: A very general special function that encompasses most named special functions.
- **Rising factorial / Pochhammer**: `rf(a, n) = a(a+1)...(a+n-1)`.
- **Denesting**: Simplifying nested radicals, e.g. `sqrt(2 + sqrt(3))` → closed form.

---

## Core Simplification

### `__init__.py`

Public API surface for the submodule. Re-exports all user-facing functions: `simplify`, `cse`, `fu`, `trigsimp`, `powsimp`, `radsimp`, `collect`, `sqrtdenest`, `hyperexpand`, `combsimp`, `ratsimp`, etc.

### `simplify.py`

The main heuristic simplification engine and assorted utilities.

- `simplify(expr, ratio=1.7, measure=count_ops, fu=False)` — The flagship simplifier.
  - Chains `signsimp → cancel → together → factor_terms → hyperexpand → trigsimp → logcombine → combsimp → powsimp → exptrigsimp` and picks the shortest result by `measure`.
  - `ratio` guards against blowup: returns original if `measure(result)/measure(input) > ratio`.
- `separatevars(expr, symbols, dict, force)` — Factor an expression into pieces each depending on disjoint subsets of symbols.
- `posify(eq)` — Replace assumption-free symbols with positive dummies; returns `(new_expr, back_subs)`.
- `hypersimp(f, k)` — Compute `f(k+1)/f(k)` for hypergeometric term ratio tests.
- `hypersimilar(f, g, k)` — Test if `f/g` is rational in `k` (hyper-similarity).
- `signsimp(expr)` — Canonicalize signs in Add sub-expressions using `could_extract_minus_sign`.
- `logcombine(expr, force)` — Combine logarithms: `log(x) + log(y) → log(x*y)`, `a*log(x) → log(x**a)`.
- `besselsimp(expr)` — Simplify Bessel J/I functions by removing polar factors and expanding half-integer / integer orders.
- `nsimplify(expr, constants, tolerance, full, rational)` — Find a simple closed-form (rational or involving given constants) matching a numerical expression via PSLQ / `mpmath.identify`.
- `nthroot(expr, n)` — Compute real nth-root of a sum of surds using minimal polynomials.
- `bottom_up(rv, F, atoms, nonbasic)` — Apply `F` to every subexpression bottom-up.
- `clear_coefficients(expr, rhs)` — Strip rational additive/multiplicative coefficients.
- `sum_simplify`, `product_simplify` — Combine Sum / Product terms with compatible limits.

---

## Trigonometric

### `trigsimp.py`

Trigonometric / hyperbolic simplification with multiple strategies.

- `trigsimp(expr, method='matching', ...)` — Main entry point.
  - `method='matching'` (default): recursive pattern matching via `futrig`.
  - `method='groebner'`: Gröbner basis algorithm (via `trigsimp_groebner`).
  - `method='combined'`: Gröbner then matching.
  - `method='fu'`: direct Fu-transform pipeline.
- `exptrigsimp(expr)` — Convert between exponential and trig/hyperbolic forms, picking the shortest.
- `trigsimp_groebner(expr, hints, quick, order, polynomial)` — Gröbner-based simplification.
  - Builds a prime ideal from trig identities (`sin²+cos²=1`, angle-multiple relations, sum formulas).
  - Calls `ratsimpmodprime` to find a lower-degree equivalent.
  - `hints` can specify multipliers, functions, or angle-sum generators.
- `futrig(e)` — Fu-transform subset mimicking `trigsimp` behavior; default backend for `trigsimp`.
- `trigsimp_old(expr)` — Legacy implementation kept for `old=True`.

### `fu.py`

Implementation of the Fu et al. trigonometric simplification algorithm (~2000 lines).

**Transform rules** (each is a function operating bottom-up on expression trees):

| Rule | Effect |
|------|--------|
| `TR0` | Rational polynomial simplify |
| `TR1` | `sec/csc → 1/cos, 1/sin` |
| `TR2` / `TR2i` | `tan/cot ↔ sin/cos` ratios |
| `TR3` | Angle canonicalization (negative args, complementary angles) |
| `TR4` | Special-angle evaluation |
| `TR5` / `TR6` | `sin²→1-cos²` / `cos²→1-sin²` |
| `TR7` | `cos²→(1+cos 2x)/2` |
| `TR8` | Products of sin/cos → sums |
| `TR9` | Sums of sin/cos → products |
| `TR10` / `TR10i` | Separate / collect sin/cos arguments |
| `TR11` | Double angle → product of single angles |
| `TR12` / `TR12i` | Separate / collect tan arguments |
| `TR13` | Products of `tan·tan` or `cot·cot` |
| `TRmorrie` | `∏cos(2^k·x) → sin(2^n·x)/(2^n·sin x)` |
| `TR14` | Factored `(cos±1)(cos∓1) → -sin²` |
| `TR15` / `TR16` | Negative powers of sin/cos → cot²/tan² |
| `TR22` | `tan²→sec²-1`, `cot²→csc²-1` |
| `TR111` | Negative trig powers → reciprocal functions |

- `fu(rv, measure)` — Top-level: applies rule lists RL1, RL2 via greedy tree search, minimizing `measure`.
- `L(rv)` — Count of trigonometric functions in `rv`.
- `CTR1`–`CTR4` — Combination transforms (chains/choices of TRs).
- `RL1`, `RL2` — Rule lists (ordered sequences of transforms).
- `trig_split(a, b, two)` — Decompose a two-term sum into gcd · (s1·f(a1) ± s2·f(a2)).
- `process_common_addends(rv, do, key2, key1)` — Group addends by coefficient and apply `do`.
- `hyper_as_trig(rv)` — Osborne substitution: mask trig, convert hyperbolic → trig, return undo function.
- `as_f_sign_1(e)` — Match `g*(a ± 1)` pattern.

---

## Radical & Denominator Simplification

### `radsimp.py`

Radical simplification, term collection, and denominator rationalization.

- `collect(expr, syms, func, evaluate, exact)` — Collect additive terms by powers of `syms`.
  - Works with polynomials, derivatives, wildcards; supports `exact` matching mode.
  - Returns expression or dict of `{power: coefficient}`.
- `rcollect(expr, *vars)` — Recursively collect sums.
- `collect_sqrt(expr)` — Group terms sharing common square-root factors.
- `collect_const(expr, *vars)` — Collect terms with common numerical coefficients.
- `radsimp(expr, symbolic, max_terms)` — Rationalize denominators by removing square roots.
  - Handles 1–4 radical terms via conjugate multiplication; uses `sqrtdenest` internally.
- `fraction(expr, exact)` — Split into `(numerator, denominator)` pair.
- `numer(expr)`, `denom(expr)` — Shorthand for `fraction`.
- `rad_rationalize(num, den)` — Recursive rationalization for sums of surds.
- `split_surds(expr)` — Partition surd terms by GCD of their radicands.

### `sqrtdenest.py`

Denesting of nested square roots. Based on Fagin et al. (1985) and Jeffrey & Rich.

- `sqrtdenest(expr, max_iter=3)` — Main entry; iteratively denests `sqrt(a + b*sqrt(r))`.
- `sqrt_depth(p)` — Maximum nesting depth of square roots.
- `is_sqrt(expr)`, `is_algebraic(p)` — Predicates.
- `sqrt_biquadratic_denest(expr, a, b, r, d2)` — Denest via the biquadratic equation `4A⁴ - 4aA² + b²r = 0`.
- `_sqrtdenest_rec(expr)` — Recursive denester for three or more surds.
- `_denester(nested, av0, h, max_depth_level)` — Core Fagin-style denesting with polynomial complexity via shared bottom-level radicand evaluation.

---

## Power Simplification

### `powsimp.py`

Combine and denest powers.

- `powsimp(expr, deep, combine, force, measure)` — Combine powers with similar bases/exponents.
  - `combine='exp'`: merge `x^a · x^b → x^(a+b)`.
  - `combine='base'`: merge `x^a · y^a → (xy)^a`.
  - `combine='all'`: both (exp first).
  - Handles non-commutative terms, Rational exponent manipulation, and Mul-base radical joining.
- `powdenest(eq, force, polar)` — Collect exponents: `(b^e1)^e2 → b^(e1·e2)` when assumptions allow.
  - `force=True`: treat symbols as positive.
  - `polar=True`: simplify on the Riemann surface of log.
  - Also denests `exp(a*log(b))` → `b^a`.

---

## Combinatorial Simplification

### `combsimp.py`

Simplify expressions with factorials, binomials, and gamma functions.

- `combsimp(expr)` — Rewrite everything through rising factorials (`_rf`), apply recurrences, then convert back.
  - Reduces arguments of combinatorial functions.
  - For gamma-only expressions: applies reflection theorem, multiplication theorem, and factor absorption.
- `_rf` (class) — Internal rising factorial `Function` with auto-evaluation for integer arguments and additive decomposition.

---

## Rational Function Simplification

### `ratsimp.py`

Simplify rational functions.

- `ratsimp(expr)` — Put over common denominator, cancel, reduce via polynomial division.
- `ratsimpmodprime(expr, G, *gens, polynomial, quick)` — Simplify a rational expression modulo a prime ideal (Gröbner basis `G`).
  - Minimizes total degree of numerator + denominator.
  - `polynomial=True`: fast canonical form without degree minimization.
  - Based on Monagan & Pearce (2006).

---

## Common Subexpression Elimination

### `cse_main.py`

The CSE algorithm: find and extract repeated subexpressions.

- `cse(exprs, symbols, optimizations, postprocess, order)` — Main entry.
  - Returns `(replacements, reduced_exprs)` where each replacement is `(symbol, subexpr)`.
  - Supports Matrix inputs (dense and sparse); preserves type and mutability.
  - `optimizations='basic'`: apply `sub_pre`/`sub_post` and `factor_terms`.
  - `order='none'`: faster but hash-dependent ordering.
- `opt_cse(exprs, order)` — Find optimization opportunities (negative coefficients, common Add/Mul args).
- `tree_cse(exprs, symbols, opt_subs, order)` — Core tree-walk: find repeated subexpressions and rebuild with substitutions.
- `preprocess_for_cse`, `postprocess_for_cse` — Apply/undo optimization transforms.
- `reps_toposort(r)` — Topologically sort replacements so dependencies come first.
- `cse_separate(r, e)` — Postprocessor that extracts `Eq(symbol, expr)` from results.

### `cse_opts.py`

Pre/post-processing helpers for CSE.

- `sub_pre(e)` — Replace `y - x` with `-(x - y)` when minus sign is extractable. Enables better CSE matching.
- `sub_post(e)` — Undo `sub_pre`: replace `1·(-1)·x` back to `-x`.

---

## Hypergeometric Expansion

### `hyperexpand.py`

Expand hypergeometric and Meijer G-functions into named special functions (~2400 lines).

Based on Kelly B. Roach (1997): lookup-table + shift-operator algorithm.

**Key classes:**

- `Hyper_Function(ap, bq)` — Represents a generalized hypergeometric function `pFq(ap; bq; z)`.
  - `build_invariants()` — Compute mod-1 parameter signature for formula matching.
  - `difficulty(func)` — Estimate shift-operator steps to reach another function.
- `G_Function(an, ap, bm, bq)` — Represents a Meijer G-function.
  - `compute_buckets()` — Sort parameters by mod-1 residues.
- `Formula` — Stores a closed-form formula with differential system `z·d/dz B = M·B`, `closed_form = C·B`.
  - `find_instantiations(func)` — Match free parameters against a target function.
- `FormulaCollection` / `MeijerFormulaCollection` — Knowledge bases of formulae indexed by `(p, q)` / signature.
- `Operator` — Differential operators for shifting indices (polynomial in `z·d/dz`).
  - `ShiftA/B`, `UnShiftA/B` — Increment/decrement upper/lower hypergeometric indices.
  - `MeijerShiftA/B/C/D`, `MeijerUnShiftA/B/C/D` — Same for Meijer G parameters.
  - `ReduceOrder` — Cancel matching upper/lower index pairs.
  - `MultOperator` — Multiply by a constant.

**Key functions:**

- `hyperexpand(f, allow_hyper, rewrite)` — Top-level: expand `hyper()` and `meijerg()` in an expression.
- `_hyperexpand(func, z, ...)` — Core algorithm: reduce order → try polynomial → try shifted sum → lookup formula → devise plan → carry out plan.
- `_meijergexpand(func, z0, ...)` — Meijer G expansion via Slater's theorem (residue sums of gamma quotients).
- `devise_plan(target, origin, z)` — Compute a sequence of shift/unshift operators transforming `origin` to `target`.
- `reduce_order(func)` / `reduce_order_meijer(func)` — Cancel matching parameter pairs.
- `add_formulae(formulae)` — Populate the knowledge base with 0F0, 1F0, 2F1, 1F1, 0F1, 3F2, 2F2, 0F3, 1F2, 2F3, 3F3 formulae.
- `try_polynomial(func, z)` / `try_shifted_sum(func, z)` / `try_lerchphi(func)` — Recognize special cases.
- `hyperexpand_special(ap, bq, z)` — Classical summation formulae (Gauss, Kummer) at `z=1` or `z=-1`.

### `hyperexpand_doc.py`

Auto-generates a Sphinx docstring listing all hypergeometric formulae from `FormulaCollection` as LaTeX equations. No runtime functionality.

---

## Expression Path Tools

### `epathtools.py`

XPath-like manipulation of expression trees.

- `EPath(path)` — Compiled path object. Grammar supports type selectors, attribute queries, slices, and wildcards.
  - `select(expr)` — Retrieve matching subexpressions.
  - `apply(expr, func)` — Transform matching subexpressions in place.
- `epath(path, expr, func)` — Convenience wrapper; compiles path on each call.

### `traversaltools.py`

- `use(expr, func, level)` — Apply `func` to `expr` at a given depth level in the expression tree.

---

## Appendix

### File size reference

| File | Lines | Complexity |
|------|-------|------------|
| `hyperexpand.py` | ~2456 | Very high — formula tables, operator algebra, Slater's theorem |
| `fu.py` | ~2137 | High — 20+ transform rules, greedy search |
| `simplify.py` | ~1307 | High — heuristic orchestration of many strategies |
| `trigsimp.py` | ~1174 | High — Gröbner basis, pattern matching, Fu pipeline |
| `radsimp.py` | ~1079 | Moderate — conjugate rationalization, term collection |
| `powsimp.py` | ~702 | Moderate — power combination with radical joining |
| `sqrtdenest.py` | ~611 | Moderate — recursive denesting algorithms |
| `cse_main.py` | ~539 | Moderate — tree-walk CSE |
| `combsimp.py` | ~514 | Moderate — gamma function manipulation |
| `epathtools.py` | ~357 | Low — path parsing and tree traversal |
| `ratsimp.py` | ~219 | Low — polynomial ideal simplification |
| `cse_opts.py` | ~44 | Trivial |
| `traversaltools.py` | ~42 | Trivial |
| `hyperexpand_doc.py` | ~18 | Trivial — doc generation only |
