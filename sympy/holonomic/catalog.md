# sympy/holonomic — Catalog

> Part of [SymPy](../catalog.md). Holonomic functions represented via linear differential equations with polynomial coefficients.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package initializer that re-exports the public API (`DifferentialOperators`, `HolonomicFunction`, `DifferentialOperator`, `from_hyper`, `from_meijerg`, `expr_to_holonomic`, `RecurrenceOperators`, `RecurrenceOperator`, `HolonomicSequence`). |
| `holonomic.py` | Core module implementing `DifferentialOperator`, `DifferentialOperatorAlgebra`, and `HolonomicFunction` with arithmetic, composition, integration (including definite-integral evaluation with fallback logic: symbolic substitution → limit when result is NaN → numerical evaluation when series conversion fails), differentiation, series expansion, and conversion utilities (`from_hyper`, `from_meijerg`, `expr_to_holonomic`). |
| `holonomicerrors.py` | Custom exception hierarchy for the holonomic module, including `NotPowerSeriesError`, `NotHolonomicError`, `SingularityError`, and `NotHyperSeriesError`. |
| `linearsolver.py` | Provides `NewMatrix`, a `MutableDenseMatrix` subclass that supports non-sympifiable elements and implements `gauss_jordan_solve` for use within holonomic computations. |
| `numerical.py` | Low-level numerical ODE stepping methods (Euler and Runge-Kutta 4th order) used as a backend when `HolonomicFunction` in `holonomic.py` needs to numerically evaluate a holonomic function at specified points in the complex plane. Does not contain integration logic or fallback strategies itself. |
| `recurrence.py` | Implements `RecurrenceOperatorAlgebra`, `RecurrenceOperator`, and `HolonomicSequence` for holonomic recurrence relations. `RecurrenceOperator` carries its own full set of arithmetic and comparison dunder methods (`__add__`, `__mul__`, `__sub__`, `__pow__`, `__eq__`, etc.) — these are independent from the analogous methods on `DifferentialOperator` in `holonomic.py`. When a bug involves comparing a *recurrence/difference* operator to a scalar, or shift-operator arithmetic, look here rather than in `holonomic.py`. |
