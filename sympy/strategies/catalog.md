# sympy/strategies — Catalog

> Part of [SymPy](../catalog.md). Rule-based expression rewriting strategies (tree traversal, branching, and composition).

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package init that selectively re-exports symbols from submodules. Imports rules from `rl` (`rm_id`, `flatten`, `sort`, etc.), selected strategies from `core` (`chain`, `exhaust`, `do_one`, `minimize`, `tryit`, `condition`, `debug`, `null_safe`), and tools (`canon`, `typed`). Notably excludes `memoize` and `switch` from `core.py` — questions about missing or inaccessible package-level exports belong here. |
| `core.py` | Generic, SymPy-independent strategy combinators that each accept and return a single expression (not generators): `exhaust`, `condition`, `chain`, `debug`, `null_safe`, `tryit`, `do_one`, `switch`, `minimize`, and `memoize` (caching decorator for rules; not re-exported by `__init__.py`). |
| `rl.py` | Concrete rewrite rules for SymPy expressions: `rm_id`, `glom`, `sort`, `distribute`, `subs`, `unpack`, `flatten`, and `rebuild`. |
| `tools.py` | Higher-level SymPy-aware strategies built on the core combinators: `subs` (full simultaneous substitution via tree traversal), `canon` (bottom-up canonicalization), and `typed` (dispatch rules by expression type). |
| `tree.py` | Utilities for executing strategic trees (nested list/tuple structures where lists denote alternative choices and tuples denote sequential composition): `treeapply` (recursively apply join functions over nested lists/tuples), `greedy` (select alternatives that minimize an objective), `allresults` (enumerate all combinatorial outcomes of a strategic tree by multiplexing alternatives and chaining sequences), and `brute` (brute-force best result). Does not perform convergence or fixpoint iteration. |
| `traverse.py` | Single-result (non-generator) tree-traversal strategies for SymPy expressions: `top_down`, `bottom_up`, `top_down_once`, `bottom_up_once`, and `sall` (applies a rule to all children, returns a single rewritten expression; for leaf/atomic expressions with no children, returns the expression unchanged). |
| `util.py` | Shared utilities: the `new` constructor alias for `Basic.__new__`, an `assoc` helper for dict copying, and predefined function dicts (`basic_fns`, `expr_fns`) used by traversal strategies. |
| `branch/__init__.py` | Package init for the branching (generator-based) strategy subpackage; re-exports core branching combinators, traversal, and `canon`. |
| `branch/core.py` | Generator-based (branching) strategy combinators that yield multiple results via generators: `identity`, `exhaust` (repeatedly applies a branching rule until convergence — tracks seen expressions and falls back to yielding the original expression unchanged when the rule produces nothing new), `onaction`, `debug`, `multiplex`, `condition`, `sfilter`, `notempty` (guarantees non-empty output by falling back to original expr), `do_one`, `chain` (recursive decomposition with explicit empty-sequence base case), and `yieldify`. |
| `branch/tools.py` | Branching canonicalization strategy: `canon` applies branching rules top-down, multiplexes alternatives, and exhausts until stable. |
| `branch/traverse.py` | Generator-based (branching) tree-traversal strategies: `top_down` and `sall`. Unlike the single-result `traverse.py`, `sall` here is a generator that yields multiple possible results using `itertools.product` over children; for leaf/atomic expressions with no sub-expressions, it yields the expression as-is without decomposition. |
