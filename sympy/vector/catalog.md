# vector — 3-D symbolic vector algebra and calculus

## Core Representation

### [`basisdependent.py`](basisdependent.py)
Abstract base for coordinate-frame-dependent quantities (vectors and dyadics).
- `BasisDependent(Expr)` — superclass providing arithmetic (+, -, *, /), `evalf`/`n` (numerical evaluation), `simplify`, `trigsimp`, `factor`, `diff` (rejects `BasisDependent` args with `TypeError`), `doit`.
- `evalf` decomposes into scalar coefficients and basis units, evaluates each scalar via `components` mapping, then reassembles.
- `BasisDependentAdd` — represents sums of basis-dependent terms.
- `BasisDependentMul.__new__` — scalar × basis-dependent product; counts non-scalar operands and raises `ValueError` if more than one (prevents e.g. vector×vector); distributes scalar over `BasisDependentAdd` sums.
- `BasisDependentZero` — zero element for basis-dependent quantities.

### [`vector.py`](vector.py)
Concrete vector classes built on `BasisDependent`.
- `Vector` — superclass for 3-D vectors; `dot`, `cross`, `outer`, `magnitude`, `normalize`, `to_matrix` (vector → 3×1 column matrix of components), `separate`.
  - `dot` dispatches on operand type: Vector → scalar, Dyadic → Vector (left-multiplies vector into dyadic, contracting the first basis index), Del → returns a **callable** (directional derivative operator).
  - The Del-dispatch closure converts scalar `0` to `Vector.zero` when the input field is a Vector, preventing type mismatches.
  - `cross` dispatches on operand type: Vector → Vector (via custom inline 3×3 determinant), Dyadic → Dyadic (distributes cross over each dyadic component's first basis vector, then re-forms outer products).
- `BaseVector` — unit basis vector (i, j, or k) tied to a coordinate system.
- `VectorAdd`, `VectorMul`, `VectorZero` — sum, scalar product, and zero specializations.
- `_vect_div` — division dispatch helper; raises `TypeError` if both operands are vectors, `ValueError` on divide-by-zero, otherwise returns `VectorMul` with inverse scalar.

### [`dyadic.py`](dyadic.py)
Dyadic tensor classes built on `BasisDependent`.
- `Dyadic` — superclass for dyadic tensors (tensor/outer products of vectors); `dot` (right-multiplies only: Dyadic·Vector→Vector, Dyadic·Dyadic→Dyadic), `cross` (Dyadic × Vector → Dyadic; crosses each component's second basis vector with the operand), `to_matrix`.
- `BaseDyadic` — outer product of two base vectors.
- `DyadicAdd`, `DyadicMul`, `DyadicZero` — sum, scalar product, and zero specializations.
- `_dyad_div` — division dispatch helper; raises `TypeError` if both operands are dyadics or if dividing by a dyadic, otherwise returns `DyadicMul` with inverse scalar.

### [`scalar.py`](scalar.py)
Coordinate variable symbols.
- `BaseScalar` — a symbolic coordinate variable (x, y, or z) bound to a specific coordinate system; marked `_diff_wrt = True` so it can appear as a differentiation variable.
- `_eval_derivative(s)` — returns `S.One` if differentiating w.r.t. itself, `S.Zero` otherwise (identity/zero derivative rule for coordinate components).

## Coordinate Systems & Orientation

### [`coordsysrect.py`](coordsysrect.py)
Cartesian coordinate system definition and creation — the **user-facing API** for building and orienting 3-D frames.
- `CoordSysCartesian` — 3-D Cartesian frame with base vectors (i, j, k), base scalars (x, y, z), and an origin `Point`.
- `orient_new_axis` — create a new system rotated about an arbitrary axis.
- `orient_new_body` — create a new system via body-fixed (Euler/Tait-Bryan) rotations; each successive rotation is about the *moving* frame's axes.
  - Accepts `rotation_order` string (length-3, XYZ or 123); consecutive same-axis forbidden ('XYX' ok, 'XXY' invalid).
- `orient_new_space` — create a new system via space-fixed rotations; each successive rotation is about the *parent* (fixed) frame's unit vectors (rotation matrices applied in reversed order compared to `orient_new_body`).
- `orient_new_quaternion` — create a new system oriented by quaternion parameters (q0=cos(θ/2), q1–q3=direction-sine components λ·sin(θ/2)); the user-facing API for finite-rotation frame creation via quaternions.
- `orient_new` — generic factory accepting a single `Orienter` or an iterable of orienters; composes multiple rotation matrices sequentially. Applies `trigsimp` only for a single orienter (not for iterable case).
- `locate_new` — create a translated system sharing the same orientation.
- `rotation_matrix` — direction cosine matrix between two systems.
- `scalar_map` — returns substitution dict mapping this system's base scalars to another's (used internally by `express`).

### [`orienters.py`](orienters.py)
Internal rotation-parameterization objects that construct direction cosine matrices; consumed by `CoordSysCartesian.orient_new_*` methods in `coordsysrect.py`.
- `Orienter` — base class; `rotation_matrix(system)`.
- `AxisOrienter` — rotation about an arbitrary axis by an angle.
- `ThreeAngleOrienter` — base for three-angle orienters; `_in_order` flag controls elementary rotation matrix multiplication order.
- `BodyOrienter(_in_order=True)` — body-fixed (Euler) rotations; matrices multiplied in given order (a1·a2·a3).
- `SpaceOrienter(_in_order=False)` — space-fixed rotations; matrices multiplied in reversed order (a3·a2·a1).
- `QuaternionOrienter` — constructs a 3×3 direction cosine matrix from four quaternion parameters (finite rotation about a unit axis).

### [`point.py`](point.py)
Spatial point representation with parent-child hierarchy.
- `Point` — 3-D point linked to an optional parent point, forming a tree of spatial locations.
- `position_wrt(other)` — computes relative displacement vector to another point; handles direct parent/child as special cases.
  - For non-adjacent points, traverses the point tree up to the common ancestor, accumulating and subtracting stored displacements along the path.
- `locate_new(name, position)` — creates a new child point at a given displacement from this point.
- `express_coordinates(coord_sys)` — returns (x, y, z) tuple of this point's position relative to a coordinate system's origin.
- Each `Point` stores `_parent`, `_pos` (displacement from parent), and `_root` (tree root); tree structure enables `position_wrt` between any two points sharing a common root.

## Operations & Calculus

### [`functions.py`](functions.py)
Vector calculus operations and coordinate re-expression.
- `express(expr, system, system2=None, variables=False)` — re-express vectors, dyadics, or scalars in a different coordinate system.
  - For dyadics, accepts an optional `system2` for the second basis index; defaults to `system` when omitted. Raises `ValueError` if `system2` is given for non-dyadic expressions.
  - When `variables=True`, substitutes foreign-frame coordinate variables (base scalars) via each foreign system's `scalar_map`.
- `curl`, `divergence`, `gradient` — convenience wrappers; computation logic lives in the `Del` operator class.
- `is_conservative`, `is_solenoidal` — field property tests; both short-circuit to `True` for the zero vector without computing curl/divergence.
- `scalar_potential`, `scalar_potential_difference` — potential computations.
- `matrix_to_vector` — inverse of `Vector.to_matrix`: takes a 3×1 column matrix and returns a Vector by combining its elements with the system's basis vectors (i, j, k).
- `orthogonalize` — Gram-Schmidt orthogonalization of vectors.
- `_path(from_object, to_object)` — traverses the parent-hierarchy tree to find the route between two coordinate systems/points; raises `ValueError` if they don't share a common root.

### [`deloperator.py`](deloperator.py)
Vector differential operator (∇), bound to a specific `CoordSysCartesian`.
- `Del` — symbolic nabla operator tied to a coordinate system; `__call__` is aliased to `gradient`.
  - `gradient` — re-expresses the scalar field into the operator's own coordinate system (via `express`) before taking partial derivatives along each axis.
  - `dot` (divergence), `cross` (curl) — analogous vector calculus operations on vector fields.
- `_diff_conditional` — re-expresses an expr into a coordinate system, returns `S(0)` if the base scalar is absent, else returns `Derivative`. Used only by `Del.dot`.

## Package Init

### [`__init__.py`](__init__.py)
Public API re-exports for the vector package.
