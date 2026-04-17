# vector — 3-D symbolic vector algebra and calculus

## Core Representation

### [`basisdependent.py`](basisdependent.py)
Abstract base for coordinate-frame-dependent quantities (vectors and dyadics).
- `BasisDependent(Expr)` — superclass providing arithmetic (+, -, *, /), `evalf`/`n` (numerical evaluation), `simplify`, `trigsimp`, `factor`, `diff` (rejects `BasisDependent` args with `TypeError`), `doit`.
- `evalf` decomposes into scalar coefficients and basis units, evaluates each scalar via `components` mapping, then reassembles.
- `BasisDependentAdd` — represents sums of basis-dependent terms.
- `BasisDependentMul` — represents scalar × basis-dependent products.
- `BasisDependentZero` — zero element for basis-dependent quantities.

### [`vector.py`](vector.py)
Concrete vector classes built on `BasisDependent`.
- `Vector` — superclass for 3-D vectors; `dot`, `cross`, `outer`, `magnitude`, `normalize`, `to_matrix` (vector → 3×1 column matrix of components), `separate`.
  - `cross` uses a custom inline 3×3 determinant because SymPy's `Matrix` cannot hold basis-dependent vector elements.
- `BaseVector` — unit basis vector (i, j, or k) tied to a coordinate system.
- `VectorAdd`, `VectorMul`, `VectorZero` — sum, scalar product, and zero specializations.
- `_vect_div` — division dispatch helper; raises `TypeError` if both operands are vectors, `ValueError` on divide-by-zero, otherwise returns `VectorMul` with inverse scalar.

### [`dyadic.py`](dyadic.py)
Dyadic tensor classes built on `BasisDependent`.
- `Dyadic` — superclass for dyadic tensors; `dot`, `cross`, `to_matrix`.
- `BaseDyadic` — outer product of two base vectors.
- `DyadicAdd`, `DyadicMul`, `DyadicZero` — sum, scalar product, and zero specializations.

### [`scalar.py`](scalar.py)
Coordinate variable symbols.
- `BaseScalar` — a symbolic coordinate variable (x, y, or z) bound to a specific coordinate system.

## Coordinate Systems & Orientation

### [`coordsysrect.py`](coordsysrect.py)
Cartesian coordinate system definition and creation — the **user-facing API** for building and orienting 3-D frames.
- `CoordSysCartesian` — 3-D Cartesian frame with base vectors (i, j, k), base scalars (x, y, z), and an origin `Point`.
- `orient_new_axis` — create a new system rotated about an arbitrary axis.
- `orient_new_body` — create a new system via body-fixed (Euler) rotations; each successive rotation is about the *moving* frame's axes.
- `orient_new_space` — create a new system via space-fixed rotations; each successive rotation is about the *parent* (fixed) frame's unit vectors.
- `orient_new_quaternion` — create a new system oriented by quaternion parameters; wrapper that accepts four scalars and returns a new frame.
- `orient_new` — generic factory accepting a single `Orienter` or an iterable of orienters; composes multiple rotation matrices sequentially. Applies `trigsimp` only for a single orienter (not for iterable case).
- `locate_new` — create a translated system sharing the same orientation.
- `rotation_matrix` — direction cosine matrix between two systems.
- `scalar_map` — returns substitution dict mapping this system's base scalars to another's (used internally by `express`).

### [`orienters.py`](orienters.py)
Rotation-parameterization objects that construct direction cosine matrices from rotation parameters.
- `Orienter` — base class; `rotation_matrix(system)`.
- `AxisOrienter` — rotation about an arbitrary axis by an angle.
- `ThreeAngleOrienter` — base for three-angle orienters; `_in_order` flag controls elementary rotation matrix multiplication order.
- `BodyOrienter(_in_order=True)` — body-fixed (Euler) rotations; matrices multiplied in given order (a1·a2·a3).
- `SpaceOrienter(_in_order=False)` — space-fixed rotations; matrices multiplied in reversed order (a3·a2·a1).
- `QuaternionOrienter` — constructs a 3×3 direction cosine matrix from four quaternion parameters (finite rotation about a unit axis).

### [`point.py`](point.py)
Spatial point representation.
- `Point` — 3-D point with `position_wrt`, `locate_new`, `distance`.

## Operations & Calculus

### [`functions.py`](functions.py)
Vector calculus operations and coordinate re-expression.
- `express(expr, system, variables=False)` — re-express vectors, dyadics, or scalars in a different coordinate system.
  - When `variables=True`, substitutes foreign-frame coordinate variables (base scalars) via each foreign system's `scalar_map`.
- `curl`, `divergence`, `gradient` — standard differential operators on fields.
- `is_conservative`, `is_solenoidal` — field property tests.
- `scalar_potential`, `scalar_potential_difference` — potential computations.
- `matrix_to_vector` — inverse of `Vector.to_matrix`: takes a 3×1 column matrix and returns a Vector by combining its elements with the system's basis vectors (i, j, k).
- `orthogonalize` — Gram-Schmidt orthogonalization of vectors.
- `_path(from_object, to_object)` — traverses the parent-hierarchy tree to find the route between two coordinate systems/points; raises `ValueError` if they don't share a common root.

### [`deloperator.py`](deloperator.py)
Vector differential operator (∇).
- `Del` — symbolic nabla; `gradient`, `dot` (divergence), `cross` (curl).
- `_diff_conditional` — re-expresses an expr into a coordinate system, returns `S(0)` if the base scalar is absent, else returns `Derivative`. Used only by `Del.dot` (divergence).

## Package Init

### [`__init__.py`](__init__.py)
Public API re-exports for the vector package.
