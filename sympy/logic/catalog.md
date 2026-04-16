# sympy/logic — Catalog

> Part of [SymPy](../catalog.md). Boolean algebra, propositional logic, satisfiability (DPLL), and inference.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package entry point; re-exports core Boolean operations (And, Or, Not, Xor, Implies, etc.), normal-form converters (to_cnf, to_dnf, to_nnf), and the `satisfiable` function. |
| `boolalg.py` | Core Boolean algebra module defining the `Boolean` base class and all gate classes (And, Or, Not, Xor, Nand, Nor, Implies, Equivalent, ITE), plus normal-form conversions, simplification, truth tables, SOP/POS form construction, and utilities to convert CNF clauses into compact signed-integer set representations (`to_int_repr`) and to convert between integer and binary-term formats. |
| `inference.py` | Propositional logic inference providing `satisfiable` (delegates to DPLL backends), `valid`, `entails`, `pl_true` (evaluate under a model), `literal_symbol`, and a simple `PropKB` knowledge base. |
| `algorithms/dpll.py` | Original recursive-backtracking DPLL satisfiability solver with both symbolic (`dpll`) and integer-encoded (`dpll_int_repr`) variants. After exhausting forced assignments (unit-clause propagation and pure-symbol elimination), the algorithm picks an unassigned variable, forks the partial model, and recursively tries both truth values with short-circuit disjunction — this is the core branching/backtracking mechanism. Includes helper functions (`find_unit_clause`, `find_pure_symbol`, `unit_propagate`, and their `_int_repr` counterparts) and a three-valued truth evaluator for partial assignments. |
| `algorithms/dpll2.py` | Improved **iterative** (non-recursive) DPLL solver (`SATSolver` class) with clause learning, VSIDS branching heuristic, and two-watched-literal (sentinel) scheme. Uses an explicit decision-level stack instead of recursive backtracking. Supports enumerating all models. |
| `utilities/__init__.py` | Utilities sub-package init; re-exports `load_file` from the DIMACS module. |
| `utilities/dimacs.py` | Parser for the DIMACS CNF standard benchmark file format. `load(s)` parses a string and `load_file` reads from disk, both producing SymPy Boolean expressions (And of Or clauses). Handles DIMACS format details: skips comment lines (`c …`), discards the problem header (`p cnf …`), treats zero literals as clause terminators (skips them), and maps signed integers to positive/negated `cnf_N` Symbols. |
