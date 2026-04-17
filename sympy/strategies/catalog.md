# strategies — Module Catalog

## Architecture Overview

This module provides a framework for expression rewriting via composable strategies.
A **rule** is a function `Expr -> Expr`; a **strategy** composes rules into higher-order rules.
Two parallel APIs exist:
- **Deterministic** (top-level): rules return a single expression.
- **Branching** (`branch/`): rules are generators that `yield` multiple alternative results (non-deterministic).

## Deterministic Strategies

### [`core.py`](core.py)
Generic, SymPy-independent deterministic strategy combinators. Each rule takes and returns a single expression.
- `exhaust(rule)` — apply a rule repeatedly in a while-loop until output equals input (fixed point); deterministic, returns one result.
- `memoize(rule)` — cache-wrapped rule.
- `condition(cond, rule)` — apply rule only when predicate holds.
- `chain(*rules)` — compose rules sequentially.
- `do_one(*rules)` — try each rule in order; return the first that changes the expression.
- `switch(key, ruledict)` — dispatch to a rule based on `key(expr)`.
- `minimize(*rules, objective=)` — apply all rules; return the result that minimizes the objective.
- `debug`, `null_safe`, `tryit` — wrappers for logging, None-safety, and exception-safety.

### [`rl.py`](rl.py)
Concrete rewriting rules (SymPy-aware) that operate on expression args.
- `rm_id(isid)` — remove identity elements from args.
- `glom(key, count, combine)` — conglomerate identical args (e.g., x + x → 2x).
- `sort(key)` — sort args by key.
- `distribute(A, B)` — distribute container A over B (e.g., Mul over Add).
- `subs(a, b)` — exact expression substitution.
- `unpack` — unwrap singleton args.
- `flatten` — flatten nested containers of the same type.
- `rebuild` — recursively reconstruct a SymPy tree to force canonicalization.

### [`traverse.py`](traverse.py)
Deterministic tree-traversal strategies that apply a rule across an expression tree.
- `top_down(rule)`, `bottom_up(rule)` — apply rule to every node top-down or bottom-up.
- `top_down_once(rule)`, `bottom_up_once(rule)` — stop after the first successful application.
- `sall(rule)` — apply rule to all args of one node.

### [`tools.py`](tools.py)
Composite deterministic strategies that depend on SymPy.
- `subs(d)` — full simultaneous exact substitution via top-down traversal.
- `canon(*rules)` — canonicalization: bottom-up exhaust of do_one, repeated until fixed point.
- `typed(ruletypes)` — dispatch rules by expression type via `switch`.

### [`tree.py`](tree.py)
Strategic trees: nested list/tuple structures encoding alternative and sequential rule composition.
- `treeapply(tree, join, leaf=)` — recursively apply join functions to a nested container.
- `greedy(tree, objective=)` — execute strategic tree greedily, picking the result that minimizes objective.
- `allresults(tree)` — execute strategic tree exhaustively; returns a lazy iterator of all possible results (uses `branch` combinators).
- `brute(tree, objective=)` — exhaustive search then pick minimum.

### [`util.py`](util.py)
Shared helpers: `new` (Basic constructor), `basic_fns` and `expr_fns` (tree-access function dicts).

## Branching (Non-Deterministic) Strategies — `branch/`

Generator-based counterparts of the deterministic strategies. Every branching rule is a **generator** that `yield`s zero or more alternative results.

### [`branch/core.py`](branch/core.py)
Generic, SymPy-independent branching strategy combinators. All rules are generators (`yield`-based).
- `exhaust(brule)` — recursively apply a branching rule until fixed point, tracking `seen` set to avoid revisiting results.
- `multiplex(*brules)` — merge outputs of multiple branching rules, deduplicating via `seen` set.
- `chain(*brules)` — compose branching rules sequentially; yields all combinations.
- `do_one(*brules)` — try branching rules in order; yield all results from the first that produces output.
- `condition(cond, brule)` — apply branching rule only when predicate holds.
- `sfilter(pred, brule)` — filter yielded results by predicate.
- `notempty(brule)` — guarantee at least one yield (original expr if rule yields nothing).
- `yieldify(rl)` — adapt a deterministic rule into a branching rule (single yield).
- `identity`, `onaction`, `debug` — branching versions of identity, action hooks, and debug logging.

### [`branch/traverse.py`](branch/traverse.py)
Branching tree-traversal strategies.
- `top_down(brule)` — apply branching rule top-down through the tree.
- `sall(brule)` — apply branching rule to all args; yields Cartesian product of per-arg results.

### [`branch/tools.py`](branch/tools.py)
Composite branching strategies.
- `canon(*rules)` — branching canonicalization: exhaust of multiplexed top-down traversals.
