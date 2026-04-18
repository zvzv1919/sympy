# geometry — Module Catalog

## Core / Base

### [`__init__.py`](__init__.py)
Package entry point; re-exports all public geometric entities and utility functions into the top-level `sympy.geometry` namespace.
- Exports: `Point`, `Point2D`, `Point3D`, `Line`, `Ray`, `Segment`, `Line3D`, `Ray3D`, `Segment3D`, `Plane`, `Ellipse`, `Circle`, `Polygon`, `RegularPolygon`, `Triangle`, `Curve`, `Parabola`.
- Utility exports: `are_similar`, `centroid`, `convex_hull`, `idiff`, `intersection`, `closest_points`, `farthest_points`.
- Also exports: `GeometryError`, `rad`, `deg`.

### [`entity.py`](entity.py)
Base classes for all geometric entities.
- `GeometryEntity` — abstract base; provides `intersection()`, `translate()`, `rotate()`, `scale()`, `reflect()`, `encloses_point()`, `equals()`.
  - `reflect(line)` — mirrors across a line; optimizes axis-aligned cases (x-axis → `scale(y=-1)`, y-axis → `scale(x=-1)`), uses point-by-point translation for offset horizontal/vertical lines, and compose translate-rotate-scale-rotate-translate for arbitrary slopes.
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
  - `Point.is_concyclic(*points)` — static method testing if points are concyclic; 0 points → False, ≤2 points → True, 3 points checks non-collinearity, 4+ constructs a Circle from first three and checks containment.
  - `is_scalar_multiple(p1, p2)` — checks linear dependence via matrix rank.
- `Point2D` — 2D specialization; adds `x`, `y` coordinate properties, `transform(Matrix)`, and overrides `rotate(angle, pt)`, `scale(x, y, pt)`, `translate(x, y)`.
  - `scale(x, y, pt)` — scales coordinates by `x`, `y` relative to reference point `pt`; uses `if pt:` (falsy check) instead of `if pt is not None:`, so passing the origin as `pt` is silently ignored.
  - `rotate(angle, pt)` — rotates counterclockwise about `pt`; uses `if pt is not None:` to guard the reference-point shift.
- `Point3D` — 3D specialization; adds `x`, `y`, `z` coordinate properties, `direction_ratio()`, `direction_cosine()`, `scale(x, y, z, pt)`, `translate(x, y, z)`, `transform(matrix)`.
  - `scale(x, y, z, pt)` — multiplies each coordinate by the respective factor; when a reference point `pt` is given, translates to origin first, scales, then translates back.
  - `direction_cosine(point)` — divides displacement components by magnitude; no guard against zero magnitude (identical points → division by zero).
  - `are_coplanar(*points)` — static; tests coplanarity of `Point3D` only (not mixed entity types); deduplicates inputs, raises `ValueError` if <3 distinct points or all are collinear. For mixed-entity coplanarity, use `util.are_coplanar`.
  - `are_collinear(*points)` — static; delegates to `Point.is_collinear`.

## Lines & Segments

### [`line.py`](line.py)
2D linear entities: lines, rays, and segments.
- `LinearEntity` — abstract base for 2D linear entities; constructed from two distinct points (raises error on coincident points).
  - `are_concurrent(*lines)` — static; tests concurrency.
  - `is_parallel(l1, l2)`, `is_perpendicular(l1, l2)` — compare via `coefficients`; return `False` (no error) if either entity lacks `coefficients`.
  - `angle_between(l1, l2)`.
  - `parallel_line(p)`, `perpendicular_line(p)`, `perpendicular_segment(p)`.
  - `projection(o)` — projects a `Point` or `LinearEntity` onto this line; raises `GeometryError` for any other geometry type (e.g., `Circle`).
  - `intersection(o)` — full intersection logic for line/ray/segment pairs; uses Cramer's rule for crossing point, then validates via coordinate-betweenness (segments) and direction-consistency (rays) instead of fragile containment tests.
  - `arbitrary_point(parameter='t')` — raises `ValueError` if parameter name collides with a free symbol already in the line's definition.
  - `random_point()` — generates a random point on the entity; switches from x-based to y-based randomization when slope is infinite (vertical line); adjusts bounds for `Ray` (half-open) and `Segment` (closed).
  - `contains()`.
- `Line` — infinite 2D line through two points.
- `Ray` — 2D ray (half-line) from a source point in a direction.
  - `distance(o)` — shortest distance to a point; falls back to distance from the ray's source when the perpendicular foot lies outside the ray.
  - `contains(o)` — membership test using direction-consistency; raises `Undecidable` for unresolvable symbolic coordinates.
- `Segment` — 2D segment between two endpoints; `length`, `midpoint`, `perpendicular_bisector()`.

### [`line3d.py`](line3d.py)
3D linear entities: lines, rays, and segments.
- `LinearEntity3D` — abstract base for 3D linear entities.
  - `are_concurrent(*lines)`, `is_parallel(l1, l2)`, `is_perpendicular(l1, l2)`.
  - `parallel_line(p)`, `perpendicular_line(p)`, `perpendicular_segment(p)`.
  - `projection(o)` — projects a `Point3D` or `LinearEntity3D` onto this line (not onto a plane); for linear entities, if both endpoints project to the same point, returns that single point instead of preserving the entity type.
  - `is_similar(other)` — checks if `self` and `other` are on the same line; only handles `Line3D` as `other`, raises `NotImplementedError` for `Ray3D`/`Segment3D`.
  - `direction_ratio`, `direction_cosine`, `angle_between(l1, l2)`.
- `Line3D`, `Ray3D`, `Segment3D` — 3D counterparts of the 2D entities.

## Curves & Conics

### [`curve.py`](curve.py)
Explicit parametric curves in the 2D plane (not 3D surfaces).
- `Curve` — defined by `(f(t), g(t))` over a parameter interval; supports `arbitrary_point()`, `plot_interval()`, `rotate()`, `scale()`, `translate()`.

### [`ellipse.py`](ellipse.py)
Elliptical entities in 2D.
- `Ellipse` — defined by center, horizontal radius, vertical radius (or eccentricity). Properties: `foci`, `eccentricity`, `area`, `circumference`, `apoapsis`, `periapsis`. Methods: `tangent_lines()`, `normal_lines()`, `equation()`, `intersection()`.
  - `intersection(o)` — type-dispatched: handles `Point`, `LinearEntity`, `Circle`, `Ellipse`; for unrecognized types, falls back to `o.intersection(self)` (reverse dispatch).
  - `is_tangent(o)` — type-dispatched tangency test: for `Ellipse` checks single intersection point (coincident ellipses → False); for `LinearEntity` checks single intersection in segment; for `Polygon` iterates over all sides counting edge–ellipse intersection points and returns `True` iff total count is 1.
  - `reflect(line)` — overrides `GeometryEntity.reflect`; handles axis-aligned lines only; raises `NotImplementedError` (with reflected equation) for diagonal lines.
  - `rotate(angle, pt)` — overrides `GeometryEntity.rotate`; only supports multiples of π/2; raises `NotImplementedError` otherwise.
  - `__contains__(o)` — Python `in` operator: for `Point` checks if point satisfies ellipse equation; for another `Ellipse` checks equality only (not geometric containment).
  - `_do_line_intersection(o)` — line–ellipse intersection via quadratic discriminant; handles symbolic discriminants by allowing indeterminate-sign cases.
  - `_do_ellipse_intersection(o)` — ellipse–ellipse / ellipse–circle intersection via solving simultaneous conic equations.
- `Circle` — `Ellipse` subclass; constructed from center+radius, three points, or center+point. Adds `radius`, `circumference`, `equation()`.
  - Three-point construction: validates collinearity (raises `GeometryError` if collinear), then computes center/radius via `Triangle.circumcenter`/`circumradius`.
  - `scale(x, y, pt)` — overrides `GeometryEntity.scale`; uniform scaling (x == y) preserves `Circle` type; non-uniform scaling returns an `Ellipse` instead.
  - `reflect(line)` — overrides `GeometryEntity.reflect` since radius is not a `GeometryEntity`.

### [`parabola.py`](parabola.py)
Parabolic entities defined by focus and directrix.
- `Parabola` — supports vertical/horizontal parabolas only; `__new__` raises `NotImplementedError` if directrix is diagonal (neither horizontal nor vertical).
  - Properties: `focus`, `directrix`, `vertex`, `eccentricity`. Methods: `equation()`, `intersection()`.
  - `vertex` — extremal (turning) point of the parabola; computed by subtracting `p_parameter` from the appropriate focal coordinate based on axis orientation.
  - `eccentricity` — always returns 1 (unit eccentricity by definition for all parabolas).
  - `p_parameter` — signed semi-latus rectum; sign convention depends on orientation (horizontal vs vertical) with inverted comparison logic between the two cases.
  - `focal_length` — unsigned distance from vertex to focus (half the vertex-to-directrix distance).

## Planes & Surfaces

### [`plane.py`](plane.py)
3D planar surfaces.
- `Plane` — defined by point + normal or three points. Methods: `equation()`, `normal_vector`, `parallel_plane()`, `perpendicular_plane()`, `distance()`, `angle_between()`, `projection()`, `projection_line()`, `intersection()`.
  - `projection_line(line)` — projects a 2D or 3D linear entity onto the plane; returns a `Point3D` (not a line) when the line is parallel to the plane's normal (both endpoints map to the same location).
  - `are_concurrent(*planes)` — static; tests whether multiple planes all share a single common line of intersection; deduplicates inputs, returns False for <2 planes.
  - `is_coplanar(o)` — instance method; tests whether a single entity (`Plane`, `Point3D`, `LinearEntity3D`, or 2D `GeometryEntity`) is coplanar with this plane. Distinct from `util.are_coplanar` which is a standalone multi-entity test.
  - `arbitrary_point(t)` — returns a parametric `Point3D` that traces a unit circle on the plane around `p1` as `t` varies from 0 to 2π; handles axis-aligned normals directly, general normals via projection and symbolic solve.
  - `random_point(seed)` — evaluates `arbitrary_point` at a random parameter value.

## Polygons

### [`polygon.py`](polygon.py)
Polygonal entities in 2D.
- `Polygon` — defined by ordered vertices. Properties: `area`, `perimeter`, `centroid`, `sides`, `vertices`, `angles`, `bounds`. Methods: `is_convex()`, `encloses_point()`, `arbitrary_point()`, `distance(o)`.
  - `arbitrary_point(parameter='t')` — parameterized perimeter point (0→1); raises `ValueError` if parameter name collides with a free symbol in the polygon's vertex coordinates.
  - `__contains__(o)` — Python `in` operator: for `Polygon` checks equality only (not geometric containment); for `Segment` checks if it matches a side; for `Point` checks boundary membership.
  - `intersection(o)` — iterates over each side, collects per-edge intersections with the other entity, and deduplicates results via `uniq`.
  - `_do_poly_distance(e2)` — minimum boundary separation between two convex polygons via rotating calipers. Pre-checks bounding circles around centroids; if they overlap, emits a warning (does not abort or raise) and continues computation, potentially returning erroneous results for intersecting polygons.
- `RegularPolygon` — `Polygon` subclass for regular n-gons; stored as center + radius + n (not explicit vertices). Adds `radius`, `interior_angle`, `exterior_angle`, `incircle`, `circumcircle`, `spin()`, `rotate()`.
  - `reflect(line)` — overrides `GeometryEntity.reflect`; reflects center and first vertex, computes angular spin at the new center, and negates the radius to encode the mirror-flip in orientation.
  - `scale(x, y, pt)` — overrides base; uniform scaling (x == y) preserves `RegularPolygon` type by scaling the radius; non-uniform scaling degrades to a plain `Polygon` with explicit vertices.
  - `encloses_point(p)` — optimized containment: rejects if distance ≥ circumradius, accepts if distance < inradius, falls back to general `Polygon.encloses_point` only for the annular region between.
  - `__eq__(o)` — cross-type equality: if compared to a plain `Polygon`, delegates to `Polygon.__eq__` to resolve center/radius vs explicit-vertices mismatch.
- `Triangle` — `Polygon` subclass; rich set of triangle-specific properties: `altitudes`, `orthocenter`, `circumcenter`, `circumcircle`, `incircle`, `medians`, `medial`, `nine_point_circle`, `bisectors`. Helper constructors: `_sss()`, `_sas()`, `_asa()`.
  - `medial` — returns the medial triangle (the triangle formed by connecting the midpoints of the three sides).
  - `medians` — returns dict mapping each vertex to the median segment (line from vertex to midpoint of opposite side).
  - `is_similar(t2)` — triangle similarity test; returns `False` immediately if `t2` is not a `Polygon`; otherwise checks all 6 side-length-ratio permutations for uniform scaling match.
  - `is_equilateral()`, `is_isosceles()`, `is_right()`, `is_scalene()` — triangle classification predicates.

## Utilities

### [`util.py`](util.py)
Standalone geometric utility functions.
- `intersection(*entities)` — convenience dispatcher; delegates to each entity's own `.intersection()` method. Contains no intersection math itself.
- `convex_hull(*points)` — computes 2D convex hull via Andrew's monotone chain algorithm; returns `Polygon` for non-degenerate hulls, `Segment` when all points are collinear, or `Point` for single-point input.
- `closest_points(*points)` — sweep-line nearest-pair search for 2D points; computes distances internally (not via `Point.distance`).
  - Adapts distance calculation per coordinate type: uses `math.sqrt` for rational coordinates, switches to SymPy `sqrt` for symbolic/irrational values.
- `farthest_points(*points)` — farthest pair(s) among 2D points via convex-hull rotating calipers.
  - Adapts distance calculation per coordinate type: uses `math.sqrt` for rational coordinates, switches to SymPy `sqrt` for symbolic/irrational values.
- `are_coplanar(*entities)` — standalone coplanarity test for 3D points/lines; returns `False` when all points are collinear (no unique plane). Converts 2D geometry objects to 3D (z=0) before checking.
- `are_similar(e1, e2)` — convenience dispatcher for geometric similarity: tries `e1.is_similar(e2)`, falls back to `e2.is_similar(e1)`, raises `GeometryError` if neither supports the check. Contains no similarity logic itself; all algorithms live in each entity's `is_similar` method (e.g., `Triangle.is_similar`).
- `centroid(*args)` — weighted center of mass for a homogeneous collection of Points (equal weight), Segments (weighted by length), or Polygons (weighted by area). Returns None for mixed types.
- `idiff(eq, y, x, n=1)` — implicit differentiation: computes dy/dx (up to order `n`) assuming `eq == 0`.
  - `y` must be a `Symbol` or list of `Symbol`s (first element is primary dependent variable); raises `ValueError` if `y` is neither.
