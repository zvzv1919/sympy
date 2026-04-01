# sympy/holonomic — Catalog

> Part of [SymPy](../catalog.md). Holonomic functions represented via linear differential equations with polynomial coefficients.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package initializer that re-exports the public API (`DifferentialOperators`, `HolonomicFunction`, `DifferentialOperator`, `from_hyper`, `from_meijerg`, `expr_to_holonomic`, `RecurrenceOperators`, `RecurrenceOperator`, `HolonomicSequence`). |
| `holonomic.py` | Core module implementing `DifferentialOperator`, `DifferentialOperatorAlgebra`, and `HolonomicFunction` with arithmetic, composition, integration, differentiation, series expansion, and conversion utilities (`from_hyper`, `from_meijerg`, `expr_to_holonomic`). |
| `holonomicerrors.py` | Custom exception hierarchy for the holonomic module, including `NotPowerSeriesError`, `NotHolonomicError`, `SingularityError`, and `NotHyperSeriesError`. |
| `linearsolver.py` | Provides `NewMatrix`, a `MutableDenseMatrix` subclass that supports non-sympifiable elements and implements `gauss_jordan_solve` for use within holonomic computations. |
| `numerical.py` | Numerical integration methods (Euler and Runge-Kutta 4th order) for evaluating holonomic functions at specified points in the complex plane. |
| `recurrence.py` | Implements `RecurrenceOperatorAlgebra`, `RecurrenceOperator`, and `HolonomicSequence` for representing and manipulating holonomic recurrence relations. |
