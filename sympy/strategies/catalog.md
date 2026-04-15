# sympy/strategies — Catalog

> Part of [SymPy](../catalog.md). Rule-based expression rewriting strategies (tree traversal, branching, and composition).

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package init that re-exports core rules (`rm_id`, `flatten`, `sort`, etc.), strategies (`chain`, `exhaust`, `do_one`, etc.), traversal helpers, and the `branch` subpackage. |
| `core.py` | Generic, SymPy-independent strategy combinators: `exhaust`, `memoize`, `condition`, `chain`, `debug`, `null_safe`, `tryit`, `do_one`, `switch` (dispatch on the result of a key function applied to an expression, falling back to identity), and `minimize`. Does **not** handle recursive/nested container structures — see `tree.py` for that. |
| `rl.py` | Concrete rewrite rules for SymPy expressions: `rm_id`, `glom`, `sort`, `distribute`, `subs`, `unpack`, `flatten`, and `rebuild`. |
| `tools.py` | Higher-level SymPy-aware strategies built on the core combinators: `subs` (full simultaneous substitution via tree traversal), `canon` (bottom-up canonicalization), and `typed` (dispatch rules by expression type). |
| `tree.py` | Utilities for executing strategic trees: `treeapply` (recursively walk nested containers, dispatching on each node's **container type** via `isinstance` against a dict of type→function; nodes whose type is not in the dispatch dict fall through to a `leaf` handler instead of recursing), `greedy` (select alternatives that minimize an objective), `allresults` (exhaustively enumerate all outcomes), and `brute` (brute-force best result). Unlike `core.py::switch`, which dispatches on a computed key, `treeapply` dispatches on the structural type of the node itself and recurses into children. |
| `traverse.py` | Tree-traversal strategies for SymPy expressions: `top_down`, `bottom_up`, `top_down_once`, `bottom_up_once`, and `sall` (apply a rule to all children). |
| `util.py` | Shared utilities: the `new` constructor alias for `Basic.__new__`, an `assoc` helper for dict copying, and predefined function dicts (`basic_fns`, `expr_fns`) used by traversal strategies. |
| `branch/__init__.py` | Package init for the branching (generator-based) strategy subpackage; re-exports core branching combinators, traversal, and `canon`. |
| `branch/core.py` | Generator-based (branching) strategy combinators that yield multiple results: `identity`, `exhaust`, `onaction`, `debug`, `multiplex`, `condition`, `sfilter`, `notempty`, `do_one`, `chain`, and `yieldify`. |
| `branch/tools.py` | Branching canonicalization strategy: `canon` applies branching rules top-down, multiplexes alternatives, and exhausts until stable. |
| `branch/traverse.py` | Branching tree-traversal strategies: `top_down` and `sall`, which yield all possible results from applying a branching rule across a SymPy tree using `itertools.product`. |
