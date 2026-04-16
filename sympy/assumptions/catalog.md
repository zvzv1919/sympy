# sympy/assumptions — Assumption System

## Glossary

- **Predicate**: A named boolean-valued property (e.g. `Q.positive`). Wraps its argument as an `AppliedPredicate`.
- **Handler**: A class with static methods keyed by expression type name (e.g. `Add`, `Mul`, `Symbol`). Resolves a predicate query for that type.
- **`ask()`**: The main entry point — queries whether a proposition holds given assumptions, using handler dispatch, known-fact lookup, logical inference, and SAT fallback.
- **Known facts**: A static set of implications/equivalences among predicates (e.g. `Q.integer ⟹ Q.rational`), compiled to CNF for inference.
- **SAT fallback**: When handler dispatch and simple inference are inconclusive, `satask` encodes the problem as a SAT instance.

---

## Core Framework

### `__init__.py`

Public API re-exports: `AppliedPredicate`, `Predicate`, `AssumptionsContext`, `assuming`, `Q`, `ask`, `register_handler`, `remove_handler`, `refine`.

### `assume.py`

Core data model for assumptions.

- **`AssumptionsContext`** — `set` subclass holding global (or local) assumptions. Singleton instance: `global_assumptions`.
- **`AppliedPredicate(Boolean)`** — Result of calling a `Predicate` on an expression (e.g. `Q.positive(x)`). Marked `is_Atom = True` to prevent decomposition. Delegates evaluation to `self.func.eval(...)`.
- **`Predicate(Boolean)`** — Named boolean function. Key members:
  - `handlers` list — registered handler class paths.
  - `eval(expr, assumptions)` — walks the expression's MRO, dispatching to each handler's matching static method. Raises `ValueError` on conflicting resolutions.
  - `add_handler` / `remove_handler`.
- **`assuming(*assumptions)`** — Context manager that temporarily augments `global_assumptions`.

### `ask.py`

Main query engine and predicate catalogue.

- **`AssumptionKeys`** — Container whose `@predicate_memo` properties define every supported predicate (`Q.real`, `Q.positive`, `Q.invertible`, …). Memoized so each `Predicate` is a singleton (handlers register on the singleton).
  - *Scalar predicates*: `hermitian`, `antihermitian`, `real`, `extended_real`, `imaginary`, `complex`, `algebraic`, `transcendental`, `integer`, `rational`, `irrational`, `finite`, `infinite`, `positive`, `negative`, `zero`, `nonzero`, `nonnegative`, `nonpositive`, `even`, `odd`, `prime`, `composite`, `commutative`, `is_true`.
  - *Matrix predicates*: `symmetric`, `invertible`, `orthogonal`, `unitary`, `positive_definite`, `upper_triangular`, `lower_triangular`, `diagonal`, `fullrank`, `square`, `integer_elements`, `real_elements`, `complex_elements`, `singular`, `normal`, `triangular`, `unit_triangular`.
  - *Deprecated*: `bounded` → `finite`, `infinity` → `infinite`, `infinitesimal` → `zero`.
- **`Q`** — Module-level `AssumptionKeys()` instance.
- **`_extract_facts(expr, symbol)`** — Extracts assumption sub-expressions relevant to a given symbol from a compound assumption. Handles `Not(And(...))` ↔ `Or(...)` de Morgan rewrites.
- **`ask(proposition, assumptions, context)`** — Main entry point:
  1. Type-checks proposition and assumptions.
  2. Converts assumptions to CNF, extracts local facts for the expression.
  3. Checks consistency of local facts against known-facts CNF.
  4. Tries direct handler resolution via `key(expr)._eval_ask(...)`.
  5. Tries fast known-facts-dict lookup (single atom or conjunction of atoms).
  6. Falls back to `ask_full_inference` (full SAT check), then to `satask`.
- **`ask_full_inference(proposition, assumptions, known_facts_cnf)`** — Returns `True`/`False`/`None` by checking satisfiability of `proposition` and `¬proposition` against known facts + assumptions.
- `register_handler(key, handler)` / `remove_handler(key, handler)`
- **`compute_known_facts(known_facts, known_facts_keys)`** — Code-generates the contents of `ask_generated.py` (CNF + dict). Invoked by `bin/ask_update.py`.
- `single_fact_lookup` — Builds the fast single-key → implied-keys mapping.
- `get_known_facts()` — Returns the canonical `And(Implies(...), ...)` encoding all predicate relationships.
- `get_known_facts_keys()` — All non-deprecated predicates.
- `_handlers` list + registration loop — maps each predicate name to its handler class path and registers them at import time.

### `ask_generated.py`

Auto-generated file (via `bin/ask_update.py`). **Do not edit manually.**

- `get_known_facts_cnf()` — Returns the known-facts knowledge base as a single `And(Or(...), ...)` in CNF.
- `get_known_facts_dict()` — Returns `{predicate: set_of_implied_predicates}` for fast single-fact lookup.

### `refine.py`

Simplifies expressions using assumptions.

- **`refine(expr, assumptions)`** — Recursively refines sub-expressions, then dispatches to `_eval_refine` methods or the `handlers_dict`. Re-applies itself on changed expressions.
- **`refine_abs(expr, assumptions)`** — Simplifies `Abs(x)`: returns `x` if nonneg, `-x` if neg.
- **`refine_Pow(expr, assumptions)`** — Simplifies `Pow` under sign/parity knowledge:
  - `|base|^exp` when `base` is real and `exp` even.
  - Strips even/odd terms from exponents of `(-1)^(sum)`.
  - Handles `(-1)^((-1)^n/2 + m/2)` patterns.
- **`refine_atan2(expr, assumptions)`** — Reduces `atan2(y,x)` to `atan(y/x) ± π` (or `±π/2`, `nan`) based on sign assumptions.
- **`refine_Relational(expr, assumptions)`** — Delegates to `ask(Q.is_true(expr), ...)`.
- **`handlers_dict`** — Maps class names (`'Abs'`, `'Pow'`, `'atan2'`, relational types) to their refine handlers.

---

## SAT-Based Inference

### `satask.py`

SAT-solver fallback for `ask()`.

- **`satask(proposition, assumptions, context, ...)`** — Collects all relevant facts, then tests satisfiability of `proposition ∧ facts` and `¬proposition ∧ facts`. Returns `True`/`False`/`None` or raises `ValueError` on inconsistency.
- **`get_relevant_facts(proposition, assumptions, ...)`** — For each expression appearing in the proposition/assumptions, instantiates the known-facts CNF template and collects registered class-level facts from `fact_registry`. Returns `(relevant_facts, new_exprs)`.
- **`get_all_relevant_facts(...)`** — Fixed-point loop over `get_relevant_facts` — new facts may introduce new sub-expressions (e.g. `Q.zero(x*y)` → `Q.zero(x)`, `Q.zero(y)`), so it iterates until stable.

### `sathandlers.py`

Predicate vectorization helpers and the class-level fact registry used by `satask`.

- **`UnevaluatedOnFree(BooleanFunction)`** — Base class that remains unevaluated on free predicates and calls `apply()` when predicates are singly applied to one expression (via `.rcall(expr)`).
- **`AllArgs(UnevaluatedOnFree)`** — Vectorizes a predicate over all `.args`. E.g. `AllArgs(Q.positive).rcall(x*y)` → `And(Q.positive(x), Q.positive(y))`.
- **`AnyArgs(UnevaluatedOnFree)`** — Like `AllArgs` but with `Or`.
- **`ExactlyOneArg(UnevaluatedOnFree)`** — Exactly-one (xor-like) vectorization.
- **`_old_assump_replacer(obj)`** — Translates new-assumption predicates to old `.is_*` attribute checks (handles semantic gaps: e.g. old `real` includes infinity).
- **`evaluate_old_assump(pred)`** — Applies `_old_assump_replacer` via `xreplace`.
- **`CheckOldAssump(UnevaluatedOnFree)`** — Wraps `evaluate_old_assump` as an equivalence for SAT encoding.
- **`CheckIsPrime(UnevaluatedOnFree)`** — Checks `isprime()` for `Integer` instances.
- **`CustomLambda`** — Thin wrapper giving a lambda an `.rcall()` interface.
- **`ClassFactRegistry(MutableMapping)`** — Maps classes to frozensets of facts, with subclass-aware lookup.
- **`fact_registry`** — Module-level `ClassFactRegistry` instance. Populated at import time with ~50 rules covering `Mul`, `Add`, `Pow`, `Abs`, `Integer`, `Number`, `NumberSymbol`, `ImaginaryUnit`, and `MatMul`.

---

## Handlers

### `handlers/__init__.py`

Re-exports: `AskHandler`, `CommonHandler`, `AskCommutativeHandler`, `TautologicalHandler`, `test_closed_group`.

### `handlers/common.py`

Base handler classes and generic handlers.

- **`AskHandler`** — Empty base class all handlers inherit.
- **`CommonHandler(AskHandler)`** — Provides `AlwaysTrue`, `AlwaysFalse`, and `NaN = AlwaysFalse` static methods.
- **`AskCommutativeHandler(CommonHandler)`** — Handles `commutative` key. `Symbol` defaults to `True` unless explicitly non-commutative. `Basic` recurses over args.
- **`TautologicalHandler(AskHandler)`** — Handles `is_true` key. Implements boolean connective evaluation (`Not`, `Or`, `And`, `Implies`, `Equivalent`) by recursively calling `ask()`.
- **`test_closed_group(expr, assumptions, key)`** — Tests whether *all* args of `expr` satisfy `key` (group closure under the current operation). Uses `_fuzzy_group`.

### `handlers/calculus.py`

Handler for boundedness.

- **`AskFiniteHandler(CommonHandler)`** — Handles `finite` key.
  - `Symbol` — checks `.is_finite` or explicit `Q.finite` in assumptions.
  - `Add` — detailed truth-table logic considering bounded/unbounded/unknown terms with sign analysis.
  - `Mul` — truth-table for bounded × unbounded × unknown with nonzero checks.
  - `Pow` — rules like `Unbounded^NonZero → Unbounded`, `Bounded^Bounded → Bounded`, `|base|≤1 ^ Positive_exp → Bounded`.
  - `log`, `exp` — delegates to finiteness of argument.
  - Constants (`cos`, `sin`, `Number`, `Pi`, etc.) → `AlwaysTrue`.
  - `Infinity`, `NegativeInfinity` → `AlwaysFalse`.

### `handlers/ntheory.py`

Handlers for number-theoretic predicates.

- **`AskPrimeHandler(CommonHandler)`** — `prime` key.
  - `_number` helper — rounds and delegates to `isprime()`.
  - `Mul` — product of integers → not prime.
  - `Pow` — `Integer^Integer` → not prime.
  - `Integer` — direct `isprime()`.
- **`AskCompositeHandler(CommonHandler)`** — `composite` key. Derived from positive + integer + ¬prime (with special case for 1).
- **`AskEvenHandler(CommonHandler)`** — `even` key.
  - `Mul` — tracks even/odd/irrational factors with accumulator trick for pairs of unknowns.
  - `Add` — parity flip per odd addend.
  - `Pow` — even base with positive integer exp → even; odd base → not even.
  - `Abs`, `re`, `im` — delegates through realness.
- **`AskOddHandler(CommonHandler)`** — `odd` key. Derived as `integer ∧ ¬even`.

### `handlers/order.py`

Handlers for sign / order predicates.

- **`AskNegativeHandler(CommonHandler)`** — `negative` key.
  - `_number` — evaluates real/imag parts numerically.
  - `Add` — all-negative-or-nonpositive → negative.
  - `Mul` — sign parity (count negatives).
  - `Pow` — even exp → not negative; odd exp → same as base.
  - `exp` — real arg → not negative.
- **`AskNonNegativeHandler(CommonHandler)`** — `nonnegative` key via `fuzzy_not(negative) ∧ real`.
- **`AskNonZeroHandler(CommonHandler)`** — `nonzero` key.
  - `Add` — all positive or all negative → nonzero.
  - `Mul` — all args nonzero → nonzero.
  - `Pow` — base nonzero → nonzero.
- **`AskZeroHandler(CommonHandler)`** — `zero` key via `¬nonzero ∧ real`. `Mul` — any arg zero → zero.
- **`AskNonPositiveHandler(CommonHandler)`** — `nonpositive` key via `fuzzy_not(positive) ∧ real`.
- **`AskPositiveHandler(CommonHandler)`** — `positive` key.
  - `_number` — numeric evaluation of real/imag parts.
  - `Mul` — sign parity.
  - `Add` — all-positive-or-nonnegative → positive.
  - `Pow` — positive base → positive; negative base with even/odd exp.
  - `exp`, `log`, `factorial`, `Abs`, `Trace`, `Determinant`, `MatrixElement` — special-case rules.
  - Trig functions: `atan`, `asin`, `acos`, `acot`.

### `handlers/sets.py`

Handlers for set-membership predicates.

- **`AskIntegerHandler(CommonHandler)`** — `integer` key.
  - `Add`/`Pow` — closed group test.
  - `Mul` — checks for even product of half-integer, irrational factor.
  - `Abs`, `MatrixElement` / `Determinant` / `Trace` — delegates.
- **`AskRationalHandler(CommonHandler)`** — `rational` key.
  - `Add`/`Mul` — closed group.
  - `Pow` — rational if integer exponent; irrational if prime base with rational exp.
  - `exp`, `log`, trig — Hermite–Lindemann–Weierstrass-style rules.
- **`AskIrrationalHandler(CommonHandler)`** — `irrational` key via `real ∧ ¬rational`.
- **`AskRealHandler(CommonHandler)`** — `real` key.
  - `Mul` — real×real → real; real×imaginary parity.
  - `Pow` — extensive case analysis (imaginary base, imaginary exp via `log`, exp base with `I·π` multiples, etc.).
  - `sin`, `cos`, `exp`, `log` — special cases.
- **`AskExtendedRealHandler(AskRealHandler)`** — `extended_real` key. Overrides `Add`/`Mul`/`Pow` with closed-group test; `Infinity`/`NegativeInfinity` → True.
- **`AskHermitianHandler(AskRealHandler)`** — `hermitian` key. `Mul` tracks hermitian/antihermitian parity with max one noncommutative factor.
- **`AskComplexHandler(CommonHandler)`** — `complex` key. Closed group for `Add`/`Mul`/`Pow`. All numbers and standard functions → True.
- **`AskImaginaryHandler(CommonHandler)`** — `imaginary` key.
  - `Add` — all imaginary → imaginary; any real → not imaginary.
  - `Mul` — imaginary parity.
  - `Pow` — imaginary base with integer exp (odd/even), exp of `I·π` multiples, negative base with half-integer exp.
  - `log`, `exp` — special cases.
- **`AskAntiHermitianHandler(AskImaginaryHandler)`** — `antihermitian` key. Hermitian/antihermitian parity in `Mul`; integer-power rules in `Pow`.
- **`AskAlgebraicHandler(CommonHandler)`** — `algebraic` key.
  - `Add`/`Mul` — closed group.
  - `Pow` — algebraic if rational exponent and algebraic base.
  - `exp`, `log`, trig — Lindemann–Weierstrass consequences.

### `handlers/matrices.py`

Handlers for matrix predicates. Each handler resolves queries across `MatMul`, `MatAdd`, `MatrixSymbol`, `Identity`, `ZeroMatrix`, `Transpose`, `Inverse`, `MatrixSlice`, and (where applicable) `Factorization`, `DiagonalMatrix`, `DFT`.

- `_Factorization(predicate, expr, assumptions)` — checks if `predicate` is in `expr.predicates`.
- **`AskSquareHandler`** — `shape[0] == shape[1]`.
- **`AskSymmetricHandler`** — `MatMul`: all symmetric or `A…Aᵀ` pattern. `Transpose`/`Inverse` delegate. `ZeroMatrix` → square check.
- **`AskInvertibleHandler`** — `MatMul`: all invertible → True, any non-invertible → False. `Identity`/`Inverse` → True; `ZeroMatrix` → False.
- **`AskOrthogonalHandler`** — `MatMul`: all orthogonal with unit factor. `Identity` → True.
- **`AskUnitaryHandler`** — `MatMul`: all unitary with `|factor|=1`. `DFT` → True.
- **`AskFullRankHandler`** — `MatMul`: all fullrank → True.
- **`AskPositiveDefiniteHandler`** — `MatMul`: all PD or `AᵀBA` pattern. `MatAdd`: all PD → True.
- **`AskUpperTriangularHandler`** / **`AskLowerTriangularHandler`** — Closed under `MatMul`/`MatAdd`. `Transpose` cross-delegates.
- **`AskDiagonalHandler`** — Closed under `MatMul`/`MatAdd`. `DiagonalMatrix` → True.
- Helper functions: `BM_elements`, `MS_elements`, `MatMul_elements` — used by element-type handlers.
- **`AskIntegerElementsHandler`** / **`AskRealElementsHandler`** / **`AskComplexElementsHandler`** — Element-type predicates. Closed under `MatAdd`, `HadamardProduct`, `Determinant`, `Trace`, `Transpose`. `MatMul` uses `MatMul_elements` (splits scalar vs matrix factors).
