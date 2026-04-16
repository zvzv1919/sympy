# sympy/sets — Catalog

> Part of [SymPy](../catalog.md). Set theory: intervals, unions, intersections, finite sets, and conditional sets.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package entry point that re-exports the public API: `Set`, `Interval`, `Union`, `Intersection`, `EmptySet`, `FiniteSet`, `ProductSet`, `Complement`, `SymmetricDifference`, `ImageSet`, `Range`, `ComplexRegion`, `Contains`, `ConditionSet`, and the `imageset` helper. |
| `sets.py` | Core set-theory classes including the `Set` base class, `Interval`, `Union`, `Intersection`, `Complement`, `SymmetricDifference`, `EmptySet`, `UniversalSet`, `FiniteSet`, and `ProductSet`. The `Set` base class defines the `contains` method which handles element-membership queries: it delegates to `_contains` and, when membership is indeterminate (returns `None`), returns an unevaluated symbolic `Contains` expression instead of raising an error. Also contains the `imageset` helper which composes nested `ImageSet` mappings into a single transformation over the base set. |
| `fancysets.py` | Extended ("fancy") set types: `Naturals`, `Naturals0`, `Integers`, `Reals`, `ImageSet` (the unevaluated symbolic representation of {f(x) | x ∈ S}), `Range` (finite or infinite integer sequences), `ComplexRegion` (polar/rectangular regions of the complex plane), and `normalize_theta_set`. `Integers` includes `_eval_imageset` which simplifies image-set expressions over ℤ — e.g., choosing between f(n) and f(−n) by preferring the form with fewer negative coefficients, and canonicalizing linear expressions a·n+b. |
| `contains.py` | Defines the `Contains` Boolean function — a symbolic element-of predicate. Its `eval` classmethod delegates to the set's `contains` method and guards against infinite recursion by returning `None` if the result is already a `Contains` instance. |
| `conditionset.py` | Defines `ConditionSet`, a set comprehension class representing `{x | condition(x) is True for x in S}`, with support for intersection and membership testing. |
