# interactive — Interactive Session Setup

## [printing.py](printing.py)
Core print-system initialization for interactive sessions (both plain Python and IPython).
- `init_printing` — main entry point for configuring display of SymPy objects in any interactive session.
  - Auto-detects environment type (terminal vs rich GUI like notebook/QtConsole) and enables unicode and LaTeX accordingly.
  - Delegates to `_init_ipython_printing` or `_init_python_printing` based on detected environment.
  - Configures pretty-printer settings: order, unicode, wrap, column width.
- `_init_ipython_printing` — registers IPython display formatters (PNG/LaTeX/SVG/pretty-text) for SymPy types.
- `_init_python_printing` — installs a `sys.displayhook` for plain Python REPL sessions.
- `_is_ipython` — checks whether a given shell object is an IPython instance.

## [ipythonprinting.py](ipythonprinting.py)
**Deprecated** IPython extension shim (deprecated since 0.7.3). Delegates to `init_printing` in `printing.py`.
- `load_ipython_extension` — deprecated wrapper; issues a `SymPyDeprecationWarning` and calls `init_printing`.

## [session.py](session.py)
Interactive session bootstrapping and configuration.
- `init_session` — top-level entry to start an interactive SymPy session (IPython or plain Python).
- `init_ipython_session` — creates and configures an IPython app instance.
- `init_python_session` — creates a plain Python `code.InteractiveConsole`.
- `enable_automatic_symbols` — IPython hook that auto-creates undefined names as SymPy Symbols.
- `enable_automatic_int_sympification` — IPython hook that converts integer literals to `Integer`.
- `int_to_Integer` — source transformer converting int literals in code strings to `Integer(...)`.
