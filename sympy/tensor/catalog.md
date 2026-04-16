# sympy/tensor — Catalog

> Part of [SymPy](../catalog.md). Tensor algebra: indexed objects, index conventions, and multi-dimensional array operations.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package initializer; exports `IndexedBase`, `Idx`, `Indexed`, index-method helpers, and all N-dim array classes and operations. |
| `indexed.py` | Defines `IndexedBase`, `Indexed`, and `Idx` classes for representing simple indexed objects (e.g. matrix elements `M[i, j]`) with optional shape and range information. Does not handle abstract tensor algebra, auto-matrix indices, or symmetry-aware index notation (those are in `tensor.py`). |
| `index_methods.py` | Provides functions for analyzing indices of `Indexed`-based (not abstract tensor) expressions: `get_indices` dispatches to expression-type-specific handlers to determine free/dummy indices; `get_contraction_structure` maps out implicit summations in `Indexed` expressions. For abstract tensor index assembly and contraction detection during construction, see `tensor.py`. |
| `tensor.py` | Implements tensors with abstract index notation (Penrose formalism). Core classes: `TIDS` (internal data structure assembling components and indices, partitioning index slots among heads and detecting contractions during construction); `TensExpr` (base for tensor expressions, with `get_matrix` to convert ndarray component data into a Matrix for rank ≤ 2); `TensorHead` (named tensor with index types, including auto-matrix index logic that assigns auto-left/negated-auto-right indices when `True` placeholders are passed); `TensMul`/`TensAdd` (products and sums of tensors). Also covers tensor index types, symmetries, canonicalization via Butler-Portugal, Einstein summation, and index replacement. |
| `array/__init__.py` | Package initializer for the N-dim array submodule; exports dense/sparse and mutable/immutable array classes along with `tensorproduct`, `tensorcontraction`, `derive_by_array`, and `permutedims`. |
| `array/ndim_array.py` | Defines the `NDimArray` base class with common functionality for N-dimensional arrays such as indexing, shape handling, arithmetic, and transpose. Operates on concrete numeric arrays, not abstract indexed tensor expressions (for converting abstract tensor data to matrices, see `tensor.py`). |
| `array/dense_ndim_array.py` | Implements `ImmutableDenseNDimArray` and `MutableDenseNDimArray`, which store all elements in a flat list for dense N-dimensional array representations. |
| `array/sparse_ndim_array.py` | Implements `ImmutableSparseNDimArray` and `MutableSparseNDimArray`, which store only non-zero elements via a dictionary-backed sparse representation. |
| `array/arrayop.py` | Provides array-level operations: `tensorproduct`, `tensorcontraction`, `derive_by_array` (elementwise differentiation), and `permutedims` (axis permutation). |
| `array/mutable_ndim_array.py` | Defines the `MutableNDimArray` base class that serves as the mixin for mutable N-dimensional arrays. |
