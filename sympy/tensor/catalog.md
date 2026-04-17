# tensor — Module Catalog

Symbolic objects with indices: abstract tensor algebra and indexed array expressions.

---

## indexed.py

Defines basic indexed objects for representing array elements like `M[i, j]`.

- `Indexed` — represents a complete indexed object (base + indices); properties: `base`, `indices`, `rank`, `shape`, `ranges`.
- `IndexedBase` — the stem/base of a concrete array-element expression (e.g., `A` in `A[i,j]`); supports `__getitem__` to create `Indexed`. Not related to abstract tensor algebra.
- `Idx` — integer index with optional range; properties: `label`, `lower`, `upper`.
- `IndexException` — raised for indexing errors.
- No index analysis or contraction logic; purely data-model classes.

---

## index_methods.py

Functions that **analyze indices** on `Indexed`/`IndexedBase` expressions: shape conformance, outer-index determination, and contraction-structure discovery.

- `get_indices(expr)` — returns the outer (non-summation) indices of an expression together with symmetry information.
- `get_contraction_structure(expr)` — maps summation (dummy) indices to the terms they apply to; returns nested dicts describing hierarchical contractions in products, powers, and sub-expressions.
- `_get_indices_Mul(expr)` — determines outer indices of a `Mul`; repeated indices across factors become dummies.
- `_get_indices_Pow(expr)` — determines outer indices of a `Pow`; treats power as element-wise (universal) function, so `x[i]**2` is NOT self-contraction; exponent indices are kept separate from base indices.
- `_get_indices_Add(expr)` — determines outer indices of an `Add`; checks conformance across terms.
- `_remove_repeated(inds)` — splits index list into unique set and repeated (dummy) tuple.
- `IndexConformanceException` — raised when indices across terms are inconsistent.

---

## tensor.py

Abstract index notation tensors (Penrose-style) with Einstein summation, canonicalization, and symmetry.

- `TensorIndexType` — characterizes a family of indices (name, metric, dimension, delta, epsilon).
- `TensorIndex` — abstract tensor index; carries covariant/contravariant flag (`is_up`).
- `TensorHead` — named tensor ("head" of an indexed tensor expression) with index types, rank, symmetry, and commutation properties.
  - `__new__` validates the `name` argument (must be string or Symbol; raises `ValueError` otherwise).
  - `_check_auto_matrix_indices_in_call` — when `True` is passed as an index placeholder, auto-fills slots: first occurrence of a type gets `auto_left`, second gets negated `auto_right`.
  - `__call__` — returns a `Tensor` with indices; supports auto-matrix index behavior via `True` placeholders or omitted trailing indices.
- `TensorSymmetry` — symmetry specification for tensor indices.
- `TensorType` — pairs a list of `TensorIndexType`s with a `TensorSymmetry`.
- `TIDS` — internal tensor-index data structure holding components, free indices, and dummy indices.
  - `TIDS.mul(f, g)` — multiplies two TIDS; contracts matching free indices of opposite variance; raises `ValueError` if both indices share the same covariant/contravariant orientation.
  - `TIDS.from_components_and_indices` — constructs TIDS from component list and index list.
  - `TIDS.get_components_with_free_indices` — returns list of (component, free-indices) pairs; maps each factor to its uncontracted indices; returns all-empty lists when every index is contracted.
  - `TIDS._check_matrix_indices` — handles matrix-style auto-indices during multiplication.
- `Tensor` — single tensor (head + indices).
  - `equals(other)` — structural equality via canonicalization: compares `(coeff, components, sorted free, sorted dum)` tuples after `canon_bp`.
- `TensMul` — product of tensors with a scalar coefficient.
  - `__mul__` — Einstein summation: contracts matching upper/lower index pairs across factors.
  - `__div__` / `__truediv__` — division by scalars only; raises `ValueError('cannot divide by a tensor')` if divisor is a `TensExpr`.
  - `equals(other)` — structural equality: handles zero (checks coeff), plain scalars (asserts no components), and general expressions via canonicalization tuple `(coeff, components, sorted free, sorted dum)`.
  - `__call__(*indices)` — substitutes ordered free indices; if new indices form contraction pairs (index and its negation), rebuilds the expression so those pairs become dummy/summation indices.
- `TensAdd` — sum of tensors in canonical form.
  - `__new__` flattens nested sums, coerces plain-scalar addends into index-free tensor products, canonicalizes each term, sorts and collects like terms.
  - `_tensAdd_flatten` — separates scalar (non-tensor-expression) addends from indexed ones, sums the scalars, wraps result as an index-free product, and flattens any nested `TensAdd`.
- `TensExpr` — abstract base for tensor expressions.
  - `get_matrix()` — converts attached ndarray component data to a `Matrix`; supports rank ≤ 2, raises `NotImplementedError` for higher ranks.
- `canon_bp(p)` — canonicalize tensor via Butler-Portugal algorithm.
- `tensor_indices(s, typ)` — create `TensorIndex` objects from comma-separated string; returns a **single object** for one name, a **list** for multiple.
- `tensorhead(name, typ, sym)` — shorthand to create a `TensorHead`.
- `contract_metric(t, g)` — contract a tensor with a metric tensor.
- `riemann_cyclic(t)` — apply cyclic identity to Riemann tensor expressions.
- `_TensorDataLazyEvaluator` — maps tensor expressions to numerical (ndarray) component data; computes lazily on `.data` access.
  - Retrieves data per-factor for `TensMul` products; raises `ValueError` if some factors have data and others do not.
  - For `TensAdd` sums, transposes each summand's ndarray so free-index axes align before element-wise addition.
  - Handles metric tensors specially via covariant/contravariant signature lookup.
- `_TensorManager` — singleton managing commutation groups and global tensor settings.

---

## array/ (subpackage)

Concrete N-dimensional array types (dense/sparse, mutable/immutable) and array operations (product, contraction, derivative, permutation).

### array/ndim_array.py

Base class for all N-dim arrays.

- `NDimArray` — abstract base providing shape, rank, `_parse_index`, `applyfunc`, `tolist`.
  - `diff(*args)` — differentiates each element w.r.t. given symbol(s); returns a new array of the **same shape**.
- `ImmutableNDimArray` — immutable base (SymPy `Basic` subclass).

### array/dense_ndim_array.py

Dense storage N-dim arrays backed by a flat internal list (`_array`).

- `DenseNDimArray` — constructor alias, returns `ImmutableDenseNDimArray`.
- `ImmutableDenseNDimArray` / `MutableDenseNDimArray` — immutable and mutable dense variants.
- `__getitem__` — tuple-of-slices indexes shape-aware; a plain (non-tuple) slice operates directly on the flat `_array`.
- `tomatrix()` — converts rank-2 array to `Matrix`; raises `ValueError` for other ranks.
- `zeros`, `reshape`.

### array/sparse_ndim_array.py

Sparse storage N-dim arrays backed by a dict (`_sparse_array`).

- `SparseNDimArray` — constructor alias, returns `ImmutableSparseNDimArray`.
- `ImmutableSparseNDimArray` / `MutableSparseNDimArray` — immutable and mutable sparse variants.
- `__getitem__` — same tuple-of-slices logic as dense; missing keys default to zero.
- `tomatrix()`, `zeros`, `reshape`.

### array/arrayop.py

Standalone functions for tensor-style operations on N-dim arrays.

- `tensorproduct(*args)` — outer (tensor) product of arrays/scalars; result rank = sum of input ranks.
- `tensorcontraction(array, *contraction_axes)` — contracts (sums) over specified axis pairs; validates axes are distinct and dimensions match (raises `ValueError` on mismatch).
- `derive_by_array(expr, dx)` — partial derivative of array w.r.t. array/scalar; result rank = rank(dx) + rank(expr) (shape is **extended**, not preserved).
- `permutedims(expr, perm)` — reorders axes of an N-dim array by a permutation.

### array/mutable_ndim_array.py

- `MutableNDimArray` — mixin base adding `__setitem__` for in-place element modification.

### array/__init__.py

Re-exports: `Array` (alias for `ImmutableDenseNDimArray`), dense/sparse array classes, `tensorproduct`, `tensorcontraction`, `derive_by_array`, `permutedims`.

---

## __init__.py

Re-exports: `IndexedBase`, `Idx`, `Indexed` from `indexed`; `get_contraction_structure`, `get_indices` from `index_methods`.
