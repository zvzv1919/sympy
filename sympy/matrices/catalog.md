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
- **Arithmetic**: `__add__`, `__mul__`, `__pow__` (integer: square-and-multiply; symbolic/float: Jordan), `multiply`, `add`, `exp` (matrix exponential via Jordan).
- **Dot / element-wise products**: `dot` (relaxed-dimension inner product — auto-transposes when row/column counts match the length of b; returns scalar for vectors, list otherwise), `multiply_elementwise`, `cross`.
- **Row reduction / spaces**: `rref`, `rank`, `nullspace`, `columnspace`.
- **Eigenvalue analysis**: `eigenvals`, `eigenvects`, `left_eigenvects`, `berkowitz_eigenvals`, `berkowitz`.
- **Diagonalization**: `is_diagonalizable`, `diagonalize`, `jordan_form`, `jordan_cells`.
- **Decompositions**: `cholesky`, `LDLdecomposition`, `QRdecomposition` (orthogonal-triangular via Gram-Schmidt; validates column rank via rref before factoring), `LUdecomposition`.
- `LUdecompositionFF`: fraction-free LU returning PA=LD⁻¹U; keeps all entries in the original integral domain by dividing each update by the previous pivot.
- **Solvers**: `solve`, `LUsolve`, `QRsolve`, `LDLsolve` (symmetric→direct LDL; overdetermined rows≥cols→normal equations A^T·A before decomposing; underdetermined→raises), `cholesky_solve`, `gauss_jordan_solve`, `solve_least_squares`, `pinv`, `pinv_solve`.
- **Determinant/inverse**: `det` (returns `S.One` for empty 0×0 matrix), `det_bareis`, `det_LU_decomposition`, `berkowitz_det`, `inv`, `adjugate`, `cofactor`, `cofactorMatrix`.
- **Inversion strategies**: `inverse_ADJ`, `inverse_LU`, `inverse_GE`.
- **Norms**: `norm` (Frobenius, spectral, p-norms).
- **Block structure**: `get_diag_blocks` — decomposes a concrete square matrix into independent square sub-matrices along the main diagonal by verifying off-block regions are zero (recursive expansion).
- **Structure / indexing**: `row_join`, `col_join`, `row_insert`, `col_insert`, `extract`, `reshape`, `key2bounds`, `key2ij`, `_setitem`.
- **Element-wise symbolic operations**: `subs`, `xreplace`, `expand`, `simplify` — each delegates to `applyfunc`, applying the operation to every entry.
- **Dynamic calculus dispatch** (`__getattr__`): lookups for `diff`, `integrate`, `limit` are intercepted and return a function that applies the operation element-wise via `applyfunc`.
- **Predicates**: `is_square`, `is_diagonal`, `is_upper`, `is_lower`, `is_hermitian`, `is_zero` (three-valued), `is_nilpotent` (characteristic polynomial = x^n via `charpoly`).
- `is_symmetric`: computes self−transpose, simplifies, checks zero; `simplify=False` skips reduction → may yield false negatives. `is_anti_symmetric`: similar simplify flag.
- **Construction**: `_handle_creation_inputs` — normalizes all constructor forms (nested list, flat list+dims, callable, NumPy array, MatrixBase) into (rows, cols, flat_list).
  - Validates uniform row lengths; skips 0×0 sub-matrices when tracking column widths.
- **Display**: `print_nonzero` (marks non-zero entries), `table` (tabular string with alignment), `_format_str` (str representation; embeds explicit dimensions for zero-row/zero-col matrices).
- `MatrixError`, `ShapeError`, `NonSquareMatrixError`: exception hierarchy.

### [`dense.py`](dense.py)
Dense matrix implementation — stores elements in a flat Python list (`_mat`).

- `DenseMatrix`: concrete dense storage; element access, `tolist`, `row`, `col`, `applyfunc`, `reshape`.
- `as_immutable`: converts to `ImmutableMatrix`; special-cases zero-row or zero-col matrices (passes shape+empty list instead of `tolist`). `as_mutable`: converts to mutable `Matrix`.
- `equals`: element-wise symbolic equivalence check using three-valued logic — returns True if all pairs proven equal, False if any pair provably unequal, None if indeterminate.
- `_eval_inverse`: dense matrix inversion dispatching to GE/LU/ADJ methods; supports `try_block_diag` flag to decompose into independent diagonal blocks via `get_diag_blocks()`, invert each block separately, and reassemble.
- Internal solver backends: `_cholesky`, `_LDLdecomposition`, `_lower_triangular_solve` (forward substitution for lower-triangular systems), `_upper_triangular_solve` (backward substitution for upper-triangular systems), `_diagonal_solve`.
- `MutableDenseMatrix`: mutable variant with in-place mutation — `row_swap`, `col_swap`, `row_del`, `col_del`, `row_op`, `col_op`, `fill`.
- `copyin_matrix(key, value)`: copies a Matrix into the sub-region defined by `key`; raises `ShapeError` if source matrix dimensions don't match target slice dimensions.
- `copyin_list(key, value)`: copies elements from an iterable into the sub-region defined by `key`; raises `TypeError` if `value` is not an ordered iterable (e.g. a plain scalar).
- `__setitem__`: thin wrapper; actual assignment logic (including list→Matrix conversion) is `MatrixBase._setitem` in `matrices.py`.
- `_force_mutable(x)`: operand coercion helper used by all mutable arithmetic operators; converts matrices to mutable, sympifies 0-d numpy arrays (scalars), wraps other array-like objects as `Matrix`.
- `matrix_multiply_elementwise(A, B)`: concrete Hadamard (element-wise) product; raises `ShapeError` on dimension mismatch.
- Factory methods: `zeros`, `eye`, `ones`, `rot_axis1`, `rot_axis2`, `rot_axis3`.
- `hessian(f, varlist, constraints=[])`: (bordered) Hessian of `f` wrt `varlist`; optionally bordered by constraint gradients.
  - `varlist` accepts a sequence or row/column matrix (columns transposed; non-vector matrices raise `ShapeError`). Validates differentiability.
- `diag(*values)`: builds a concrete block-diagonal matrix from a mix of scalars, plain lists, and existing Matrix objects; auto-converts lists to Matrix, accumulates total rows/cols from each block, places blocks along the diagonal of a sparse intermediate, then converts to the target class.
- `wronskian(functions, var, method)`: computes the Wronskian determinant (derivatives matrix det) for testing linear independence of differential functions; returns 1 for an empty input list.
- `casoratian(seqs, n)`: computes the Casoratian determinant for testing linear independence of sequences (used in recurrence solving).
- `randMatrix(r, c, ...)`: generates a random matrix with optional symmetry and sparsity control.

### [`sparse.py`](sparse.py)
Sparse matrix implementation — stores entries in a dictionary-of-keys (`_smat`) mapping `(row, col)` to value.

- `SparseMatrix`: DOK-based sparse storage; `row_list`, `col_list`, `nnz`.
- `row_join`: horizontal concatenation (`[A B]`); handles both sparse (dict iteration) and dense (flat-list `_mat` iteration) operands.
- `col_join`: vertical concatenation (`[A; B]`); similarly handles mixed sparse/dense operands.
- `MutableSparseMatrix`: mutable variant with in-place `row_swap`, `col_swap`, `row_del`, `col_del`, `row_op`, `col_op`.
- `row_structure_symbolic_cholesky`: pre-computes non-zero row structure via elimination trees; used by sparse decompositions to skip zero entries.
- Sparse decompositions exploiting pre-computed non-zero pattern: `_cholesky_sparse`, `_LDL_sparse` (unit lower-triangular + diagonal factorization).
- Sparse triangular solvers exploiting sparsity: `_lower_triangular_solve` (forward substitution), `_upper_triangular_solve` (backward substitution; reverses each row's entries to process columns right-to-left), `_diagonal_solve`.
- `_eval_inverse`: sparse inversion dispatch (internal to sparse storage layer); symmetrizes via M^T·M when needed.
- Sparse composite solvers: `_cholesky_solve` (Cholesky factorization + triangular solves), `_LDL_solve` (L·D·L^T factorization then forward substitution → diagonal solve → backward substitution).

### [`immutable.py`](immutable.py)
Hashable, immutable matrix types usable as dictionary keys and in SymPy expression trees.

- `ImmutableMatrix` (`ImmutableDenseMatrix`): combines `MatrixExpr` and `DenseMatrix`; hashable concrete dense matrix.
- `_eval_Eq`: three-way equality — returns False for shape mismatch, None (defer to default `Eq` handling) when comparing against a non-immutable `MatrixExpr`, otherwise `sympify(diff.is_zero)`.
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
- `MatrixElement`: represents a single symbolic entry M[i,j] as an `Expr` node; `doit(deep=True)` recursively evaluates parent matrix and indices before indexing, `doit(deep=False)` indexes with raw args.
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
- `BlockDiagMatrix`: unevaluated symbolic block-diagonal expression (does NOT detect/extract blocks from concrete matrices — see `get_diag_blocks` in `matrices.py`); `_eval_inverse` inverts each pre-declared diagonal block independently.
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
- `refine_MatMul`: assumption-based simplification of matrix products — reduces X·Xᵀ→Identity when X is orthogonal, X·conjugate(X)→Identity when X is unitary. Registered as `handlers_dict['MatMul']` for the refine system.

### [`expressions/matadd.py`](expressions/matadd.py)
- `MatAdd`: unevaluated symbolic matrix sum A+B+C…; `doit()` evaluates.

### [`expressions/determinant.py`](expressions/determinant.py)
- `Determinant`: unevaluated symbolic determinant det(M); `doit()` delegates to `_eval_determinant`.
- `det(matexpr)`: convenience function returning `Determinant(matexpr).doit()`.
- `refine_Determinant`: assumption-based simplification — returns 1 for orthogonal/unit-triangular, 0 for singular matrices.

### [`expressions/trace.py`](expressions/trace.py)
- `Trace`: unevaluated symbolic trace tr(M); `doit(deep=True)` recursively evaluates arg then delegates to `_eval_trace`; `doit(deep=False)` returns concrete trace for `MatrixBase` instances or unevaluated `Trace` otherwise.
- `trace(expr)`: convenience function returning `Trace(expr).doit()`.

### [`expressions/diagonal.py`](expressions/diagonal.py)
- `DiagonalMatrix`, `DiagMatrix`: symbolic diagonal matrix expressions.

### [`expressions/dotproduct.py`](expressions/dotproduct.py)
- `DotProduct`: symbolic dot product of two vector matrices (1×n or n×1); `doit()` auto-transposes arguments based on row/column orientation before multiplying and extracting the scalar.

### [`expressions/hadamard.py`](expressions/hadamard.py)
- `HadamardProduct`: unevaluated symbolic element-wise matrix product (concrete version is `matrix_multiply_elementwise` in `dense.py`).

### [`expressions/funcmatrix.py`](expressions/funcmatrix.py)
- `FunctionMatrix`: matrix defined by a lambda `f(i,j)` for each entry; has custom `_eval_trace` that delegates to `Trace._eval_rewrite_as_Sum` (symbolic summation over diagonal) rather than iterating entries directly.

### [`expressions/fourier.py`](expressions/fourier.py)
- `DFT`: discrete Fourier transform matrix.

### [`expressions/factorizations.py`](expressions/factorizations.py)
- Symbolic matrix factorization nodes: `LofLU`, `UofLU`, `LofCholesky`, `UofCholesky`.

### [`expressions/slice.py`](expressions/slice.py)
- `MatrixSlice`: symbolic submatrix slice expression M[i:j, k:l]; `__new__` auto-detects when the parent is itself a `MatrixSlice` and delegates to `mat_slice_of_slice` to collapse nesting.
- `mat_slice_of_slice`: collapses nested matrix slices into a single `MatrixSlice` referencing the original parent by composing row/col range triples.
- `normalize`: converts various index forms (int, slice, tuple) into a canonical (start, stop, step) triple with negative-index handling.

---

## Low-Level Backends

### [`densesolve.py`](densesolve.py)
Low-level solvers operating on raw list-of-lists (not matrix objects).

- `row_echelon`: forward elimination on raw nested lists.
- `rref`: reduced row echelon form on raw nested lists; back-substitution phase only eliminates upward from rows whose diagonal is 1, skipping rank-deficient rows.
- `LU`, `cholesky`, `LDL` (L·D·Lᵀ factorization for hermitian matrices; rational entries only): decomposition routines on raw nested lists.
- `rref_solve`, `LU_solve`, `cholesky_solve` (symmetric positive-definite solve via Cholesky factorization into L and L* then two-pass substitution): solver routines on raw nested-list data.
- `forward_substitution`, `backward_substitution`: standalone lower/upper-triangular solvers on raw nested lists; mutate the `variable` list in-place and return it.
- These are internal backends (standalone functions, not methods); the public API lives in `MatrixBase` (`matrices.py`).

### [`densearith.py`](densearith.py)
Low-level arithmetic on raw nested lists (list-of-lists representation, not matrix objects).

- `add`, `addrow`: element-wise addition on nested lists. `sub`: subtraction implemented as `negate` + `add` (not direct element-wise difference). `negate`, `negaterow`: sign inversion.
- `mulmatmat`: matrix-matrix product — transposes the second operand (rows→columns via `zip`) then computes row-column dot products via `mulrowcol`.
- `mulrowcol(row, col, K)`: inner product of a row and column represented as flat lists; columns are flat (not nested single-element lists) for performance.
- `mulmatscaler`: scalar-matrix product on nested lists.

### [`densetools.py`](densetools.py)
Low-level matrix utilities on list-of-lists: `trace`, `transpose`, `conjugate`, `conjugate_row` (element-wise conjugation with AttributeError fallback for entries lacking `.conjugate()`), `conjugate_transpose`, `eye`, `augment`, `rowadd`, `rowmul`.

### [`sparsetools.py`](sparsetools.py)
Sparse format conversion utilities: `_doktocsr` (DOK→CSR), `_csrtodok` (CSR→DOK).
