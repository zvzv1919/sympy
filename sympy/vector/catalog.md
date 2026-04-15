# sympy/vector — Catalog

> Part of [SymPy](../catalog.md). Vector algebra and calculus in curvilinear coordinate systems.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Public API surface for the vector module; re-exports all core classes and helper functions (Vector, Dyadic, CoordSys3D, Del, Point, orienters, operators, etc.). |
| `basisdependent.py` | Abstract base class `BasisDependent` providing shared arithmetic, simplification, and printing logic for vectors and dyadics. |
| `coordsysrect.py` | Defines `CoordSys3D` (and deprecated `CoordSysCartesian`), the 3-D Cartesian coordinate system class. Owns the `rotation_matrix` method that computes direction cosine matrices (DCMs) between any two frames by traversing the parent-chain tree and composing intermediate transformations. Also provides `orient_new_*` factory methods (`orient_new_axis`, `orient_new_body`, `orient_new_space`, `orient_new_quaternion`) that **create a new derived reference frame** from an existing one using various rotation parameterizations: axis-angle, body-fixed (Tait-Bryan / Euler intrinsic) rotations about the moving frame's axes, space-fixed (extrinsic) rotations about the original parent frame's axes, and quaternion (four-parameter finite-rotation). This is the authoritative file for any question about *constructing or orienting a new coordinate system*; `orienters.py` only supplies the low-level rotation-matrix math these factory methods delegate to. |
| `deloperator.py` | Implements the `Del` (nabla) vector differential operator with gradient, divergence, and curl methods. |
| `dyadic.py` | Defines the `Dyadic` class hierarchy (`BaseDyadic`, `DyadicAdd`, `DyadicMul`, `DyadicZero`) for representing and manipulating dyadic tensors. |
| `functions.py` | Utility functions for vector analysis: `express` (re-express a vector/dyadic/scalar entirely in a single target coordinate system), `matrix_to_vector`, `is_conservative`, `is_solenoidal`, `scalar_potential`, `scalar_potential_difference`, `directional_derivative`, and `laplacian`. Note: `express` converts to one frame; to decompose a vector into per-frame constituents without re-expressing, see `vector.py::Vector.separate`. |
| `operators.py` | Defines unevaluated symbolic operator classes (`Gradient`, `Divergence`, `Curl`) and their evaluated counterparts (`gradient`, `divergence`, `curl`). |
| `orienters.py` | Orienter classes (`AxisOrienter`, `BodyOrienter`, `SpaceOrienter`, `QuaternionOrienter`) that each compute a single rotation matrix for one orientation step. These are low-level building blocks—they do **not** create new coordinate systems, new reference frames, or compose multi-step DCMs between arbitrary frames. If the question is about creating/constructing a new frame via body-fixed (Tait-Bryan/Euler), space-fixed, axis-angle, or quaternion rotations, the answer is `coordsysrect.py` which owns the `orient_new_*` factory methods that instantiate new `CoordSys3D` objects. |
| `point.py` | Defines the `Point` class representing a point in 3-D space, with methods for computing position vectors between points. |
| `scalar.py` | Defines `BaseScalar`, the coordinate symbol (base scalar) class attached to a `CoordSys3D`, with custom pretty-printing and LaTeX support. |
| `vector.py` | Defines the `Vector` class hierarchy (`BaseVector`, `VectorAdd`, `VectorMul`, `VectorZero`, `Cross`, `Dot`) with dot/cross product operations, outer product, projection, matrix conversion (`to_matrix`), and the `separate` method that decomposes a vector spanning multiple reference frames into a per-frame mapping of constituent parts. For re-expressing a vector entirely in one frame, see `functions.py::express`. |
