# diffgeom — Differential Geometry Module

## Core Library

### [`diffgeom.py`](diffgeom.py)
Class and function definitions for differential geometry on manifolds.
- `Manifold` — container for patches; no topological analysis.
- `Patch` — region of a manifold; container for coordinate systems.
- `CoordSystem` — coordinate chart on a patch; owns coord functions, base vectors, base oneforms.
  - `connect_to(other, from_coords, to_exprs, inverse=True, fill_in_gaps=True)` — register a transformation to another coord system.
- `Point` — point on a manifold, defined by coordinates in a given system.
- `BaseScalarField`, `BaseVectorField` — coordinate component scalar/vector fields.
- `Commutator` — commutator of two vector fields.
- `Differential` — exterior derivative operator on forms.
- `TensorProduct`, `WedgeProduct` — tensor and wedge products of forms.
- `LieDerivative` — Lie derivative along a vector field.
- `BaseCovarDerivativeOp`, `CovarDerivativeOp` — covariant derivative operators.
- `intcurve_series`, `intcurve_diffequ` — integral curve utilities.
- `metric_to_Christoffel_1st`, `metric_to_Christoffel_2nd` — Christoffel symbols from a metric.
- `metric_to_Riemann_components`, `metric_to_Ricci_components` — curvature tensors from a metric.
- `twoform_to_matrix`, `vectors_in_basis` — conversion helpers.

## Predefined Manifolds

### [`rn.py`](rn.py)
Module-level predefined instances of R² and R³ manifolds with standard coordinate systems and preregistered transformations.
- Instantiates `R2` (rectangular, polar) and `R3` (rectangular, cylindrical, spherical) manifolds, patches, and coord systems.
- Registers all pairwise coordinate transformations via `connect_to` with `inverse=False, fill_in_gaps=False` (automatic inverse derivation explicitly disabled; both directions registered manually).
- Exposes coordinate functions, basis vectors, and basis oneforms as attributes on coord system objects.
- For R2 only: also aliases coordinate functions, basis vectors, and oneforms onto the manifold and patch objects.
