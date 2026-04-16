# sympy/unify — Unification

## Core Engine

### core.py
Generic unification algorithm for expression trees with lists of children. Direct translation of AIMA (Russell & Norvig) §9.2, extended with associative/commutative matching (combinatorial) and lazy exploration.

- **`Compound(op, args)`** — Interior tree node (analogous to `Basic` for non-Atoms). Stores an operator and a tuple of arguments.
- **`Variable(arg)`** — Wild token that matches anything.
- **`CondVariable(arg, valid)`** — Wild token that matches only when `valid(match)` returns true.
- **`unify(x, y, s=None, **fns)`** — Main unification entry point. Yields lazy mappings `{Variable: subtree}`.
  - Handles plain equality, variable binding, `Compound` structural matching, and recursive list/tuple matching.
  - Accepts `is_commutative` and `is_associative` callbacks to enable AC-matching via `allcombinations`.
- **`unify_var(var, x, s, **fns)`** — Helper: binds a variable, respecting existing substitutions and `CondVariable` guards.
- **`occur_check(var, x)`** — Prevents infinite substitutions (standard occur-check).
- **`allcombinations(A, B, ordered)`** — Restructures two argument tuples so the longer one is partitioned into sublists matching the shorter's length; `ordered` selects associative vs commutative enumeration (uses `kbins`).
- `partition(it, part)`, `index(it, ind)` — Fancy indexing/partitioning helpers for `allcombinations`.
- `assoc`, `is_args`, `unpack` — trivial helpers.

## SymPy Integration

### usympy.py
SymPy-specific interface to the unification engine. Bridges SymPy expression trees (`Basic`) and the generic `Compound`/`Variable` representation.

- **`deconstruct(s, variables=())`** — Converts a SymPy object into a `Compound` tree; symbols listed in `variables` become `Variable` nodes.
- **`construct(t)`** — Inverse of `deconstruct`; rebuilds a SymPy object from a `Compound` tree.
  - Respects `eval_false_legal` (passes `evaluate=False` for `AssocOp`, `Pow`, `FiniteSet`) and `basic_new_legal` (uses `Basic.__new__` for `MatrixExpr`).
- **`unify(x, y, s=None, variables=(), **kwargs)`** — Top-level SymPy unifier. Deconstructs both sides, calls `core.unify` with SymPy-aware AC predicates, then reconstructs results.
- **`rebuild(s)`** — Round-trips through `deconstruct`/`construct` to normalize expressions damaged by Expr-Rules interactions.
- `sympy_associative(op)`, `sympy_commutative(op)` — Check if a SymPy operator class is associative/commutative (hard-coded lists: `Add`, `Mul`, `MatAdd`, `MatMul`, set ops, etc.).
- `is_associative(x)`, `is_commutative(x)` — Compound-level wrappers passed as callbacks to `core.unify`.
- `mk_matchtype(typ)` — Factory returning a predicate that matches a given SymPy type or a `Compound` whose op subclasses it.

**Caveats:** `is_commutative` for `Mul` compounds falls back to checking each arg's `.is_commutative` attribute via `construct`, mixing tree representations mid-check.

### rewrite.py
Builds rewrite rules on top of the SymPy unifier.

- **`rewriterule(source, target, variables=(), condition=None, assume=None)`** — Returns a callable `rewrite_rl(expr, assumptions=True)` that yields all target expressions obtainable by unifying `expr` against `source` and substituting into `target`.
  - `condition` — optional lambda filter on matched variable values.
  - `assume` — optional assumptions predicate checked via `ask`.

## Package Init

### \_\_init\_\_.py
Re-exports `unify`, `rebuild` (from `usympy`) and `rewriterule` (from `rewrite`).
