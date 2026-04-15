# sympy/tensor — Catalog

> Part of [SymPy](../catalog.md). Tensor algebra: indexed objects, index conventions, and multi-dimensional array operations.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package initializer; exports `IndexedBase`, `Idx`, `Indexed`, index-method helpers, and all N-dim array classes and operations. |
| `indexed.py` | Defines `IndexedBase`, `Indexed`, and `Idx` classes for representing indexed objects (e.g. matrix elements `M[i, j]`) with optional shape and range information. |
| `index_methods.py` | Provides purely symbolic/structural analysis functions for `IndexedBase`/`Indexed` expressions only (not abstract tensor algebra classes like `TensMul`/`TensAdd`): `get_indices` determines free and dummy indices, and `get_contraction_structure` maps out implicit summation patterns (does not perform numerical contractions). For automatic index contraction during abstract tensor multiplication, see `TIDS.mul` in `tensor.py`. |
| `tensor.py` | Implements tensors with abstract index notation (Penrose formalism), including tensor index types, symmetries, canonicalization via the Butler-Portugal algorithm, and Einstein summation. Core classes: `TensMul` (tensor products) with arithmetic operations (multiply, divide — division by a tensor raises `ValueError`), `TensAdd` (tensor sums) with equality comparison that short-circuits when the other operand has a zero scalar coefficient, and `TIDS` (tensor-index data structure) whose `mul` method detects matching covariant/contravariant index pairs across two operands for automatic dummy-index contraction and raises `ValueError` if two same-variance indices are mistakenly paired. Also handles numerical component data: attaching arrays to tensor heads, computing scalar magnitudes by contracting component arrays with associated bilinear-form (metric) data, and exponentiation of tensors with numeric components via iterative metric contraction and root extraction. Note: this module is for *abstract* tensor algebra — for numeric N-dimensional array operations see `array/` subpackage; for symbolic index analysis of `IndexedBase` expressions see `index_methods.py`. |
| `array/__init__.py` | Package initializer for the N-dim array submodule; exports dense/sparse and mutable/immutable array classes along with `tensorproduct`, `tensorcontraction`, `derive_by_array`, and `permutedims`. |
| `array/ndim_array.py` | Defines the `NDimArray` base class with common functionality for numeric N-dimensional arrays such as indexing, shape handling, arithmetic (including element-wise division), and conversion utilities. This is for concrete numeric arrays — for abstract tensor algebra division/arithmetic see `TensMul` in `tensor.py`. |
| `array/dense_ndim_array.py` | Implements `ImmutableDenseNDimArray` and `MutableDenseNDimArray`, which store all elements in a flat list for dense N-dimensional array representations. |
| `array/sparse_ndim_array.py` | Implements `ImmutableSparseNDimArray` and `MutableSparseNDimArray`, which store only non-zero elements via a dictionary-backed sparse representation. |
| `array/arrayop.py` | Provides numeric array-level operations: `tensorproduct`, `tensorcontraction`, `derive_by_array` (elementwise differentiation), and `permutedims` (axis permutation). These operate on `NDimArray` objects — for abstract tensor product/sum algebra and equality comparison see `TensMul`/`TensAdd` in `tensor.py`. |
| `array/mutable_ndim_array.py` | Defines the `MutableNDimArray` base class that serves as the mixin for mutable N-dimensional arrays. |
