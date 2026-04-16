# sympy/holonomic — Holonomic Functions

## Glossary

- **Holonomic function**: A solution to a linear homogeneous ODE with polynomial coefficients.
- **Annihilator**: A differential operator `L` such that `L.f = 0`.
- **Weyl Algebra / Ore Algebra**: The noncommutative algebra of differential operators with the rule `Dx * a = a * Dx + a'`.
- **DMF**: Dense Multivariate Fraction — internal polynomial fraction representation used for rational coefficient manipulation.

---

## Differential Operators and Holonomic Functions

### holonomic.py

Core module: defines differential operator algebras, differential operators, holonomic functions, and conversions from hypergeometric / Meijer G-functions / arbitrary expressions.

**Algebra & Operators**

- `DifferentialOperators(base, generator)` — factory returning a `DifferentialOperatorAlgebra` and its `Dx` operator.
- `DifferentialOperatorAlgebra` — Weyl Algebra (Ore Algebra with σ=id, δ=d/dx). Stores base ring and `Dx`.
- `DifferentialOperator` — element of the Weyl Algebra, stored as a list of polynomial coefficients for each power of `Dx`.
  - Full arithmetic (`+`, `-`, `*`, `**`, `/`), respecting the commutation rule `Dx*a = a*Dx + a'`.
  - `is_singular(x0)` — checks if the leading coefficient vanishes at `x0`.

**HolonomicFunction**

Central class representing a holonomic function via its annihilator and initial conditions `y0` at a point `x0`.

- Arithmetic: `+`, `-`, `*`, `**` (closure under these operations). Annihilators are combined via ansatz + linear-system solve using `NewMatrix.gauss_jordan_solve`.
- `integrate(limits)` — indefinite/definite integration (multiplies annihilator by `Dx` on the right).
- `diff(*args)` — symbolic differentiation.
- `composition(expr, *args)` — annihilator after composing with an algebraic function.
- `to_sequence(lb=True)` — converts ODE to a recurrence relation on power-series coefficients; returns `(HolonomicSequence, n0)`.
- `series(n=6)` — power-series expansion about `x0` up to order `n`.
- `to_hyper()` — converts to a hypergeometric representation (linear combination of `hyper()` terms).
- `to_expr()` — converts back to elementary functions via `hyperexpand`.
- `evalf(points, method='RK4')` — numerical evaluation along a path using Euler or RK4.
- `change_ics(b)` — re-derives initial conditions at a new point `b`.
- `change_x(z)` / `shift_x(a)` — variable substitution helpers.
- `unify(other)` — unifies ground domains of two holonomic functions.
- `degree()` — highest power of `x` in the annihilator.
- `_indicial()` — roots of the indicial equation at singular points.

**Conversion Functions**

- `from_hyper(func, x0=0)` — `hyper()` → `HolonomicFunction` via the generalized hypergeometric ODE.
- `from_meijerg(func, x0=0)` — `meijerg()` → `HolonomicFunction`.
- `expr_to_holonomic(func, x=None, x0=0)` — general-purpose converter: tries polynomial/rational/algebraic first, then Meijer G lookup table, then recursive decomposition over `Add`/`Mul`/`Pow`.

**Internal Helpers**

- `_normalize(list_of, parent)` — normalizes an annihilator (clears denominators, divides by GCD).
- `_derivate_diff_eq(listofpoly)` — differentiates both sides of a homogeneous ODE.
- `_add_lists(list1, list2)` — adds two polynomial-coefficient lists of possibly different lengths.
- `_extend_y0(Holonomic, n)` — extends initial conditions by substituting `x0` into the ODE.
- `DMFdiff(frac)` — differentiates a `DMF` (quotient rule on polynomial fraction).
- `DMFsubs(frac, x0)` — evaluates a `DMF` at a point.
- `_convert_poly_rat_alg(func, x)` — handles polynomial, rational, and algebraic-power conversion.
- `_convert_meijerint(func, x)` — converts via `meijerint._rewrite1` to Meijer G sums.
- `_create_table(table)` — builds a lookup table mapping elementary functions (sin, cos, exp, log, erf, etc.) to pre-computed holonomic representations.
- `_find_conditions(func, x, x0, order)` — computes `[f(x0), f'(x0), ...]` for initial conditions.

---

## Recurrences

### recurrence.py

Recurrence operator algebra and holonomic sequences, mirroring the differential operator structure for discrete (shift) operators.

- `RecurrenceOperators(base, generator)` — factory returning a `RecurrenceOperatorAlgebra` and the shift operator `Sn`.
- `RecurrenceOperatorAlgebra` — noncommutative algebra with the rule `Sn * a(n) = a(n+1) * Sn`.
- `RecurrenceOperator` — element of the recurrence algebra; full arithmetic (`+`, `-`, `*`, `**`) with shift-commutation.
- `HolonomicSequence` — a sequence satisfying a linear recurrence with polynomial coefficients, stored as a `RecurrenceOperator` plus initial values `u0`.

---

## Numerical Methods

### numerical.py

Numerical integration of holonomic functions along paths in the complex plane.

- `_evalf(func, points, derivatives=False, method='RK4')` — main driver; converts annihilator coefficients to DMF field elements, then steps through each segment.
- `_euler(red, x0, x1, y0, a)` — single Euler step from `x0` to `x1`.
- `_rk4(red, x0, x1, y0, a)` — single Runge-Kutta 4th-order step from `x0` to `x1`.

All arithmetic is done in `mpmath` precision.

---

## Linear Algebra Support

### linearsolver.py

Specialized matrix class for the holonomic module's linear-system solves.

- `NewMatrix(MutableDenseMatrix)` — subclass that bypasses `sympify` (supports elements from polynomial fraction fields that aren't standard SymPy objects).
  - `row_join` / `col_join` — overridden to handle null matrices.
  - `gauss_jordan_solve(b)` — reduced-row-echelon-form solver; free variables are set to 1 (returns `(sol, tau[, free_var_index])`).

**Caveat**: Free parameters `tau` are hard-coded to 1 rather than left symbolic — this is intentional for the ansatz-based annihilator computation.

---

## Errors

### holonomicerrors.py

Custom exceptions for the holonomic module.

- `BaseHolonomicError(Exception)` — abstract base.
- `NotPowerSeriesError` — power series does not exist at the given point.
- `NotHolonomicError` — expression is not holonomic.
- `SingularityError` — operation fails due to a singularity.
- `NotHyperSeriesError` — power series expansion is not hypergeometric.

---

## Package Init

### __init__.py

Re-exports: `DifferentialOperator`, `HolonomicFunction`, `DifferentialOperators`, `from_hyper`, `from_meijerg`, `expr_to_holonomic`, `RecurrenceOperators`, `RecurrenceOperator`, `HolonomicSequence`.
