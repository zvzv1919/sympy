# Holonomic Module Catalog

Holonomic functions are solutions to linear homogeneous ODEs with polynomial coefficients.
This module implements their symbolic algebra, numerical evaluation, recurrence relations, and conversions.

---

## Core

### [`holonomic.py`](holonomic.py)
Main module: differential operator algebra, `HolonomicFunction` class, and conversion utilities.

- `DifferentialOperatorAlgebra` — parent ring (Weyl algebra) for differential operators.
- `DifferentialOperator` — element of the Weyl algebra; list of polynomial coefficients + parent.
- `DifferentialOperators(base, generator)` — factory returning algebra and derivative operator `Dx`.
- `HolonomicFunction` — solution to L.f = 0; holds annihilator, variable, initial-condition point and values.
  - Arithmetic (`__add__`, `__mul__`, etc.): closure operations that build an ansatz matrix from operator derivatives, solve a homogeneous linear system, and iteratively increase matrix dimension until a nontrivial annihilator is found.
  - `integrate(limits)` / `diff()` — symbolic integral/derivative returning new `HolonomicFunction`.
  - `composition(expr)` — compose with another expression.
  - `series(n)` — power series expansion.
  - `to_sequence()` — convert ODE to recurrence relation by substituting each `x^k Dx^j` term into a rising-factorial shifted-index expression; returns `HolonomicSequence`.
  - `to_hyper()` / `to_expr()` — convert to hypergeometric or closed-form symbolic expression.
  - `evalf(points, method, h, derivatives)` — **user-facing numerical evaluation entry point**.
    - Accepts a single scalar point **or** list of mesh points.
    - Single-point mode: generates integration path from initial point using step `h`.
    - **Edge case:** if target point equals initial-condition point, returns immediately without mesh or singularity check.
    - Performs singularity detection (roots of leading coefficient) before delegating to `numerical._evalf`.
  - `change_x(z)` / `shift_x(a)` — variable substitution helpers.
- `from_hyper`, `from_meijerg` — construct `HolonomicFunction` from hypergeometric / Meijer G-function.
- `expr_to_holonomic(func)` — convert arbitrary symbolic expression to holonomic form.
- `DMFsubs`, `DMFdiff` — evaluate/differentiate domain multivariate fractions (used by numerical code).
- `_extend_y0` — extend initial-condition vector to required length.
- `_find_conditions` — compute initial conditions for a given function at a point.

### [`recurrence.py`](recurrence.py)
Recurrence (shift) operator algebra and holonomic sequences — defines algebraic structures only; ODE-to-recurrence conversion lives in `holonomic.py`.

- `RecurrenceOperators(base, generator)` — factory returning algebra and shift operator `Sn`.
- `RecurrenceOperator` — element of recurrence algebra; commutation rule `Sn * a(n) = a(n+1) * Sn`.
- `HolonomicSequence` — sequence satisfying a linear recurrence with polynomial coefficients.

---

## Numerical

### [`numerical.py`](numerical.py)
Low-level numerical ODE integration routines. **Not the user entry point** — called by `HolonomicFunction.evalf`.

- `_evalf(func, points, derivatives, method)` — converts annihilator to reduced ODE form, chains integration across points, dispatches to solver.
- `_euler(red, x0, x1, y0, a)` — Euler method (first-order).
- `_rk4(red, x0, x1, y0, a)` — Runge-Kutta 4th order (default, higher accuracy).

---

## Errors

### [`holonomicerrors.py`](holonomicerrors.py)
Custom exceptions for the holonomic module.

- `SingularityError` — raised when evaluation hits a singularity.
- `NotPowerSeriesError` — power series does not exist at given point.
- `NotHolonomicError` — function is not holonomic.
- `NotHyperSeriesError` — power series is not hypergeometric.

---

## Utilities

### [`linearsolver.py`](linearsolver.py)
Matrix solving utilities for internal holonomic computations.

- `NewMatrix` — thin `MutableDenseMatrix` wrapper adding `gauss_jordan_solve` for non-Sympified elements. Passive utility only; all ansatz construction and dimension-increase logic lives in `holonomic.py`.
