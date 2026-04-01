# sympy/deprecated -- Catalog

> Part of [SymPy](../catalog.md). Deprecated modules with import-time warnings pointing to replacements.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package init that documents why certain deprecations live here (import cycles, removed modules) and re-exports `C` and `ClassRegistry` from `class_registry`. |
| `class_registry.py` | Defines `ClassRegistry`, a deprecated namespace registry (`C`) that allowed looking up SymPy classes by name (e.g. `C.Add`) to avoid cyclic imports; deprecated since SymPy 1.0 in favor of direct imports. |
