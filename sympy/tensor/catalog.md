# tensor — Module Catalog

Symbolic objects with indices: abstract tensor algebra and indexed array expressions.

---

## indexed.py

Defines basic indexed objects for representing array elements like `M[i, j]`.

- `Indexed` — represents a complete indexed object (base + indices); properties: `base`, `indices`, `rank`, `shape`, `ranges`.
  - `shape` — returns base shape if defined; otherwise infers dimensions from each `Idx`'s `upper - lower + 1`. Raises `IndexException` if indices lack bounds attributes or if bounds are None.
  - `_eval_derivative(wrt)` — derivative w.r.t. another `Indexed`: returns product of KroneckerDeltas if same base and equal index count; raises `IndexException` if index counts differ; returns zero for different bases.
- `IndexedBase` — the stem/base of a concrete array-element expression (e.g., `A` in `A[i,j]`); supports `__getitem__` to create `Indexed`. Not related to abstract tensor algebra.
- `Idx` — integer index with optional range; properties: `label`, `lower`, `upper`.
- `IndexException` — raised for indexing errors.
- No index analysis, contraction logic, canonicalization, or structural equality (`equals`) methods; purely data-model classes.

---

## index_methods.py

Functions that **analyze indices** on `Indexed`/`IndexedBase` expressions only (not abstract `TensMul`/`TIDS` tensors): shape conformance, outer-index determination, and contraction-structure discovery.

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
  - `.data` setter — assigns numerical component data; accepts rank-1 (auto-expanded to diagonal square matrix) or rank-2 arrays; validates dimension match and square shape.
- `TensorIndex` — abstract tensor index; carries covariant/contravariant flag (`is_up`). Negation (`-idx`) returns a new index with flipped variance (upper ↔ lower).
- `TensorHead` — named tensor ("head" of an indexed tensor expression) with index types, rank, symmetry, and commutation properties.
  - `__new__` validates the `name` argument (must be string or Symbol; raises `ValueError` otherwise).
  - `_check_auto_matrix_indices_in_call` — when `True` is passed as an index placeholder, auto-fills slots: first occurrence of a type gets `auto_left`, second gets negated `auto_right`.
  - `__call__` — returns a `Tensor` with indices; supports auto-matrix index behavior via `True` placeholders or omitted trailing indices.
- `TensorSymmetry` — symmetry specification for tensor indices.
- `TensorType` — pairs a list of `TensorIndexType`s with a `TensorSymmetry`.
- `TIDS` — internal tensor-index data structure holding components, free indices, and dummy indices for a product of abstract tensors.
  - `TIDS.mul(f, g)` — multiplies two TIDS; contracts matching free indices of opposite variance; raises `ValueError` if both indices share the same covariant/contravariant orientation.
  - `TIDS.from_components_and_indices` — constructs TIDS from component list and index list.
  - `TIDS.get_indices` — reconstructs full index list from internal free/dum representation, generating fresh `TensorIndex` objects for contracted pairs; bumps the auto-name counter past any free indices already using the `dummy_fmt` pattern to avoid naming collisions.
  - `TIDS.get_tensors` — decomposes the stored product back into individual `Tensor` objects by slicing the full index list per component rank; preserves both contracted and uncontracted indices.
  - `TIDS.get_components_with_free_indices` — returns list of (component, free-indices) pairs; maps each factor to its uncontracted indices; returns all-empty lists when every index is contracted.
  - `TIDS._check_matrix_indices` — handles matrix-style auto-indices during multiplication.
- `Tensor` — single tensor (head + indices).
  - `__call__(*indices)` — substitutes ordered free indices; if new indices form contraction pairs (same label, opposite variance), rebuilds so those pairs become dummy/summation indices.
  - `equals(other)` — structural equality: handles zero (checks coeff), plain scalars (asserts no components), and general expressions via canonicalization tuple `(coeff, components, sorted free, sorted dum)` after `canon_bp`.
- `TensMul` — product of tensors with a scalar coefficient.
  - `__mul__` — Einstein summation: contracts matching upper/lower index pairs across factors.
  - `__div__` / `__truediv__` — division by scalars only; raises `ValueError('cannot divide by a tensor')` if divisor is a `TensExpr`.
  - `equals(other)` — structural equality: handles zero (checks coeff), plain scalars (asserts no components), and general expressions via canonicalization tuple `(coeff, components, sorted free, sorted dum)`.
  - `__call__(*indices)` — substitutes ordered free indices; if new indices form contraction pairs (index and its negation), rebuilds the expression so those pairs become dummy/summation indices.
- `TensAdd` — sum of tensors in canonical form.
  - `__new__` flattens nested sums, coerces plain-scalar addends into index-free tensor products, canonicalizes each term, sorts and collects like terms.
  - `_tensAdd_flatten` — separates scalar (non-tensor-expression) addends from indexed ones, sums the scalars, wraps result as an index-free product, and flattens any nested `TensAdd`.
  - `equals(other)` — equality for sums: compares argument sets when both are `TensAdd`; otherwise falls back to computing `self - other` and checking all coefficients are zero.
- `TensExpr` — abstract base for tensor expressions.
  - `__pow__(other)` — requires `.data` (ndarray); contracts the expression with itself via each free index's metric (numpy tensordot), producing a scalar norm², then returns `norm² ** (other/2)`.
  - `get_matrix()` — converts abstract tensor expression's attached ndarray component data to a `Matrix`; handles rank-1 (flat list) and rank-2 (nested lists); raises `NotImplementedError` for rank > 2.
- `canon_bp(p)` — canonicalize tensor via Butler-Portugal algorithm.
- `tensor_indices(s, typ)` — create `TensorIndex` objects from comma-separated string; returns a **single object** for one name, a **list** for multiple.
- `tensorhead(name, typ, sym)` — shorthand to create a `TensorHead`.
- `contract_metric(t, g)` — contract a tensor with a metric tensor.
- `riemann_cyclic(t)` — apply cyclic identity to Riemann tensor expressions.
- `get_lines(ex, index_type)` — analyzes contracted dummy indices in a product of matrix-valued tensors (e.g., spinor/gamma-matrix contractions); returns open multiplication chains, closed loops (traces), and remaining unmatched components. Raises `NotImplementedError` when contraction pattern requires transposition.
- `_TensorDataLazyEvaluator` — maps tensor expressions to numerical (ndarray) component data; computes lazily on `.data` access.
  - `__getitem__` — retrieves component data; unwraps zero-dimensional arrays to scalar (`dat[()]`) and single-element 1-d arrays to their sole element (`dat[0]`).
  - Retrieves data per-factor for `TensMul` products; raises `ValueError` if some factors have data and others do not.
  - `_correct_signature_from_indices` — adjusts ndarray values for covariant/contravariant index positions: lowers covariant indices via metric matrix, then contracts dummy (paired) index axes via numpy `trace`.
  - `data_product_tensors` — iteratively multiplies a list of ndarray factors via `reduce`; at each step pairs arrays with `TensMul` metadata, contracts matching indices, and accumulates the result.
  - For `TensAdd` sums, transposes each summand's ndarray so free-index axes align before element-wise addition.
  - Handles metric tensors specially via covariant/contravariant signature lookup.
- `_TensorManager` — singleton managing commutation groups and global tensor settings.
  - `comm_symbols2i(i)` — maps a label (symbol/string/number) to its internal commutation group number; **auto-registers** unseen labels by appending a new group that commutes with group 0.
  - `set_comm(i, j, c)` — registers commutation parameter (0=commuting, 1=anticommuting, None=no relation) between two named groups; new groups auto-commute with the universal group 0.
  - `get_comm(i, j)` — returns commutation parameter for group pair; defaults to 0 if either is group 0, else None for unregistered pairs.
  - `TensorManager` — module-level instance of `_TensorManager`.

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
- `tomatrix()` — converts a concrete rank-2 `NDimArray` to `Matrix`; raises `ValueError` for other ranks. Not for abstract tensor expressions.
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
- Module docstring serves as the primary tutorial for the array subpackage: array construction, element-wise ops, conversion (`tolist`, `tomatrix`), and combining operations.
- "Products and contractions" section: shows how to combine outer product + axis contraction to reproduce matrix multiplication, compute traces, etc.
- "Derivatives by array" section: documents `derive_by_array` usage patterns.

---

## __init__.py

Re-exports: `IndexedBase`, `Idx`, `Indexed` from `indexed`; `get_contraction_structure`, `get_indices` from `index_methods`.
