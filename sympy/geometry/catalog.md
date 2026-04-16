# sympy/geometry — Catalog

> Part of [SymPy](../catalog.md). Computational geometry: points, lines, polygons, circles, ellipses, and curves.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package initializer that imports and re-exports all public geometry classes (Point, Line, Ellipse, Circle, Polygon, Plane, Curve, Parabola) and utility functions. |
| `entity.py` | Defines `GeometryEntity`, the base class for all geometric objects, and `GeometrySet`, a base for entities that can also act as SymPy Sets. Provides common methods such as rotation, scaling, translation, reflection, intersection, and symbolic substitution (`_eval_subs` handles sequence-to-Point conversion, dispatching to Point3D or Point2D based on the entity type). |
| `point.py` | Implements `Point`, `Point2D`, and `Point3D` classes representing points in n-dimensional Euclidean space, with operations like distance, midpoint, collinearity checks (`is_collinear`, `are_collinear`), coplanarity checks (`Point3D.are_coplanar` — raises ValueError when all points are collinear), and coordinate transformations. |
| `line.py` | Implements 2D line-like entities: `LinearEntity`, `Line`, `Ray`, `Segment` (all 2D only). Supports intersection, perpendicularity, parallelism, angle computation, projections, and containment checks for 2D rays/segments. |
| `line3d.py` | Implements 3D line-like entities: `LinearEntity3D`, `Line3D`, `Ray3D`, `Segment3D`. Provides 3D-specific direction properties, distance calculations, perpendicular segments (`perpendicular_segment` for computing shortest segment from a point to a 3D line), perpendicular lines, projections onto 3D lines, containment checks, and parallel/perpendicular tests in 3D space. |
| `plane.py` | Implements the `Plane` class for 3D planes, constructed from three points or a point and normal vector. Supports intersection with lines and other planes, distance calculations, and projections. |
| `polygon.py` | Implements `Polygon`, `RegularPolygon`, and `Triangle` classes. Provides area, perimeter, centroid, angle computations, point enclosure testing (`encloses_point` — returns False for boundary points, None for symbolic coordinates), polygon-to-polygon minimum distance via rotating calipers (`_do_poly_distance`), and specialized triangle properties like incircle, circumcircle, and medians. |
| `ellipse.py` | Implements `Ellipse` and `Circle` classes with support for tangent lines, focal properties, eccentricity, and arbitrary point generation. `Circle.__new__` supports construction from three points (raises GeometryError if collinear) or center+radius. `Ellipse.reflect` overrides base reflection: handles axis-aligned lines directly but raises NotImplementedError with the reflected equation for non-axis-aligned lines. Each class has its own `intersection` method with type-based dispatch (handling Point, Line, Segment, Ray, Circle, Ellipse, and identity checks), delegating to internal helpers like `_do_line_intersection` and `_do_ellipse_intersection`. |
| `curve.py` | Implements the `Curve` class for parametrically-defined 2D curves, supporting arc length computation, arbitrary point generation, and plotting-related methods. |
| `parabola.py` | Implements the `Parabola` class defined by a focus and directrix. Supports vertex, axis of symmetry, focal length, and equation generation for vertical and horizontal parabolas. |
| `util.py` | Module-level utility functions (not class methods): `intersection` (convenience wrapper that calls the `intersection` method on the first entity), `convex_hull`, `closest_points`, `farthest_points`, `are_coplanar`, `are_similar`, `centroid`, and `idiff` (implicit differentiation). |
| `exceptions.py` | Defines `GeometryError`, the exception class raised by geometry module classes. |
