# sympy/deprecated — Deprecated Modules

Houses deprecations that cannot remain in their original modules (e.g., removed modules or import-cycle issues). SymPy imports this last, after all other modules.

## Package Init

### `__init__.py`
Re-exports `C` and `ClassRegistry` from `class_registry`.

## Class Registry

### `class_registry.py`
Provides a deprecated namespace (`C`) for accessing SymPy classes by name, originally used to avoid cyclic imports.

- **`ClassRegistry(Registry)`** — Registry subclass that keeps `all_classes` in sync.
  - `__setattr__` — registers a class and adds it to `all_classes`.
  - `__delattr__` — unregisters a class, removing it from `all_classes` only if no other name still maps to it.
  - `__getattr__` — **deprecated since 1.0**; looks up a class by name from `all_classes`. Users should import classes directly instead.
- **`C`** — module-level `ClassRegistry` singleton, pre-seeded with `BasicMeta`.
