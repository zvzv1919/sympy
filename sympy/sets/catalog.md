# sets — Module Catalog

Symbolic set theory: construction, membership, and operations on mathematical sets.

## Core Set Framework

### [`sets.py`](sets.py)
Foundation of all set types and operations.

- `Set` — abstract base class for all sets; defines `union`, `intersect`, `contains`, `complement`, `is_subset`, `is_superset`, `is_disjoint`
- `Interval` — continuous real interval with open/closed endpoint flags; `_eval_imageset` computes images via calculus
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
- `Range` — discrete integer range (start, stop, step); supports slicing, iteration, and Diophantine-based intersection
- `ComplexRegion` — region in the complex plane defined by product sets in rectangular (re × im) or polar (r × θ) form
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

- `Contains` — `BooleanFunction` representing `x ∈ S`; evaluates via `S.contains(x)` or stays unevaluated
