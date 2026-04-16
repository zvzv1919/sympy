# sympy/holonomic — Catalog

> Part of [SymPy](../catalog.md). Holonomic functions represented via linear differential equations with polynomial coefficients.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package initializer that re-exports the public API (`DifferentialOperators`, `HolonomicFunction`, `DifferentialOperator`, `from_hyper`, `from_meijerg`, `expr_to_holonomic`, `RecurrenceOperators`, `RecurrenceOperator`, `HolonomicSequence`). |
| `holonomic.py` | Core module implementing `DifferentialOperator` (differential operator `Dx`, with arithmetic, `__eq__` equality comparison against other operators or plain scalars using `listofpoly`, and `is_singular`), `DifferentialOperatorAlgebra` (continuous ODE operator algebra, defaults generator to `'Dx'`), and `HolonomicFunction` — including closure-property arithmetic (`__add__`, `__mul__`, etc.) that computes combined ODE annihilators, plus composition, integration, differentiation, series expansion, numerical evaluation entry point (`evalf`), and conversion utilities (`from_hyper`, `from_meijerg`, `expr_to_holonomic`). |
| `holonomicerrors.py` | Custom exception hierarchy for the holonomic module, including `NotPowerSeriesError`, `NotHolonomicError`, `SingularityError`, and `NotHyperSeriesError`. |
| `linearsolver.py` | Provides `NewMatrix`, a `MutableDenseMatrix` subclass that supports non-sympifiable elements and implements `gauss_jordan_solve`; a low-level utility consumed by `holonomic.py` (not the orchestration logic). |
| `numerical.py` | Low-level numerical stepping routines (Euler and Runge-Kutta 4th order) called by `HolonomicFunction.evalf` in `holonomic.py`; operates on a pre-built mesh of points. |
| `recurrence.py` | Implements `RecurrenceOperatorAlgebra` (linear difference equation operator algebra whose `__init__` builds a shift operator from base ring elements and defaults the noncommutative generator symbol to `'Sn'` when none is provided), `RecurrenceOperator` (sequence-shifting operator with arithmetic and scalar multiplication), and `HolonomicSequence` for representing and manipulating holonomic recurrence relations. |
