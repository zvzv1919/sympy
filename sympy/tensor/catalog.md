# sympy/tensor — Catalog

> Part of [SymPy](../catalog.md). Tensor algebra: indexed objects, index conventions, and multi-dimensional array operations.

## Python Files

| File | Summary |
|------|----------|
| `__init__.py` | Package initializer; exports `IndexedBase`, `Idx`, `Indexed`, index-method helpers, and all N-dim array classes and operations. |
| `indexed.py` | Defines `IndexedBase`, `Indexed`, and `Idx` classes for representing indexed objects (e.g. matrix elements `M[i, j]`) with optional shape and range information. |
| `index_methods.py` | Provides functions for analyzing indexed expressions: `get_indices` determines free and dummy indices, and `get_contraction_structure` maps out implicit summations. |
| `tensor.py` | Implements the `Tensor` class for abstract index notation (Penrose formalism), tensor index types, symmetries, and the Butler-Portugal canonicalization algorithm via the `canon_bp()` method (which returns `S.Zero` if canonicalization determines the tensor vanishes). Also includes Einstein summation and related utilities. |
| `array/__init__.py` | Package initializer for the N-dim array submodule; exports dense/sparse and mutable/immutable array classes along with `tensorproduct`, `tensorcontraction`, `derive_by_array`, and `permutedims`. |
| `array/ndim_array.py` | Defines the `NDimArray` base class with common functionality for N-dimensional arrays such as indexing, shape handling, arithmetic, and conversion utilities. |
| `array/dense_ndim_array.py` | Implements `ImmutableDenseNDimArray` and `MutableDenseNDimArray`, which store all elements in a flat list for dense N-dimensional array representations. |
| `array/sparse_ndim_array.py` | Implements `ImmutableSparseNDimArray` and `MutableSparseNDimArray`, which store only non-zero elements via a dictionary-backed sparse representation. |
| `array/arrayop.py` | Provides array-level operations: `tensorproduct`, `tensorcontraction`, `derive_by_array` (elementwise differentiation), and `permutedims` (axis permutation). |
| `array/mutable_ndim_array.py` | Defines the `MutableNDimArray` base class that serves as the mixin for mutable N-dimensional arrays. |
