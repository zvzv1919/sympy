# geometry — Module Catalog

## Core / Base

### [`entity.py`](entity.py)
Base classes for all geometric entities.
- `GeometryEntity` — abstract base; provides `intersection()`, `translate()`, `rotate()`, `scale()`, `reflect()`, `encloses_point()`, `equals()`.
  - `encloses(o)` — type-dispatching containment check; delegates to each subclass's `encloses_point` method.
  - `_eval_subs(old, new)` — substitution hook; converts sequence arguments to `Point3D` if entity is 3D, else `Point`.
- `GeometrySet` — extends `GeometryEntity` with set-theoretic operations (`union`, `intersection`, `difference`, `contains`).

### [`exceptions.py`](exceptions.py)
`GeometryError` — `ValueError` subclass for geometry-specific errors.

## Points

### [`point.py`](point.py)
Point representations in n-dimensional Euclidean space.
- `Point` — n-dimensional point; supports arithmetic (`+`, `-`, `*`, `/`), `distance()`, `taxicab_distance()`, `midpoint()`, `dot()`.
  - `intersection(o)` — returns `[self]` if `o` is an equal Point, `[]` if different Point; delegates to `o.intersection(self)` when `o` is not a Point.
  - `equals(other)` — component-wise symbolic equality via `.equals()`; distinct from `__eq__` which does structural tuple comparison.
  - `Point.is_collinear(*points)` — static method testing if points are collinear; deduplicates inputs, returns `True` for ≤2 unique points.
  - `Point.is_concyclic(*points)` — static method testing if points are concyclic.
  - `is_scalar_multiple(p1, p2)` — checks linear dependence via matrix rank.
- `Point2D` — 2D specialization; adds `x`, `y` coordinate properties and `transform(Matrix)`.
- `Point3D` — 3D specialization; adds `x`, `y`, `z` coordinate properties, `direction_ratio()`, `direction_cosine()`.
  - `are_coplanar(*points)` — static; tests coplanarity by trying to construct a `Plane` from triples; raises `ValueError` if all points are collinear.
  - `are_collinear(*points)` — static; delegates to `Point.is_collinear`.

## Lines & Segments

### [`line.py`](line.py)
2D linear entities: lines, rays, and segments.
- `LinearEntity` — abstract base for 2D linear entities; constructed from two distinct points (raises error on coincident points).
  - `are_concurrent(*lines)` — static; tests concurrency.
  - `is_parallel(l1, l2)`, `is_perpendicular(l1, l2)`, `angle_between(l1, l2)`.
  - `parallel_line(p)`, `perpendicular_line(p)`, `perpendicular_segment(p)`, `projection(o)`.
  - `intersection(o)` — full intersection logic for line/ray/segment pairs; uses Cramer's rule for crossing point, then validates via coordinate-betweenness (segments) and direction-consistency (rays) instead of fragile containment tests.
  - `arbitrary_point(parameter='t')` — raises `ValueError` if parameter name collides with a free symbol already in the line's definition.
  - `random_point()`, `contains()`.
- `Line` — infinite 2D line through two points.
- `Ray` — 2D ray (half-line) from a source point in a direction.
  - `distance(o)` — shortest distance to a point; falls back to distance from the ray's source when the perpendicular foot lies outside the ray.
  - `contains(o)` — membership test using direction-consistency; raises `Undecidable` for unresolvable symbolic coordinates.
- `Segment` — 2D segment between two endpoints; `length`, `midpoint`, `perpendicular_bisector()`.

### [`line3d.py`](line3d.py)
3D linear entities: lines, rays, and segments.
- `LinearEntity3D` — abstract base for 3D linear entities.
  - `are_concurrent(*lines)`, `is_parallel(l1, l2)`, `is_perpendicular(l1, l2)`.
  - `parallel_line(p)`, `perpendicular_line(p)`, `perpendicular_segment(p)`, `projection(o)`.
  - `direction_ratio`, `direction_cosine`, `angle_between(l1, l2)`.
- `Line3D`, `Ray3D`, `Segment3D` — 3D counterparts of the 2D entities.

## Curves & Conics

### [`curve.py`](curve.py)
Parametric curves in 2D space.
- `Curve` — defined by `(f(t), g(t))` over a parameter interval; supports `arbitrary_point()`, `plot_interval()`, `rotate()`, `scale()`, `translate()`.

### [`ellipse.py`](ellipse.py)
Elliptical entities in 2D.
- `Ellipse` — defined by center, horizontal radius, vertical radius (or eccentricity). Properties: `foci`, `eccentricity`, `area`, `circumference`, `apoapsis`, `periapsis`. Methods: `tangent_lines()`, `normal_lines()`, `is_tangent()`, `equation()`.
  - `reflect(line)` — overrides `GeometryEntity.reflect`; handles axis-aligned lines only; raises `NotImplementedError` (with reflected equation) for diagonal lines.
  - `rotate(angle, pt)` — overrides `GeometryEntity.rotate`; only supports multiples of π/2; raises `NotImplementedError` otherwise.
  - `__contains__(o)` — Python `in` operator: for `Point` checks if point satisfies ellipse equation; for another `Ellipse` checks equality only (not geometric containment).
  - `_do_line_intersection(o)` — line–ellipse intersection via quadratic discriminant; handles symbolic discriminants by allowing indeterminate-sign cases.
  - `_do_ellipse_intersection(o)` — ellipse–ellipse / ellipse–circle intersection via solving simultaneous conic equations.
- `Circle` — `Ellipse` subclass; constructed from center+radius, three points, or center+point. Adds `radius`, `circumference`, `equation()`.
  - Three-point construction: validates collinearity (raises `GeometryError` if collinear), then computes center/radius via `Triangle.circumcenter`/`circumradius`.

### [`parabola.py`](parabola.py)
Parabolic entities defined by focus and directrix.
- `Parabola` — supports vertical/horizontal parabolas. Properties: `focus`, `directrix`, `vertex`, `p_parameter`, `eccentricity`. Methods: `equation()`, `intersection()`.

## Planes & Surfaces

### [`plane.py`](plane.py)
3D planar surfaces.
- `Plane` — defined by point + normal or three points. Methods: `equation()`, `normal_vector`, `is_coplanar()`, `parallel_plane()`, `perpendicular_plane()`, `distance()`, `angle_between()`, `projection()`, `intersection()`, `arbitrary_point()`.

## Polygons

### [`polygon.py`](polygon.py)
Polygonal entities in 2D.
- `Polygon` — defined by ordered vertices. Properties: `area`, `perimeter`, `centroid`, `sides`, `vertices`, `angles`, `bounds`. Methods: `is_convex()`, `encloses_point()`, `arbitrary_point()`, `distance(o)`.
  - `__contains__(o)` — Python `in` operator: for `Polygon` checks equality only (not geometric containment); for `Segment` checks if it matches a side; for `Point` checks boundary membership.
  - `intersection(o)` — iterates over each side, collects per-edge intersections with the other entity, and deduplicates results via `uniq`.
  - `_do_poly_distance(e2)` — minimum boundary separation between two convex polygons via angular-sweep over edge pairs (rotating calipers).
- `RegularPolygon` — `Polygon` subclass for regular n-gons; stored as center + radius + n (not explicit vertices). Adds `radius`, `interior_angle`, `exterior_angle`, `incircle`, `circumcircle`, `spin()`, `rotate()`.
  - `encloses_point(p)` — optimized containment: rejects if distance ≥ circumradius, accepts if distance < inradius, falls back to general `Polygon.encloses_point` only for the annular region between.
  - `__eq__(o)` — cross-type equality: if compared to a plain `Polygon`, delegates to `Polygon.__eq__` to resolve center/radius vs explicit-vertices mismatch.
- `Triangle` — `Polygon` subclass; rich set of triangle-specific properties: `altitudes`, `orthocenter`, `circumcenter`, `circumcircle`, `incircle`, `medians`, `medial`, `nine_point_circle`, `bisectors`. Helper constructors: `_sss()`, `_sas()`, `_asa()`.

## Utilities

### [`util.py`](util.py)
Standalone geometric utility functions.
- `intersection(*entities)` — convenience dispatcher; delegates to each entity's own `.intersection()` method. Contains no intersection math itself.
- `convex_hull(*points)` — returns convex hull as a `Polygon`, `Segment`, or `Point`.
- `closest_points(*points)` — sweep-line nearest-pair search for 2D points; computes distances internally (not via `Point.distance`).
  - Adapts distance calculation per coordinate type: uses `math.sqrt` for rational coordinates, switches to SymPy `sqrt` for symbolic/irrational values.
- `farthest_points(*points)` — farthest pair(s) among 2D points via convex-hull rotating calipers.
- `are_coplanar(*entities)` — tests coplanarity of points/lines in 3D.
- `are_similar(e1, e2)` — tests geometric similarity via double dispatch: tries `e1.is_similar(e2)`, falls back to `e2.is_similar(e1)`, raises `GeometryError` if neither supports the check.
- `centroid(*args)` — weighted centroid of geometric entities.
- `idiff(eq, y, x, n=1)` — implicit differentiation for curves defined by equations.
