# sympy/logic — Catalog

> Part of [SymPy](../catalog.md). Boolean algebra, propositional logic, satisfiability (DPLL), and inference.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package entry point; re-exports core Boolean operations (And, Or, Not, Xor, Implies, etc.), normal-form converters (to_cnf, to_dnf, to_nnf), and the `satisfiable` function. |
| `boolalg.py` | Core Boolean algebra module defining the `Boolean` base class and all gate classes (And, Or, Not, Xor, Nand, Nor, Implies, Equivalent, ITE), plus normal-form conversions, simplification, truth tables, and SOP/POS form construction. |
| `inference.py` | Propositional logic inference providing `satisfiable` (delegates to DPLL backends), `valid`, `entails`, `pl_true` (evaluate under a model), `literal_symbol`, and a simple `PropKB` knowledge base. |
| `algorithms/dpll.py` | Original DPLL satisfiability solver with unit propagation, pure-symbol elimination, and both symbolic and integer-representation variants. Includes lightweight helper functions for integer-repr clauses: a clause truth evaluator (`pl_true_int_repr`) that checks literal truth against a partial model (with careful None-guarding for negated/unassigned literals), integer-repr unit propagation, and pure-symbol/unit-clause finders. |
| `algorithms/dpll2.py` | Improved DPLL solver (`SATSolver` class) featuring clause learning, watched-literal (sentinel) scheme, and VSIDS branching heuristic; supports enumerating all models. Literal assignment and BCP are handled via sentinel-based watched-literal data structures, not the helper-function approach in `dpll.py`. |
| `utilities/__init__.py` | Utilities sub-package init; re-exports `load_file` from the DIMACS module. |
| `utilities/dimacs.py` | Parser for the DIMACS CNF file format; converts DIMACS text or files into SymPy Boolean expressions (conjunctions of clauses). |
