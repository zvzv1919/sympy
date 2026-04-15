# sympy/interactive -- Catalog

> Part of [SymPy](../catalog.md). Interactive session helpers (isympy startup, printing configuration, IPython integration).

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package init that re-exports `init_printing` and `init_session`, the two main entry points for setting up interactive SymPy sessions. |
| `ipythonprinting.py` | Deprecated IPython extension (`%load_ext`) that delegates to `init_printing`. Exists for backward compatibility and emits a deprecation warning when loaded. |
| `printing.py` | Configures pretty-printing for interactive sessions, including Python `sys.displayhook` setup and IPython formatters for plain-text, LaTeX, PNG, SVG, and MathJax output. Also contains shell-type detection helpers (e.g. checking whether a shell is an IPython instance) that attempt multiple import paths for the IPython shell base class across different versions, with fallback handling when import paths break due to backward-incompatible changes. |
| `session.py` | Builds and launches interactive SymPy sessions (IPython or plain Python), handling banner messages, automatic symbol creation, automatic int-to-Integer sympification, and pre-executed imports. Does **not** perform shell-type detection or import-path compatibility checks — see `printing.py` for that. |
