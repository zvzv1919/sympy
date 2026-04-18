# Deprecated Module Catalog

Holds deprecations that could not remain in their original module (e.g., import-cycle avoidance).

## Class Registry

### `class_registry.py`

Global namespace (`C`) mapping string aliases to SymPy classes, avoiding cyclic imports.

- `ClassRegistry` — subclass of `Registry` backed by the global `all_classes` set.
  - `__setattr__` — registers a class by name and adds it to `all_classes`.
  - `__delattr__` — unregisters a name; only removes the class from `all_classes` if no other alias still maps to it (guards against premature removal of multi-aliased classes).
  - `__getattr__` — (deprecated) looks up a class by name in `all_classes`.
- Module-level singleton `C = ClassRegistry()` is the public entry point.
