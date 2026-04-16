# sympy/strategies — Rule-based Strategies

> **Experimental module.** The interface is subject to change.

A *rule* is a function `Expr -> Expr` that transforms one expression into another. A *strategy* is a higher-order function that controls *how* rules are applied to a syntax tree, separating mathematical transformations from algorithmic plumbing.

The `branch` sub-package mirrors the top-level API but works with *branching rules* (`Expr -> Iterator[Expr]`) that can yield multiple results.

## Glossary

| Term | Meaning |
|------|---------|
| **rule** | `Expr -> Expr` — a deterministic rewrite |
| **branching rule (brule)** | `Expr -> Iterator[Expr]` — a non-deterministic rewrite yielding multiple results |
| **strategy** | Higher-order function: takes rule(s), returns a new rule |
| **strategic tree** | Nested list/tuple structure encoding alternative and sequential strategy composition |

---

## Core Strategies

### `core.py`
Generic, SymPy-independent strategy combinators.

- **`exhaust(rule)`** — Apply a rule repeatedly until it has no effect (fixed-point).
- **`memoize(rule)`** — Wrap a rule with a dictionary cache.
- **`condition(cond, rule)`** — Guard: only apply `rule` when `cond(expr)` is true.
- **`chain(*rules)`** — Compose rules sequentially (pipeline).
- **`debug(rule, file=None)`** — Logging wrapper: prints before/after when the expression changes.
- **`null_safe(rule)`** — If the rule returns `None`, return the original expression instead.
- **`tryit(rule)`** — If the rule raises an exception, return the original expression.
- **`do_one(*rules)`** — Try each rule in order; return the first result that differs from the input.
- **`switch(key, ruledict)`** — Dispatch: select a rule from `ruledict` based on `key(expr)`.
- **`minimize(*rules, objective=identity)`** — Apply all rules, return the result that minimizes `objective`.
- `identity` — The identity function (lambda).

### `rl.py`
Concrete rewrite rules that assume knowledge of `Basic`.

- **`rm_id(isid)`** — Rule factory: remove identity elements from args (e.g. zeros from Add). Keeps at least one arg.
- **`glom(key, count, combine)`** — Rule factory: conglomerate identical args by grouping on `key`, summing via `count`, and recombining with `combine` (e.g. `x + x → 2x`).
- **`sort(key)`** — Rule factory: sort `expr.args` by `key`.
- **`distribute(A, B)`** — Rule factory: distribute container `A` over container `B` (e.g. `Mul` over `Add`).
- **`subs(a, b)`** — Rule factory: exact substitution of expression `a` with `b`.
- `unpack(expr)` — Unwrap singleton args: `T(x) → x`.
- `flatten(expr)` — Flatten nested containers of the same type: `T(a, T(b, c)) → T(a, b, c)`.
- `rebuild(expr)` — Recursively reconstruct an expression tree via its constructors, forcing re-canonicalization.

---

## Traversal

### `traverse.py`
Strategies that walk a SymPy expression tree, applying a rule at each node.

- **`top_down(rule, fns=basic_fns)`** — Apply rule at the root first, then recurse into children.
- **`bottom_up(rule, fns=basic_fns)`** — Recurse into children first, then apply rule at the root.
- `top_down_once(rule)` — Like `top_down` but stops as soon as the rule fires.
- `bottom_up_once(rule)` — Like `bottom_up` but stops as soon as the rule fires.
- **`sall(rule, fns=basic_fns)`** — Apply a rule to every child arg; reconstruct the node. Building block for the above.

---

## Composite Strategies (tools)

### `tools.py`
Higher-level strategies that combine primitives from `core`, `rl`, and `traverse`.

- **`subs(d)`** — Full simultaneous exact substitution over a tree. Builds a `top_down(do_one(...))` pipeline from a mapping dict.
- **`canon(*rules)`** — Canonicalization strategy: `exhaust(top_down(exhaust(do_one(*rules))))`.
- **`typed(ruletypes)`** — Dispatch rules by expression type via `switch(type, ruletypes)`.

---

## Strategic Trees

### `tree.py`
Encode complex strategy plans as nested Python data structures (lists = alternatives, tuples = sequences).

- **`treeapply(tree, join, leaf=identity)`** — Generic recursive applicator over a nested list/tuple tree, dispatching on container type.
- **`greedy(tree, objective=identity)`** — Execute a strategic tree greedily: at each list node, pick the alternative that minimizes `objective`. Uses `minimize` for lists, `chain` for tuples.
- **`allresults(tree, leaf=yieldify)`** — Execute a strategic tree exhaustively, returning a lazy iterator of all possible results (can cause combinatorial blowup). Uses branching `multiplex`/`chain`.
- **`brute(tree, objective=identity)`** — Compute all results via `allresults`, then pick the global minimum under `objective`.

---

## Branching Sub-package (`branch/`)

Mirrors the deterministic API but every combinator works with *branching rules* that `yield` multiple results.

### `branch/core.py`
Branching analogues of `core.py`.

- `identity(x)` — Yield `x` unchanged.
- **`exhaust(brule)`** — Repeatedly apply a branching rule, tracking seen expressions to avoid cycles, until no new results appear.
- **`onaction(brule, fn)`** — Callback wrapper: call `fn(brule, expr, result)` for every result that differs from the input.
- **`debug(brule, file=None)`** — Logging wrapper built on `onaction`.
- **`multiplex(*brules)`** — Merge multiple branching rules into one, deduplicating results.
- **`condition(cond, brule)`** — Guard: only yield from `brule` when `cond(expr)` is true.
- **`sfilter(pred, brule)`** — Filter: yield only results satisfying `pred`.
- **`notempty(brule)`** — Guarantee at least one yield (the original expr if the rule produces nothing).
- **`do_one(*brules)`** — Try branching rules in order; yield from the first one that produces results.
- **`chain(*brules)`** — Sequential composition: pipe each intermediate result through the remaining rules.
- **`yieldify(rl)`** — Adapter: wrap a deterministic rule as a branching rule (single yield).

### `branch/traverse.py`
Branching tree-traversal strategies.

- **`top_down(brule, fns=basic_fns)`** — Branching top-down traversal using `do_one` + `sall`.
- **`sall(brule, fns=basic_fns)`** — Apply a branching rule to every child; yield all combinations via `itertools.product`.

### `branch/tools.py`
Composite branching strategies.

- **`canon(*rules)`** — Branching canonicalization: `exhaust(multiplex(*map(top_down, rules)))`.

---

## Utilities

### `util.py`
Shared helpers for tree operations.

- `new` — Alias for `Basic.__new__`; used as the default node constructor.
- `assoc(d, k, v)` — Non-mutating dict update (returns a copy with `k → v`).
- `basic_fns` — Dict of tree-access functions (`op`, `new`, `children`, `leaf`) for `Basic` trees.
- `expr_fns` — Variant of `basic_fns` whose `new` calls `op(*args)` directly (triggers evaluation).

### `__init__.py`
Public API re-exports: `rm_id`, `unpack`, `flatten`, `sort`, `glom`, `distribute`, `rebuild`, `new`, `condition`, `debug`, `chain`, `null_safe`, `do_one`, `exhaust`, `minimize`, `tryit`, `canon`, `typed`, plus the `rl`, `traverse`, and `branch` submodules.
