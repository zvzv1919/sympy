# sympy/logic — Catalog

> Part of [SymPy](../catalog.md). Boolean algebra, propositional logic, satisfiability (DPLL), and inference.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package entry point; re-exports core Boolean operations (And, Or, Not, Xor, Implies, etc.), normal-form converters (to_cnf, to_dnf, to_nnf), and the `satisfiable` function. |
| `boolalg.py` | Core Boolean algebra module defining the `Boolean` base class and all gate classes (And, Or, Not, Xor, Nand, Nor, Implies, Equivalent, ITE), plus normal-form conversions, simplification, truth tables, SOP/POS form construction, and utilities to convert CNF clauses into compact signed-integer set representations (`to_int_repr`) and to convert between integer and binary-term formats. |
| `inference.py` | Propositional logic inference providing `satisfiable` (delegates to DPLL backends), `valid`, `entails`, `pl_true` (evaluate under a model), `literal_symbol`, and a simple `PropKB` knowledge base. |
| `algorithms/dpll.py` | Original DPLL satisfiability solver with unit propagation, pure-symbol elimination, and both symbolic and integer-representation variants; includes a lightweight three-valued truth evaluator for integer-encoded disjunctive clauses that handles negated literals (negative ints) and partial variable assignments. |
| `algorithms/dpll2.py` | Improved DPLL solver (`SATSolver` class) featuring clause learning, watched-literal scheme, and VSIDS branching heuristic; supports enumerating all models. |
| `utilities/__init__.py` | Utilities sub-package init; re-exports `load_file` from the DIMACS module. |
| `utilities/dimacs.py` | Parser for the DIMACS CNF file format; reads DIMACS-formatted text or files and produces SymPy Boolean expressions (conjunctions of clauses). Does not convert expressions back to integer representations. |
