# sympy/assumptions — Catalog

> Part of [SymPy](../catalog.md). Assumption system for declaring and querying properties of symbols (positive, integer, etc.).

## Python Files

| File | Summary |
|------|----------|
| `__init__.py` | Package entry point; re-exports `AppliedPredicate`, `Predicate`, `AssumptionsContext`, `assuming`, `Q`, `ask`, `register_handler`, `remove_handler`, and `refine`. |
| `ask.py` | Defines the `AssumptionKeys` class (the `Q` object) with all supported predicate keys (prime, composite, integer, positive, negative, real, complex, rational, irrational, even, odd, commutative, symmetric, etc.) and the `ask()` function that queries expressions against assumptions using registered handlers. Each predicate is a method decorated with `@predicate_memo` that returns a `Predicate` object with documentation and examples. |
| `ask_generated.py` | Auto-generated file (via `bin/ask_update.py`) containing precomputed known-fact implications between predicates in CNF and a single-fact lookup dict. |
| `assume.py` | Core infrastructure for the assumption system: `AssumptionsContext` (set of active assumptions), `AppliedPredicate` (a predicate applied to an expression), `Predicate`, global assumptions store, and the `assuming` context manager. |
| `refine.py` | Implements `refine()`, which simplifies expressions using assumptions, along with specialized refinement handlers for `Abs`, `Pow`, `exp`, `Q.real`, `atan2`, `Piecewise`, and sign. |
| `satask.py` | SAT-based assumption querying via `satask()`: converts a proposition and assumptions into a satisfiability problem and iteratively gathers all relevant known facts to determine truth values. |
| `sathandlers.py` | Defines helper Boolean-function classes (`UnevaluatedOnFree`, `AllArgs`, `AnyArgs`, `ExactlyOneArg`, `CheckOldAssump`) and a `ClassFactRegistry` that maps expression types to SAT-based inference rules for `satask`. |
| `handlers/__init__.py` | Package init for assumption handlers; re-exports `AskHandler`, `CommonHandler`, `AskCommutativeHandler`, `TautologicalHandler`, and `test_closed_group`. |
| `handlers/calculus.py` | Query handlers for calculus-related predicates (`finite`, `infinite`) that determine boundedness of expressions built from `Add`, `Mul`, `Pow`, `Symbol`, and special constants. |
| `handlers/common.py` | Base handler classes: `AskHandler` (abstract base), `CommonHandler` (always-true/false helpers), `AskCommutativeHandler`, and `TautologicalHandler` (evaluates boolean assumption expressions). Also provides `test_closed_group` for checking closure under an operation. |
| `handlers/matrices.py` | Query handlers for matrix-related predicates such as square, symmetric, invertible, orthogonal, unitary, positive-definite, upper/lower-triangular, diagonal, fullrank, and singular. |
| `handlers/ntheory.py` | Query handlers for number-theory predicates: prime, composite, even, odd, and rational/irrational classification of expressions. |
| `handlers/order.py` | Query handlers for order-relation predicates (negative, nonnegative, nonzero, zero, positive, nonpositive, extended_real) applied to `Add`, `Mul`, `Pow`, and numeric types. |
| `handlers/sets.py` | Query handlers for set-membership predicates: integer, rational, irrational, real, extended_real, hermitian, complex, imaginary, antihermitian, and algebraic. |
