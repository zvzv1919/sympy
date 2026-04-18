# Solvers Module Catalog

## Glossary
- **Legacy solver** (`solvers.py`): returns lists/dicts (or `set=True` for `(keys, {tuples})` format); heuristic dispatch; handles Piecewise branch logic directly.
- **Set-based solver** (`solveset.py`): returns Set objects (FiniteSet, ConditionSet, ImageSet); explicit domain; systematic inversion strategy.

## Notes
- Piecewise/conditional-expression solving (branch priority, piecewise_fold) lives in `solvers.py`, not `solveset.py`.
- `solveset.py` represents conditional/unsolved results as ConditionSet, not Piecewise.

---

## Package Entry Point

### [`__init__.py`](__init__.py)
Public namespace for the solvers package. Assembles and re-exports all user-facing functions from submodules: general solving (`solvers.py`), integer-only/Diophantine (`diophantine.py`), recurrence relations (`recurr.py`), ODEs (`ode.py`), PDEs (`pde.py`), polynomial systems (`polysys.py`), inequalities (`inequalities.py`), functional decomposition (`decompogen.py`), set-based solving (`solveset.py`), and DE utilities (`deutils.py`).

---

## Core Equation Solvers

### [`solvers.py`](solvers.py)
Legacy general-purpose algebraic equation solver. Returns solutions as lists or dicts.

- `solve(f, *symbols, **flags)` — primary entry point for equations and systems; dispatches to `_solve`, `_solve_system`, or linear helpers. Accepts non-Symbol solve targets (Indexed elements, derivatives, sub-expressions like `x+2`) — isolates them algebraically via substitution with a temporary Symbol.
  - Output format flags: `dict=True` returns list of {symbol: value} dicts; `set=True` returns `(sorted_keys, {value_tuples})` tuple built from those dicts. Default returns plain list.
  - Preprocessing: rewrites hyperbolics as exp; splits real/imag parts; rewrites Abs as Piecewise (raises NotImplementedError if argument's real/imaginary status is unknown); rewrites `arg` as `atan(im/re)`.
  - Solution validation: automatically excludes candidates that make any denominator zero (via `denoms`); `check=False` flag bypasses both denominator filtering and assumption checks, recovering all raw candidates.
- `_solve_system(exprs, symbols)` — internal system solver (used by `solve` for multi-equation inputs); handles:
  - Linear systems: converts each expression to Poly, extracts monomial coefficients to build an augmented matrix in-place, then dispatches to `solve_linear_system`.
  - Nonlinear polynomial systems via `solve_poly_system`.
  - Underdetermined nonlinear systems: enumerates variable subsets sized to match equation count, solves each subset via `solve_poly_system`, discards solutions whose free symbols overlap previously solved variables (dependent-solution filter); does **not** produce parametric infinite-family solutions (see `linsolve` for that).
  - Residual non-polynomial equations: iteratively solves remaining symbols one at a time after polynomial pass.
- `_solve(f, symbol, **flags)` — internal single-equation solver; handles:
  - Multi-symbol sequential resolution: solves for each symbol in turn; discards solutions whose free symbols depend on a previously solved symbol.
  - Piecewise/conditional expressions: iterates branches, enforces branch-priority (earlier-branch exclusion) via `piecewise_fold`.
  - Linear equations via `solve_linear`.
  - Polynomial dispatch via `Poly` and generator inspection.
  - Multi-generator same-base handling: when generators share one base but differ in power (e.g. exp(x), exp(-x)), expands powers before substituting the base with a dummy variable.
  - Multi-generator different-base handling: for nested transcendental functions (e.g. log(x) and log(log(x)-1)), substitutes the shallowest function with a dummy variable, solves the simplified equation, then inverts to recover solutions.
  - Transcendental fallback via `_tsolve`.
- `solve_linear(lhs, rhs)` — fast linear-equation solver for one or more variables. Accepts Equality as `lhs` (extracts sides internally); raises ValueError if `lhs` is an Equality and `rhs` is nonzero (ambiguous RHS). Returns `(symbol, solution)` if linear, `(0, 1)` if trivially zero, `(0, 0)` if no solution, or `(numer, denom)` if not linear.
- `solve_linear_system(matrix, *syms)` — internal linear system solver from augmented matrix (use `linsolve` in `solveset.py` for the user-facing equivalent).
- `solve_undetermined_coeffs(equ, coeffs, sym)` — solves for unknown algebraic coefficients in a polynomial identity (not ODE-related; see `ode.py` for the ODE undetermined coefficients method).
- Post-solve assumption filtering: checks each candidate against the symbol's declared properties (e.g. positive, real) via `check_assumptions`.
  - Drops solutions that definitively violate assumptions (test=False); keeps solutions where verification is inconclusive (test=None) with optional warning.
  - Relational/inequality solutions (Relational, And, Or): assumptions on the variable are **not** verified — only a warning is emitted. Raises ValueError if more than one symbol is involved.
- `checksol(f, symbol, sol)` — validates a candidate algebraic-equation solution by substitution; does not handle ODE verification (see `checkodesol`) or domain/singularity checks (see `domain_check`).
- `nsolve(*args, **kwargs)` — numerical root-finding via mpmath.
- `_invert(eq, *symbols)` — algebraic inversion loop returning `(independent, dependent)` scalar tuple by recursively peeling additive/multiplicative layers, function inverses (single-arg via `.inverse()`), and special-case atan2 rewriting. Handles Pow with principal roots.
- `_tsolve(eq, sym)` — transcendental equation solver (exp, log, trig inversions, Pow); delegates exp/log-to-Lambert-W reduction to `bivariate._solve_lambert`.
  - Pow handling: integer exponents, symbol-free exponents, and `f(x)**g(x)=0` (solves base, excludes solutions where exponent is also zero to avoid 0^0).
  - Lambert W fallback orchestration: classifies generators into exp/log vs algebraic, factors polynomial part, attempts `_solve_lambert`.
  - On Lambert W failure with exactly 2 generators, falls back to `bivariate_type` reduction within `_tsolve` itself (not in bivariate.py).
  - Last-resort `force` fallback: calls `posify` to re-express the equation with all symbols assumed positive, then re-solves. If the target variable is absent from the positified expression, returns None (no solution).
- `unrad(eq, *syms)` — removes radicals from equations.
- `denoms(eq, symbols)` — extracts denominators; used by `solve` post-validation to auto-discard solutions causing zero denominators.

### [`solveset.py`](solveset.py)
Modern set-based solver with explicit domain handling. Returns FiniteSet, Interval, ConditionSet, or ImageSet.

- `solveset(f, symbol, domain=S.Complexes)` — core solver; dispatches by domain.
  - Boolean vs numeric edge case: boolean `True` (from a relational always satisfied) returns full domain; `False` returns EmptySet.
  - Numeric `0` also returns domain (Eq(0,0) is true), but `1` returns EmptySet. Boolean/numeric dispatch occurs before expression analysis.
  - Constant (variable-free) expressions: returns domain if equal to 0, EmptySet if nonzero; raises NotImplementedError if equality to zero is undetermined.
  - Relational/inequality inputs (real domain only): delegates to `solve_univariate_inequality`, then subtracts denominator-zero points (via `_invalid_solutions`) from the result; falls back to ConditionSet on NotImplementedError.
- `_invalid_solutions(f, symbol, domain)` — collects zeros of all denominators in `f` to exclude undefined points from solution sets.
- `solveset_real(f, symbol)` / `solveset_complex(f, symbol)` — domain-specific wrappers.
- `linsolve(system, *symbols)` — primary user-facing linear system solver (Gauss-Jordan elimination) returning FiniteSet of ordered solution tuples. Validates that all `symbols` are actual Symbol instances; raises ValueError if non-symbolic values (e.g. integers, strings) are passed.
  - Accepts three input forms: (A, b) matrix pair, list of equations, or augmented matrix.
  - Underdetermined systems: replaces internally generated placeholder parameters with the caller's original symbols, so the parametric solution tuple is expressed in the user's own unknowns.
- `linear_eq_to_matrix(equations, *symbols)` — standalone utility that converts linear equations to (A, b) matrix pair for external use. Does not solve; just extracts coefficients. Accepts both expressions (implicit =0) and Eq() relations.
- `domain_check(f, symbol, p)` — solveset-internal singularity check; walks expression tree for infinite subexpressions at a candidate point. Not used by legacy `solve` (which has its own denominator-zero filter). Caveat: misses singularities if auto-simplification has already reduced the expression (e.g. x/x → 1).
- `_invert(f_x, y, x, domain)` — set-based function inversion; reduces f(x)=y to simpler form. Returns solution sets (FiniteSet/ImageSet). Distinct from `solvers._invert` which uses algebraic peeling and returns scalar tuples.
  - Caveat: domain intersection (filtering against reals/complexes) is applied only when the result is a FiniteSet; infinite/continuous solution sets (ImageSet, Union, etc.) pass through unfiltered.
- `invert_real` / `invert_complex` — domain-specific inversion helpers.
  - `_invert_real` recursively inverts real-valued functions; Abs handling: splits into positive branch ([0,∞) kept as-is) and negative branch ((-∞,0] negated), then unions results.
  - `_invert_complex` exp handling: maps each target value to an ImageSet over Integers (adding 2nπi branches); requires `g_ys` to be a FiniteSet.
  - Caveat: silently returns the expression unchanged for infinite target sets (Integers, Union, etc.) — exp inversion only proceeds for finite discrete inputs.
- `_solve_as_poly(f, symbol, domain)` — solves via polynomial techniques (roots, Poly.all_roots); falls back to ConditionSet when root count is incomplete.
  - Post-processing: simplifies complex solutions via `expand_complex` (e.g. `-sqrt(-I)` → `sqrt(2)/2 - sqrt(2)*I/2`) only when all solutions are fully numeric (no free symbols) and none are `RootOf` objects; skips simplification otherwise to avoid complicating parametric/algebraic results.
- `_is_function_class_equation(func_class, f, symbol)` — tests whether an equation is composed exclusively of a given function family (e.g. TrigonometricFunction, HyperbolicFunction) with linear-in-symbol arguments, combined only with symbol-independent terms. Recursively checks Add/Mul/Pow structure.
- `_solve_as_rational`, `_solve_trig` — type-specific internal solvers.
- `_solveset(f, symbol, domain, _check=False)` — internal helper that dispatches to type-specific solvers and optionally validates results.
  - Product decomposition: decomposes `f*g == 0` into `Union(f==0, g==0)` only when all factors are verified finite for finite inputs (`_is_finite_with_finite_vars`); prevents spurious solutions where one factor diverges at zeros of another.
  - Post-solve validation (`_check=True`): for FiniteSet results, filters out invalid candidates via `domain_check`, but exempts `RootOf` (implicit algebraic root) objects from validation. ConditionSet results bypass checking entirely.
- `_solve_radical(f, symbol, solveset_solver)` — solves equations with radicals via `unrad`; when a cover (substitution) variable is returned, tests whether it can equal I — if not, replaces it with a real-constrained dummy before solving. Filters final candidates through `checksol` to eliminate extraneous solutions introduced by radical removal (e.g. squaring both sides).
- `_solve_abs(f, symbol, domain)` — solves equations involving Abs; real domain only (raises ValueError for complex domain). Decomposes `p*|q| + r` into two cases: solves with `q` non-negative and with `q` negative, intersecting each solution with the corresponding sign condition on the argument.
- Represents unsolved/conditional results as ConditionSet (not Piecewise).

---

## Specialized Solvers

### [`bivariate.py`](bivariate.py)
Solves bivariate equations by structural reduction to single-variable problems.

- `_linab(arg, symbol)` — decomposes expression into `a*X + b` where `X` is symbol-dependent and `a`, `b` are independent; normalizes negative leading sign on `X` by negating both `a` and `X`.
- `_mostfunc(lhs, func, X=None)` — selects the most deeply nested occurrence of a given function type (exp, log, Pow, etc.) in an expression; ties broken by highest nesting count; optional variable filter restricts candidates.
- `_lambert(eq, x)` — low-level Lambert W solver for equations in the form `a*log(b*X+c) + d*X + f = 0`. Evaluates both real branches of LambertW (k=0 and k=-1), discarding k=-1 when the result is not real. Handles nested-log edge case: if the non-log remainder is itself a negated log, unwraps one layer and rewrites the equation before solving.
- `_solve_lambert(f, symbol, gens)` — reduces transcendental equations mixing exp/log/symbolic-exponent powers to Lambert W form. Cascades through log-dominant, exp-dominant, and power-with-symbolic-exponent cases, branching on additive vs multiplicative structure.
  - Log-dominant + additive lhs: computes log-difference differently when rhs==0 (`log(other) - log(other - lhs)`) vs nonzero (`log(lhs - other) - log(rhs - other)`).
  - Exp-dominant + additive lhs: isolates exp-containing term; if both sides are negatable (`could_extract_minus_sign`), negates both before taking log. This negation check is absent in the analogous power-with-symbolic-exponent additive case.
  - Calls `_lambert` for final resolution.
- `bivariate_type(f, x, y)` — classifies a two-variable expression into composite substitution forms: product (`x*y`), linear sum (`a*x+b*y`), or mixed (`a*x*y+b*y`/`a*x*y+b*x`).
  - Returns `(u(x,y), P(u), u)` so solving `P(u)=0` then equating `u(x,y)=solution` recovers solutions for x or y.
  - On first call, replaces original symbols with temporary placeholders (Dummy) before recursing, preventing symbol-name collisions during classification.

### [`diophantine.py`](diophantine.py)
Solves Diophantine equations (polynomial equations over integers).

- `diophantine(eq, param, syms)` — main entry; factors equation into terms, dispatches each to `diop_solve()`, and merges partial solutions into full-length tuples via `merge_solution`.
  - When the expression has unknowns in the denominator, solves numerator and denominator independently and filters out solutions that make the denominator vanish.
- `merge_solution(var, var_t, solution)` — constructs full solution tuples from sub-equation solutions that involve only a subset of variables. Fills missing variables with fresh integer parameters (`n1, n2, …`). Returns empty tuple if any value violates its symbol's declared assumptions (e.g. positivity).
- `classify_diop(eq)` — classifies equation type (linear, quadratic, ternary, Pell, etc.).
- Integer arithmetic helpers: `_nint_or_floor` (nearest-integer rounding with floor as tie-breaker), `_rational_pq`, `_remove_gcd`.
- Descent solvers for ternary quadratics: `ldescent(A, B)` — finds non-trivial solution to w²=Ax²+By² via Lagrange's method; returns None when no solution exists (e.g. both A and B are -1). `descent(A, B)` — same problem but uses Gaussian lattice reduction for speed.
- Type solvers: `diop_linear`, `diop_quadratic`, `diop_ternary_quadratic` / `_diop_ternary_quadratic`, `diop_DN`, `cornacchia`.
  - `_diop_ternary_quadratic`: cross-product-only case (no squared terms): if xz coefficient is nonzero, reduces to binary quadratic and picks solution minimizing |x|+|z|; if xz coefficient is zero, swaps variables and recurses.
  - `diop_linear` / `_diop_linear` — solves linear Diophantine equations (a₁x₁+…+aₙxₙ=c) by recursively reducing n-variable problems to two-variable GCD sub-problems; returns parametric solutions with integer parameters.
  - `diop_quadratic` / `_diop_quadratic` — solves binary quadratic Diophantine equations (Ax²+Bxy+Cy²+Dx+Ey+F=0) by discriminant-based case dispatch: simple-hyperbolic (A=C=0), parabolic (B²−4AC=0, including variable-swap when A=0), square discriminant, and general case.
- Sum-of-powers solvers: `diop_general_sum_of_squares`, `diop_general_sum_of_even_powers` — solve x₁^e+…+xₙ^e=k over integers; respects variable assumptions (e.g. nonpositive) by flipping signs on results.
- `power_representation(n, p, k)` — generates representations of n as sum of k p-th powers. `partition(n, k)` — integer partition generator.

### [`inequalities.py`](inequalities.py)
Solves inequality constraints and returns interval-based solutions.

- `reduce_inequalities(inequalities, symbols)` — general entry point for mixed inequality systems.
- `solve_univariate_inequality(expr, gen)` — solves a single real-valued univariate inequality. Substitutes a real-constrained dummy for the generator to decouple from user assumptions on the original symbol.
- `solve_poly_inequality(poly, rel)` — polynomial inequality → interval list.
- `solve_rational_inequalities(eqs)` — rational expression inequalities.
- `reduce_abs_inequality` / `reduce_abs_inequalities` — absolute value inequalities.

### [`polysys.py`](polysys.py)
Solves zero-dimensional (fully determined) systems of polynomial equations via Groebner bases. Does not handle underdetermined subset enumeration — that logic lives in `_solve_system` in `solvers.py`.

- `solve_poly_system(seq, *gens)` — general polynomial system solver; requires #equations ≥ #variables for a finite solution set.
- `solve_biquadratic(f, g, opt)` — two bivariate quadratic equations via Groebner basis. Returns None if the basis is a single constant (inconsistent system, no solutions). Raises `SolveFailed` if the basis has more than 2 elements (caller falls back to `solve_generic`).
- `solve_generic(polys, opt)` — zero-dimensional systems via Groebner basis elimination.
- `solve_triangulated(polys, *gens)` — Gianni-Kalkbrenner triangulation algorithm.

---

## Differential & Recurrence Equation Solvers

### [`ode.py`](ode.py)
Solves ordinary differential equations via classification and hint-based dispatch.

- `dsolve(eq, func, hint, ics)` — main ODE solver; classifies then applies best method.
- `classify_ode(eq, func)` — classifies ODE into applicable solving hints without solving.
- `checkodesol(ode, sol, func)` — validates ODE solution via multi-pass verification:
  - If `func` is omitted, auto-detects via `_preprocess`; on failure, falls back to extracting applied undefined functions from the solution(s) (raises ValueError if not exactly one found).
  - Pass 1: direct substitution of solved f(x) into the ODE.
  - Pass 2: compares nth derivatives of both sides (for exact ODEs).
  - Pass 3: computes successive derivatives of candidate, solves for each d^n f/dx^n, then back-substitutes into ODE in decreasing order (n, n-1, …, 0).
- `homogeneous_order(expr, *symbols)` — computes homogeneity order.
- Methods: separable, exact, linear (1st/nth), Bernoulli, Lie group, variation of parameters, undetermined coefficients, power series.
- `_solve_variation_of_parameters(eq, func, order, match)` — builds particular solution for nonhomogeneous linear ODEs via parameter variation. Computes Wronskian of homogeneous solutions (with trig simplification); raises NotImplementedError if Wronskian is zero (linearly dependent solutions) or solution count is insufficient.
- System-of-ODE solvers: `sysode_nonlinear_2eq_order1`, `sysode_nonlinear_3eq_order1` — dispatch to type-specific solvers for coupled nonlinear first-order systems (2-eq and 3-eq).
  - Includes Clairaut system solver (type5 for 2-eq): pattern-matches `x = t*x' + F(x',y')` in multiple algebraic forms; swaps dependent variables if initial match fails.
- `_undetermined_coefficients_match(expr, x)` — tests applicability and builds trial solution terms for the undetermined coefficients method.
  - `_get_trial_set` generates candidate terms by repeated differentiation until the set stabilizes; dispatches recursively when a derivative produces a sum.

### [`pde.py`](pde.py)
Solves partial differential equations via method dispatch.

- `pdsolve(eq, func, hint)` — main PDE solver; supports meta-hints "all"/"all_Integral" returning a dict where failed strategies store the NotImplementedError exception object as value.
- `classify_pde(eq, func)` — classifies PDE into applicable hints.
  - Pre-classification normalization: if the unknown function raised to some power multiplies highest-order derivative coefficients, divides the entire equation by the smallest such power to reduce to standard form.
  - First-order two-variable classification: uses two-pass pattern matching — first attempts with wildcards excluding independent variables (constant coefficients); on failure, relaxes wildcards to allow dependence on independent variables (variable coefficients).
- `_helper_simplify(eq, hint, func, order, match, solvefun)` — internal dispatch that routes to the correct `pde_<hint>` solver function via `globals()` lookup. Strips `_Integral` suffix before lookup so integral-form hints reuse the same solver with unevaluated integrals.
- `checkpdesol(pde, sol, func)` — validates PDE solution by substitution.
  - If `func` is omitted, auto-detects via `_preprocess`; on failure, falls back to extracting applied undefined functions from the solution's atoms (raises ValueError if not exactly one found).
  - When the candidate is not isolated for the dependent function, attempts `solve` to isolate; if multiple roots, recursively checks each one.
- `_handle_Integral(expr, func, order, hint)` — post-processes PDE solutions containing unevaluated integrals.
  - Hint suffix `_Integral` preserves raw integral form; `1st_linear_constant_coeff` triggers `doit()` + `simplify`; all others return unchanged.
- `pde_separate(eq, fun, sep, strategy)` — separates a PDE by substituting a product (or sum) of single-variable functions for the dependent variable.
  - Multiplicative mode: after substitution, divides each term by the full product to normalize into a form where variables can be isolated to different sides.
  - Delegates to `_separate`, which uses a two-pass algorithm: first extracts derivative terms depending only on the target variable and finds divisors, then splits remaining terms into left/right sides.
- `pde_separate_add` / `pde_separate_mul` — convenience wrappers calling `pde_separate` with additive or multiplicative strategy.

### [`recurr.py`](recurr.py)
Solves recurrence (difference) equations with polynomial/rational coefficients.

- `rsolve(f, y, init)` — main entry; dispatches to type solver.
- `rsolve_poly`, `rsolve_ratio` — polynomial and rational RHS solvers.
- `rsolve_hyper` — hypergeometric RHS solver. Groups pairwise hyper-similar terms (consecutive-term ratio is rational) in the forcing function.
  - Finds particular solutions for each group via Abramov's algorithm. Returns None if any summand is not hypergeometric.

---

## Utilities

### [`decompogen.py`](decompogen.py)
Functional decomposition for solving via composition chain reduction.

- `decompogen(f, symbol)` — decomposes a single-variable expression f into a composition chain f = f₁∘f₂∘…∘fₙ. Pure decomposition utility; does not classify multivariate substitution forms (see `bivariate_type`) or solve equations.

### [`deutils.py`](deutils.py)
Utilities for classifying and manipulating differential equations.

- `ode_order(expr, func)` — returns the order of a differential equation.
- `_preprocess(expr, func, hint)` — controls derivative evaluation before ODE/PDE solving. Evaluates unevaluated derivatives selectively based on hint suffix; if hint is None, bypasses all derivative evaluation entirely. Auto-detects the target function from derivatives if func is omitted (raises ValueError if ambiguous).
- `_desolve(eq, func, hint, ics)` — shared dispatch helper used by both `dsolve` (ODE) and `pdsolve` (PDE).
  - Delegates to `classify_ode` or `classify_pde` based on `type` kwarg.
  - Handles meta-hints `all`, `all_Integral`, and `best`: iterates matching hints, collects solutions into a dict.
  - `all_Integral` filtering: removes non-Integral variants that have an `_Integral` counterpart, and explicitly excludes strategies without an `_Integral` form (e.g. power series, Lie group, homogeneous coeff best).
  - On recursive calls, accepts `classify=False` to skip re-classification and reuse previously computed hints/match/order from kwargs.
  - Validates order > 0; raises ValueError if order is 0 (not a DE). Three-way error branching when no default hint: unrecognized hint → ValueError, non-matching hint → ValueError, no method works → NotImplementedError.
