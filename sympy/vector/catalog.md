# sympy/vector — Vector Algebra & Calculus

Symbolic 3-D vector algebra and calculus built on top of `sympy.core`. Provides Cartesian coordinate systems, vectors, dyadics, differential operators (nabla), and standard vector-calculus operations (gradient, divergence, curl, scalar potential).

---

## Coordinate Systems & Geometry

### coordsysrect.py

Defines `CoordSysCartesian`, the central 3-D Cartesian coordinate system that owns base vectors (i, j, k), base scalars (x, y, z), an origin `Point`, and a `Del` operator.

- **`CoordSysCartesian`** — Core coordinate system class.
  - Construction accepts optional `location`, `rotation_matrix`, `parent`, and custom `vector_names`/`variable_names`.
  - Properties: `i/j/k` (base vectors), `x/y/z` (base scalars), `origin`, `delop`.
  - `base_vectors()` / `base_scalars()` — return the 3-tuples.
  - `rotation_matrix(other)` — cached DCM computation; walks the parent tree via `_path` when systems are not directly related.
  - `position_wrt(other)` — delegates to `origin.position_wrt`.
  - `scalar_map(other)` — maps own coordinate variables to another system's variables (accounts for rotation + translation).
  - `locate_new(name, position)` — create a translated child system.
  - `orient_new(name, orienters, ...)` — general orientation via one or more `Orienter` instances; applies rotation matrices in order and calls `trigsimp` for canonical form.
  - Convenience wrappers: `orient_new_axis`, `orient_new_body`, `orient_new_space`, `orient_new_quaternion`.
- `_check_strings(arg_name, arg)` — validates iterable-of-3-strings for custom naming.

**Caveat:** Coincident systems positioned/oriented wrt different parents may compare as unequal even though they share the same global pose.

### point.py

Represents named points in 3-D space, forming a tree rooted at the coordinate-system origin.

- **`Point`** — constructed with `(name, position_vector, parent_point)`.
  - `position_wrt(other)` — cached; walks the point tree via `_path`.
  - `locate_new(name, position)` — create a child point offset by a position vector.
  - `express_coordinates(coord_sys)` — returns `(x, y, z)` tuple wrt a coordinate system's origin.

### scalar.py

Coordinate variable symbol bound to a specific axis of a `CoordSysCartesian`.

- **`BaseScalar(Expr)`** — atomic, commutative, differentiable-wrt symbol.
  - Tracks `system`, `index` (0–2), and rendering forms (pretty/LaTeX).
  - `_eval_derivative` returns `S.One` when differentiating wrt itself.

---

## Core Vector & Dyadic Types

### basisdependent.py

Abstract base providing arithmetic, simplification, differentiation, and integration that both `Vector` and `Dyadic` inherit.

- **`BasisDependent(Expr)`** — operator overloads (`+`, `-`, `*`, `/`), plus:
  - `evalf`, `simplify`, `trigsimp`, `factor` — apply to each scalar component.
  - `diff`, `doit`, `_eval_Integral` — component-wise calculus.
  - `as_numer_denom`, `as_coeff_Mul`, `as_coeff_add`.
- **`BasisDependentAdd(BasisDependent, Add)`** — flattens / merges component dicts; returns zero when all components cancel.
- **`BasisDependentMul(BasisDependent, Mul)`** — scalar × base-instance product; validates that at most one basis-dependent factor appears.
- **`BasisDependentZero(BasisDependent)`** — singleton-style zero; custom `__eq__`, `__hash__`, and identity arithmetic.

### vector.py

Concrete vector types and operations (dot, cross, outer, projection).

- **`Vector(BasisDependent)`**
  - `components` — `{BaseVector: measure_number}` dict.
  - `magnitude()` / `normalize()`
  - `dot(other)` — handles `Vector` (scalar result), `Dyadic` (vector result), and `Del` (returns directional-derivative closure).
  - `cross(other)` — handles `Vector` (vector result) and `Dyadic` (dyadic result); uses a 3×3 symbolic determinant internally.
  - `outer(other)` — returns a `Dyadic`.
  - `projection(other, scalar=False)` — vector or scalar projection.
  - `to_matrix(system)` — 3×1 `ImmutableMatrix`.
  - `separate()` — splits into per-system constituent vectors.
- **`BaseVector(Vector, AtomicExpr)`** — unit basis vector tied to a `CoordSysCartesian`.
- **`VectorAdd`** / **`VectorMul`** / **`VectorZero`** — sum, scalar-product, and zero singletons.
- `_vect_div` — helper enforcing vector division rules.

Operator aliases: `&` → `dot`, `^` → `cross`, `|` → `outer`.

### dyadic.py

Second-order tensor (dyadic) types, parallel to the vector hierarchy.

- **`Dyadic(BasisDependent)`**
  - `dot(other)` — Dyadic·Dyadic → Dyadic, Dyadic·Vector → Vector.
  - `cross(other)` — Dyadic×Vector → Dyadic.
  - `to_matrix(system, second_system=None)` — 3×3 matrix representation.
- **`BaseDyadic(Dyadic, AtomicExpr)`** — outer product of two `BaseVector`s.
- **`DyadicMul`** / **`DyadicAdd`** / **`DyadicZero`** — sum, scalar-product, and zero singletons.
- `_dyad_div` — helper enforcing dyadic division rules.

Operator aliases: `&` → `dot`, `^` → `cross`.

---

## Differential Operators

### deloperator.py

The nabla (∇) operator, bound to a coordinate system.

- **`Del(Basic)`** — constructed from a `CoordSysCartesian`.
  - `gradient(scalar_field, doit=False)` — returns ∇f as a `Vector` of `Derivative` instances (or evaluated if `doit=True`). Also callable via `Del(field)`.
  - `dot(vect, doit=False)` — divergence (∇·v). Alias: `&`.
  - `cross(vect, doit=False)` — curl (∇×v). Alias: `^`.
- `_diff_conditional(expr, base_scalar)` — re-expresses `expr` in the scalar's system and differentiates only if the scalar appears; returns `S(0)` otherwise.

---

## Orientation

### orienters.py

Rotation-matrix builders used by `CoordSysCartesian.orient_new`.

- **`Orienter(Basic)`** — abstract base; `rotation_matrix()` returns stored DCM.
- **`AxisOrienter(Orienter)`** — rotation by `angle` about an arbitrary `axis` vector.
  - `rotation_matrix(system)` — Rodrigues' formula (normalized axis, cos/sin decomposition).
- **`ThreeAngleOrienter(Orienter)`** — base for body/space three-angle rotations.
  - Validates rotation order string (e.g. `'123'`, `'ZXZ'`); builds DCM from three elementary rotations.
  - `_in_order` class flag selects multiplication order.
- **`BodyOrienter(ThreeAngleOrienter)`** — `_in_order = True` (Euler / Tait-Bryan, body-fixed axes).
- **`SpaceOrienter(ThreeAngleOrienter)`** — `_in_order = False` (space-fixed axes, reversed order).
- **`QuaternionOrienter(Orienter)`** — builds DCM directly from quaternion parameters `(q0, q1, q2, q3)`.
- `_rot(axis, angle)` — returns elementary DCM for axis 1/2/3.

---

## Vector Calculus Utilities

### functions.py

Top-level functions for re-expression, differential calculus, and orthogonalization.

- **`express(expr, system, system2=None, variables=False)`** — re-expresses a Vector, Dyadic, or scalar field in a target coordinate system. When `variables=True`, also substitutes coordinate variables via `scalar_map`.
- **`curl(vect, coord_sys)`** — `coord_sys.delop.cross(vect).doit()`.
- **`divergence(vect, coord_sys)`** — `coord_sys.delop.dot(vect).doit()`.
- **`gradient(scalar, coord_sys)`** — `coord_sys.delop(scalar).doit()`.
- **`is_conservative(field)`** — checks `curl(field) == 0` (simplifies first).
- **`is_solenoidal(field)`** — checks `divergence(field) == 0`.
- **`scalar_potential(field, coord_sys)`** — integrates a conservative field to recover φ such that ∇φ = field (no integration constant).
- **`scalar_potential_difference(field, coord_sys, point1, point2)`** — evaluates φ(point2) − φ(point1); accepts either a scalar field or a conservative vector field.
- **`matrix_to_vector(matrix, system)`** — converts a 3×1 matrix to a `Vector` in the given system.
- **`orthogonalize(*vlist, orthonormal=False)`** — Gram–Schmidt process on a sequence of vectors; raises on linear dependence.
- `_path(from_object, to_object)` — walks the parent tree to find the common ancestor; returns `(root_index, path_list)`.

---

## Package Initialization

### \_\_init\_\_.py

Re-exports the public API: `Vector`, `VectorAdd`, `VectorMul`, `BaseVector`, `VectorZero`, `Dyadic`, `DyadicAdd`, `DyadicMul`, `BaseDyadic`, `DyadicZero`, `BaseScalar`, `Del`, `CoordSysCartesian`, `Point`, the four `Orienter` subclasses, and the function-level calculus utilities (`express`, `matrix_to_vector`, `curl`, `divergence`, `gradient`, `is_conservative`, `is_solenoidal`, `scalar_potential`, `scalar_potential_difference`).
