# sympy/external — Catalog

> Part of [SymPy](../catalog.md). Utilities for importing and probing optional external dependencies.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package init that re-exports `import_module` from `importtools`, providing a single entry point for optional-dependency imports. |
| `importtools.py` | Implements `import_module()`, which conditionally imports an external module with support for minimum version checks, minimum Python version checks, and configurable warning behavior. |
