# sympy/logic — Catalog

> Part of [SymPy](../catalog.md). Boolean algebra, propositional logic, satisfiability (DPLL), and inference.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package entry point; re-exports core Boolean operations (And, Or, Not, Xor, Implies, etc.), normal-form converters (to_cnf, to_dnf, to_nnf), and the `satisfiable` function. |
| `boolalg.py` | Core Boolean algebra module defining the `Boolean` base class and all gate classes (And, Or, Not, Xor, Nand, Nor, Implies, Equivalent, ITE), plus normal-form conversions, simplification, truth tables, and SOP/POS form construction. |
| `inference.py` | Propositional logic inference providing `satisfiable` (delegates to DPLL backends), `valid`, `entails`, `pl_true` (evaluate under a model), `literal_symbol`, and a simple `PropKB` knowledge base. |
| `algorithms/dpll.py` | Original DPLL satisfiability solver using **recursive backtracking**: after deterministic simplifications (unit-clause forced-assignment propagation and pure-symbol/single-polarity elimination), it picks an unassigned variable, recursively tries `True` then `False` via short-circuit OR, and backtracks on failure. Provides both symbolic (`dpll`) and integer-representation (`dpll_int_repr`) recursive variants, plus lightweight helpers (unit propagate, pure-symbol finder, unit-clause finder, `pl_true_int_repr`). |
| `algorithms/dpll2.py` | Improved DPLL solver (`SATSolver` class) using an **iterative** (non-recursive) search loop with clause learning, watched-literal (sentinel) scheme, and VSIDS branching heuristic; supports enumerating all models. Backtracking is handled by flipping variable assignments inside `_find_model`'s loop, not by recursive calls. |
| `utilities/__init__.py` | Utilities sub-package init; re-exports `load_file` from the DIMACS module. |
| `utilities/dimacs.py` | Parser for the DIMACS CNF file format; converts DIMACS text or files into SymPy Boolean expressions (conjunctions of clauses). |
