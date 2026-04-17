# tensor — Module Catalog

Symbolic objects with indices: abstract tensor algebra and indexed array expressions.

---

## indexed.py

Defines basic indexed objects for representing array elements like `M[i, j]`.

- `Indexed` — represents a complete indexed object (base + indices); properties: `base`, `indices`, `rank`, `shape`, `ranges`.
- `IndexedBase` — the stem/base of an indexed object (e.g., `A` in `A[i,j]`); supports `__getitem__` to create `Indexed`.
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
- `TensorHead` — named tensor with index types, rank, symmetry, and commutation properties.
- `TensorSymmetry` — symmetry specification for tensor indices.
- `TensorType` — pairs a list of `TensorIndexType`s with a `TensorSymmetry`.
- `TIDS` — internal tensor-index data structure holding components, free indices, and dummy indices.
  - `TIDS.mul(f, g)` — multiplies two TIDS; contracts matching free indices of opposite variance; raises `ValueError` if both indices share the same covariant/contravariant orientation.
  - `TIDS.from_components_and_indices` — constructs TIDS from component list and index list.
  - `TIDS._check_matrix_indices` — handles matrix-style auto-indices during multiplication.
- `Tensor` — single tensor (head + indices).
- `TensMul` — product of tensors with a scalar coefficient.
- `TensAdd` — sum of tensors in canonical form.
- `TensExpr` — abstract base for tensor expressions.
- `canon_bp(p)` — canonicalize tensor via Butler-Portugal algorithm.
- `tensor_indices(s, typ)` — create tensor index objects from string.
- `tensorhead(name, typ, sym)` — shorthand to create a `TensorHead`.
- `contract_metric(t, g)` — contract a tensor with a metric tensor.
- `riemann_cyclic(t)` — apply cyclic identity to Riemann tensor expressions.
- `_TensorManager` — singleton managing commutation groups and global tensor settings.

---

## __init__.py

Re-exports: `IndexedBase`, `Idx`, `Indexed` from `indexed`; `get_contraction_structure`, `get_indices` from `index_methods`.
