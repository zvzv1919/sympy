# matrices — Module Catalog

## Architecture Overview

The `matrices` module has two layers:
- **Concrete matrices** (`matrices.py`, `dense.py`, `sparse.py`): store actual numerical/symbolic entries and perform direct computation (solving, decomposition, eigenvalues, nullspace).
- **Matrix expressions** (`expressions/`): unevaluated symbolic representations of matrix operations; converted to concrete form via `as_explicit`.
- **Low-level backends** (`densesolve.py`, `densearith.py`, `densetools.py`): operate on raw Python list-of-lists, called internally by concrete matrix classes.

---

## Concrete Matrix Core

### [`matrices.py`](matrices.py)
Central base class `MatrixBase` — defines the full matrix API inherited by both dense and sparse types.

- `MatrixBase`: base for all concrete matrix types; not instantiated directly.
- **Arithmetic**: `__add__`, `__mul__`, `__pow__`, `multiply`, `add`.
- **Row reduction**: `rref` (reduced row echelon form with pivot tracking), `rank`.
- **Null/column space**: `nullspace` (kernel basis via rref; handles pivot vs free variable classification, errors on unexpected pivot-column entries), `columnspace`.
- **Eigenvalue analysis**: `eigenvals`, `eigenvects`, `left_eigenvects`, `berkowitz_eigenvals`, `berkowitz`.
- **Diagonalization**: `is_diagonalizable` (checks P·D·P⁻¹ decomposability; supports `reals_only` flag to reject complex eigenvalues), `diagonalize`, `jordan_form`, `jordan_cells`.
- **Decompositions**: `cholesky`, `LDLdecomposition`, `QRdecomposition`, `LUdecomposition`.
- **Solvers**: `solve`, `LUsolve`, `QRsolve`, `cholesky_solve`, `gauss_jordan_solve`, `solve_least_squares`, `pinv`.
- **Determinant/inverse**: `det`, `inv`, `adjugate`, `cofactor`, `cofactorMatrix`.
- **Structure**: `row_join`, `col_join`, `row_insert`, `col_insert`, `extract`, `reshape`.
- **Predicates**: `is_square`, `is_symmetric`, `is_hermitian`, `is_diagonal`, `is_upper`, `is_lower`, `is_zero`, `is_symbolic`.
- `MatrixError`, `ShapeError`, `NonSquareMatrixError`: exception hierarchy.

### [`dense.py`](dense.py)
Dense matrix implementation — stores elements in a flat Python list (`_mat`).

- `DenseMatrix`: concrete dense storage; element access, `tolist`, `row`, `col`, `applyfunc`, `reshape`.
- Internal solver backends: `_cholesky`, `_LDLdecomposition`, `_lower_triangular_solve`, `_upper_triangular_solve`.
- `MutableDenseMatrix`: mutable variant with in-place mutation — `row_swap`, `col_swap`, `row_del`, `col_del`, `row_op`, `col_op`, `__setitem__`, `copyin_matrix`, `fill`.
- Factory methods: `zeros`, `eye`, `ones`, `diag`.

### [`sparse.py`](sparse.py)
Sparse matrix implementation — stores entries in a dictionary-of-keys (`_smat`) mapping `(row, col)` to value.

- `SparseMatrix`: DOK-based sparse storage; `row_list`, `col_list`, `nnz`.
- `row_join`: horizontal concatenation (`[A B]`); handles both sparse (dict iteration) and dense (flat-list `_mat` iteration) operands.
- `col_join`: vertical concatenation (`[A; B]`); similarly handles mixed sparse/dense operands.
- `MutableSparseMatrix`: mutable variant with in-place `row_swap`, `col_swap`, `row_del`, `col_del`, `row_op`, `col_op`.
- Internal solver backends: `_cholesky_sparse`, `_LDL_sparse`, `_lower_triangular_solve_sparse`, `_upper_triangular_solve_sparse`.

### [`immutable.py`](immutable.py)
Hashable, immutable matrix types usable as dictionary keys and in SymPy expression trees.

- `ImmutableMatrix` (`ImmutableDenseMatrix`): combines `MatrixExpr` and `DenseMatrix`; hashable concrete dense matrix.
- `ImmutableSparseMatrix`: hashable concrete sparse matrix.

---

## Matrix Expressions (Symbolic / Unevaluated)

### [`expressions/matexpr.py`](expressions/matexpr.py)
Base class for all symbolic (unevaluated) matrix expressions.

- `MatrixExpr`: abstract symbolic matrix type; defines shape, arithmetic operators, and conversion.
- **Operator dispatch**: `__pow__` handles special exponents (0→Identity, 1→self, -1→Inverse, non-square→ShapeError) before delegating to `MatPow`.
- **Conversion to concrete form**: `as_explicit` iterates all (i,j) entries and returns an `ImmutableMatrix`; `as_mutable` converts further to mutable dense.
- Properties: `shape`, `rows`, `cols`, `is_square`, `T` (transpose).
- `MatrixElement`: represents a single symbolic entry M[i,j] as an `Expr` node.

### [`expressions/matpow.py`](expressions/matpow.py)
Unevaluated matrix power node `MatPow(base, exp)`.

- `MatPow`: symbolic representation of M^n; `doit()` evaluates when possible.
- Properties: `base`, `exp`, `shape`.
- Note: special-case exponent handling (0, 1, -1) lives in `MatrixExpr.__pow__`, not here.

### [`expressions/blockmatrix.py`](expressions/blockmatrix.py)
Block-structured symbolic matrices.

- `BlockMatrix`: matrix composed of a 2D grid of sub-matrices; `blocks`, `blockshape`, `rowblocksizes`, `colblocksizes`.
- `BlockDiagMatrix`: block-diagonal specialization.
- Block arithmetic rules: `bc_matmul`, `bc_block_plus_ident`, `bc_dist`, `bc_transpose`.

### [`expressions/inverse.py`](expressions/inverse.py)
- `Inverse`: unevaluated symbolic matrix inverse M⁻¹.

### [`expressions/transpose.py`](expressions/transpose.py)
- `Transpose`: unevaluated symbolic transpose Mᵀ.

### [`expressions/adjoint.py`](expressions/adjoint.py)
- `Adjoint`: unevaluated conjugate transpose M*.

### [`expressions/matmul.py`](expressions/matmul.py)
- `MatMul`: unevaluated symbolic matrix product A·B·C…; `doit()` evaluates.

### [`expressions/matadd.py`](expressions/matadd.py)
- `MatAdd`: unevaluated symbolic matrix sum A+B+C…; `doit()` evaluates.

### [`expressions/determinant.py`](expressions/determinant.py)
- `Determinant`: unevaluated symbolic determinant det(M).

### [`expressions/trace.py`](expressions/trace.py)
- `Trace`: unevaluated symbolic trace tr(M).

### [`expressions/diagonal.py`](expressions/diagonal.py)
- `DiagonalMatrix`, `DiagMatrix`: symbolic diagonal matrix expressions.

### [`expressions/dotproduct.py`](expressions/dotproduct.py)
- `DotProduct`: symbolic dot product of two vectors.

### [`expressions/hadamard.py`](expressions/hadamard.py)
- `HadamardProduct`: element-wise (Hadamard) matrix product.

### [`expressions/funcmatrix.py`](expressions/funcmatrix.py)
- `FunctionMatrix`: matrix defined by a lambda `f(i,j)` for each entry.

### [`expressions/fourier.py`](expressions/fourier.py)
- `DFT`: discrete Fourier transform matrix.

### [`expressions/factorizations.py`](expressions/factorizations.py)
- Symbolic matrix factorization nodes: `LofLU`, `UofLU`, `LofCholesky`, `UofCholesky`.

### [`expressions/slice.py`](expressions/slice.py)
- `MatrixSlice`: symbolic submatrix slice expression M[i:j, k:l].

---

## Low-Level Backends

### [`densesolve.py`](densesolve.py)
Low-level solvers operating on raw list-of-lists (not matrix objects).

- `row_echelon`, `rref`: row reduction on raw nested lists.
- `rref_solve`, `LU_solve`, `cholesky_solve`: solver routines on raw data.
- These are internal backends; the public API lives in `MatrixBase` (`matrices.py`).

### [`densearith.py`](densearith.py)
Low-level arithmetic on list-of-lists: `add`, `sub`, `negate`, `mulmatmat`, `mulmatscaler`.

### [`densetools.py`](densetools.py)
Low-level matrix utilities on list-of-lists: `trace`, `transpose`, `conjugate`, `eye`, `augment`, `rowadd`, `rowmul`.

### [`sparsetools.py`](sparsetools.py)
Sparse format conversion utilities: `_doktocsr` (DOK→CSR), `_csrtodok` (CSR→DOK).
