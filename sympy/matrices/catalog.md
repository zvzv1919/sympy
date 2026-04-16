# sympy/matrices — Matrix Algebra

## Glossary

- **DOK**: Dictionary of Keys — sparse storage format mapping `(row, col)` to value.
- **CSR**: Compressed Sparse Row — compact storage format for sparse matrices.
- **Mutable vs Immutable**: Mutable matrices allow element assignment; immutable matrices are hashable SymPy `Basic` objects suitable for use in expressions.
- **MatrixExpr**: Symbolic matrix expression tree node — never materialised until `.as_explicit()` or `.doit()` is called.

---

## Core Matrix Infrastructure

### `matrices.py`
The foundational module (~4500 lines). Defines `MatrixBase` and all shared matrix logic.

- `class MatrixBase` — base class for all concrete matrix types. Key capabilities:
  - **Construction**: `_handle_creation_inputs` — polymorphic constructor accepting nested lists, dicts, callables, flat lists, etc.
  - **Decompositions**: `LUdecomposition`, `LUdecomposition_Simple`, `LUdecompositionFF` (fraction-free), `QRdecomposition`, `cholesky`, `LDLdecomposition`
  - **Solvers**: `LUsolve`, `QRsolve`, `cholesky_solve`, `LDLsolve`, `diagonal_solve`, `lower_triangular_solve`, `upper_triangular_solve`, `solve`, `solve_least_squares`, `pinv_solve`, `gauss_jordan_solve`
  - **Determinant**: `det` (Bareis/Berkowitz/LU), `det_bareis`, `det_LU_decomposition`, `berkowitz`, `berkowitz_det`, `berkowitz_minors`, `berkowitz_charpoly`
  - **Eigenproblems**: `eigenvals`, `eigenvects`, `left_eigenvects`, `singular_values`, `condition_number`, `diagonalize`, `is_diagonalizable`, `jordan_form`, `jordan_cells`
  - **Row-reduction**: `rref`, `rank`, `nullspace`, `columnspace`
  - **Properties**: `is_square`, `is_zero`, `is_symmetric`, `is_anti_symmetric`, `is_hermitian`, `is_diagonal`, `is_upper`, `is_lower`, `is_upper_hessenberg`, `is_lower_hessenberg`, `is_nilpotent`, `is_symbolic`
  - **Calculus**: `jacobian`, `diff`, `integrate`, `limit`
  - **Vector ops**: `cross`, `dot`, `norm`, `normalized`, `project`
  - **Structural**: `hstack`, `vstack`, `row_join`, `col_join`, `row_insert`, `col_insert`, `extract`, `reshape`, `get_diag_blocks`, `vec`, `vech`
  - **Misc**: `cofactorMatrix`, `cofactor`, `minorEntry`, `minorMatrix`, `adjugate`, `inverse_LU`, `inverse_GE`, `inverse_ADJ`, `pinv`, `dual`, `exp`, `permuteBkwd`, `permuteFwd`, `replace`
- `class DeferredVector(Symbol)` — vector whose components are lazily resolved by index (used with `lambdify`).
- `class ShapeError`, `class NonSquareMatrixError`, `class MatrixError`
- `def classof(A, B)` — returns the higher-priority class of two matrices.
- `def a2idx(j, n=None)` — converts a value to a valid matrix index.

### `__init__.py`
Public API surface — re-exports classes and factory functions from submodules. Also defines convenience aliases (`MutableDenseMatrix = MutableMatrix = Matrix`, `SparseMatrix = MutableSparseMatrix`, `ImmutableDenseMatrix = ImmutableMatrix`).

---

## Dense Matrices

### `dense.py`
Mutable dense matrix implementation. Storage: flat Python list (`self._mat`).

- `class DenseMatrix(MatrixBase)` — base dense class providing:
  - Element access (`__getitem__`), trace, transpose, conjugate, adjoint, inverse (GE/LU/ADJ)
  - `_cholesky`, `_LDLdecomposition`, triangular/diagonal solvers
  - `applyfunc`, `reshape`, `equals`, `as_mutable`, `as_immutable`
  - Arithmetic operators delegate through `_force_mutable`
- `class MutableDenseMatrix(DenseMatrix, MatrixBase)` — the primary user-facing `Matrix`. Adds:
  - `__setitem__`, `copyin_matrix`, `copyin_list`
  - In-place row/col operations: `row_op`, `col_op`, `zip_row_op`, `row_swap`, `col_swap`, `row_del`, `col_del`
  - `simplify` (in-place), `fill`
- **Factory functions**: `zeros`, `ones`, `eye`, `diag`, `jordan_cell`, `randMatrix`
- **Mathematical functions**: `hessian`, `GramSchmidt`, `wronskian`, `casoratian`, `matrix_multiply_elementwise`
- **NumPy interop**: `list2numpy`, `matrix2numpy`, `symarray`
- **Rotation matrices**: `rot_axis1`, `rot_axis2`, `rot_axis3`

### `densearith.py`
Low-level dense arithmetic on list-of-lists representation (domain-ring `K` parameter).

- `add` / `addrow` — matrix/row addition
- `sub` — subtraction via negation + addition
- `negate` / `negaterow`
- `mulmatmat` / `mulmatscaler` / `mulrowscaler` / `mulrowcol`

### `densesolve.py`
Equation-solving routines on list-of-lists representation.

- `row_echelon`, `rref`
- `LU`, `cholesky`, `LDL`
- `upper_triangle`, `lower_triangle`
- `rref_solve`, `LU_solve`, `cholesky_solve`
- `forward_substitution`, `backward_substitution`

### `densetools.py`
Utility operations on list-of-lists representation.

- `trace`, `transpose`, `conjugate`, `conjugate_row`, `conjugate_transpose`
- `augment` — append column to matrix
- `eye`, `row`, `col`
- `rowswap`, `rowmul`, `rowadd`
- `isHermitian`

**Caveat**: `densearith.py`, `densesolve.py`, and `densetools.py` operate on raw `list[list]` representations with an explicit domain ring `K` (e.g. `ZZ`, `QQ`). They are independent of `MatrixBase` and serve as a lower-level algebra layer.

---

## Sparse Matrices

### `sparse.py`
Mutable and immutable sparse matrix implementation. Storage: DOK dict (`self._smat`).

- `class SparseMatrix(MatrixBase)` — base sparse class:
  - `__getitem__` with dict lookup, `copy`, `is_Identity`, `tolist`, `row`, `col`
  - `row_list` / `col_list` (and `RL` / `CL` properties) — sorted non-zero element accessors
  - Arithmetic: `multiply` (sparse-aware), `scalar_multiply`, `add`, `__mul__`, `__add__`, `__neg__`
  - Decompositions: `_cholesky_sparse`, `_LDL_sparse`, `cholesky`, `LDLdecomposition`
  - Solvers: `_lower_triangular_solve`, `_upper_triangular_solve`, `_diagonal_solve`, `_cholesky_solve`, `_LDL_solve`, `solve`, `solve_least_squares`
  - Sparsity analysis: `liupc` (Liu's elimination-tree algorithm), `row_structure_symbolic_cholesky`
  - Properties: `is_hermitian`, `is_symmetric`, `has`, `nnz`, `extract`
  - `_eval_inverse` (CH/LDL methods), `applyfunc`, `reshape`
- `class MutableSparseMatrix(SparseMatrix, MatrixBase)` — mutable variant:
  - `__setitem__`, `row_del`, `col_del`, `row_swap`, `col_swap`
  - `row_join`, `col_join`, `copyin_list`, `copyin_matrix`
  - In-place ops: `row_op`, `col_op`, `zip_row_op`, `fill`

### `sparsetools.py`
Conversion between sparse storage formats.

- `_doktocsr(dok)` — DOK `SparseMatrix` → CSR representation `[A, JA, IA, shape]`
- `_csrtodok(csr)` — CSR → DOK `SparseMatrix`

---

## Immutable Matrices

### `immutable.py`
Hashable, expression-compatible matrix types.

- `class ImmutableMatrix(MatrixExpr, DenseMatrix)` — immutable dense matrix. Stored as a `Tuple` of elements inside a `Basic` node. Supports `_eval_Eq` for `Eq()` comparisons. Delegates most operations to `DenseMatrix`.
- `class ImmutableSparseMatrix(Basic, SparseMatrix)` — immutable sparse matrix. Wraps a `Dict` of entries.
- `def sympify_matrix(arg)` — registered converter so `sympify()` auto-converts mutable matrices to immutable.

**Caveat**: `ImmutableMatrix` inherits from both `MatrixExpr` and `DenseMatrix`, meaning it lives in the symbolic expression tree while also supporting concrete element access.

---

## Matrix Expressions (`expressions/`)

Symbolic (lazy) representation of matrix operations — no concrete elements until explicitly materialised.

### `matexpr.py`
Core expression infrastructure.

- `class MatrixExpr(Basic)` — abstract superclass for all symbolic matrix expressions. Provides:
  - Arithmetic operators, `shape`, `rows`, `cols`, `is_square`
  - `transpose`, `conjugate`, `adjoint`, `inverse` (→ `Inverse`)
  - `__getitem__` → `MatrixElement`
  - `as_explicit()` — materialises into `ImmutableMatrix`
  - `as_mutable()` — materialises into mutable `Matrix`
  - `as_coeff_mmul`
- `class MatrixElement(Expr)` — symbolic `M[i, j]` element access.
- `class MatrixSymbol(MatrixExpr)` — named symbolic matrix (e.g. `MatrixSymbol('A', 3, 3)`).
- `class Identity(MatrixExpr)` — symbolic identity matrix.
- `class ZeroMatrix(MatrixExpr)` — symbolic zero matrix.
- `def matrix_symbols(expr)` — extract all `MatrixSymbol` instances from an expression.

### `matadd.py`
- `class MatAdd(MatrixExpr)` — symbolic sum of matrix expressions.
- `def merge_explicit(matadd)` — fold adjacent explicit `MatrixBase` arguments.
- Canonicalisation rules: remove zeros, flatten, combine like terms, sort.

### `matmul.py`
- `class MatMul(MatrixExpr)` — symbolic product of matrix expressions.
- `def merge_explicit(matmul)` — fold adjacent explicit matrices.
- Canonicalisation rules: `any_zeros`, `remove_ids`, `xxinv` (X·X⁻¹ → I), `factor_in_front`, flatten.
- `def only_squares(*matrices)` — partition a chain into maximal square sub-products.
- `def refine_MatMul` — simplify under assumptions (e.g. orthogonal ⇒ X·Xᵀ = I).

### `matpow.py`
- `class MatPow(MatrixExpr)` — symbolic matrix power `base ** exp`. Handles identity/zero/explicit cases in `doit`.

### `inverse.py`
- `class Inverse(MatPow)` — symbolic matrix inverse (`exp = -1`). `doit` calls `arg.inverse()`.
- `def refine_Inverse` — simplify under assumptions (orthogonal ⇒ transpose, unitary ⇒ conjugate).

### `transpose.py`
- `class Transpose(MatrixExpr)` — symbolic transpose.
- `def transpose(expr)` — convenience function.
- `def refine_Transpose` — symmetric ⇒ identity.

### `adjoint.py`
- `class Adjoint(MatrixExpr)` — symbolic Hermitian adjoint (conjugate transpose).

### `trace.py`
- `class Trace(Expr)` — symbolic matrix trace. Can rewrite as `Sum`.
- `def trace(expr)` — convenience function.

### `determinant.py`
- `class Determinant(Expr)` — symbolic matrix determinant.
- `def det(matexpr)` — convenience function.
- `def refine_Determinant` — orthogonal ⇒ 1, singular ⇒ 0, unit-triangular ⇒ 1.

### `blockmatrix.py`
Block-structured matrix expressions.

- `class BlockMatrix(MatrixExpr)` — matrix composed of sub-matrices arranged in a grid.
  - `blockshape`, `rowblocksizes`, `colblocksizes`
  - `_blockmul`, `_blockadd`, `transpose`, `_eval_trace`, `_eval_determinant`
  - `structurally_equal`, `is_structurally_symmetric`
- `class BlockDiagMatrix(BlockMatrix)` — block-diagonal specialisation with efficient `_eval_inverse`.
- `def block_collapse(expr)` — recursively simplify block-matrix expressions.
- Rewrite rules: `bc_unpack`, `bc_matadd`, `bc_block_plus_ident`, `bc_dist`, `bc_matmul`, `bc_transpose`, `bc_inverse`
- `def blockinverse_1x1`, `def blockinverse_2x2` — analytic block inverses.
- `def deblock(B)` — flatten nested block matrices.
- `def reblock_2x2(B)` — re-partition into 2×2 block structure.
- `def blockcut(expr, rowsizes, colsizes)` — cut a matrix expression into a `BlockMatrix`.

### `slice.py`
- `class MatrixSlice(MatrixExpr)` — symbolic slice of a matrix expression (start, stop, step).
- `def normalize(i, parentsize)` — normalise slice indices.
- `def mat_slice_of_slice` — collapse nested slices.

### `diagonal.py`
- `class DiagonalMatrix(MatrixExpr)` — wraps a column vector as a diagonal matrix.
- `class DiagonalOf(MatrixExpr)` — extracts the diagonal of a square matrix as a column vector.

### `dotproduct.py`
- `class DotProduct(MatrixExpr)` — symbolic dot product of two vector matrices. `doit` delegates to transpose + multiply.

### `hadamard.py`
- `class HadamardProduct(MatrixExpr)` — symbolic elementwise (Hadamard) product.
- `def hadamard_product(*matrices)` — convenience constructor with shape validation.

### `funcmatrix.py`
- `class FunctionMatrix(MatrixExpr)` — matrix defined by a `Lambda(i, j)` function. Entries evaluated lazily.

### `fourier.py`
- `class DFT(MatrixExpr)` — Discrete Fourier Transform matrix. Entry `(i,j) = ω^(ij) / √n`.
- `class IDFT(DFT)` — Inverse DFT. Mutually inverse with `DFT`.

### `factorizations.py`
Symbolic placeholders for matrix factorisations (used by assumption/query system).

- `class Factorization(MatrixExpr)` — base class.
- Subclasses: `LofLU`, `UofLU`, `LofCholesky`, `UofCholesky`, `QofQR`, `RofQR`, `EigenVectors`, `EigenValues`, `UofSVD`, `SofSVD`, `VofSVD`
- Factory functions: `lu`, `qr`, `eig`, `svd`

### `expressions/__init__.py`
Re-exports all expression classes and convenience functions.

---

## Benchmarks

### `benchmarks/bench_matrix.py`
ASV-style benchmark functions for matrix operations: element access (`M[3,3]`, `M[i3,i3]`), full-matrix slice (`M[:,:]`), and `zeros(100, 100)` construction.

---

## Appendix

### Inheritance Hierarchy (simplified)

```
MatrixBase (matrices.py)
├── DenseMatrix (dense.py)
│   ├── MutableDenseMatrix  (= Matrix)
│   └── ImmutableMatrix     (also inherits MatrixExpr)
└── SparseMatrix (sparse.py)
    ├── MutableSparseMatrix (= SparseMatrix alias)
    └── ImmutableSparseMatrix

MatrixExpr (expressions/matexpr.py)
├── Identity, ZeroMatrix, MatrixSymbol
├── MatAdd, MatMul, MatPow, Inverse
├── Transpose, Adjoint, Trace, Determinant
├── BlockMatrix, BlockDiagMatrix
├── FunctionMatrix, MatrixSlice
├── HadamardProduct, DotProduct
├── DiagonalMatrix, DiagonalOf
├── DFT, IDFT
├── Factorization subclasses
└── ImmutableMatrix (also inherits DenseMatrix)
```

### Dual Algebra Layers

The module provides two independent algebra layers:
1. **High-level** (`MatrixBase` hierarchy) — symbolic element types, integrates with SymPy core.
2. **Low-level** (`densearith`, `densesolve`, `densetools`) — operates on raw `list[list]` with explicit domain rings (`ZZ`, `QQ`). Useful for exact arithmetic in polynomial/number-theory contexts.
