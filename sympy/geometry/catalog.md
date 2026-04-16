# sympy/geometry — Catalog

> Part of [SymPy](../catalog.md). Computational geometry: points, lines, polygons, circles, ellipses, and curves.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package initializer that imports and re-exports all public geometry classes (Point, Line, Ellipse, Circle, Polygon, Plane, Curve, Parabola) and utility functions. |
| `entity.py` | Defines `GeometryEntity`, the base class for all geometric objects, and `GeometrySet`, a base for entities that can also act as SymPy Sets. Provides common methods such as rotation, scaling, translation, and intersection. |
| `point.py` | Implements `Point`, `Point2D`, and `Point3D` classes representing points in n-dimensional Euclidean space, with operations like distance, midpoint, collinearity checks, and coordinate transformations. |
| `line.py` | Implements 2D line-like entities: `LinearEntity`, `Line`, `Ray`, `Segment` (all 2D only). Supports intersection, perpendicularity, parallelism, angle computation, projections, and containment checks for 2D rays/segments. |
| `line3d.py` | Implements 3D line-like entities: `LinearEntity3D`, `Line3D`, `Ray3D`, `Segment3D`. Provides 3D-specific direction properties (xdirection, ydirection, zdirection), distance calculations, containment checks (`Ray3D.contains`, `Segment3D.contains`), and parallel/perpendicular tests in 3D space. |
| `plane.py` | Implements the `Plane` class for 3D planes, constructed from three points or a point and normal vector. Supports intersection with lines and other planes, distance calculations, and projections. |
| `polygon.py` | Implements `Polygon`, `RegularPolygon`, and `Triangle` classes. Provides area, perimeter, centroid, angle computations, and specialized triangle properties like incircle, circumcircle, and medians. |
| `ellipse.py` | Implements `Ellipse` and `Circle` classes with support for tangent lines, focal properties, eccentricity, and arbitrary point generation. Each class has its own `intersection` method with type-based dispatch (handling Point, Line, Segment, Ray, Circle, Ellipse, and identity checks), delegating to internal helpers like `_do_line_intersection` and `_do_ellipse_intersection`. |
| `curve.py` | Implements the `Curve` class for parametrically-defined 2D curves, supporting arc length computation, arbitrary point generation, and plotting-related methods. |
| `parabola.py` | Implements the `Parabola` class defined by a focus and directrix. Supports vertex, axis of symmetry, focal length, and equation generation for vertical and horizontal parabolas. |
| `util.py` | Module-level utility functions (not class methods): `intersection` (convenience wrapper that calls the `intersection` method on the first entity), `convex_hull`, `closest_points`, `farthest_points`, `are_coplanar`, `are_similar`, `centroid`, and `idiff` (implicit differentiation). |
| `exceptions.py` | Defines `GeometryError`, the exception class raised by geometry module classes. |
