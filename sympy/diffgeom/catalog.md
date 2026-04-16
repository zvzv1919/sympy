# sympy/diffgeom — Catalog

> Part of [SymPy](../catalog.md). Differential geometry: manifolds, coordinate systems, differential forms.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package init that re-exports all public classes and functions from `diffgeom.py` (Manifold, Patch, CoordSystem, differential operators, metric utilities, etc.). |
| `diffgeom.py` | Core implementation of the differential geometry module, defining Manifold, Patch, CoordSystem, Point, scalar/vector/oneform fields, tensor products, wedge products, Lie derivatives, covariant derivatives, integral curve computation (series expansion and differential equations, with coordinate-system defaulting logic), and metric-to-curvature utilities (Christoffel symbols, Riemann/Ricci components). |
| `rn.py` | Predefined R^2 and R^3 manifold instances with common coordinate systems (rectangular, polar, cylindrical, spherical) and their transformation laws, basis vectors, and basis oneforms. Does not contain computational logic for curves or field operations. |
