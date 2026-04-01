# sympy/unify — Catalog

> Part of [SymPy](../catalog.md). Unification algorithms (structural and expression-level pattern matching).

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package entry point; re-exports `unify` and `rebuild` from `usympy` and `rewriterule` from `rewrite`. |
| `core.py` | Generic unification engine for expression trees, supporting associative and commutative matching. Defines `Compound`, `Variable`, and `CondVariable` node types and the core `unify` algorithm. |
| `usympy.py` | SymPy-specific interface to the core unification engine. Converts SymPy expressions to/from `Compound` trees (`deconstruct`/`construct`) and provides a SymPy-aware `unify` that handles commutativity and associativity of SymPy operators. |
| `rewrite.py` | Builds rewrite rules from unification: `rewriterule` takes a source pattern, target pattern, and variables, and returns a function that rewrites matching expressions, with optional lambda conditions and assumption-based guards. |
