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
- `to_int_repr` — convert CNF clauses to integer-set representation (used by DPLL solvers).
- `SOPform`, `POSform` — build canonical sum-of-products / product-of-sums from truth tables.
- `simplify_logic` — simplify boolean expressions via Quine-McCluskey or normal forms.
- `bool_map` — check equivalence of two boolean expressions by truth-table comparison.
- `_find_predicates` — extract atomic predicates from an expression.

## Inference and Evaluation

### [`inference.py`](inference.py)
Propositional-logic inference: truth evaluation, satisfiability dispatch, entailment, and knowledge bases.
- `pl_true(expr, model, deep)` — evaluate a propositional expression under a (possibly partial) truth assignment; returns True/False/None (three-valued).
  - With `deep=True`, tests all remaining unassigned atoms to determine if the expression is a tautology or contradiction.
- `satisfiable(expr, algorithm, all_models)` — check satisfiability; dispatches to `dpll` or `dpll2` algorithm backends.
- `valid(expr)` — check if expression is a tautology (true under every assignment).
- `entails(expr, formula_set)` — check logical entailment of expr from a set of formulas.
- `literal_symbol(literal)` — extract the underlying symbol from a (possibly negated) literal.
- `KB` — abstract base class for knowledge bases (tell/ask/retract interface).
- `PropKB(KB)` — simple propositional knowledge base backed by a clause set; ask delegates to `entails`.

## Algorithms

### [`algorithms/dpll.py`](algorithms/dpll.py)
Classic DPLL satisfiability solver with simple recursive backtracking; both symbolic and integer-encoded representations.
- `dpll_satisfiable(expr)` — top-level SAT solver entry point; converts to CNF then calls `dpll_int_repr`.
- `dpll(clauses, symbols, model)` — recursive DPLL on symbolic clause lists.
- `dpll_int_repr(clauses, symbols, model)` — recursive DPLL on integer-encoded clause sets.
  - After exhausting unit-propagation and pure-literal rules, picks an unassigned variable and recursively tries both True/False assignments (backtracking via short-circuit OR).
- `pl_true_int_repr(clause, model)` — lightweight three-valued truth evaluator for a single integer-encoded disjunctive clause.
  - Negative integers represent negated propositions (looks up absolute value and flips boolean).
  - Returns True/False/None depending on whether the clause is satisfied, falsified, or indeterminate under partial assignment.
- `unit_propagate` / `unit_propagate_int_repr` — simplify clauses by propagating unit clauses.
- `find_pure_symbol` / `find_pure_symbol_int_repr` — find symbols appearing with only one polarity.
- `find_unit_clause` / `find_unit_clause_int_repr` — find clauses with exactly one unbound literal.

### [`algorithms/dpll2.py`](algorithms/dpll2.py)
Modern iterative CDCL SAT solver (not recursive backtracking); uses clause learning, watched-literal scheme, and VSIDS heuristic.
- `dpll_satisfiable(expr, all_models)` — entry point; supports generating all satisfying models.
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
