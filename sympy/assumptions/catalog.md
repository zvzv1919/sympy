# sympy/assumptions — Catalog

> Part of [SymPy](../catalog.md). Assumption system for declaring and querying properties of symbols (positive, integer, etc.).

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package entry point; re-exports `AppliedPredicate`, `Predicate`, `AssumptionsContext`, `assuming`, `Q`, `ask`, `register_handler`, `remove_handler`, and `refine`. |
| `ask.py` | Defines the `AssumptionKeys` class (the `Q` object) with predicate key definitions (e.g. `Q.integer`, `Q.rational`, `Q.algebraic`, `Q.transcendental`, `Q.real`, `Q.complex`, `Q.prime`, `Q.positive`, etc.) including their mathematical docstrings and classification semantics. Also defines the `ask()` function that queries expressions against assumptions using registered handlers. |
| `ask_generated.py` | Auto-generated file (via `bin/ask_update.py`) containing precomputed known-fact implications between predicates in CNF and a single-fact lookup dict. |
| `assume.py` | Core infrastructure for the assumption system: `AssumptionsContext` (set of active assumptions), `AppliedPredicate` (a predicate applied to an expression), `Predicate` (with `eval` method that dispatches to registered handlers via MRO and raises `ValueError` when multiple resolution strategies yield contradictory truth values), global assumptions store, and the `assuming` context manager. |
| `refine.py` | Implements `refine()`, which simplifies expressions using assumptions, along with specialized refinement handlers for `Abs`, `Pow`, `exp`, `Q.real`, `atan2`, `Piecewise`, and sign. |
| `satask.py` | SAT-based assumption querying via `satask()`: converts a proposition and assumptions into a satisfiability problem and iteratively gathers all relevant known facts to determine truth values. |
| `sathandlers.py` | Defines helper Boolean-function classes (`UnevaluatedOnFree`, `AllArgs`, `AnyArgs`, `ExactlyOneArg`, `CheckOldAssump`) and a `ClassFactRegistry` that maps expression types to SAT-based inference rules for `satask`. |
| `handlers/__init__.py` | Package init for assumption handlers; re-exports `AskHandler`, `CommonHandler`, `AskCommutativeHandler`, `TautologicalHandler`, and `test_closed_group`. |
| `handlers/calculus.py` | Query handlers for calculus-related predicates (`finite`, `infinite`) that determine boundedness of expressions built from `Add`, `Mul`, `Pow`, `Symbol`, and special constants. |
| `handlers/common.py` | Base handler classes: `AskHandler` (abstract base), `CommonHandler` (always-true/false helpers), `AskCommutativeHandler`, and `TautologicalHandler` which implements three-valued logic (True/False/None) for evaluating boolean connectives (`Or`, `And`, `Not`, `Implies`, `Equivalent`) under incomplete knowledge—e.g., `Or` short-circuits on True, propagates None if any arg is unknown, returns False only when all args are definitively False. Also provides `test_closed_group` for checking closure under an operation. |
| `handlers/matrices.py` | Query handlers for matrix-related predicates such as square, symmetric, invertible, orthogonal, unitary, positive-definite, upper/lower-triangular, diagonal, fullrank, and singular. |
| `handlers/ntheory.py` | Query handlers for number-theory predicates: prime, composite, even, odd, and rational/irrational classification of expressions. |
| `handlers/order.py` | Query handlers for order-relation predicates (negative, nonnegative, nonzero, zero, positive, nonpositive, extended_real) applied to `Add`, `Mul`, `Pow`, numeric types, and transcendental functions (`log`, `exp`, `factorial`, `Abs`). Positivity of `log` first requires a real argument, then checks whether arg − 1 is positive or negative. |
| `handlers/sets.py` | Query handlers (evaluation logic) for set-membership predicates: integer, rational, irrational, real, extended_real, hermitian, complex, imaginary, antihermitian, and algebraic. Contains handler classes that evaluate whether expressions satisfy these predicates, not the predicate definitions themselves. |
