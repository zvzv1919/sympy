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
- **Arithmetic**: `__add__`, `__mul__` (returns `NotImplemented` when the right operand is matrix-like but its transpose lacks `tolist`, e.g. MatrixSymbol), `__pow__` (integer exponents: square-and-multiply; symbolic/float exponents: Jordan decomposition of each cell), `multiply`, `add`.
- `exp`: matrix exponential via Jordan decomposition; catches `MatrixError` from `jordan_cells` and raises `NotImplementedError`.
- **Row reduction**: `rref` (reduced row echelon form with pivot tracking), `rank`.
- **Null/column space**: `nullspace` (kernel basis via rref; handles pivot vs free variable classification, errors on unexpected pivot-column entries), `columnspace`.
- **Eigenvalue analysis**: `eigenvals`, `eigenvects`, `left_eigenvects`, `berkowitz_eigenvals`, `berkowitz`.
- **Diagonalization**: `is_diagonalizable` (checks P·D·P⁻¹ decomposability; supports `reals_only` flag to reject complex eigenvalues), `diagonalize`, `jordan_form`, `jordan_cells`.
- **Decompositions**: `cholesky`, `LDLdecomposition`, `QRdecomposition`, `LUdecomposition`, `LUdecompositionFF` (fraction-free LU returning P, L, D, U).
- **Solvers**: `solve`, `LUsolve`, `QRsolve`, `cholesky_solve`, `gauss_jordan_solve`, `solve_least_squares`.
- `pinv`: Moore-Penrose pseudoinverse; raises `NotImplementedError` for rank-deficient matrices (catches `ValueError` from `inv()`). `pinv_solve`: least-squares solver via pseudoinverse.
- **Determinant/inverse**: `det`, `det_bareis` (Bareiss fraction-free Gaussian elimination for determinant; divides by previous pivot to avoid fractions; swaps rows when current pivot is zero), `det_LU_decomposition`, `berkowitz_det`, `inv`, `adjugate`, `cofactor`, `cofactorMatrix`.
- **Inversion strategies** (concrete implementations): `inverse_ADJ` (adjugate/determinant method; falls back to rref diagonal check when `equals(0)` is indeterminate), `inverse_LU`, `inverse_GE`.
- **Norms**: `norm` (vector and matrix norms — Frobenius, spectral, p-norms; for 2D matrices with default ord, reshapes to column vector via `vec()` and recurses).
- **Structure**: `row_join` (horizontal concat; null/empty self → returns rhs), `col_join` (vertical concat; null/empty self → returns bott), `row_insert`, `col_insert`, `extract`, `reshape`.
- **Indexing helpers**: `key2bounds` (converts mixed int/slice keys to row/col boundaries; handles zero-dimension edge case), `key2ij`.
- `_setitem`: item-assignment logic shared by all mutable subclasses; for integer keys with a plain sequence value, auto-wraps into a dense `Matrix` then delegates to `copyin_matrix`. Slice keys delegate to `copyin_matrix`/`copyin_list` directly.
- **Predicates (shape)**: `is_square`, `is_diagonal`, `is_upper`, `is_lower`, `is_upper_hessenberg` (zero below first subdiagonal), `is_lower_hessenberg` (zero above first superdiagonal), `is_zero`, `is_symbolic`.
- **Display**: `print_nonzero` (text-based sparsity visualization — prints configurable symbol at non-zero entry positions, space at zeros).
- **Predicates (symmetry)**: `is_symmetric` (simplifies entries before comparison to avoid false negatives on algebraically equivalent expressions).
- `is_hermitian`: checks self-adjoint property (equality to conjugate transpose) via fuzzy three-valued logic; returns None when symbolic entries make result indeterminate.
- `MatrixError`, `ShapeError`, `NonSquareMatrixError`: exception hierarchy.

### [`dense.py`](dense.py)
Dense matrix implementation — stores elements in a flat Python list (`_mat`).

- `DenseMatrix`: concrete dense storage; element access, `tolist`, `row`, `col`, `applyfunc`, `reshape`.
- `as_immutable`: converts to `ImmutableMatrix`; special-cases zero-row or zero-col matrices (passes shape+empty list instead of `tolist`). `as_mutable`: converts to mutable `Matrix`.
- `equals`: element-wise symbolic equivalence check using three-valued logic — returns True if all pairs proven equal, False if any pair provably unequal, None if indeterminate.
- `_eval_inverse`: dense matrix inversion dispatching to GE/LU/ADJ methods; supports `try_block_diag` flag to decompose into independent diagonal blocks via `get_diag_blocks()`, invert each block separately, and reassemble.
- Internal solver backends: `_cholesky`, `_LDLdecomposition`, `_lower_triangular_solve` (forward substitution for lower-triangular systems), `_upper_triangular_solve` (backward substitution for upper-triangular systems), `_diagonal_solve`.
- `MutableDenseMatrix`: mutable variant with in-place mutation — `row_swap`, `col_swap`, `row_del`, `col_del`, `row_op`, `col_op`, `copyin_matrix`, `fill`.
- `__setitem__`: thin wrapper; actual assignment logic (including list→Matrix conversion) is `MatrixBase._setitem` in `matrices.py`.
- `_force_mutable(x)`: operand coercion helper used by all mutable arithmetic operators; converts matrices to mutable, sympifies 0-d numpy arrays (scalars), wraps other array-like objects as `Matrix`.
- `matrix_multiply_elementwise(A, B)`: concrete Hadamard (element-wise) product; raises `ShapeError` on dimension mismatch.
- Factory methods: `zeros`, `eye`, `ones`, `rot_axis1`, `rot_axis2`, `rot_axis3`.
- `diag(*values)`: builds a concrete block-diagonal matrix from a mix of scalars, plain lists, and existing Matrix objects; auto-converts lists to Matrix, accumulates total rows/cols from each block, places blocks along the diagonal of a sparse intermediate, then converts to the target class.

### [`sparse.py`](sparse.py)
Sparse matrix implementation — stores entries in a dictionary-of-keys (`_smat`) mapping `(row, col)` to value.

- `SparseMatrix`: DOK-based sparse storage; `row_list`, `col_list`, `nnz`.
- `row_join`: horizontal concatenation (`[A B]`); handles both sparse (dict iteration) and dense (flat-list `_mat` iteration) operands.
- `col_join`: vertical concatenation (`[A; B]`); similarly handles mixed sparse/dense operands.
- `MutableSparseMatrix`: mutable variant with in-place `row_swap`, `col_swap`, `row_del`, `col_del`, `row_op`, `col_op`.
- `row_structure_symbolic_cholesky`: pre-computes non-zero row structure via elimination trees; used by sparse decompositions to skip zero entries.
- Sparse decompositions exploiting pre-computed non-zero pattern: `_cholesky_sparse`, `_LDL_sparse` (unit lower-triangular + diagonal factorization).
- Sparse triangular solvers exploiting sparsity: `_lower_triangular_solve` (forward substitution), `_upper_triangular_solve` (backward substitution; reverses each row's entries to process columns right-to-left), `_diagonal_solve`.
- `_eval_inverse`: sparse inversion; symmetrizes non-symmetric matrices via M^T·M before LDL/Cholesky solve, then applies scaling correction to the result.
- Sparse composite solvers: `_cholesky_solve` (Cholesky factorization + triangular solves), `_LDL_solve` (L·D·L^T factorization then forward substitution → diagonal solve → backward substitution).

### [`immutable.py`](immutable.py)
Hashable, immutable matrix types usable as dictionary keys and in SymPy expression trees.

- `ImmutableMatrix` (`ImmutableDenseMatrix`): combines `MatrixExpr` and `DenseMatrix`; hashable concrete dense matrix.
- `ImmutableSparseMatrix`: hashable concrete sparse matrix.

---

## Matrix Expressions (Symbolic / Unevaluated)

### [`expressions/matexpr.py`](expressions/matexpr.py)
Base class for all symbolic (unevaluated) matrix expressions.

- `_sympifyit`: module-level decorator wrapping matrix arithmetic operators; always attempts strict `sympify` on the second operand (unlike core `_sympifyit` which skips objects with `_op_priority`); returns `retval` on `SympifyError`.
- `MatrixExpr`: abstract symbolic matrix type; defines shape, arithmetic operators, and conversion.
- **Operator dispatch**: `__pow__` handles special exponents (0→Identity, 1→self, -1→Inverse, non-square→ShapeError) before delegating to `MatPow`.
- **Conversion to concrete form**: `as_explicit` iterates all (i,j) entries and returns an `ImmutableMatrix`; `as_mutable` converts further to mutable dense.
- Properties: `shape`, `rows`, `cols`, `is_square`, `T` (transpose).
- `MatrixElement`: represents a single symbolic entry M[i,j] as an `Expr` node.
- `Identity(n)`: symbolic n×n identity matrix (square, `is_Identity`); `_eval_inverse` returns self.
- `ZeroMatrix(m, n)`: symbolic m×n zero matrix — additive identity (`is_ZeroMatrix`).
- `ZeroMatrix.__pow__`: own exponent dispatch — 0→Identity, ≥1→self, <1→`ValueError` (det==0; not invertible), non-square with exp≠1→`ShapeError`.

### [`expressions/matpow.py`](expressions/matpow.py)
Unevaluated matrix power node `MatPow(base, exp)`.

- `MatPow`: unevaluated symbolic node for M^n; `doit()` delegates to concrete `__pow__` (actual numeric/Jordan computation lives in `matrices.py`).
- Properties: `base`, `exp`, `shape`.
- Note: special-case exponent handling (0, 1, -1) lives in `MatrixExpr.__pow__`, not here.

### [`expressions/blockmatrix.py`](expressions/blockmatrix.py)
Block-structured symbolic matrices.

- `BlockMatrix`: matrix composed of a 2D grid of sub-matrices; `blocks`, `blockshape`, `rowblocksizes`, `colblocksizes`.
- `BlockMatrix._entry`: resolves element access by locating the correct sub-block; uses `!= False` comparisons to handle symbolic (non-concrete) row/column indices.
- `BlockDiagMatrix`: unevaluated symbolic block-diagonal expression (not a concrete constructor); `_eval_inverse` inverts each diagonal block independently.
- Block arithmetic rules: `bc_matmul`, `bc_block_plus_ident`, `bc_dist`, `bc_transpose`.

### [`expressions/inverse.py`](expressions/inverse.py)
- `Inverse`: unevaluated symbolic matrix inverse M⁻¹.

### [`expressions/transpose.py`](expressions/transpose.py)
- `Transpose`: unevaluated symbolic transpose Mᵀ.

### [`expressions/adjoint.py`](expressions/adjoint.py)
- `Adjoint`: unevaluated symbolic expression node for conjugate transpose M*; represents the operation lazily, does not verify self-adjoint properties.

### [`expressions/matmul.py`](expressions/matmul.py)
- `MatMul`: unevaluated symbolic matrix product A·B·C…; `doit()` evaluates via `canonicalize`.
- `_eval_inverse`: reverses factor order and inverts each; falls back to `Inverse(self)` on ShapeError (incompatible dimensions).
- `_eval_transpose`, `_eval_adjoint`: reverse factor order and apply transpose/adjoint to each factor.
- `_eval_determinant`: extracts scalar coefficient and delegates to `Determinant` per square sub-factor.
- `as_coeff_matrices`: splits args into scalar coefficient and matrix factors.
- `validate`: checks adjacent factor dimension compatibility.
- **Simplification rules** (module-level functions applied during canonicalization):
  - `xxinv`: cancels adjacent matrix-inverse pairs X·X⁻¹→Identity; catches `ValueError` if a factor is not invertible.
  - `remove_ids`: strips Identity factors. `any_zeros`: collapses product to ZeroMatrix if any factor is zero.
  - `merge_explicit`: multiplies adjacent concrete `MatrixBase` factors. `factor_in_front`: moves scalar coefficient to front.

### [`expressions/matadd.py`](expressions/matadd.py)
- `MatAdd`: unevaluated symbolic matrix sum A+B+C…; `doit()` evaluates.

### [`expressions/determinant.py`](expressions/determinant.py)
- `Determinant`: unevaluated symbolic determinant det(M); `doit()` delegates to `_eval_determinant`.
- `det(matexpr)`: convenience function returning `Determinant(matexpr).doit()`.
- `refine_Determinant`: assumption-based simplification — returns 1 for orthogonal/unit-triangular, 0 for singular matrices.

### [`expressions/trace.py`](expressions/trace.py)
- `Trace`: unevaluated symbolic trace tr(M).

### [`expressions/diagonal.py`](expressions/diagonal.py)
- `DiagonalMatrix`, `DiagMatrix`: symbolic diagonal matrix expressions.

### [`expressions/dotproduct.py`](expressions/dotproduct.py)
- `DotProduct`: symbolic dot product of two vectors.

### [`expressions/hadamard.py`](expressions/hadamard.py)
- `HadamardProduct`: unevaluated symbolic element-wise matrix product (concrete version is `matrix_multiply_elementwise` in `dense.py`).

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
- `LU`, `cholesky`, `LDL` (L·D·Lᵀ factorization for hermitian matrices; rational entries only): decomposition routines on raw nested lists.
- `rref_solve`, `LU_solve`, `cholesky_solve` (symmetric positive-definite solve via Cholesky factorization into L and L* then two-pass substitution): solver routines on raw nested-list data.
- `forward_substitution`: lower-triangular solve on raw nested lists. `backward_substitution`: upper-triangular solve on raw nested lists.
- These are internal backends; the public API lives in `MatrixBase` (`matrices.py`).

### [`densearith.py`](densearith.py)
Low-level arithmetic on raw nested lists (list-of-lists representation, not matrix objects).

- `add`, `sub`, `negate`: element-wise addition, subtraction, and negation on nested lists.
- `mulmatmat`: matrix-matrix product — transposes the second operand (rows→columns via `zip`) then computes row-column dot products.
- `mulmatscaler`: scalar-matrix product on nested lists.

### [`densetools.py`](densetools.py)
Low-level matrix utilities on list-of-lists: `trace`, `transpose`, `conjugate`, `conjugate_row` (element-wise conjugation with AttributeError fallback for entries lacking `.conjugate()`), `conjugate_transpose`, `eye`, `augment`, `rowadd`, `rowmul`.

### [`sparsetools.py`](sparsetools.py)
Sparse format conversion utilities: `_doktocsr` (DOK→CSR), `_csrtodok` (CSR→DOK).
