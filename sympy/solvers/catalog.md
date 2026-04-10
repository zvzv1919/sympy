# sympy/solvers — Catalog

> Part of [SymPy](../catalog.md). Equation solving: algebraic, ODE, PDE, recurrence, inequality, and Diophantine solvers.

## Python Files

| File | Summary |
|------|----------|
| `__init__.py` | Package init that re-exports the public API from submodules (solve, dsolve, pdsolve, rsolve, diophantine, solveset, etc.). |
| `solvers.py` | Core equation-solving module providing `solve()` for algebraic/transcendental equations, `nsolve()` for numerical solving, and helpers for linear systems and undetermined coefficients. Includes `_solve_system()` for iterative symbol-by-symbol solving of equation systems with circular-dependency detection (discarding solutions where a candidate value references previously-solved symbols) and `solve_linear()` for linear equation analysis. |
| `solveset.py` | Set-based equation solving with `solveset()`, `linsolve()`, `nonlinsolve()`, and `linear_eq_to_matrix()`, operating over real or complex domains and returning solutions as SymPy sets. |
| `ode.py` | Ordinary differential equation solver (`dsolve`) supporting separable, homogeneous, exact, linear, Bernoulli, Liouville, power series, and nth-order constant-coefficient methods. Lie group symmetry methods include multiple heuristics: scanning for non-integer power subexpressions and unevaluated two-variable functions in the right-hand side, computing partial derivative ratios, applying variable separation to construct candidate infinitesimal generators (xi, eta), and verifying them against the determining PDE. Also includes ODE classification and solution checking. |
| `pde.py` | Partial differential equation solver (`pdsolve`) for first-order linear PDEs with constant or variable coefficients, with classification and additive/multiplicative variable separation utilities. |
| `recurr.py` | Recurrence relation (difference equation) solver providing `rsolve()` and lower-level routines (`rsolve_poly`, `rsolve_ratio`, `rsolve_hyper`) for linear inhomogeneous recurrences with polynomial or rational coefficients. |
| `polysys.py` | Solvers for systems of polynomial equations using Groebner bases, including specialized handling of bivariate biquadratic systems. |
| `inequalities.py` | Tools for solving polynomial, rational, and absolute-value inequalities and reducing systems of inequalities to solution sets. |
| `diophantine.py` | Solver for Diophantine equations (integer-valued polynomial equations) covering linear, quadratic, Pythagorean, and sum-of-powers forms, with classification and parametric solution support. |
| `decompogen.py` | General functional decomposition of expressions, decomposing `f(x)` into a composition chain `f_1(f_2(...f_n(x)))` for both polynomial and non-polynomial functions. |
| `bivariate.py` | Helper routines for solving bivariate equations involving transcendental functions (LambertW, exp, log), used internally by `solve()`. |
| `deutils.py` | Shared utility functions for ODE and PDE solvers, including `_preprocess` for expression preparation, `ode_order` for computing differential equation order, and `_desolve` as a common dispatch wrapper. |
| `benchmarks/bench_solvers.py` | Performance benchmark for `solve_linear_system` on a trivial identity-matrix system. |
