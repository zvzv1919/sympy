# sympy/holonomic — Catalog

> Part of [SymPy](../catalog.md). Holonomic functions represented via linear differential equations with polynomial coefficients.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package initializer that re-exports the public API (`DifferentialOperators`, `HolonomicFunction`, `DifferentialOperator`, `from_hyper`, `from_meijerg`, `expr_to_holonomic`, `RecurrenceOperators`, `RecurrenceOperator`, `HolonomicSequence`). |
| `holonomic.py` | Core module implementing `DifferentialOperator` (differential operator `Dx`, with arithmetic, `__eq__` equality comparison against other operators or plain scalars using `listofpoly`, and `is_singular`), `DifferentialOperatorAlgebra` (continuous ODE operator algebra, defaults generator to `'Dx'`), and `HolonomicFunction` — including closure-property arithmetic (`__add__`, `__mul__`, etc.) that computes combined ODE annihilators, plus composition, integration, differentiation, series expansion, numerical evaluation entry point (`evalf`), and conversion utilities (`from_hyper`, `from_meijerg`, `expr_to_holonomic`). |
| `holonomicerrors.py` | Custom exception hierarchy for the holonomic module. Each exception class defines its own `__str__` method that formats human-readable error messages (e.g. singularity location, failed series expansion). Includes `NotPowerSeriesError`, `NotHolonomicError`, `SingularityError` (formats message when a function is not analytic at a point), and `NotHyperSeriesError`. |
| `linearsolver.py` | Provides `NewMatrix`, a `MutableDenseMatrix` subclass that supports non-sympifiable elements and implements `gauss_jordan_solve`; a low-level utility consumed by `holonomic.py` (not the orchestration logic). |
| `numerical.py` | Contains `_evalf`, the main numerical integration orchestrator that converts a holonomic ODE into a reduced first-order system, validates that enough initial conditions are provided (raises `TypeError` if fewer than the ODE order), and iterates through complex-plane mesh points using either Euler (`_euler`) or Runge-Kutta 4th order (`_rk4`) stepping. Called by `HolonomicFunction.evalf` in `holonomic.py`. |
| `recurrence.py` | Implements `RecurrenceOperatorAlgebra` (discrete/sequence operator algebra whose `__init__` type-checks the generator argument: accepts `None` (defaults to `'Sn'`), a `str`, or a `Symbol` — other types silently leave `gen_symbol` unset), `RecurrenceOperator` (sequence-shifting operator with arithmetic and scalar multiplication), and `HolonomicSequence` for representing and manipulating holonomic recurrence relations. |
