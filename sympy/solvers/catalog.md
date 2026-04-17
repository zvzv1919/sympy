# Solvers Module Catalog

## Glossary
- **Legacy solver** (`solvers.py`): returns lists/dicts; heuristic dispatch; handles Piecewise branch logic directly.
- **Set-based solver** (`solveset.py`): returns Set objects (FiniteSet, ConditionSet, ImageSet); explicit domain; systematic inversion strategy.

## Notes
- Piecewise/conditional-expression solving (branch priority, piecewise_fold) lives in `solvers.py`, not `solveset.py`.
- `solveset.py` represents conditional/unsolved results as ConditionSet, not Piecewise.

---

## Core Equation Solvers

### [`solvers.py`](solvers.py)
Legacy general-purpose algebraic equation solver. Returns solutions as lists or dicts.

- `solve(f, *symbols, **flags)` — primary entry point for equations and systems; dispatches to `_solve`, `_solve_system`, or linear helpers. Can target non-symbol objects (numeric literals, compound expressions) via implicit substitution.
  - Preprocessing: rewrites hyperbolics as exp; splits real/imag parts; rewrites Abs as Piecewise (raises NotImplementedError if argument's real/imaginary status is unknown); rewrites `arg` as `atan(im/re)`.
- `_solve_system(exprs, symbols)` — internal system solver; handles:
  - Linear systems via augmented matrix construction → `solve_linear_system`.
  - Nonlinear polynomial systems via `solve_poly_system`.
  - Underdetermined systems (more unknowns than equations): enumerates variable subsets sized to match equation count, solves each subset, discards solutions that reference previously determined variables.
  - Residual non-polynomial equations: iteratively solves remaining symbols one at a time after polynomial pass.
- `_solve(f, symbol, **flags)` — internal single-equation solver; handles:
  - Multi-symbol sequential resolution: solves for each symbol in turn; discards solutions whose free symbols depend on a previously solved symbol.
  - Piecewise/conditional expressions: iterates branches, enforces branch-priority (earlier-branch exclusion) via `piecewise_fold`.
  - Linear equations via `solve_linear`.
  - Polynomial dispatch via `Poly` and generator inspection.
  - Transcendental fallback via `_tsolve`.
- `solve_linear(lhs, rhs)` — fast linear-equation solver for one or more variables.
- `solve_linear_system(matrix, *syms)` — linear system from augmented matrix.
- `solve_undetermined_coeffs(equ, coeffs, sym)` — determines polynomial coefficients.
- `checksol(f, symbol, sol)` — validates a candidate solution by substitution.
- `nsolve(*args, **kwargs)` — numerical root-finding via mpmath.
- `_invert(eq, *symbols)` — algebraic inversion loop returning `(independent, dependent)` scalar tuple by recursively peeling additive/multiplicative layers, function inverses (single-arg via `.inverse()`), and special-case atan2 rewriting. Handles Pow with principal roots.
- `_tsolve(eq, sym)` — transcendental equation solver (exp, log, trig inversions, Pow); delegates exp/log-to-Lambert-W reduction to `bivariate._solve_lambert`.
  - Pow handling: integer exponents, symbol-free exponents, and `f(x)**g(x)=0` (solves base, excludes solutions where exponent is also zero to avoid 0^0).
- `unrad(eq, *syms)` — removes radicals from equations.
- `denoms(eq, symbols)` — extracts denominators for solution validation.

### [`solveset.py`](solveset.py)
Modern set-based solver with explicit domain handling. Returns FiniteSet, Interval, ConditionSet, or ImageSet.

- `solveset(f, symbol, domain=S.Complexes)` — core solver; dispatches by domain.
  - Relational/inequality inputs (real domain only): delegates to `solve_univariate_inequality`; falls back to ConditionSet on NotImplementedError.
- `solveset_real(f, symbol)` / `solveset_complex(f, symbol)` — domain-specific wrappers.
- `linsolve(system, *symbols)` — linear system solver returning set of solution tuples.
- `linear_eq_to_matrix(equations, *symbols)` — converts linear equations to augmented matrix form (A, b). Accepts both expressions (implicit =0) and Eq() relations.
- `domain_check(f, symbol, p)` — validates candidate solution point by walking the expression tree for singularities (infinite subexpressions). Caveat: misses singularities if auto-simplification has already reduced the expression (e.g. x/x → 1).
- `_invert(f_x, y, x, domain)` — set-based function inversion; reduces f(x)=y to simpler form. Returns solution sets (FiniteSet/ImageSet). Distinct from `solvers._invert` which uses algebraic peeling and returns scalar tuples.
- `invert_real` / `invert_complex` — domain-specific inversion helpers.
- `_solve_as_poly`, `_solve_as_rational`, `_solve_trig`, `_solve_radical`, `_solve_abs` — type-specific internal solvers.
- Represents unsolved/conditional results as ConditionSet (not Piecewise).

---

## Specialized Solvers

### [`bivariate.py`](bivariate.py)
Solves bivariate equations by structural reduction to single-variable problems.

- `_linab(arg, symbol)` — decomposes expression into `a*X + b` where `X` is symbol-dependent and `a`, `b` are independent; normalizes negative leading sign on `X` by negating both `a` and `X`.
- `_mostfunc(lhs, func, X=None)` — selects the most deeply nested occurrence of a given function type (exp, log, Pow, etc.) in an expression; ties broken by highest nesting count; optional variable filter restricts candidates.
- `_solve_lambert(f, symbol, gens)` — reduces transcendental equations mixing exp/log/symbolic-exponent powers to Lambert W form. Cascades through log-dominant, exp-dominant, and power-with-symbolic-exponent cases, branching on additive vs multiplicative structure.
- `bivariate_type(f, x, y)` — classifies bivariate equation structure.

### [`diophantine.py`](diophantine.py)
Solves Diophantine equations (polynomial equations over integers).

- `diophantine(eq, param, syms)` — main entry; classifies and dispatches to type-specific solvers.
- `classify_diop(eq)` — classifies equation type (linear, quadratic, ternary, Pell, etc.).
- Type solvers: `diop_linear`, `diop_quadratic`, `diop_ternary_quadratic`, `diop_DN`, `cornacchia`.
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
Solves systems of polynomial equations via Groebner bases.

- `solve_poly_system(seq, *gens)` — general polynomial system solver.
- `solve_biquadratic(f, g, opt)` — two bivariate quadratic equations.
- `solve_generic(polys, opt)` — arbitrary systems via elimination.
- `solve_triangulated(polys, *gens)` — Gianni-Kalkbrenner triangulation algorithm.

---

## Differential & Recurrence Equation Solvers

### [`ode.py`](ode.py)
Solves ordinary differential equations via classification and hint-based dispatch.

- `dsolve(eq, func, hint, ics)` — main ODE solver; classifies then applies best method.
- `classify_ode(eq, func)` — classifies ODE into applicable solving hints without solving.
- `checkodesol(ode, sol)` — validates ODE solution by substitution.
- `homogeneous_order(expr, *symbols)` — computes homogeneity order.
- Methods: separable, exact, linear (1st/nth), Bernoulli, Lie group, variation of parameters, undetermined coefficients, power series.

### [`pde.py`](pde.py)
Solves partial differential equations via method dispatch.

- `pdsolve(eq, func, hint)` — main PDE solver; supports meta-hints "all"/"all_Integral" returning a dict where failed strategies store the NotImplementedError exception object as value.
- `classify_pde(eq, func)` — classifies PDE into applicable hints.
- `checkpdesol(pde, sol)` — validates PDE solution by substitution. When the candidate is not isolated for the dependent function, attempts `solve` to isolate; if multiple roots, recursively checks each one.
- `_handle_Integral(expr, func, order, hint)` — post-processes PDE solutions containing unevaluated integrals.
  - Hint suffix `_Integral` preserves raw integral form; `1st_linear_constant_coeff` triggers `doit()` + `simplify`; all others return unchanged.
- `pde_separate`, `pde_separate_add`, `pde_separate_mul` — variable separation methods.

### [`recurr.py`](recurr.py)
Solves recurrence (difference) equations with polynomial/rational coefficients.

- `rsolve(f, y, init)` — main entry; dispatches to type solver.
- `rsolve_poly`, `rsolve_ratio`, `rsolve_hyper` — polynomial, rational, and hypergeometric RHS solvers.

---

## Utilities

### [`decompogen.py`](decompogen.py)
Functional decomposition for solving via composition chain reduction.

- `decompogen(f, symbol)` — decomposes f into composition chain f = f₁∘f₂∘…∘fₙ.

### [`deutils.py`](deutils.py)
Utilities for classifying and manipulating differential equations.

- `ode_order(expr, func)` — returns the order of a differential equation.
- `_preprocess(expr, func, hint)` — prepares expressions for ODE solving.
- `_desolve(eq, func, hint, ics)` — shared dispatch helper used by both `dsolve` (ODE) and `pdsolve` (PDE).
  - Delegates to `classify_ode` or `classify_pde` based on `type` kwarg.
  - On recursive calls, accepts `classify=False` to skip re-classification and reuse previously computed hints/match/order from kwargs.
  - Validates order > 0; raises ValueError if order is 0 (not a DE). Three-way error branching when no default hint: unrecognized hint → ValueError, non-matching hint → ValueError, no method works → NotImplementedError.
