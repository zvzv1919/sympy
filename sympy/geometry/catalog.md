# sympy/geometry — Computational Geometry

2D and 3D Euclidean geometry: points, lines, rays, segments, polygons, ellipses, circles, parabolas, curves, and planes. All entities are symbolic and integrate with SymPy's CAS.

## Glossary

- **GeometryEntity** — base class for all geometric objects; provides `rotate`, `scale`, `translate`, `reflect`, `encloses`.
- **GeometrySet** — `GeometryEntity` that doubles as a `sympy.sets.Set` (supports `_contains`, `_union`, `_intersect`). All entities except `Point` inherit from this.
- **LinearEntity / LinearEntity3D** — abstract base for `Line`, `Ray`, `Segment` (and their 3D variants).

---

## Infrastructure

### `__init__.py`
Public API surface. Re-exports every user-facing class and utility function.

### `entity.py`
Base classes and affine-transform helpers for all geometry objects.

- **`GeometryEntity(Basic)`** — root of the hierarchy.
  - `intersection(o)` — stub; subclasses with higher `ordering_of_classes` index must implement.
  - `rotate(angle, pt=None)`, `scale(x, y, pt=None)`, `translate(x, y)` — generic affine transforms that recurse into args.
  - `reflect(line)` — reflection about an arbitrary 2D line; builds a composite translate→rotate→scale→rotate→translate transform.
  - `encloses(o)` — dispatches to `encloses_point` for Points; checks vertices/center for Segments, Ellipses, Polygons.
  - `is_similar(other)`, `equals(o)`, `bounds`, `ambient_dimension` — interface stubs.
  - `_svg(...)` / `_repr_svg_()` — IPython/Jupyter SVG rendering with auto-computed viewBox.
- **`GeometrySet(GeometryEntity, Set)`** — bridges `sympy.sets`; implements `_contains`, `_union`, `_intersect`.
- `ordering_of_classes` — list defining `__cmp__` order and intersection dispatch priority.
- **Module-level helpers:**
  - `translate(x, y)` → 3×3 matrix
  - `scale(x, y, pt=None)` → 3×3 matrix
  - `rotate(th)` → 3×3 rotation matrix

### `exceptions.py`
Single class: `GeometryError(ValueError)`.

---

## Points

### `point.py`
N-dimensional point with automatic dispatch to `Point2D` / `Point3D` for 2- and 3-coordinate cases.

- **`Point(GeometryEntity)`** — n-dimensional.
  - `is_collinear(*points)` — uses rank of displacement matrix.
  - `is_scalar_multiple(p1, p2)` — checks linear dependence via `Matrix.rank`.
  - `distance(p)`, `taxicab_distance(p)`, `midpoint(p)`, `dot(p2)`
  - Arithmetic: `__add__`, `__sub__`, `__mul__`, `__div__`, `__neg__`, `__abs__`
  - `evalf(prec)` / `n` — numeric evaluation.
- **`Point2D(Point)`** — 2D specialization.
  - Properties: `x`, `y`, `bounds`
  - `is_concyclic(*points)` — tests concyclicity via `Circle` construction.
  - `rotate`, `scale`, `translate` — optimized 2D overrides.
  - `transform(matrix)` — apply a 3×3 affine matrix.
- **`Point3D(Point)`** — 3D specialization.
  - Properties: `x`, `y`, `z`
  - `direction_ratio(point)`, `direction_cosine(point)`
  - `are_collinear(*points)`, `are_coplanar(*points)` — static; coplanarity uses `Plane`.
  - `scale(x, y, z, pt=None)`, `translate(x, y, z)`, `transform(matrix)` — 4×4 affine matrix.

#### Caveats
- `Point.__new__` auto-converts `Float` coords to `Rational` unless `evaluate=False`.
- Imaginary coordinates raise `ValueError`.

---

## Lines (2D)

### `line.py`
2D line, ray, and segment built on two `Point`s.

- **`LinearEntity(GeometrySet)`** — abstract base.
  - Properties: `p1`, `p2`, `coefficients` (a, b, c for ax+by+c=0), `slope`, `points`, `length`, `bounds`
  - `are_concurrent(*lines)` — checks via `sympy.sets.Intersection`.
  - `is_parallel(l2)`, `is_perpendicular(l2)`, `angle_between(l2)`
  - `parallel_line(p)`, `perpendicular_line(p)`, `perpendicular_segment(p)`
  - `projection(o)` — project a Point or LinearEntity onto this line.
  - `intersection(o)` — exhaustive case analysis for Line/Ray/Segment combinations (parallel overlap, single point, etc.).
  - `arbitrary_point(parameter)`, `random_point()`
  - `is_similar(other)`, `contains(other)` — interface for subclasses.
- **`Line(LinearEntity)`** — infinite 2D line.
  - Constructed from two points, a point+slope, or another `LinearEntity`.
  - `equation(x, y)` → symbolic expression `ax + by + c`.
  - `distance(o)` — perpendicular distance to a point.
  - `contains(o)` — uses `solveset`.
- **`Ray(LinearEntity)`** — semi-infinite line with a source.
  - Constructed from two points or a point+angle.
  - Properties: `source`, `direction`, `xdirection`, `ydirection`
  - `distance(o)`, `contains(o)` — respects ray directionality.
- **`Segment(LinearEntity)`** — finite line segment.
  - Auto-reorders endpoints for canonical form (smaller x first, then y).
  - Returns `Point` if endpoints coincide.
  - Properties: `length`, `midpoint`
  - `perpendicular_bisector(p=None)`
  - `distance(o)` — clamps projection parameter to [0, 1].
  - `contains(other)` — uses parametric test.

- `Undecidable(ValueError)` — raised when containment cannot be determined symbolically.

---

## Lines (3D)

### `line3d.py`
3D counterparts of `line.py`: `LinearEntity3D`, `Line3D`, `Ray3D`, `Segment3D`.

- **`LinearEntity3D(GeometryEntity)`** — abstract base for 3D linear entities.
  - Properties: `p1`, `p2`, `direction_ratio`, `direction_cosine`, `length`, `points`
  - `are_concurrent(*lines)` — checks pairwise intersection.
  - `is_parallel`, `is_perpendicular` — via direction cosines / dot product.
  - `angle_between(l2)`
  - `parallel_line(p)`, `perpendicular_line(p)`, `perpendicular_segment(p)`
  - `projection(o)` — project Point3D or LinearEntity3D.
  - `intersection(o)` — solves parametric equations via `linsolve`; handles parallel, skew, and intersecting cases.
  - `arbitrary_point(parameter)`, `is_similar`, `contains`
- **`Line3D`** — infinite 3D line; accepts `direction_ratio` keyword.
  - `equation(x, y, z, k)` — returns tuple of parametric ratios.
  - `distance(o)`, `contains(o)`, `equals(other)`
- **`Ray3D`** — 3D ray with `source`, `xdirection`, `ydirection`, `zdirection`.
  - `distance(o)`, `contains(o)`, `equals(other)`
- **`Segment3D`** — 3D segment with `length`, `midpoint`.
  - `distance(o)`, `contains(other)`

---

## Conics

### `ellipse.py`
Ellipse and Circle (as a degenerate Ellipse).

- **`Ellipse(GeometrySet)`** — defined by center, hradius, vradius (or eccentricity).
  - Returns `Circle` automatically when `hradius == vradius`.
  - Properties: `center`, `hradius`, `vradius`, `major`, `minor`, `area`, `circumference`, `eccentricity`, `periapsis`, `apoapsis`, `focus_distance`, `foci`, `bounds`
  - `rotate(angle, pt)` — only supports multiples of π/2 (general rotated ellipse unsupported).
  - `scale`, `reflect` — override base; reflect raises `NotImplementedError` for non-axis-aligned cases.
  - `encloses_point(p)` — uses foci distance sum.
  - `tangent_lines(p)` — via implicit differentiation (`idiff`); returns 1 or 2 lines.
  - `normal_lines(p, prec=None)` — solves quartic; optional numeric approximation.
  - `is_tangent(o)` — for Ellipse, LinearEntity, or Polygon.
  - `arbitrary_point`, `random_point(seed)`, `plot_interval`
  - `equation(x, y)`, `evolute(x, y)`
  - `intersection(o)` — dispatches to `_do_line_intersection` / `_do_ellipse_intersection`.
- **`Circle(Ellipse)`** — constructed from center+radius or three points (via `Triangle.circumcircle`).
  - Properties: `radius`, `vradius`, `circumference`
  - `equation(x, y)` — `(x-h)² + (y-k)² - r² = 0`.
  - `intersection(o)` — optimized circle-circle via geometric formula.
  - `scale` — returns `Ellipse` for non-uniform scaling.
  - `reflect(line)` — preserves circle form.

#### Caveats
- General rotated ellipses are **not** supported; `rotate` raises `NotImplementedError` for non-π/2 angles.
- `Ellipse.reflect` on non-axis-aligned lines gives only the zero-set equation.

### `parabola.py`
Vertical or horizontal parabola defined by focus and directrix.

- **`Parabola(GeometrySet)`**
  - Properties: `focus`, `directrix`, `axis_of_symmetry`, `focal_length`, `p_parameter`, `vertex`, `eccentricity` (always 1)
  - `equation(x, y)` — returns symbolic expression.

#### Caveats
- Only horizontal or vertical directrix lines are supported; oblique raises `NotImplementedError`.

---

## Curves

### `curve.py`
Parametric 2D curves defined by `(x(t), y(t))` over a parameter interval.

- **`Curve(GeometrySet)`** — `Curve((f_x, f_y), (t, t_min, t_max))`.
  - Properties: `functions`, `parameter`, `limits`, `free_symbols`
  - `arbitrary_point(parameter)` — substitutes parameter into functions.
  - `plot_interval(parameter)` — returns `[t, t_min, t_max]`.
  - `rotate`, `scale`, `translate` — override base to apply directly to parametric functions.
  - Substitution (`subs`) with the curve parameter evaluates to a `Point`.

---

## Polygons

### `polygon.py`
General polygon, regular polygon, and triangle with rich triangle-center support.

- **`Polygon(GeometrySet)`** — arbitrary simple polygon from a sequence of vertices.
  - Auto-removes consecutive duplicates and collinear interior points; degenerates to `Triangle`, `Segment`, or `Point`.
  - Validates no self-intersecting sides (via convex hull check).
  - Properties: `area` (signed, via shoelace), `angles`, `perimeter`, `vertices`, `centroid`, `sides`, `bounds`
  - `is_convex()`, `encloses_point(p)` — ray-casting for concave polygons.
  - `arbitrary_point(parameter)` — piecewise parametric around perimeter.
  - `intersection(o)`, `distance(o)`
  - `_do_poly_distance(e2)` — rotating calipers algorithm for convex polygon-polygon distance.
- **`RegularPolygon(Polygon)`** — center, circumradius, n sides, rotation.
  - Properties: `center`/`centroid`, `radius`/`circumradius`, `rotation`, `apothem`/`inradius`, `interior_angle`, `exterior_angle`, `circumcircle`, `incircle`, `length` (side), `angles`, `vertices` (computed trigonometrically)
  - `spin(angle)` — mutates rotation in-place.
  - `rotate`, `scale`, `reflect` — overrides handle the virtual vertex representation.
  - `encloses_point(p)` — fast-path via incircle/circumcircle radii.
- **`Triangle(Polygon)`** — 3-vertex polygon with construction from `sss`, `sas`, `asa` keywords.
  - Type tests: `is_equilateral()`, `is_isosceles()`, `is_scalene()`, `is_right()`
  - `is_similar(t2)` — checks all 6 side-ratio permutations.
  - Properties: `altitudes`, `orthocenter`, `circumcenter`, `circumradius`, `circumcircle`, `incenter`, `inradius`, `incircle`, `medians`, `medial`, `nine_point_circle`, `eulerline`
  - `bisectors()` — angle bisector segments.

- Module-level helpers:
  - `rad(d)`, `deg(r)` — degree↔radian conversion.
  - `_asa`, `_sss`, `_sas` — triangle construction helpers.

---

## 3D Geometry

### `plane.py`
Infinite plane in 3D space, defined by a point and normal vector (or three non-collinear points).

- **`Plane(GeometryEntity)`**
  - Properties: `p1`, `normal_vector`
  - `equation(x, y, z)` — returns linear expression.
  - `projection(pt)` — project Point3D onto plane along normal.
  - `projection_line(line)` — project 2D or 3D linear entity onto plane; useful for 2D↔3D interop.
  - `is_parallel(l)`, `is_perpendicular(l)` — works with `LinearEntity3D` or `Plane`.
  - `distance(o)` — to Point3D, LinearEntity3D, or Plane.
  - `angle_between(o)` — `asin` for lines, `acos` for planes.
  - `are_concurrent(*planes)` — static; checks if all intersections share a common line.
  - `perpendicular_line(pt)`, `parallel_plane(pt)`, `perpendicular_plane(*pts)`
  - `arbitrary_point(t)`, `random_point(seed)`
  - `intersection(o)` — handles Point, LinearEntity, and Plane (plane-plane yields `Line3D`).
  - `is_coplanar(o)` — for Plane, Point3D, or LinearEntity3D.

---

## Utilities

### `util.py`
Standalone geometric algorithms and helper functions.

- **`intersection(*entities)`** — pairwise intersection reduction across arbitrary geometry entities.
- **`convex_hull(*args, polygon=True)`** — Andrew's monotone chain algorithm; returns `Polygon`, `Segment`, or `Point`.
- **`closest_points(*args)`** — sweep-line algorithm for nearest pair in 2D; O(n log n).
- **`farthest_points(*args)`** — rotating calipers on convex hull for diameter.
- **`are_coplanar(*e)`** — tests coplanarity for mixed 3D entities (points, lines, planes).
- **`are_similar(e1, e2)`** — delegates to `is_similar` methods.
- **`centroid(*args)`** — weighted centroid for collections of Points, Segments, or Polygons.
- **`idiff(eq, y, x, n=1)`** — implicit differentiation: computes dy/dx from an implicit equation, supporting higher-order and multiple dependent variables.
- `_symbol(s, matching_symbol=None)` — coerce string to `Symbol(real=True)`.
- `_uniquely_named_symbol(xname, *exprs)` — generate a symbol name not clashing with existing free symbols.
- `_ordered_points(p)` — canonical ordering of point pairs.
