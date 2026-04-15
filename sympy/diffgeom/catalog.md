# sympy/diffgeom — Catalog

> Part of [SymPy](../catalog.md). Differential geometry: manifolds, coordinate systems, differential forms.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package init that re-exports all public classes and functions from `diffgeom.py` (Manifold, Patch, CoordSystem, differential operators, metric utilities, etc.). |
| `diffgeom.py` | Core implementation of the differential geometry module, defining Manifold, Patch, CoordSystem, Point, scalar/vector/oneform fields, tensor products, wedge products, Lie derivatives, covariant derivatives, and metric-to-curvature utilities (Christoffel symbols, Riemann/Ricci components). |
| `rn.py` | Predefined R^2 and R^3 manifolds with common coordinate systems (rectangular, polar, cylindrical, spherical). All coordinate chart transitions are manually specified in both directions with automatic inverse deduction disabled (`inverse=False, fill_in_gaps=False`); for R^3 this means all six pairwise directional transformations among rectangular, cylindrical, and spherical are hand-coded. Also defines basis coordinate functions, basis vectors, and basis oneforms as convenient attributes. See `diffgeom.py` for the underlying `CoordSystem.connect_to` API. |
