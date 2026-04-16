# sympy/sets — Set Theory

Core set-theoretic types, operations, and membership predicates used throughout SymPy.

## Glossary

- **Set** — abstract base for all set types; provides `union`, `intersect`, `complement`, `contains`, topological properties, and operator overloads (`+`, `|`, `&`, `-`, `*`, `^`).
- **ImageSet** — the image {f(x) | x ∈ S} of a set under a Lambda transformation.
- **ConditionSet** — the set {x ∈ S | condition(x)}.

---

## Core Set Types

### `sets.py`

Defines the `Set` base class and all fundamental concrete set types, plus the `imageset()` convenience constructor.

#### `Set` (base class)
- Public API: `union`, `intersect` / `intersection`, `complement`, `symmetric_difference`, `contains`, `is_subset` / `is_superset` / `is_proper_*`, `powerset`.
- Topological properties: `inf`, `sup`, `measure`, `boundary`, `is_open`, `is_closed`, `closure`, `interior`.
- Operator overloads: `+`/`|` → union, `&` → intersect, `-` → complement, `*` → product, `^` → symmetric difference, `**` → repeated product.
- Internal hooks for subclasses: `_intersect`, `_union`, `_complement`, `_symmetric_difference`, `_contains`, `_eval_imageset`, `_eval_powerset`.

#### `Interval`
- Real interval `[a, b]` with optional open endpoints; convenience constructors `open`, `Lopen`, `Ropen`.
- Properties: `start`/`end`, `left_open`/`right_open`, `is_left_unbounded`/`is_right_unbounded`.
- `_eval_imageset` — computes image of monotonic/piecewise functions over the interval using calculus (derivatives, singularities, limits).
- `as_relational(x)` — rewrites as inequality chain.
- `to_mpi` / `_eval_evalf` — mpmath interval conversion.

#### `FiniteSet`
- Discrete set of symbolic/numeric elements, backed by a `frozenset`.
- Supports iteration, `len`, `powerset`, relational comparison operators (`<=`, `<`, `>=`, `>`).
- `_complement` handles splitting `Reals` into open intervals around sorted numeric elements.

#### `ProductSet`
- Cartesian product of sets; flattens nested products.
- Membership test checks element-wise containment.
- Iterates via `itertools.product`.

#### `EmptySet` / `UniversalSet`
Singletons (`S.EmptySet`, `S.UniversalSet`) with trivial set-operation identities.

#### `Union`
- `reduce` — merges `FiniteSet`s, then iteratively applies pair-wise `_union` rules.
- Measure via inclusion-exclusion principle.
- `_complement` uses De Morgan's law.
- Iteration round-robins across constituent sets.

#### `Intersection`
- `reduce` — short-circuits on `EmptySet`, distributes over `Union`, handles `Complement` arguments, then applies pair-wise `_intersect` rules.
- `_handle_finite_sets` — special-cases intersections involving `FiniteSet`s with fuzzy membership logic.

#### `Complement`
Set difference A \ B. `reduce` delegates to `B._complement(A)`.

#### `SymmetricDifference`
A △ B = (A \ B) ∪ (B \ A).

#### `imageset(*args)`
- Convenience function accepting `(var, expr, set)`, `(Lambda, set)`, `(callable, set)`.
- Attempts eager evaluation via `set._eval_imageset(f)`; falls back to `ImageSet`.
- Composes nested `ImageSet` lambdas when possible.

---

## Special / Infinite Sets

### `fancysets.py`

Defines named infinite number sets, `ImageSet`, `Range`, and `ComplexRegion`.

#### `Naturals` / `Naturals0` / `Integers`
Singleton infinite sets (`S.Naturals`, `S.Naturals0`, `S.Integers`). All are iterable.
- `Integers._eval_imageset` — canonicalizes linear maps `a*n + b` by normalizing sign and shifting `b` modulo `a`.

#### `Reals`
Singleton (`S.Reals`), implemented as `Interval(-oo, oo)`.

#### `ImageSet`
- Image of a base set under a `Lambda`; inherits iterability from base set.
- `_contains` — solves `f(x) = other` (via `solveset` / `linsolve`) and checks if any solution lies in `base_set`. Handles both univariate and multivariate lambdas.
- `_intersect`:
  - Two integer `ImageSet`s — uses Diophantine solver.
  - With `Reals` — separates real/imaginary parts and solves.
  - With `Interval` — inverts the function and clips the domain.

#### `Range`
- Discrete range of integers, analogous to Python `range` but supporting symbolic/infinite bounds.
- `__new__` canonicalizes stop to ensure equivalent ranges have identical args.
- `reversed` — equivalent range in opposite order.
- `_intersect`:
  - With `Interval` — ceiling/floor to integer bounds.
  - With another `Range` — Diophantine linear solver to find coincident points, then LCM step.
- `_contains` — modular arithmetic check against step.
- `__getitem__` — supports integer indexing and slicing, including infinite ranges with negative indices.
- `_eval_imageset` — rewrites linear functions of the variable into a new `Range`.

#### `normalize_theta_set(theta)`
Normalizes an angular set into [0, 2π). Handles `Interval`, `FiniteSet`, and `Union` inputs; requires endpoints to be rational multiples of π.

#### `ComplexRegion`
- Represents a region of the complex plane in rectangular (`x + iy`) or polar (`r·e^{iθ}`) form, built on `ProductSet` of intervals.
- Properties: `sets`, `psets`, `a_interval`, `b_interval`, `polar`.
- `_contains` — decomposes into (re, im) or (r, θ) and checks against constituent `ProductSet`s.
- `_intersect` — intersects matching-form regions; intersects with real subsets by projecting.
- `_union` — unions matching-form regions.

#### `Complexes`
Singleton (`S.Complexes`), shorthand for `ComplexRegion(Reals × Reals)`.

---

## Membership & Conditions

### `contains.py`

#### `Contains`
- `BooleanFunction` subclass asserting `x ∈ S`.
- `eval` delegates to `S.contains(x)`; returns unevaluated `Contains` when membership is indeterminate.

### `conditionset.py`

#### `ConditionSet`
- `{x ∈ base_set | condition(x)}`.
- `__new__` eagerly simplifies: returns `EmptySet` if condition is false, `base_set` if true, and filters `FiniteSet` bases via `sift` + `fuzzy_bool`.
- `_intersect(other)` — wraps the base set in an `Intersection` with `other` (when `other` is not itself a `ConditionSet`).
- `contains(other)` — evaluates condition as a `Lambda` applied to `other`, AND-ed with `base_set.contains(other)`.

---

## Package Init

### `__init__.py`

Re-exports the public API: `Set`, `Interval`, `Union`, `EmptySet`, `FiniteSet`, `ProductSet`, `Intersection`, `imageset`, `Complement`, `SymmetricDifference`, `ImageSet`, `Range`, `ComplexRegion`, `Contains`, `ConditionSet`.
