# sympy/interactive -- Catalog

> Part of [SymPy](../catalog.md). Interactive session helpers (isympy startup, printing configuration, IPython integration).

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package init that re-exports `init_printing` and `init_session`, the two main entry points for setting up interactive SymPy sessions. |
| `ipythonprinting.py` | Deprecated IPython extension (`%load_ext`) that delegates to `init_printing`. Exists for backward compatibility and emits a deprecation warning when loaded. |
| `printing.py` | Configures pretty-printing output formatters for interactive sessions, including Python `sys.displayhook` setup and IPython formatters for plain-text, LaTeX, PNG, SVG, and MathJax rendering of symbolic expressions. |
| `session.py` | Builds and launches interactive SymPy sessions (IPython or plain Python) via `init_session`. Handles IPython version detection, pylab/matplotlib plotting backend enablement (with broad exception handling for missing matplotlib or unavailable display), banner messages, automatic symbol creation, automatic int-to-Integer sympification, and pre-executed imports. |
