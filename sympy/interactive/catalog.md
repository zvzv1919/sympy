# sympy/interactive — Interactive Session

Tools for initializing and configuring interactive SymPy sessions (used by `isympy`).

## Session Setup

### session.py

Constructs and launches interactive SymPy sessions (IPython or plain Python).

- **`preexec_source`** / **`verbose_message`** / **`no_ipython`** — template strings for the default startup code, banner, and missing-IPython warning.
- **`_make_message(ipython, quiet, source)`** — builds the session banner showing SymPy version, Python version, arch, ground types, cache/debug status.
- **`int_to_Integer(s)`** — tokenize-based transform that wraps integer literals in `Integer(…)` so `1/2` yields `Rational(1, 2)` instead of `0.5`.
- **`enable_automatic_int_sympification(app)`** — monkey-patches `app.shell.run_cell` to apply `int_to_Integer` transparently.
- **`enable_automatic_symbols(app)`** — installs a custom `NameError` handler that auto-creates `Symbol` objects for undefined names (`isympy -a`).
  - Caveat: re-executes the failing cell after injecting the symbol, which can cause side-effects (e.g. double-incremented counters).
- **`init_ipython_session(argv, auto_symbols, auto_int_to_Integer)`** — creates a `TerminalIPythonApp`, optionally enables auto-symbols/auto-int, returns the shell.
- **`init_python_session()`** — returns a `SymPyConsole` (subclass of `code.InteractiveConsole`) with readline history support.
- **`init_session(…)`** — top-level entry point: detects or creates an IPython/Python session, runs `preexec_source`, calls `init_printing`, displays the banner, and enters the mainloop.

## Printing

### printing.py

Configures pretty-printing for both plain Python and IPython sessions.

- **`_init_python_printing(stringify_func, **settings)`** — replaces `sys.displayhook` with one that calls `stringify_func`.
- **`_init_ipython_printing(ip, stringify_func, use_latex, …)`** — registers IPython display formatters:
  - `_print_plain` — text/plain via `stringify_func`.
  - `_print_latex_png` — image/png via external LaTeX, falls back to matplotlib's mathtext.
  - `_print_latex_matplotlib` — image/png via matplotlib only.
  - `_print_latex_text` — text/latex for MathJax rendering.
  - `_can_print_latex(o)` — recursive check for whether an object (including containers) is LaTeX-printable.
  - `_preview_wrapper` / `_matplotlib_wrapper` — internal renderers.
  - Handles IPython ≥ 0.11 formatter API and legacy `result_display` hook.
- **`_is_ipython(shell)`** — duck-type check for `InteractiveShell` instance.
- **`init_printing(pretty_print, order, use_unicode, use_latex, …)`** — public API. Auto-detects environment (IPython GUI vs terminal vs plain Python), sets global printer settings, and delegates to the appropriate init function.

### ipythonprinting.py

Deprecated IPython extension shim (`%load_ext sympy.interactive.ipythonprinting`).

- **`load_ipython_extension(ip)`** — emits a `SymPyDeprecationWarning` (deprecated since 0.7.3) and forwards to `init_printing`.

## Package Init

### \_\_init\_\_.py

Re-exports `init_printing` and `init_session`.
