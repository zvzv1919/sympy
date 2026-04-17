# sets — Module Catalog

Symbolic set theory: construction, membership, and operations on mathematical sets.

## Core Set Framework

### [`sets.py`](sets.py)
Foundation of all set types and operations.

- `Set` — abstract base class for all sets; defines `union`, `intersect`, `contains`, `complement`, `is_subset`, `is_superset`, `is_disjoint`
  - `contains` — returns True/False for definite membership; falls back to an unevaluated `Contains` expression when membership is indeterminate
- `Interval` — continuous real interval with open/closed endpoint flags; `_eval_imageset` computes images via calculus; `_eval_Eq` returns false for non-compound sets, unevaluated for Union/Complement/Intersection/ProductSet
- `ProductSet` — Cartesian product of sets; flattens nested products
- `Union` — union of sets; `reduce()` simplifies by merging overlapping intervals and finite sets
- `Intersection` — intersection of sets; `reduce()` with `_handle_finite_sets` logic
- `Complement` — relative complement (set difference A − B)
- `SymmetricDifference` — elements in either set but not both
- `EmptySet` — singleton empty set
- `UniversalSet` — singleton universal set
- `FiniteSet` — finite collection of discrete symbolic elements; supports powerset
- **`imageset(*args)`** — standalone function; computes the image of a set under a transformation (Lambda, function, or lambda)
  - Composes nested transformations when the input is already an ImageSet; returns unevaluated `ImageSet` if it cannot simplify

## Specialized / "Fancy" Sets

### [`fancysets.py`](fancysets.py)
Named infinite sets, image sets, integer ranges, and complex-plane regions.

- `Naturals` / `Naturals0` — positive integers / non-negative integers (singletons)
- `Integers` — all integers; `_eval_imageset` canonicalizes linear expressions
- `Reals` — all reals as Interval(−∞, ∞) singleton
- `ImageSet` — the image of a base set under a Lambda; membership via `solveset`/`diophantine`; intersection uses Diophantine solver for integer bases
  - `_intersect` with `Interval`: inverts boundary values, restricts base set; converts finite `Range` to `FiniteSet` before remapping through the lambda
- `Range` — discrete integer range (start, stop, step); supports slicing, iteration, and Diophantine-based intersection
  - `_contains` — returns `S.false` for non-integers; returns `None` (propagates uncertainty) when the value's integer nature is unknown
- `ComplexRegion` — region in the complex plane defined by product sets in rectangular (re × im) or polar (r × θ) form
  - `psets` — returns constituent ProductSets as a tuple (wraps single product; unpacks Union args for multiple)
  - `_contains` checks membership by iterating over constituent product sets; supports intersection/union of same-form regions
- `Complexes` — singleton for all complex numbers (ℝ × ℝ)
- `normalize_theta_set` — normalizes angle sets to [0, 2π)

## Conditional Sets

### [`conditionset.py`](conditionset.py)
Sets defined by a boolean predicate over a base set.

- `ConditionSet` — `{x ∈ base_set | condition(x)}`; simplifies for trivially true/false conditions and filters FiniteSets eagerly

## Membership Predicate

### [`contains.py`](contains.py)
Boolean expression for symbolic set membership.

- `Contains` — symbolic `BooleanFunction` node representing `x ∈ S`; delegates to `Set.contains` for evaluation; holds unevaluated form only
