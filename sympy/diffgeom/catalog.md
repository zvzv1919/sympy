# sympy/diffgeom — Differential Geometry

## Manifolds, Patches, and Coordinate Systems

### `diffgeom.py`

Core module implementing the full differential geometry stack: manifolds, coordinate systems, scalar/vector/form fields, tensor algebra, derivatives, and curvature computations.

**Topological scaffolding:**

- `Manifold(name, dim)` — container holding a list of `Patch` instances; no topological analysis.
- `Patch(name, manifold)` — open subset of a manifold; container for `CoordSystem` charts.
- `CoordSystem(name, patch, names=None)` — coordinate chart on a patch.
  - `connect_to(to_sys, from_coords, to_exprs, inverse=True, fill_in_gaps=False)` — register coordinate transform; auto-inverts via `sympy.solve` when `inverse=True`.
  - `coord_tuple_transform_to` / `jacobian` — evaluate transforms and their Jacobians.
  - `coord_function` / `base_vector` / `base_oneform` — factory methods returning `BaseScalarField`, `BaseVectorField`, `Differential` instances.
  - `point(coords)` / `point_to_coords(point)` — create and project `Point` objects.
- `Point(coord_sys, coords)` — point on a manifold, defined by coordinates in a specific chart.

**Fields:**

- `BaseScalarField(coord_sys, index)` — coordinate function; evaluates at a `Point` to return one coordinate. Callable; applies `simplify().doit()` internally.
- `BaseVectorField(coord_sys, index)` — directional derivative operator along a coordinate line.
  - `__call__(scalar_field)` — computes the directional derivative via chain rule and Jacobian substitution.
- `Commutator(v1, v2)` — Lie bracket `[v1, v2]`; evaluates symbolically to zero when both fields share one coordinate system, otherwise deferred.

**Differential forms and tensor products:**

- `Differential(form_field)` — exterior derivative operator.
  - On 0-forms: `df(v) = v(f)`. On higher forms: uses the invariant formula with commutators.
  - `Differential(Differential(...))` returns `Zero` (d² = 0).
- `TensorProduct(*args)` — multilinear tensor product of form fields; supports partial contraction via `None` arguments.
- `WedgeProduct(*args)` — antisymmetric product of forms (subclass of `TensorProduct`); evaluates via signed permutation sums.

**Derivatives:**

- `LieDerivative(v_field, expr)` — Lie derivative; reduces to `v(f)` on scalars, `Commutator` on vector fields, deferred on higher forms.
- `BaseCovarDerivativeOp(coord_sys, index, christoffel)` — covariant derivative along a base vector, given Christoffel symbols.
- `CovarDerivativeOp(wrt, christoffel)` — covariant derivative along an arbitrary vector field; decomposes into `BaseCovarDerivativeOp` calls.

**Integral curves:**

- `intcurve_series(vector_field, param, start_point, n=6, coord_sys=None, coeffs=False)` — Taylor series expansion of the integral curve γ(t) to order n.
- `intcurve_diffequ(vector_field, param, start_point, coord_sys=None)` — returns (equations, initial_conditions) ODE system for the integral curve.

**Riemannian geometry (metric → curvature pipeline):**

- `twoform_to_matrix(expr)` — convert a 2-form to its matrix representation `M[i,j] = w(e_i, e_j)`.
- `metric_to_Christoffel_1st(expr)` — Christoffel symbols of the first kind (Γ_{ijk}).
- `metric_to_Christoffel_2nd(expr)` — Christoffel symbols of the second kind (Γ^i_{jk}); inverts the metric matrix (with a workaround for non-symbol entries).
- `metric_to_Riemann_components(expr)` — Riemann curvature tensor R^ρ_{σμν}.
- `metric_to_Ricci_components(expr)` — Ricci tensor via contraction of Riemann.

**Helpers:**

- `contravariant_order(expr)` / `covariant_order(expr)` — walk an expression tree to determine tensor order.
- `vectors_in_basis(expr, to_sys)` — re-express vector fields in a different coordinate basis via Jacobian.
- `dummyfy(args, exprs)` — replace symbols with dummies to avoid substitution collisions.

**Caveats:**
- `_fill_gaps_in_transformations` is `NotImplementedError`; transitive coordinate transforms must be registered manually.
- `Point.free_symbols` raises `NotImplementedError` (dead code after the raise).
- Simplification is fragile — many operations require manual `.doit()` or `simplify()` on results.
- `Commutator` evaluation across multiple coordinate systems is deferred (not fully evaluated).

## Predefined Manifolds

### `rn.py`

Predefined R² and R³ manifolds with standard coordinate systems and precomputed transformation laws.

- **R2**: 2-manifold with `R2_r` (rectangular: x, y) and `R2_p` (polar: r, θ) coordinate systems. Includes mutual coordinate transforms, basis coordinate functions, basis vectors, and basis 1-forms as attributes on the manifold, patch, and coordinate system objects.
- **R3**: 3-manifold with `R3_r` (rectangular), `R3_c` (cylindrical: ρ, ψ, z), and `R3_s` (spherical: r, θ, φ) coordinate systems. All six pairwise transforms registered; basis fields defined per chart.

Coordinate functions, vectors, and 1-forms are accessible as attributes (e.g. `R2.x`, `R2.e_x`, `R2.dx`, `R3_s.e_r`).

## Package Init

### `__init__.py`

Re-exports all public symbols from `diffgeom.py`. No additional logic.
