# sympy/vector — Catalog

> Part of [SymPy](../catalog.md). Vector algebra and calculus in curvilinear coordinate systems.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Public API surface for the vector module; re-exports all core classes and helper functions (Vector, Dyadic, CoordSys3D, Del, Point, orienters, operators, etc.). |
| `basisdependent.py` | Abstract base class `BasisDependent` providing shared arithmetic, simplification, and printing logic for vectors and dyadics. |
| `coordsysrect.py` | Defines `CoordSys3D` (and deprecated `CoordSysCartesian`), the 3-D coordinate system class. Owns `rotation_matrix()` for computing direction cosine matrices (DCMs) between frames with parent/child shortcuts and tree traversal, and `orient_new_*` factory methods (`orient_new_axis`, `orient_new_body`, `orient_new_space`, `orient_new_quaternion`) that create new frames oriented relative to an existing one. Also handles curvilinear coordinate transformations and origin management. |
| `deloperator.py` | Implements the `Del` (nabla) vector differential operator with gradient, divergence, and curl methods. |
| `dyadic.py` | Defines the `Dyadic` class hierarchy (`BaseDyadic`, `DyadicAdd`, `DyadicMul`, `DyadicZero`) for representing and manipulating dyadic tensors. |
| `functions.py` | Utility functions for vector analysis: `express` (re-express in another coordinate system), `matrix_to_vector`, `is_conservative`, `is_solenoidal`, `scalar_potential`, `scalar_potential_difference`, `directional_derivative`, and `laplacian`. |
| `operators.py` | Defines unevaluated symbolic operator classes (`Gradient`, `Divergence`, `Curl`) and their evaluated counterparts (`gradient`, `divergence`, `curl`). |
| `orienters.py` | Orienter parameter classes (`AxisOrienter`, `BodyOrienter`, `SpaceOrienter`, `QuaternionOrienter`) that store rotation parameters and produce a single rotation matrix. These are lightweight data objects consumed by `CoordSys3D.orient_new()` in `coordsysrect.py`; they do not create frames or compute inter-frame DCMs. |
| `point.py` | Defines the `Point` class representing a point in 3-D space, with methods for computing position vectors between points. |
| `scalar.py` | Defines `BaseScalar`, the coordinate symbol (base scalar) class attached to a `CoordSys3D`, with custom pretty-printing and LaTeX support. |
| `vector.py` | Defines the `Vector` class hierarchy (`BaseVector`, `VectorAdd`, `VectorMul`, `VectorZero`, `Cross`, `Dot`) with dot/cross product operations and outer product support. |
