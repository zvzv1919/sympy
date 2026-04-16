# sympy/simplify -- Catalog

> Part of [SymPy](../catalog.md). Expression simplification: trigsimp, radsimp, powsimp, collect, CSE, hypergeometric simplification.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package init that re-exports public APIs from submodules (simplify, trigsimp, radsimp, powsimp, combsimp, gammasimp, ratsimp, fu, sqrtdenest, cse, hyperexpand, epathtools, traversaltools). |
| `simplify.py` | Core general-purpose simplification. The main `simplify` function is the top-level expression reducer: for non-arithmetic types (not Add/Mul/Pow/Exp) it detects single-argument function-inverse compositions (e.g., asin(sin(x)) → x) and recursively simplifies sub-expressions; for arithmetic types it orchestrates multiple strategies (cancel, together, factor_terms, hyperexpand, trigsimp, logcombine, combsimp, powsimp, etc.) and picks the shortest result. Also provides `separatevars`, `nsimplify`, `besselsimp`, `hypersimp`, `hypersimilar`, `logcombine`, `posify`, `signsimp`, and `bottom_up`. |
| `trigsimp.py` | Trigonometric simplification via `trigsimp` and `exptrigsimp`. Includes a recursive `__trigsimp` helper that applies pattern-matched identity substitutions to trig/hyperbolic expressions and falls back to exponential rewriting if trig functions remain. Contains `_replace_mul_fpowxgpow` and `_match_div_rewrite` which convert products of different trig functions into quotient forms (e.g., sin(x)^n * cos(x)^m → tan(x)^k) by matching common arguments, guarded by positivity or integer-exponent conditions. Also provides `trigsimp_groebner` for polynomial Groebner-basis-based trig reduction. |
| `radsimp.py` | Radical simplification and term collection: `collect`, `rcollect`, `radsimp`, `collect_const`, `fraction`, `numer`, and `denom` for rationalizing denominators and grouping terms. |
| `powsimp.py` | Power simplification via `powsimp` (combine bases/exponents of powers) and `powdenest` (simplify nested power expressions). |
| `ratsimp.py` | Rational expression simplification via `ratsimp` (common denominator, cancel, reduce) and `ratsimpmodprime` (simplification modulo a polynomial prime ideal using Groebner bases). |
| `combsimp.py` | Combinatorial simplification via `combsimp`, which reduces expressions involving factorials, binomials, and Pochhammer symbols by rewriting through gamma functions. |
| `gammasimp.py` | Gamma function simplification via `gammasimp`, applying rising-factorial rewriting, reflection theorem, and multiplication theorem to minimize gamma function count. |
| `sqrtdenest.py` | Denesting of nested square roots via `sqrtdenest`, with helpers for measuring sqrt depth and checking algebraic structure of radical expressions. |
| `fu.py` | Implementation of the Fu et al. trigonometric simplification algorithm, providing a library of numbered transform rules (TR0--TR22, TR111, TRmorrie) applied in heuristic order. Key transforms include: `_TR56` (helper for TR5/TR6) replaces even powers of a trig function with its Pythagorean identity complement (e.g., sin²→1−cos², sin⁸→(1−cos²)⁴), controlled by a `max` exponent cap and a `pow` flag that restricts rewrites to exponents that are perfect powers of 2; TR9 converts sums of sin/cos into products via pairwise combination with recursive re-application for 3+ terms; TR14 simplifies factored sin/cos powers like (cos(x)±1)^n into simpler trig powers; TR8 expands products of sin/cos to sums. Also provides combination routines (`fu`, `CTR1`–`CTR4`) orchestrating rule sequences. |
| `cse_main.py` | Common subexpression elimination via `cse`, identifying repeated subexpressions and replacing them with temporary variables to reduce redundant computation. |
| `cse_opts.py` | Pre- and post-processing optimizations for CSE (`sub_pre`, `sub_post`) that canonicalize subtraction expressions to expose more common-subexpression opportunities. |
| `hyperexpand.py` | Expands hypergeometric and Meijer G-functions into named special functions using lookup tables and shift-operator algorithms based on Roach (1997). |
| `hyperexpand_doc.py` | Auto-generates a Sphinx docstring listing all known hypergeometric function formulas from `FormulaCollection` for documentation purposes. |
| `traversaltools.py` | Provides the `use` function for applying a transformation to an expression at a specified depth level in the expression tree. |
| `epathtools.py` | Expression path tools: the `EPath` class and `epath` function for selecting and applying functions to parts of an expression tree using a path-based DSL. |
