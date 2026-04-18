# logic — Module Catalog

## Boolean Algebra

### [`boolalg.py`](boolalg.py)
Core boolean types, operators, and normal-form conversions.
- `Boolean` — base class for symbolic boolean expressions.
- `BooleanTrue`, `BooleanFalse` — singleton truth constants (`true`/`false`).
- `BooleanFunction` — base class for `And`, `Or`, `Not`, `Xor`, `Nand`, `Nor`, `Implies`, `Equivalent`, `ITE`.
- `And`, `Or`, `Not` — fundamental boolean connectives.
- `Xor`, `Nand`, `Nor`, `Implies`, `Equivalent`, `ITE` — derived connectives.
- `to_cnf`, `to_dnf`, `to_nnf` — convert expressions to conjunctive/disjunctive/negation normal form.
- `is_cnf`, `is_dnf` — validate whether a formula is in conjunctive (product-of-sums) or disjunctive (sum-of-products) normal form.
  - Both delegate to `_is_form`, a shared two-level structural validator parameterised by outer/inner connective.
  - Handles edge cases: bare atoms, single negated atoms, and single-clause expressions all return True.
- `to_int_repr` — convert CNF clauses to integer-set representation (used by DPLL solvers).
- `SOPform`, `POSform` — build canonical sum-of-products / product-of-sums from truth tables.
- `simplify_logic(expr, form, deep)` — simplify boolean expressions; accepts `form='cnf'`, `'dnf'`, or `None`.
  - When `form=None`, auto-selects DNF (via `SOPform`) if truth table covers ≥ half of all rows, else CNF (via `POSform`).
- `_finger(eq)` — compute a 5-element structural fingerprint tuple per variable in a boolean expression (occurrence counts as bare symbol, negated, in compounds, etc.); groups variables with identical signatures as interchangeable.
- `bool_map` — check equivalence of two boolean expressions under variable renaming; uses `_finger` to match variables by structural signature.
- `_find_predicates` — extract atomic predicates from an expression.
- Quine-McCluskey minimization helpers (used internally by `SOPform`/`POSform`):
  - `_check_pair` — test if two binary term lists differ in exactly one bit position.
  - `_simplified_pairs` — one QM reduction pass: merge all pairs differing in one bit (replacing that bit with don't-care 3), collect unmerged terms.
  - `_rem_redundancy` — prime-implicant table method to eliminate redundant terms after simplification.

## Inference and Evaluation

### [`inference.py`](inference.py)
Propositional-logic inference: truth evaluation, satisfiability dispatch, entailment, and knowledge bases.
- `pl_true(expr, model, deep)` — evaluate a propositional expression under a (possibly partial) truth assignment; returns True/False/None (three-valued).
  - With `deep=True`, uses a heuristic probe: sets all unassigned atoms to True, evaluates, then branches — checks tautology (`valid`) if probe is true, checks unsatisfiability (`satisfiable`) if probe is false.
- `satisfiable(expr, algorithm, all_models)` — **public entry point** for SAT checking; thin dispatcher that selects backend (`dpll` or `dpll2`) by `algorithm` param and forwards `all_models`; all model-finding and edge-case logic lives in the backend.
- `valid(expr)` — check if expression is a tautology (true under every assignment).
- `entails(expr, formula_set)` — check logical entailment of expr from a set of formulas.
- `literal_symbol(literal)` — extract the underlying symbol from a (possibly negated) literal.
- `KB` — abstract base class for knowledge bases (tell/ask/retract interface).
- `PropKB(KB)` — simple propositional knowledge base backed by a clause set; ask delegates to `entails`.

## Algorithms

### [`algorithms/dpll.py`](algorithms/dpll.py)
Internal classic DPLL SAT backend (called by `inference.satisfiable`); simple recursive backtracking on symbolic and integer-encoded representations.
- `dpll_satisfiable(expr)` — backend entry point; converts to CNF, early-exits with `False` if any conjunct is literal `False`, then delegates to `dpll_int_repr`. Does not support `all_models`.
- `dpll(clauses, symbols, model)` — recursive DPLL on symbolic clause lists.
  - Runs unit-clause and pure-literal elimination loops, evaluates remaining clauses, then branches on an unassigned variable (True first, False on backtrack via short-circuit OR).
- `dpll_int_repr(clauses, symbols, model)` — recursive DPLL on integer-encoded clause sets; same branch-and-backtrack logic as `dpll`.
- `pl_true_int_repr(clause, model)` — lightweight three-valued truth evaluator for a single integer-encoded disjunctive clause.
  - Negative integers represent negated propositions (looks up absolute value and flips boolean).
  - Returns True/False/None depending on whether the clause is satisfied, falsified, or indeterminate under partial assignment.
- `unit_propagate(clauses, symbol)` — simplify a symbolic clause list by a unit literal: removes clauses containing the symbol; deletes ¬symbol from others.
  - Non-`Or` clauses (single literals) are kept unconditionally without symbol-matching.
- `unit_propagate_int_repr` — integer-encoded equivalent; filters out clauses containing the symbol and strips its negation from the rest.
- `find_pure_symbol` / `find_pure_symbol_int_repr` — find symbols appearing with only one polarity.
- `find_unit_clause(clauses, model)` — iterate disjunctive clauses counting literals not yet bound in `model`; when exactly one literal is unbound, return that variable and its forced truth value (positive literal → True, negated → False).
- `find_unit_clause_int_repr` — integer-encoded equivalent of `find_unit_clause`.

### [`algorithms/dpll2.py`](algorithms/dpll2.py)
Internal CDCL SAT backend (called by `inference.satisfiable`); iterative solver with clause learning, watched literals, and VSIDS heuristic.
- Standalone clause helpers (`unit_propagate`, `find_unit_clause`, `find_pure_symbol`, `pl_true_int_repr`) live in `dpll.py`, not here; this module has no three-valued clause truth evaluator.
- `dpll_satisfiable(expr, all_models)` — backend entry point; converts to CNF, handles `all_models` semantics. Early-exits for trivially false formulas (returns `False`, or a generator yielding `False` when `all_models=True`). Otherwise delegates to `SATSolver` and wraps results via `_all_models` generator.
- `_all_models(models)` — wrapper generator that yields all satisfying assignments; if the underlying solver produces no models at all, yields `False` as a sentinel to signal unsatisfiability.
- `SATSolver` — stateful SAT solver class operating on integer-encoded clauses.
  - Uses watched-literal data structures for efficient unit propagation.
  - VSIDS (Variable State Independent Decaying Sum) branching heuristic.
  - Conflict-driven clause learning via `_simple_compute_conflict`.
- `Level` — helper class representing a decision level in the search tree.

## Utilities

### [`utilities/dimacs.py`](utilities/dimacs.py)
Parser for the DIMACS CNF file format (standard SAT benchmark text format) into SymPy boolean expressions.
- `load(s)` — parse a DIMACS-format string into a conjunction-of-disjunctions (`And` of `Or`s).
  - Each line is a clause of space-separated integers (negative = negated variable, zero = clause terminator); skips comment/header lines.
- `load_file(location)` — read a DIMACS file from disk and delegate to `load`.
