# external — Optional Dependency Management

## Package Entry Point

### [`__init__.py`](__init__.py)
Unified public entry point for optional dependency loading. Re-exports `import_module` so the rest of the codebase imports it from `sympy.external`. Returns `None` when a package is absent or below a required version.

## Implementation

### [`importtools.py`](importtools.py)
Implementation of the optional-import machinery.
- `import_module()` — performs the actual import via `__import__`, checks Python/module version constraints, emits warnings, and returns the module or `None`.
- `WARN_NOT_INSTALLED`, `WARN_OLD_VERSION` — module-level flags controlling default warning behavior.
