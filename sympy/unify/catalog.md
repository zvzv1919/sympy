# sympy/unify — Catalog

> Part of [SymPy](../catalog.md). Unification algorithms (structural and expression-level pattern matching).

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package entry point; re-exports `unify` and `rebuild` from `usympy` and `rewriterule` from `rewrite`. |
| `core.py` | Low-level generic unification algorithm for abstract `Compound`/`Variable`/`CondVariable` tree nodes (not SymPy expressions). Implements the tree-walking and substitution logic; does not know about SymPy types, operator commutativity, or expression reconstruction. Used as a backend by `usympy.py`. |
| `usympy.py` | **Primary entry point** for structural pattern matching (unification) on SymPy expressions. Converts SymPy objects to/from intermediate `Compound` trees (`deconstruct`/`construct`), defines SymPy-aware commutativity and associativity predicates (e.g., `Add` is commutative, `Mul` checks factors), and provides the main `unify` function that yields all valid substitution mappings—supporting free reordering and regrouping of operands. |
| `rewrite.py` | Builds rewrite rules from unification: `rewriterule` takes a source pattern, target pattern, and variables, and returns a function that rewrites matching expressions, with optional lambda conditions and assumption-based guards. |
