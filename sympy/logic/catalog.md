# sympy/logic — Catalog

> Part of [SymPy](../catalog.md). Boolean algebra, propositional logic, satisfiability (DPLL), and inference.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package entry point; re-exports core Boolean operations (And, Or, Not, Xor, Implies, etc.), normal-form converters (to_cnf, to_dnf, to_nnf), and the `satisfiable` function. |
| `boolalg.py` | Core Boolean algebra module defining the `Boolean` base class and all gate classes (And, Or, Not, Xor, Nand, Nor, Implies, Equivalent, ITE), plus normal-form conversions, simplification, truth tables, SOP/POS form construction, and utilities to convert CNF clauses into compact signed-integer set representations (`to_int_repr`) and to convert between integer and binary-term formats. |
| `inference.py` | Propositional logic inference providing `satisfiable` (delegates to DPLL backends), `valid`, `entails`, `pl_true` (evaluate under a model), `literal_symbol`, and a simple `PropKB` knowledge base. |
| `algorithms/dpll.py` | Original recursive DPLL satisfiability solver operating on symbolic expressions or integer-encoded clauses; uses pure-symbol elimination and a separate `find_unit_clause` scan to detect unit clauses at each recursive step; includes a three-valued truth evaluator for partial assignments. |
| `algorithms/dpll2.py` | Improved iterative DPLL solver (`SATSolver` class) with clause learning, VSIDS branching heuristic, and two-watched-literal (sentinel) scheme. During clause initialization (`_initialize_clauses`), unit clauses (single-literal) are immediately queued for propagation while multi-literal clauses have their first and last literals registered as watched sentinels; literal occurrence counts are also accumulated here. Supports enumerating all models. |
| `utilities/__init__.py` | Utilities sub-package init; re-exports `load_file` from the DIMACS module. |
| `utilities/dimacs.py` | Parser for the DIMACS CNF file format; reads DIMACS-formatted text or files and produces SymPy Boolean expressions (conjunctions of clauses). Does not convert expressions back to integer representations. |
