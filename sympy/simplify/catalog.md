# sympy/simplify -- Catalog

> Part of [SymPy](../catalog.md). Expression simplification: trigsimp, radsimp, powsimp, collect, CSE, hypergeometric simplification.

## Python Files

| File | Summary |
|------|----------|
| `__init__.py` | Package init that re-exports public APIs from submodules (simplify, trigsimp, radsimp, powsimp, combsimp, gammasimp, ratsimp, fu, sqrtdenest, cse, hyperexpand, epathtools, traversaltools). |
| `simplify.py` | Core simplification routines including `simplify`, `separatevars`, `nsimplify`, `besselsimp`, `hypersimp`, `hypersimilar`, `logcombine`, `posify`, `signsimp`, `bottom_up`, and internal helpers `_real_to_rational` (converts floats to rationals with tolerance handling, maps infinity to ComplexInfinity) and `clear_coefficients` (strips rational coefficients from expressions). |
| `trigsimp.py` | Trigonometric simplification via `trigsimp` and `exptrigsimp`, using Groebner basis and Fu algorithm strategies to reduce trig and hyperbolic expressions. Internal helpers include the division-rewrite dispatcher (`_match_div_rewrite`) that maps rule indices to trigonometric/hyperbolic power product rewrites (with indices 6 and 7 intentionally skipped), and `_replace_mul_fpowxgpow` for rewriting products of trigonometric powers. |
| `radsimp.py` | Radical simplification and term collection: `collect`, `rcollect`, `radsimp`, `collect_const`, `fraction`, `numer`, and `denom` for rationalizing denominators and grouping terms. |
| `powsimp.py` | Power simplification via `powsimp` (combine bases/exponents of powers) and `powdenest` (flatten nested exponentiation (b^a)^c → b^(a*c) under assumptions: positive base, integer outer exponent, or |inner exponent| < 1; supports `force` and `polar` parameters for extended denesting). |
| `ratsimp.py` | Rational expression simplification via `ratsimp` (common denominator, cancel, reduce) and `ratsimpmodprime` (simplification modulo a polynomial prime ideal using Groebner bases). |
| `combsimp.py` | Combinatorial simplification via `combsimp`, which reduces expressions involving factorials, binomials, and Pochhammer symbols by rewriting through gamma functions. |
| `gammasimp.py` | Gamma function simplification via `gammasimp`, applying rising-factorial rewriting, reflection theorem, and multiplication theorem to minimize gamma function count. |
| `sqrtdenest.py` | Denesting of nested square roots via `sqrtdenest`, with helpers for measuring sqrt depth and checking algebraic structure of radical expressions. |
| `fu.py` | Implementation of the Fu et al. trigonometric simplification algorithm, providing a library of numbered transform rules (TR0--TR22, TR111, TRmorrie) and combination transforms that are applied in heuristic order. |
| `cse_main.py` | Common subexpression elimination via `cse`, identifying repeated subexpressions and replacing them with temporary variables to reduce redundant computation. |
| `cse_opts.py` | Pre- and post-processing optimizations for CSE (`sub_pre`, `sub_post`) that canonicalize subtraction expressions to expose more common-subexpression opportunities. |
| `hyperexpand.py` | Expands hypergeometric and Meijer G-functions into named special functions using lookup tables and shift-operator algorithms based on Roach (1997). For Meijer G-functions, applies Slater's theorem to generate two candidate closed-form representations (direct and via variable transformation t=1/z), then selects the best via a weighting function that penalizes expressions containing infinities (oo, zoo, -oo, nan) and scores based on convergence conditions and remaining special function counts. |
| `hyperexpand_doc.py` | Auto-generates a Sphinx docstring listing all known hypergeometric function formulas from `FormulaCollection` for documentation purposes. |
| `traversaltools.py` | Provides the `use` function for applying a transformation to an expression at a specified depth level in the expression tree. |
| `epathtools.py` | Expression path tools: the `EPath` class and `epath` function for selecting and applying functions to parts of an expression tree using a path-based DSL. |
