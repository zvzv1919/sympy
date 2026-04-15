# sympy/logic — Catalog

> Part of [SymPy](../catalog.md). Boolean algebra, propositional logic, satisfiability (DPLL), and inference.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package entry point; re-exports core Boolean operations (And, Or, Not, Xor, Implies, etc.), normal-form converters (to_cnf, to_dnf, to_nnf), and the `satisfiable` function. |
| `boolalg.py` | Core Boolean algebra module defining the `Boolean` base class and all gate classes (And, Or, Not, Xor, Nand, Nor, Implies, Equivalent, ITE), plus normal-form conversions (to_cnf, to_dnf, to_nnf), simplification, and truth tables. Also contains the **Quine-McCluskey truth-table minimization pipeline**: `SOPform` and `POSform` take a list of true-output (minterm) input combinations and optional don't-care conditions, derive the complement terms by enumerating all binary input combinations, then apply `_simplified_pairs` and `_rem_redundancy` to produce minimized sum-of-products or product-of-sums expressions. This is the module for constructing minimal CNF/DNF from truth tables — not the DPLL solvers in `algorithms/`. |
| `inference.py` | Propositional logic inference providing `satisfiable` (delegates to DPLL backends), `valid`, `entails`, `pl_true` (evaluate under a model), `literal_symbol`, and a simple `PropKB` knowledge base. |
| `algorithms/dpll.py` | Original DPLL **satisfiability solver** (checks whether a given Boolean formula is satisfiable and returns a model) using **recursive backtracking**: after deterministic simplifications (unit-clause forced-assignment propagation and pure-symbol/single-polarity elimination), it picks an unassigned variable, recursively tries `True` then `False` via short-circuit OR, and backtracks on failure. Does **not** construct or minimize normal forms from truth tables — see `boolalg.py` for that. Provides both symbolic (`dpll`) and integer-representation (`dpll_int_repr`) recursive variants plus `pl_true_int_repr`. |
| `algorithms/dpll2.py` | Improved DPLL solver (`SATSolver` class) using an **iterative** (non-recursive) search loop with clause learning, watched-literal (sentinel) scheme, and VSIDS branching heuristic; supports enumerating all models. Propagation is **queue-driven**: `_unit_prop` pops literals from `_unit_prop_queue` and checks each against already-assigned opposites — when a conflict is found (negation already in `var_settings`) it marks the theory unsatisfied, clears the queue, and returns `False`. `_simplify` loops unit propagation and pure-literal passes until no further changes occur. Backtracking is handled by flipping variable assignments inside `_find_model`'s loop, not by recursive calls. |
| `utilities/__init__.py` | Utilities sub-package init; re-exports `load_file` from the DIMACS module. |
| `utilities/dimacs.py` | Parser for the DIMACS CNF file format; converts DIMACS text or files into SymPy Boolean expressions (conjunctions of clauses). |
