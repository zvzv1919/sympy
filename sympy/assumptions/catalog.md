# assumptions — Module Catalog

## Architecture Overview

The assumptions system answers queries about mathematical properties of expressions.
- **Predicate layer** (`ask.py`, `assume.py`): defines predicates (`Q.positive`, `Q.real`, …) and the `ask()` inference engine.
- **Handler layer** (`handlers/`): per-predicate, per-expression-type static methods that evaluate truth values.
- **SAT layer** (`satask.py`, `sathandlers.py`): fallback inference via satisfiability when direct handlers are inconclusive.
- **Refinement** (`refine.py`): simplifies expressions using known assumptions.

When `ask(Q.property(expr), assumptions)` is called, the engine dispatches to a handler class registered for that predicate, which selects a static method by the expression's type (e.g., `Add`, `Mul`, `Pow`, `log`, `exp`).

---

## Core Query & Predicate System

### [`ask.py`](ask.py)
Main inference engine for the assumptions system.
- `AssumptionKeys` (aliased as `Q`): defines all predicate query keys as `@predicate_memo` properties returning `Predicate` objects.
  - Each predicate property's docstring documents **semantic rules and cross-predicate implications** (the authoritative source for predicate meaning).
  - Deprecated alias properties: `Q.infinitesimal` → `Predicate('zero')`, `Q.bounded` → `Predicate('finite')`, `Q.infinity` → `Predicate('infinite')`. Decorated with `@deprecated`.
  - `deprecated_predicates` list: names excluded from the set of active assumption keys used for fact derivation (e.g., `compute_known_facts`).
  - Scalar predicates: `Q.positive`, `Q.negative`, `Q.real`, `Q.imaginary`, `Q.complex`, `Q.prime`, `Q.composite`, `Q.even`, `Q.odd`, `Q.integer`, `Q.rational`, `Q.finite`, …
  - `Q.imaginary`: true iff expressible as a nonzero real times `I`; zero is explicitly excluded from imaginary numbers.
  - `Q.real` documents that "non" facts (`Q.nonnegative`, `Q.nonpositive`, `Q.nonzero`, `Q.noninteger`) imply realness, not just negation.
  - `Q.positive`, `Q.negative`, `Q.nonnegative`, `Q.nonpositive` each document the asymmetry between negation and "non" counterparts: e.g., `~Q.negative(I)` is `True` but `Q.nonnegative(I)` is `False`, because "non" predicates require realness.
  - Matrix predicates: `Q.symmetric`, `Q.invertible`, `Q.orthogonal`, `Q.unitary`, `Q.positive_definite`, `Q.upper_triangular`, `Q.lower_triangular`, `Q.triangular`, `Q.diagonal`, `Q.fullrank`, `Q.square`.
    - `Q.triangular`: general predicate true iff matrix is upper_triangular OR lower_triangular; subsumes both specific variants.
    - `Q.positive_definite`: true iff square symmetric real matrix has Z^T M Z > 0 for every nonzero column vector Z. Non-square → False.
    - `Q.orthogonal`/`Q.unitary`: true iff M^T M = I (real/complex analogue). Non-square → False.
    - `Q.fullrank`: true iff all rows and columns are linearly independent. Square matrix criterion: full rank iff determinant is nonzero.
    - Docstrings define cross-predicate inference rules (e.g., `Q.diagonal` iff both `Q.upper_triangular` and `Q.lower_triangular`; `Q.invertible` iff `Q.fullrank` ∧ `Q.square`).
  - Matrix element-type predicates: `Q.integer_elements`, `Q.real_elements`, `Q.complex_elements` — docstrings document subset implications (e.g., integer_elements → complex_elements).
- `_extract_facts(expr, symbol)`: extracts assumption predicates relevant to a given symbol from a compound Boolean expression; applies De Morgan's law to push negations inward (converting negated And/Or).
- `ask(proposition, assumptions)`: top-level query function; validates assumption consistency (raises `ValueError("inconsistent assumptions")` if local facts contradict known mathematical facts via SAT check), then dispatches to registered handlers, then falls back in two tiers.
- `ask_full_inference(proposition, assumptions, known_facts_cnf)`: first-tier SAT fallback inside `ask.py`; checks satisfiability of proposition (and its negation) against known predicate relationships to return True/False/None.
  - If indeterminate, `ask()` escalates to `satask()` (in `satask.py`) which gathers expression-specific facts.
- `register_handler(key, handler)`: registers a handler class for a predicate; if the property name doesn't exist on `Q`, dynamically creates a new `Predicate` and attaches it.
- `remove_handler(key, handler)`: removes a handler from a predicate.
- `compute_known_facts()`, `get_known_facts()`: build logical relationship tables between predicates.

### [`assume.py`](assume.py)
Predicate definitions and global assumptions context.
- `Predicate`: base class representing a named predicate with registered handlers.
  - `eval(expr, assumptions)`: dispatches to `AskHandler` static methods (not logical inference rules) by walking the expression type's MRO; raises `ValueError` on conflicting results. Relies on handler classes registered via `register_handler()`.
- `AppliedPredicate`: result of `Q.property(expr)`; a Boolean-valued object; delegates to `Predicate.eval` via `_eval_ask`.
  - `args` returns only the expression (`_args[1:]`), hiding the predicate; `func` returns the predicate (`_args[0]`). Public arg tuple differs from internal `_args`.
- `AssumptionsContext` / `global_assumptions`: mutable set of globally active assumptions.
- `assuming(*assumptions)`: context manager for temporary local assumptions.

### [`ask_generated.py`](ask_generated.py)
Auto-generated pre-computed inference tables. Generated by `bin/ask_update.py`.
- `get_known_facts_cnf()`: returns all predicate relationships as CNF clauses; includes multi-predicate clauses (e.g., `fullrank ∧ square → invertible`).
- `get_known_facts_dict()`: returns a dict mapping each predicate to the flat set of predicates it **singly** implies; cannot represent conjunctive implications.
- The gap between these two representations determines when `ask()` must fall back to SAT solving.

---

## Handlers — Direct Predicate Evaluation

Each handler class **evaluates** (not defines) a predicate via static methods keyed by expression type. Predicate query keys are **defined** in `ask.py`.

### [`handlers/__init__.py`](handlers/__init__.py)
Re-exports common handler base classes: `AskHandler`, `CommonHandler`, `AskCommutativeHandler`, `TautologicalHandler`, `test_closed_group`.

### [`handlers/common.py`](handlers/common.py)
Base classes and utilities shared by all handlers.
- `AskHandler`: abstract base for all ask handlers.
- `CommonHandler`: utility statics `AlwaysTrue`, `AlwaysFalse`, `NaN`.
- `AskCommutativeHandler`: handler for `Q.commutative`.
- `TautologicalHandler`: evaluates truth of Boolean expressions.
- `test_closed_group()`: tests membership in a group under an operation.

### [`handlers/order.py`](handlers/order.py)
Handlers that **evaluate** ordering / sign predicates for specific expression types. Predicate semantics (e.g., why "non" facts require realness) are defined in `ask.py`.
- `AskNegativeHandler`: evaluates `Q.negative` for `Basic`, `Add`, `Mul`, `Pow`, `ImaginaryUnit`, etc.
- `AskNonNegativeHandler`, `AskNonZeroHandler`, `AskZeroHandler`, `AskNonPositiveHandler`: sign-boundary predicates.
- `AskPositiveHandler`: evaluates `Q.positive` with expression-type dispatch:
  - Arithmetic types: `Basic`, `Mul`, `Add`, `Pow`.
  - Transcendental functions: `exp`, `log`, `factorial`, `atan`, `asin`, `acos`, `acot`.
  - Each method checks domain validity (e.g., realness of arguments) before evaluating sign.
  - Matrix-related: `Trace`, `Determinant`, `MatrixElement`.

### [`handlers/sets.py`](handlers/sets.py)
Handlers that **evaluate** set-membership predicates for specific expression types. Predicate definitions and implication rules are in `ask.py`.
- `AskIntegerHandler`, `AskRationalHandler`, `AskIrrationalHandler`: dispatch on `Basic`, `Add`, `Mul`, `Pow`, `Rational`, `Float`, `GoldenRatio`, `Abs`, `exp`, `ImaginaryUnit`.
  - `AskRationalHandler.Pow`: rational base + integer exp → rational; prime base + rational (non-integer) exp → irrational.
- `AskRealHandler`: dispatch includes `Add`, `Mul`, `Pow`, `cos`, `sin`, `exp`, `log`, `atan`, `asin`, `acos`.
- `AskExtendedRealHandler`, `AskComplexHandler`.
- `AskImaginaryHandler`: dispatch on `Add`, `Mul`, `Pow`, `log`, `exp`, `Number`, `ImaginaryUnit`.
  - `log`: checks realness/positivity of argument; has hardcoded workaround for `exp(I)` / `exp(-I)` when general `Q.nonpositive` query is insufficient.
  - `Pow`: handles imaginary bases, imaginary exponents, and real base/exponent combinations including half-integer exponents.
  - `exp`: checks if argument is an odd multiple of `I*pi/2`.
- `AskHermitianHandler` (extends `AskRealHandler`): evaluates `Q.hermitian` for `Add`, `Mul`, and scalar types; `Mul` tracks noncommutative factor count and returns indeterminate if more than one noncommutative term.
- `AskAntiHermitianHandler` (extends `AskImaginaryHandler`), `AskAlgebraicHandler`.

### [`handlers/ntheory.py`](handlers/ntheory.py)
Handlers for **number-theory predicates**: prime, composite, even, odd.
- `AskPrimeHandler`, `AskCompositeHandler`, `AskEvenHandler`, `AskOddHandler`.

### [`handlers/calculus.py`](handlers/calculus.py)
Handlers for **calculus predicates**: finiteness / boundedness.
- `AskFiniteHandler`: evaluates `Q.finite` with methods for `Symbol`, `Add`, `Mul`, `Pow`, `log`, `exp`, `cos`, `sin`, number constants, `Infinity`, `NegativeInfinity`.

### [`handlers/matrices.py`](handlers/matrices.py)
Handlers that **evaluate** matrix predicates — both structural properties and element-type membership — for matrix expression types.
- Does NOT handle `Q.hermitian` or `Q.antihermitian` — those are in `handlers/sets.py`.
- Cross-predicate inference rules (e.g., diagonal ↔ triangular) are defined in `ask.py`, not here.
- `AskSquareHandler`, `AskSymmetricHandler`, `AskInvertibleHandler`.
- `AskOrthogonalHandler`, `AskUnitaryHandler`, `AskFullRankHandler`.
- `AskPositiveDefiniteHandler`, `AskUpperTriangularHandler`, `AskLowerTriangularHandler`, `AskDiagonalHandler`.
- `AskIntegerElementsHandler`, `AskRealElementsHandler`, `AskComplexElementsHandler`: evaluate `Q.*_elements` for matrix expression types.
  - `MatMul_elements`: splits `MatMul` args into scalar factors vs matrix factors via `sift`.
  - Validates scalars with a scalar predicate (e.g., `Q.integer`) and matrices with element-type predicate (e.g., `Q.integer_elements`).
  - `BM_elements`: checks all blocks of a `BlockMatrix`. `MS_elements`: delegates to parent matrix.

---

## SAT-Based Inference

### [`satask.py`](satask.py)
Second-tier SAT fallback, invoked when both handlers and `ask_full_inference` (in `ask.py`) are inconclusive.
- `satask()`: gathers expression-specific relevant facts (via `sathandlers.py` registry) and checks satisfiability.
- `get_relevant_facts()`, `get_all_relevant_facts()`: extract and expand relevant facts for a proposition.

### [`sathandlers.py`](sathandlers.py)
Registry of logical inference rules (implications, equivalences) keyed by expression type, plus old-to-new assumption bridging utilities for SAT solving.
- `_old_assump_replacer` / `evaluate_old_assump`: translates new-style predicates (`Q.positive`, `Q.negative`, …) to legacy `.is_*` attribute lookups.
  - Handles semantic mismatches: e.g., `Q.positive` requires both `is_finite` and `is_positive` (legacy "positive" doesn't exclude unbounded).
  - `CheckOldAssump`: wrapper asserting equivalence between a predicate and its old-assumption evaluation.
- `UnevaluatedOnFree`: base for deferred Boolean wrappers over predicates; `__new__` validates that input is either entirely free (unapplied) or singly applied to one expression.
  - Raises `ValueError` if bare predicates are mixed with expression-bound `AppliedPredicate`s, or if applied predicates target multiple distinct expressions.
  - On free input, stores `pred` and defers evaluation; on singly applied input, reconstructs the free form, sets `.expr`, and delegates to `apply()` hook.
- `AllArgs`, `AnyArgs`, `ExactlyOneArg`: vectorize a predicate over expression arguments.
- `ClassFactRegistry` / `fact_registry`: the expression-type-keyed **registry** of logical inference rules for SAT solving; `__getitem__` returns the union of rules for the queried class and all its registered superclasses (via `issubclass`), so child types inherit parent-type rules. `register_fact()` populates this registry.
- Module-level loop registers ~40 inference rules (e.g., `Mul` → `Implies(AllArgs(Q.positive), Q.positive)`) covering `Add`, `Mul`, `Pow`, `Abs`, `Number`, `Integer`, `MatMul`.

---

## Expression Refinement

### [`refine.py`](refine.py)
Simplifies expressions using assumptions.
- `refine(expr, assumptions)`: main entry point for assumption-based simplification.
- Type-specific handlers: `refine_abs`, `refine_Pow`, `refine_atan2`, `refine_Relational`.
- `handlers_dict`: maps expression class names to refinement handlers.
