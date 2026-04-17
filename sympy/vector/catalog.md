# vector — 3-D symbolic vector algebra and calculus

## Core Representation

### [`basisdependent.py`](basisdependent.py)
Abstract base for coordinate-frame-dependent quantities (vectors and dyadics).
- `BasisDependent(Expr)` — superclass providing arithmetic (+, -, *, /), `evalf`/`n` (numerical evaluation), `simplify`, `trigsimp`, `factor`, `diff`, `doit`.
- `evalf` decomposes into scalar coefficients and basis units, evaluates each scalar via `components` mapping, then reassembles.
- `BasisDependentAdd` — represents sums of basis-dependent terms.
- `BasisDependentMul` — represents scalar × basis-dependent products.
- `BasisDependentZero` — zero element for basis-dependent quantities.

### [`vector.py`](vector.py)
Concrete vector classes built on `BasisDependent`.
- `Vector` — superclass for 3-D vectors; `dot`, `cross`, `outer`, `magnitude`, `normalize`, `to_matrix`, `separate`.
- `BaseVector` — unit basis vector (i, j, or k) tied to a coordinate system.
- `VectorAdd`, `VectorMul`, `VectorZero` — sum, scalar product, and zero specializations.

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
Cartesian coordinate system definition and creation.
- `CoordSysCartesian` — 3-D Cartesian frame with base vectors (i, j, k), base scalars (x, y, z), and an origin `Point`.
- Methods to create related systems: `locate_new`, `orient_new`, `orient_new_axis`, `orient_new_body`, `orient_new_space`, `orient_new_quaternion`.
- `rotation_matrix`, `scalar_map` — inter-system transformations.

### [`orienters.py`](orienters.py)
Orientation parameterizations for coordinate system rotations.
- `Orienter` — base class; `rotation_matrix(system)`.
- `AxisOrienter` — rotation about an arbitrary axis by an angle.
- `BodyOrienter` / `SpaceOrienter` — Euler-angle rotations (body-fixed / space-fixed).
- `QuaternionOrienter` — quaternion-based rotation.

### [`point.py`](point.py)
Spatial point representation.
- `Point` — 3-D point with `position_wrt`, `locate_new`, `distance`.

## Operations & Calculus

### [`functions.py`](functions.py)
Vector calculus operations and coordinate re-expression.
- `express` — re-express vectors, dyadics, or scalars in a different coordinate system.
- `curl`, `divergence`, `gradient` — standard differential operators on fields.
- `is_conservative`, `is_solenoidal` — field property tests.
- `scalar_potential`, `scalar_potential_difference` — potential computations.
- `matrix_to_vector` — convert a matrix to a vector in a given system.
- `orthogonalize` — Gram-Schmidt orthogonalization of vectors.

### [`deloperator.py`](deloperator.py)
Vector differential operator (∇).
- `Del` — symbolic nabla; `gradient`, `dot` (divergence), `cross` (curl).
- `_diff_conditional` — re-expresses an expr into a coordinate system, returns `S(0)` if the base scalar is absent, else returns `Derivative`. Used only by `Del.dot` (divergence).

## Package Init

### [`__init__.py`](__init__.py)
Public API re-exports for the vector package.
