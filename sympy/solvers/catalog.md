# sympy/solvers — Equation Solvers

## Glossary

- **solveset** — The newer set-theoretic solving API, returning solution sets rather than lists.
- **LambertW** — The Lambert W function, the inverse of `x*exp(x)`.
- **Pell equation** — A Diophantine equation of the form `x² - Dy² = N`.
- **Groebner basis** — A particular generating set of a polynomial ideal used to solve polynomial systems.
- **Cornacchia's algorithm** — Method for solving `ax² + by² = m` with `gcd(a,b) = 1`.

---

## Equation Solving (Core)

### solvers.py

Main entry point for algebraic and transcendental equation solving.

- `solve(f, *symbols, **flags)` — Universal solver for algebraic/transcendental equations and systems.
  - Accepts a single expression, a list of expressions, or relational objects.
  - Dispatches to polynomial solvers, transcendental solvers (`_tsolve`), Lambert-W solver, system solvers, etc.
  - Flags: `dict`, `set`, `check`, `simplify`, `rational`, `manual`, `minimal`, `quick`, `cubics`, `quartics`, `quintics`.
- `_solve(f, *symbols, **flags)` — Internal single-equation solver; handles polynomial, trig, exponential, logarithmic, and piecewise cases.
- `_solve_system(exprs, symbols, **flags)` — Solves systems of equations; tries polynomial system solvers first, then falls back to iterative substitution.
- `solve_linear(lhs, rhs, symbols, exclude)` — Fast path: isolate a symbol that appears linearly. Returns `(symbol, solution)` or `(numerator, denominator)` if non-linear.
- `nsolve(*args, **kwargs)` — Numerical solver wrapping mpmath's `findroot`. Supports multi-dimensional systems and multiple methods (secant, muller, bisect, newton, etc.).
- `checksol(f, symbol, sol, **flags)` — Verify a candidate solution by substitution, returning `True`/`False`/`None`.
- `check_assumptions(expr, **assumptions)` — Test whether an expression satisfies given assumption predicates.
- `_tsolve(eq, sym, **flags)` — Solver for transcendental equations (log, exp, LambertW).
- `_invert(eq, *symbols, **kwargs)` — Algebraic inversion of an expression to isolate a symbol; returns `(solved_symbol, rest)`.
- `unrad(eq, *syms, **flags)` — Remove radicals from an equation by raising to appropriate powers; may introduce spurious solutions.
- `solve_linear_system(system, *symbols, **flags)` — Solve a linear system given as an augmented matrix via Gauss-Jordan elimination.
- `solve_linear_system_LU(matrix, syms)` — Solve a linear system via LU decomposition.
- `solve_undetermined_coeffs(equ, coeffs, sym, **flags)` — Solve for undetermined coefficients by equating powers of `sym`.
- `minsolve_linear_system(system, *symbols, **flags)` — Find a solution to a linear system that minimizes the number of non-zero variables (supports `quick` mode).
- `denoms(eq, symbols)` — Recursively collect all denominators containing any of `symbols`.
- `det_quick(M, method)`, `inv_quick(M)` — Fast determinant/inverse for small matrices; dispatch between Bareiss and Berkowitz methods.

### solveset.py

Set-theoretic solver — the successor to `solve()`, returning `FiniteSet`, `Interval`, `ImageSet`, `ConditionSet`, etc.

- `solveset(f, symbol, domain)` — Top-level entry point. Solves equations and inequalities over a specified domain (`S.Complexes` or `S.Reals`). Claims completeness in its solution set.
- `solveset_real(f, symbol)` — Shortcut for `solveset(..., domain=S.Reals)`.
- `solveset_complex(f, symbol)` — Shortcut for `solveset(..., domain=S.Complexes)`.
- `linsolve(system, *symbols)` — Solve systems of linear equations. Accepts augmented matrix, `(A, b)` pair, or list-of-equations form. Uses Gauss-Jordan elimination; returns parametric solutions for underdetermined systems.
- `linear_eq_to_matrix(equations, *symbols)` — Convert a list of linear equations to augmented matrix form `(A, b)`.
- `_invert` / `invert_complex` — Reduce `f(x) = y` to simpler equations via algebraic inversion; handles trig, exp, log, Abs, Pow.
- `invert_real(f_x, y, x)` — Real-domain variant of `_invert`.
- `_solveset(f, symbol, domain, _check)` — Core dispatcher: routes to trig solver, polynomial solver, radical solver, rational solver, or abs solver.
- `_solve_trig(f, symbol, domain)` — Rewrite trig equations via `exp(I*x)` substitution.
- `_solve_as_poly(f, symbol, domain)` — Solve via polynomial root-finding, with change-of-variable support.
- `_solve_radical(f, symbol, solveset_solver)` — Remove radicals with `unrad`, solve, then verify.
- `_solve_abs(f, symbol, domain)` — Split `|q| = r` into positive and negative branches.
- `domain_check(f, symbol, p)` — Reject solutions where substitution gives infinity.
- `_is_function_class_equation(func_class, f, symbol)` — Test whether an equation is composed purely of a given function class (trig/hyperbolic) with linear arguments.
- `_has_rational_power(expr, symbol)` — Detect non-integer rational exponents (e.g. `sqrt`).

### bivariate.py

Solvers for bivariate equations reducible via Lambert W or composite substitution.

- `bivariate_type(f, x, y)` — Detect if `f(x,y)` can be written as `P(u)` where `u` is one of `x*y`, `x+y`, `x*y+x`, or `x*y+y`. Returns `(u(x,y), P(u), u)`.
- `_solve_lambert(f, symbol, gens)` — Solve Lambert-type equations `a*log(b*X+c) + d*X + f = 0` using the `LambertW` function. Handles six canonical forms.
- `_lambert(eq, x)` — Low-level Lambert W solution extraction.
- `_filtered_gens(poly, symbol)` — Extract generators of a polynomial that depend on `symbol`, preferring non-inverted forms.
- `_mostfunc(lhs, func, X)` — Find the term with the deepest nesting of `func` (e.g. `log(log(x))` beats `log(x)`).
- `_linab(arg, symbol)` — Decompose `arg` as `a*X + b` where `X` depends on `symbol`.

### polysys.py

Solvers for systems of polynomial equations using Groebner bases.

- `solve_poly_system(seq, *gens)` — Solve a polynomial system. Tries `solve_biquadratic` for 2×2 degree-≤2 systems, otherwise falls back to `solve_generic`.
- `solve_biquadratic(f, g, opt)` — Specialized solver for two bivariate quadratic polynomials via Groebner basis of length 2.
- `solve_generic(polys, opt)` — General zero-dimensional polynomial system solver. Computes a lex-order Groebner basis and recursively back-substitutes roots.
- `solve_triangulated(polys, *gens)` — Solve via the Gianni-Kalkbrenner algorithm: compute Groebner basis then iteratively factorize in algebraic extensions.

**Caveats:** `solve_generic` only supports zero-dimensional systems (finitely many solutions). Raises `NotImplementedError` for positive-dimensional ideals.

---

## Ordinary Differential Equations (ODE)

### ode.py

Comprehensive ODE solver supporting many classes of first-, second-, and nth-order ODEs.

- `dsolve(eq, func, hint, simplify, ics)` — Main ODE solver. Classifies the ODE, selects a method, solves, and simplifies.
  - Supports meta-hints: `"default"`, `"all"`, `"all_Integral"`, `"best"`.
  - Handles systems of ODEs via `classify_sysode`.
- `classify_ode(eq, func, dict, ics)` — Classify an ODE into applicable solution hints, ordered by preference. Returns a tuple of hint strings.
- `checkodesol(ode, sol, func, order)` — Verify a proposed ODE solution by substitution and simplification.
- `homogeneous_order(eq, *symbols)` — Return the homogeneous order of an expression, or `None`.
- `infinitesimals(eq, func, order, hint)` — Compute Lie group infinitesimals `(ξ, η)` for first-order ODEs.
- `checkinfsol(eq, infinitesimals)` — Verify computed infinitesimals.
- `classify_sysode(eq, funcs)` — Classify a system of ODEs (linear/nonlinear, order, type).
- `odesimp(eq, func, order, constants, hint)` — Post-processing: solve for `func`, simplify constants, evaluate integrals.
- `constantsimp(expr, constants)` — Simplify expressions containing arbitrary constants.
- `constant_renumber(expr, symbolname, start, end)` — Renumber arbitrary constants to canonical form.

**Solver methods (selected):**
- 1st order: `ode_separable`, `ode_1st_exact`, `ode_1st_linear`, `ode_Bernoulli`, `ode_Riccati_special_minus2`, `ode_1st_homogeneous_coeff_best/subs_dep_div_indep/subs_indep_div_dep`, `ode_1st_power_series`, `ode_lie_group`, `ode_linear_coefficients`, `ode_separable_reduced`, `ode_almost_linear`
- 2nd order: `ode_Liouville`, `ode_2nd_power_series_ordinary`, `ode_2nd_power_series_regular` (Frobenius method)
- nth order: `ode_nth_linear_constant_coeff_homogeneous`, `ode_nth_linear_constant_coeff_undetermined_coefficients`, `ode_nth_linear_constant_coeff_variation_of_parameters`, `ode_nth_linear_euler_eq_homogeneous`, `ode_nth_linear_euler_eq_nonhomogeneous_undetermined_coefficients`, `ode_nth_linear_euler_eq_nonhomogeneous_variation_of_parameters`

**System ODE solvers:**
- `sysode_linear_2eq_order1` (7 sub-types), `sysode_linear_2eq_order2` (11 sub-types)
- `sysode_linear_3eq_order1` (4 sub-types), `sysode_linear_neq_order1`
- `sysode_nonlinear_2eq_order1` (5 sub-types), `sysode_nonlinear_3eq_order1` (2 sub-types)

**Lie group heuristics:** `lie_heuristic_abaco1_simple`, `lie_heuristic_abaco1_product`, `lie_heuristic_bivariate`, `lie_heuristic_chi`, `lie_heuristic_function_sum`, `lie_heuristic_abaco2_similar`, `lie_heuristic_abaco2_unique_unknown`, `lie_heuristic_abaco2_unique_general`, `lie_heuristic_linear`

---

## Partial Differential Equations (PDE)

### pde.py

Solver for first-order linear PDEs in two independent variables.

- `pdsolve(eq, func, hint, dict, solvefun)` — Main PDE solver. Supports meta-hints `"default"`, `"all"`, `"all_Integral"`.
- `classify_pde(eq, func, dict)` — Classify a PDE into applicable hints. Supported classifications:
  - `1st_linear_constant_coeff_homogeneous`
  - `1st_linear_constant_coeff` / `1st_linear_constant_coeff_Integral`
  - `1st_linear_variable_coeff`
- `checkpdesol(pde, sol, func)` — Verify a PDE solution by direct substitution.
- `pde_separate(eq, fun, sep, strategy)` — Attempt separation of variables (`'add'` or `'mul'` strategy).
- `pde_separate_add(eq, fun, sep)` — Additive separation helper.
- `pde_separate_mul(eq, fun, sep)` — Multiplicative separation helper.
- `pde_1st_linear_constant_coeff_homogeneous(eq, func, order, match, solvefun)` — Solve `a*f_x + b*f_y + c*f = 0`.
- `pde_1st_linear_constant_coeff(eq, func, order, match, solvefun)` — Solve `a*f_x + b*f_y + c*f = G(x,y)`.
- `pde_1st_linear_variable_coeff(eq, func, order, match, solvefun)` — Solve 1st order linear PDE with variable coefficients by reducing to an ODE via the method of characteristics.

---

## Diophantine Equations

### diophantine.py

Solver for polynomial equations with integer coefficients seeking integer solutions.

- `diophantine(eq, param, syms)` — High-level solver. Factors the equation and solves each factor independently via `diop_solve`. Returns a set of solution tuples.
- `classify_diop(eq)` — Classify a Diophantine equation. Recognized types:
  - `linear`, `binary_quadratic`, `univariate`
  - `homogeneous_ternary_quadratic`, `homogeneous_ternary_quadratic_normal`
  - `inhomogeneous_ternary_quadratic`, `inhomogeneous_general_quadratic`, `homogeneous_general_quadratic`
  - `general_pythagorean`, `general_sum_of_squares`, `general_sum_of_even_powers`
  - `cubic_thue` (recognized but not solved)
- `diop_solve(eq, param)` — Dispatch to type-specific solvers:

**Type-specific solvers:**
- `diop_linear(eq, param)` — Solve `a₁x₁ + a₂x₂ + … + aₙxₙ = c` using extended GCD.
- `diop_quadratic(eq, param)` — Solve `Ax² + Bxy + Cy² + Dx + Ey + F = 0`. Handles simple-hyperbolic, parabolic, square-discriminant, and Pell equation cases.
- `diop_DN(D, N, t)` — Solve the generalized Pell equation `x² - Dy² = N` using the LMM algorithm.
- `diop_bf_DN(D, N, t)` — Brute-force Pell solver for small solution bounds.
- `diop_ternary_quadratic(eq)` — Solve `ax² + by² + cz² + fxy + gyz + hxz = 0`.
- `diop_ternary_quadratic_normal(eq)` — Solve `ax² + by² + cz² = 0` using Lagrange's descent with Holzer reduction.
- `diop_general_pythagorean(eq, param)` — Parametrize `a₁²x₁² + … - aₙ₊₁²xₙ₊₁² = 0`.
- `diop_general_sum_of_squares(eq, limit)` — Solve `x₁² + x₂² + … + xₙ² = k`.
- `diop_general_sum_of_even_powers(eq, limit)` — Solve `x₁ᵉ + x₂ᵉ + … + xₙᵉ = k` for even `e`.

**Transformation & number-theoretic utilities:**
- `transformation_to_DN(eq)` / `find_DN(eq)` — Transform binary quadratic to Pell form `X² - DY² = N`.
- `cornacchia(a, b, m)` — Solve `ax² + by² = m` (Cornacchia's algorithm).
- `PQa(P_0, Q_0, D)` — Generator yielding continued fraction sequences for `(P + √D)/Q`.
- `descent(A, B)` / `ldescent(A, B)` — Lagrange descent (with/without Gaussian reduction) for `w² = Ax² + By²`.
- `holzer(x, y, z, a, b, c)` — Reduce a solution of `ax² + by² = cz²` so that `z² ≤ |ab|`.
- `gaussian_reduce(w, a, b)` — Gaussian lattice reduction for congruence solving.
- `sqf_normal(a, b, c)` — Square-free normal form of `ax² + by² + cz²`.
- `parametrize_ternary_quadratic(eq)` — Full parametric solution `(x(p,q), y(p,q), z(p,q))`.
- `transformation_to_normal(eq)` — Transform ternary quadratic to diagonal form.
- `sum_of_three_squares(n)`, `sum_of_four_squares(n)` — Represent `n` as a sum of 3 or 4 squares.
- `prime_as_sum_of_two_squares(p)` — Represent prime `p ≡ 1 (mod 4)` as `a² + b²`.
- `power_representation(n, p, k, zeros)` — Generator for `n = n₁ᵖ + n₂ᵖ + … + nₖᵖ`.
- `sum_of_squares(n, k, zeros)` — Generator wrapping `power_representation` with `p=2`.
- `partition(n, k, zeros)` — Integer partition generator.
- `merge_solution`, `base_solution_linear`, `equivalent`, `length`, `divisible`, `check_param`, `reconstruct`, `square_factor`, `is_solution_quad`

---

## Inequalities

### inequalities.py

Solvers for polynomial, rational, and absolute-value inequalities.

- `reduce_inequalities(inequalities, symbols)` — Top-level inequality reducer. Partitions into polynomial, absolute-value, and general parts; solves each and intersects results.
- `solve_poly_inequality(poly, rel)` — Solve a single polynomial inequality with rational coefficients. Returns a list of `Interval`/`FiniteSet` objects.
- `solve_poly_inequalities(polys)` — Solve a union of polynomial inequalities.
- `solve_rational_inequalities(eqs)` — Solve a system of rational inequalities (numer/denom pairs).
- `reduce_rational_inequalities(exprs, gen, relational)` — Reduce a system of rational inequalities; converts to polynomial pairs and delegates.
- `reduce_abs_inequality(expr, rel, gen)` — Reduce a single inequality with nested `Abs` by case-splitting.
- `reduce_abs_inequalities(exprs, gen)` — Reduce a system of absolute-value inequalities.
- `solve_univariate_inequality(expr, gen, relational)` — Solve a real univariate inequality by finding roots and sign-testing intervals.
- `_solve_inequality(ie, s)` — Internal helper; handles linear case directly, falls back to `reduce_rational_inequalities` or `solve_univariate_inequality`.

---

## Recurrence Relations

### recurr.py

Solver for linear recurrences (difference equations) with polynomial or rational coefficients.

- `rsolve(f, y, init)` — Top-level recurrence solver. Accepts the recurrence as an expression in `y(n)`, `y(n+1)`, etc. Applies initial conditions if provided.
- `rsolve_poly(coeffs, f, n)` — Find polynomial solutions of `L[y] = f`. Uses undetermined coefficients for low degree, otherwise the Abramov-Bronstein-Petkovsek method.
- `rsolve_ratio(coeffs, f, n)` — Find rational function solutions by computing a universal denominator and reducing to `rsolve_poly`.
- `rsolve_hyper(coeffs, f, n)` — Find hypergeometric solutions (and combinations of dissimilar hypergeometric terms). Computes a linearly independent basis for the solution space.

---

## Utilities

### deutils.py

Shared utilities for ODE and PDE solvers.

- `_preprocess(expr, func, hint)` — Prepare a differential equation for solving: evaluate derivatives, auto-detect the unknown function.
- `ode_order(expr, func)` — Compute the order of a differential equation w.r.t. a given function.
- `_desolve(eq, func, hint, ics, simplify, **kwargs)` — Internal dispatcher shared by `dsolve` and `pdsolve`. Classifies the DE, validates hints, and routes to solvers.

### decompogen.py

General functional decomposition of expressions.

- `decompogen(f, symbol)` — Decompose `f` into `[f₁, f₂, …, fₙ]` such that `f = f₁ ∘ f₂ ∘ … ∘ fₙ`. Handles `Function`, `Pow`, and polynomial cases (delegates to `polys.decompose` for pure polynomials).

### \_\_init\_\_.py

Re-exports the public API: `solve`, `solve_linear_system`, `solve_linear_system_LU`, `solve_undetermined_coeffs`, `nsolve`, `solve_linear`, `checksol`, `det_quick`, `inv_quick`, `diophantine`, `rsolve`, `rsolve_poly`, `rsolve_ratio`, `rsolve_hyper`, `checkodesol`, `classify_ode`, `dsolve`, `homogeneous_order`, `solve_poly_system`, `solve_triangulated`, `pdsolve`, `classify_pde`, `checkpdesol`, `pde_separate`, `pde_separate_add`, `pde_separate_mul`, `ode_order`, `reduce_inequalities`, `reduce_abs_inequality`, `reduce_abs_inequalities`, `solve_poly_inequality`, `solve_rational_inequalities`, `solve_univariate_inequality`, `decompogen`, `solveset`, `linsolve`, `linear_eq_to_matrix`.

---

## Benchmarks

### benchmarks/bench_solvers.py

Contains `timeit_linsolve_trivial` — benchmarks `solve_linear_system` on an 8×8 identity-augmented matrix.
