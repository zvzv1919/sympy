# unify — Expression Tree Unification

Generic unification engine for symbolic expression trees, with SymPy adapters.

## Core Algorithm

### `core.py`
Generic unification algorithm for expression trees (language-agnostic).

- `Compound(op, args)` — interior tree node (analogous to `Basic` for non-Atoms); holds an operator and child args.
- `Variable(arg)` — wild token that matches any subtree.
- `CondVariable(arg, valid)` — wild token with an additional constraining predicate.
- `unify(x, y, s, **fns)` — main unification entry point; lazily yields variable-to-subtree mappings.
  - Accepts `is_associative` and `is_commutative` predicates as keyword args.
  - When both nodes are associative Compounds with the same op, regroups the **shorter** node's children against the longer node's children.
  - When both nodes are also commutative, uses a permutation-based enumeration; otherwise uses partition-based (order-preserving) enumeration.
- `unify_var` — handles substitution lookup and occurs-check for a single variable.
- `allcombinations(A, B, ordered)` — generates all ways to group elements of A and B; `ordered='associative'` or `'commutative'`.

## SymPy Adapter

### `usympy.py`
SymPy-specific interface that wraps `core.unify` — translates SymPy objects to/from `Compound` trees.

- `deconstruct(s, variables)` — converts a SymPy expression into a `Compound` tree; marks specified symbols as `Variable`.
- `construct(t)` — inverse of `deconstruct`; converts `Compound` back to a SymPy object.
- `unify(x, y, s, variables, **kwargs)` — public API; deconstructs inputs, calls `core.unify` with SymPy-aware associativity/commutativity predicates, reconstructs results.
- `is_associative` / `is_commutative` — predicates that classify SymPy operators (Add, Mul, MatAdd, MatMul, Union, Intersection, FiniteSet).
- `rebuild(s)` — round-trips a SymPy expression through deconstruct/construct to normalize it.

## Rewrite Rules

### `rewrite.py`
Pattern-driven expression rewriting built on top of `usympy.unify`.

- `rewriterule(source, target, variables, condition, assume)` — creates a rewrite rule that transforms expressions matching `source` into `target`.
