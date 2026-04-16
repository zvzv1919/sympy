# sympy/tensor — Tensor Algebra

## Indexed Objects

### `indexed.py`
Defines indexed mathematical objects using Einstein-like notation with implicit contraction of repeated indices.

- **`IndexException`** — raised on index errors (rank mismatch, undefined range, etc.)
- **`Indexed(Expr)`** — represents a fully-indexed object (e.g. `M[i, j]`):
  - Created via `IndexedBase.__getitem__`; requires at least one index.
  - Properties: `base`, `indices`, `rank`, `shape`, `ranges`.
  - Derivative w.r.t. another `Indexed` yields a product of Kronecker deltas.
- **`IndexedBase(Expr)`** — the "stem" of an indexed object (the array symbol):
  - Accepts optional `shape`; `__getitem__` returns an `Indexed` instance.
  - Shape on the base overrides shape inferred from index ranges.
- **`Idx(Expr)`** — integer index with optional range:
  - Range as a single integer → `(0, n-1)`; as a tuple → explicit `(lower, upper)`.
  - String labels are auto-converted to integer `Symbol`s.

### `index_methods.py`
Analysis utilities for indexed expressions — determines free/dummy indices and contraction structure.

- `get_indices(expr)` — returns `(free_indices_set, symmetry_dict)` for an expression; repeated indices are treated as summed (dummy).
- `get_contraction_structure(expr)` — returns a nested dict describing which dummy indices contract which terms, recursively for `Mul`, `Add`, `Pow`, and `Function` nodes.
- `_remove_repeated(inds)` — splits an index list into unique and repeated (dummy) sets.
- `_get_indices_Mul`, `_get_indices_Pow`, `_get_indices_Add` — type-specific index extraction helpers.
- **`IndexConformanceException`** — raised when terms in a sum have inconsistent outer indices.

**Caveats:** `get_indices` applies contraction recursively from the innermost parentheses outward. Expanding such expressions can mix outer indices with inner dummies, producing incorrect results.

## Abstract Index Tensor Algebra

### `tensor.py`
Large (~4000 lines) module implementing tensors with abstract (Penrose-style) index notation, canonicalization via Butler-Portugal, and optional NumPy data binding.

#### Internal Data Structures
- **`TIDS`** — tensor-index data structure storing components, free indices (triplets), and dummy indices (4-tuples). Not intended for end-user use.
- **`_TensorDataLazyEvaluator`** — global dict-like store mapping tensor heads/expressions to NumPy ndarrays; handles metric contractions, axis permutations, and lazy evaluation.
- **`_TensorManager`** — singleton managing commutativity relations between tensor types (commuting, anticommuting, or neither).

#### Index & Symmetry Types
- **`TensorIndexType`** — defines an index type with dimension, metric, Kronecker delta, Levi-Civita epsilon, and dummy-index format string. Supports auto-matrix indices (`auto_left` / `auto_right`).
- **`TensorIndex`** — a named index belonging to a `TensorIndexType`; negation flips covariant ↔ contravariant.
- **`TensorSymmetry`** — stores BSGS (base and strong generating set) for monoterm symmetries.
- **`TensorType`** — pairs a list of `TensorIndexType`s with a `TensorSymmetry`; callable to create `TensorHead`s.

#### Tensor Heads & Expressions
- **`TensorHead`** — the "name" of a tensor (e.g. `T`); calling it with indices produces a `Tensor` or `TensMul`. Supports optional matrix-index behavior and NumPy data binding.
- **`TensExpr`** — abstract base for tensor expressions (`TensAdd`, `TensMul`). Provides `get_matrix()`, `__neg__`, and arithmetic stubs.
- **`TensAdd`** — sum of tensor terms; canonicalizes each term and merges like terms.
- **`Tensor`** — a single tensor head applied to specific indices.
- **`TensMul`** — product of tensors with a scalar coefficient; handles dummy-index renaming, metric contraction, and canonical ordering.

#### Convenience Functions
- `tensor_indices(s, typ)` — create multiple `TensorIndex` objects from a comma-separated string.
- `tensorsymmetry(*args)` — build a `TensorSymmetry` from Young-tableau-style arguments.
- `tensorhead(name, typ, sym, comm=0)` — shortcut to create a `TensorHead`.
- `canon_bp(p)` — canonicalize a tensor expression using Butler-Portugal.
- `tensor_mul(*a)` — multiply tensors, flattening nested `TensMul`s.
- `riemann_cyclic(t2)` — enforce the first Bianchi identity on Riemann-tensor expressions.
- `contract_metric(t, g)` — raise/lower indices using metric `g`.
- `substitute_indices(t, *index_tuples)` — replace free indices by symbol (no raising/lowering).
- `perm2tensor(t, g)` — build a tensor from a permutation.
- `get_lines(ex, index_type)` — extract matrix-line and trace structure for a given index type.
- `get_indices(t)`, `get_tids(t)`, `get_coeff(t)` — lightweight accessors.

**Caveats:** Data binding (`TensorHead.data`) depends on NumPy at runtime. The `_TensorDataLazyEvaluator` is marked experimental — its API may change.

## N-dimensional Arrays (`array/`)

### `ndim_array.py`
Base class for all N-dim array variants.

- **`NDimArray`** — provides common interface: `shape`, `rank()`, `diff()`, `applyfunc()`, `tolist()`, `tomatrix()`, `transpose()`, `conjugate()`, `adjoint()`, and arithmetic (`+`, `-`, scalar `*`, `/`).
  - Construction from nested lists auto-detects shape (`_scan_iterable_shape`).
  - Indexing via `_parse_index` / `_get_tuple_index` converts between flat and multi-dim.
- **`ImmutableNDimArray(NDimArray, Basic)`** — immutable base with `_op_priority = 11.0`.

### `dense_ndim_array.py`
Dense (all-elements-stored) N-dim arrays.

- **`DenseNDimArray(NDimArray)`** — shared logic for dense arrays: `__getitem__` with slice support, `zeros()`, `tomatrix()`, `reshape()`.
- **`ImmutableDenseNDimArray(DenseNDimArray, ImmutableNDimArray)`** — immutable dense; aliased as `Array` at package level.
- **`MutableDenseNDimArray(DenseNDimArray, MutableNDimArray)`** — mutable dense; supports `__setitem__`.

### `sparse_ndim_array.py`
Sparse (only-nonzero-stored) N-dim arrays.

- **`SparseNDimArray(NDimArray)`** — shared sparse logic using an internal dict (`_sparse_array`): `__getitem__` with slice support, `zeros()`, `tomatrix()` (returns `SparseMatrix`), `reshape()`.
- **`ImmutableSparseNDimArray(SparseNDimArray, ImmutableNDimArray)`** — immutable sparse.
- **`MutableSparseNDimArray(MutableNDimArray, SparseNDimArray)`** — mutable sparse; `__setitem__` auto-removes zero entries.

### `mutable_ndim_array.py`
- **`MutableNDimArray(NDimArray)`** — empty mixin base for mutable variants.

### `arrayop.py`
Tensor-style operations on N-dim arrays.

- `tensorproduct(*args)` — outer (tensor) product of arrays/scalars; result rank is the sum of input ranks.
- `tensorcontraction(array, *contraction_axes)` — sum over specified axis pairs (generalised trace / matrix multiply).
- `derive_by_array(expr, dx)` — element-wise derivative of an array (or scalar) w.r.t. an array (or scalar) of variables.
- `permutedims(expr, perm)` — reorder axes of an N-dim array by a permutation (or `Permutation` object).

### `array/__init__.py`
Re-exports all four array classes plus `arrayop` functions. Sets convenience aliases:
`Array = NDimArray = DenseNDimArray = ImmutableDenseNDimArray`, `SparseNDimArray = ImmutableSparseNDimArray`.
