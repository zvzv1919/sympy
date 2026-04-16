# sympy/logic — Catalog

> Part of [SymPy](../catalog.md). Boolean algebra, propositional logic, satisfiability (DPLL), and inference.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package entry point; re-exports core Boolean operations (And, Or, Not, Xor, Implies, etc.), normal-form converters (to_cnf, to_dnf, to_nnf), and the `satisfiable` function. |
| `boolalg.py` | Core Boolean algebra module defining the `Boolean` base class and all gate classes (And, Or, Not, Xor, Nand, Nor, Implies, Equivalent, ITE), plus normal-form conversions, simplification, truth tables, SOP/POS form construction. Also defines the CNF-to-integer encoding used by SAT solvers: `to_int_repr` converts CNF clauses into sets of signed integers (positive index for a symbol, negated/negative index for a `Not`-wrapped atom), and `term_to_integer`/`integer_to_term` convert between integers and binary-term lists. |
| `inference.py` | Propositional logic inference providing `satisfiable` (delegates to DPLL backends), `valid`, `entails`, `pl_true` (evaluate under a model), `literal_symbol`, and a simple `PropKB` knowledge base. |
| `algorithms/dpll.py` | Original **recursive-backtracking** DPLL satisfiability solver. Top-level `dpll_satisfiable(expr)` converts the expression to CNF, checks for trivially unsatisfiable input (returns `False` immediately if the boolean constant `False` appears among the conjunct clauses), then delegates to `dpll_int_repr`. The recursive search applies unit-clause propagation and pure-symbol simplification, then branches by picking an unassigned variable and recursing on both truth values. Includes helpers `find_unit_clause`, `find_pure_symbol`, `unit_propagate`, and their `_int_repr` counterparts, plus `pl_true_int_repr`. |
| `algorithms/dpll2.py` | Improved **iterative** (non-recursive) DPLL solver built around the `SATSolver` class with clause learning, VSIDS branching heuristic, and two-watched-literal scheme. Also exposes a `dpll_satisfiable(expr, all_models=False)` wrapper (supports enumerating all models via a generator). Uses an explicit decision-level stack and conflict-driven back-jumping — no recursive branching. |
| `utilities/__init__.py` | Utilities sub-package init; re-exports `load_file` from the DIMACS module. |
| `utilities/dimacs.py` | Parser for the DIMACS CNF standard benchmark file format. `load(s)` parses a string and `load_file` reads from disk, both producing SymPy Boolean expressions (And of Or clauses). Handles DIMACS format details: skips comment lines (`c …`), discards the problem header (`p cnf …`), treats zero literals as clause terminators (skips them), and maps signed integers to positive/negated `cnf_N` Symbols. |
