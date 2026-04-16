# sympy/vector — Catalog

> Part of [SymPy](../catalog.md). Vector algebra and calculus in curvilinear coordinate systems.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Public API surface for the vector module; re-exports all core classes and helper functions (Vector, Dyadic, CoordSys3D, Del, Point, orienters, operators, etc.). |
| `basisdependent.py` | Abstract base class `BasisDependent` providing shared arithmetic, simplification, and printing logic for vectors and dyadics. |
| `coordsysrect.py` | Defines `CoordSys3D` (and deprecated `CoordSysCartesian`), the 3-D coordinate system class. Owns `rotation_matrix()` which retrieves and composes pre-built DCMs along the frame tree (parent/child shortcuts and tree traversal) — it does not construct DCMs from rotation parameters itself (that is done by orienter classes in `orienters.py`). Also provides `scalar_map()` which returns a dictionary mapping one frame's base scalars (coordinate variables) to expressions in another frame's variables accounting for both rotation and origin translation, and `orient_new_*` / `locate_new` factory methods that create new frames oriented or translated relative to an existing one. Also handles curvilinear coordinate transformations and origin management. |
| `deloperator.py` | Implements the `Del` (nabla) vector differential operator with gradient, divergence, and curl methods. |
| `dyadic.py` | Defines the `Dyadic` class hierarchy (`BaseDyadic`, `DyadicAdd`, `DyadicMul`, `DyadicZero`) for representing and manipulating dyadic tensors. Includes `to_matrix` for converting a dyadic to a numeric SymPy Matrix (not for computing vector cross products). |
| `functions.py` | Utility functions for vector analysis: `express` (re-express a Vector, Dyadic, or scalar expression in another coordinate system's basis, optionally substituting coordinate variables), `matrix_to_vector`, `is_conservative` and `is_solenoidal` (check whether a vector field is curl-free or divergence-free respectively; both short-circuit to return True immediately for the zero vector without computing curl/divergence, and raise TypeError for non-Vector inputs), `scalar_potential`, `scalar_potential_difference`, `directional_derivative`, and `laplacian`. |
| `operators.py` | Defines unevaluated symbolic operator classes (`Gradient`, `Divergence`, `Curl`) and their evaluated counterparts (`gradient`, `divergence`, `curl`). |
| `orienters.py` | Orienter classes (`AxisOrienter`, `BodyOrienter`, `SpaceOrienter`, `QuaternionOrienter`) that each construct the actual 3×3 direction-cosine matrix (DCM) from their rotation parameters (e.g., quaternion components → explicit matrix with transposition to set the parent-to-child convention). Also contains the `_rot` helper that builds elementary single-axis DCMs (likewise transposed). These constructed matrices are consumed by `CoordSys3D.orient_new()` in `coordsysrect.py`. |
| `point.py` | Defines the `Point` class representing a point in 3-D space, with methods for computing position vectors between points. |
| `scalar.py` | Defines `BaseScalar`, the coordinate symbol (base scalar) class attached to a `CoordSys3D`, with custom pretty-printing and LaTeX support. |
| `vector.py` | Defines the `Vector` class hierarchy (`BaseVector`, `VectorAdd`, `VectorMul`, `VectorZero`, `Cross`, `Dot`) with dot/cross product operations and outer product support. The cross product implementation uses a custom inline 3×3 determinant helper (not SymPy's Matrix, which cannot hold basis-dependent vector elements) and iterates over coordinate systems to handle operands from different reference frames. Also handles cross products with dyadics by iterating over dyadic basis-pair components. |
