# sympy/external — External Library Imports

Unified place for safely importing optional external dependencies with version checking and warning control.

## Package Initialization

### `__init__.py`

Re-exports `import_module` from `importtools` as the public API for the subpackage.

## Import Utilities

### `importtools.py`

Tools to assist importing optional external modules, with version gating and configurable warnings.

- **Module-level settings**
  - `WARN_NOT_INSTALLED` / `WARN_OLD_VERSION` — global overrides for warning behavior (auto-enabled when `SYMPY_DEBUG` is set).

- **`import_module(module, ...)`** — Import and return a module, or `None` if unavailable/too old.
  - Checks `min_python_version` against `sys.version_info` before attempting import.
  - Skips NumPy on PyPy (rudimentary support causes errors).
  - Imports via `__import__` with forwarded `__import__kwargs` (needed for submodule imports like `mpl_toolkits.mplot3d`).
  - Works around a matplotlib/py3k bug where `from matplotlib import collections` resolves to stdlib `collections`.
  - Catches `ImportError` plus any extra exception types passed via `catch`.
  - Compares `module_version_attr` (default `__version__`, optionally callable) against `min_module_version`.
  - Warning precedence: global override > keyword argument > default (`WARN_NOT_INSTALLED=False`, `WARN_OLD_VERSION=True`).
