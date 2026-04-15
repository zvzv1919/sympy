# sympy/simplify -- Catalog

> Part of [SymPy](../catalog.md). Expression simplification: trigsimp, radsimp, powsimp, collect, CSE, hypergeometric simplification.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package init that re-exports public APIs from submodules (simplify, trigsimp, radsimp, powsimp, combsimp, gammasimp, ratsimp, fu, sqrtdenest, cse, hyperexpand, epathtools, traversaltools). |
| `simplify.py` | Core simplification routines including `simplify`, `separatevars`, `nsimplify`, `besselsimp`, `logcombine`, `posify`, `signsimp`, and `bottom_up`. Also contains `hypersimp` (simplifies the consecutive-term ratio f(k+1)/f(k) of a combinatorial expression by rewriting in terms of gamma functions and reducing; returns None if the result is not a rational function in k) and `hypersimilar` (tests hyper-similarity). |
| `trigsimp.py` | Trigonometric and hyperbolic simplification via `trigsimp` and `exptrigsimp`. Contains ordered pattern-matching rewrite-rule tables (ratio rewrites like sin/cos→tan, angle-sum combination rules, Pythagorean-identity rewrites, and artifact cleanup) used by the pattern-driven strategy, as well as a Groebner-basis strategy that analyses generators of trig/hyperbolic terms, decomposes ratio-type functions (e.g. tan into sin and cos), constructs polynomial ideals, and reduces via Groebner bases. See also `fu.py` for the separate Fu-algorithm transform rules. |
| `radsimp.py` | Radical simplification and term collection: `collect`, `rcollect`, `radsimp`, `collect_const`, `fraction`, `numer`, and `denom` for rationalizing denominators and grouping terms. |
| `powsimp.py` | Power simplification via `powsimp` (combine bases/exponents of powers) and `powdenest` (simplify nested power expressions). |
| `ratsimp.py` | Rational expression simplification via `ratsimp` (common denominator, cancel, reduce) and `ratsimpmodprime` (simplification of rational expressions modulo a given polynomial prime ideal using Groebner bases). Does not construct ideals from trig/hyperbolic generators; for Groebner-based trig ideal construction and generator analysis see `trigsimp.py`. |
| `combsimp.py` | Combinatorial simplification via `combsimp`, which reduces expressions involving factorials, binomials, and Pochhammer symbols by rewriting through gamma functions. Does not handle consecutive-term ratio analysis (see `hypersimp` in `simplify.py` for that). |
| `gammasimp.py` | Gamma function simplification via `gammasimp`, applying rising-factorial rewriting, reflection theorem, and multiplication theorem to minimize gamma function count. |
| `sqrtdenest.py` | Denesting of nested square roots via `sqrtdenest`, with helpers for measuring sqrt depth and checking algebraic structure of radical expressions. |
| `fu.py` | Implementation of the Fu et al. trigonometric simplification algorithm, providing a library of numbered transform rules (TR0--TR22, TR111, TRmorrie) and combination transforms that are applied in heuristic order. These are expression-level transforms, not the pattern-matching rewrite-rule tables or Groebner-based generator analysis found in `trigsimp.py`. |
| `cse_main.py` | Common subexpression elimination via `cse`, identifying repeated subexpressions and replacing them with temporary variables to reduce redundant computation. |
| `cse_opts.py` | Pre- and post-processing optimizations for CSE (`sub_pre`, `sub_post`) that canonicalize subtraction expressions to expose more common-subexpression opportunities. |
| `hyperexpand.py` | Expands hypergeometric and Meijer G-functions into named special functions using lookup tables and shift-operator algorithms based on Roach (1997). |
| `hyperexpand_doc.py` | Auto-generates a Sphinx docstring listing all known hypergeometric function formulas from `FormulaCollection` for documentation purposes. |
| `traversaltools.py` | Provides the `use` function for applying a transformation to an expression at a specified depth level in the expression tree. |
| `epathtools.py` | Expression path tools: the `EPath` class and `epath` function for selecting and applying functions to parts of an expression tree using a path-based DSL. |
