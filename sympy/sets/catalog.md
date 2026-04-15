# sympy/sets — Catalog

> Part of [SymPy](../catalog.md). Set theory: intervals, unions, intersections, finite sets, and conditional sets.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package entry point that re-exports the public API: `Set`, `Interval`, `Union`, `Intersection`, `EmptySet`, `FiniteSet`, `ProductSet`, `Complement`, `SymmetricDifference`, `ImageSet`, `Range`, `ComplexRegion`, `Contains`, `ConditionSet`, and the `imageset` helper. |
| `sets.py` | Core set-theory classes including the `Set` base class, `Interval`, `Union`, `Intersection`, `Complement`, `SymmetricDifference`, `EmptySet`, `UniversalSet`, `FiniteSet`, and `ProductSet`, plus the `imageset` convenience function. Each set class implements `as_relational(symbol)` to rewrite itself as relational/logical expressions (e.g., `FiniteSet` becomes a disjunction of equalities, `Interval` becomes a conjunction of inequalities). Also includes set comparison operators and powerset evaluation. |
| `fancysets.py` | Extended ("fancy") set types: `Naturals`, `Naturals0`, `Integers`, `Reals`, `ImageSet`, `Range` (finite or infinite integer sequences), `ComplexRegion` (polar/rectangular regions of the complex plane), and `normalize_theta_set`. |
| `contains.py` | Defines the `Contains` Boolean function, which represents the symbolic assertion that an element belongs to a set (element-of predicate). Does NOT convert sets into relational/logical form — see `as_relational` in `sets.py` for that. |
| `conditionset.py` | Defines `ConditionSet`, a set comprehension class representing `{x | condition(x) is True for x in S}`, with support for intersection and membership testing. |
