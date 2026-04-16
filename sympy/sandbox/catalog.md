# sympy/sandbox — Experimental & Sandbox

Experimental/unstable code with no stability guarantees across SymPy versions.

## Package Init

### `__init__.py`
Package marker with a warning that all sandbox contents are experimental and may move.

## Indexed Integration

### `indexed_integrals.py`
Experimental integration over indexed variables (e.g. `A[i]`).

- **`IndexedIntegral(Integral)`** — Subclass of `Integral` that adds awareness of `Indexed` integration variables.
  - Replaces `Indexed` limits with `Dummy` symbols internally so the standard integration machinery works.
  - `doit()` — performs the integral, then substitutes the original indexed symbols back.
  - `_indexed_process_limits(limits)` — static helper that scans limits for `Indexed` objects and builds the replacement map.
- **Caveat:** Contraction of non-identical index symbols referring to the same `IndexedBase` is not supported.
