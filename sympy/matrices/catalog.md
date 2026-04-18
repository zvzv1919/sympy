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
- **Arithmetic**: `__add__` (concrete element-wise addition; reshapes result to preserve original dimensions when a dimension is zero), `__pow__` (integer: square-and-multiply; symbolic/float: Jordan), `multiply`, `add`.
- `__mul__`: matrix multiplication — checks `is_Matrix` on RHS; returns `NotImplemented` (defers to Python dispatch) when RHS claims matrix-ness but lacks `.T.tolist()` (e.g. `MatrixSymbol`); scalar RHS broadcasts element-wise.
- `exp`: matrix exponential — decomposes via `jordan_cells`, splits each Jordan block into diagonal + nilpotent parts, computes factorial power series for the nilpotent component, recombines via P·eJ·P⁻¹.
- **Dot / element-wise products**: `dot` (relaxed-dimension concrete inner product — accepts plain Python lists/sequences (wraps to Matrix) or Matrix; auto-transposes both operands when b has cols>1; returns scalar for vectors, list for rectangular), `multiply_elementwise`, `cross`.
- **Row reduction / spaces**: `rref` (reduced row-echelon form on `MatrixBase` objects — searches for non-zero pivots, swaps rows, scales, and eliminates; returns transformed matrix + pivot indices), `rank`, `nullspace`, `columnspace`.
- **Eigenvalue analysis**: `eigenvals`, `eigenvects`, `left_eigenvects`, `berkowitz_eigenvals`, `berkowitz`.
- `singular_values`: computes via eigenvalues of A^H·A, takes sqrt of each, returns list sorted descending. `condition_number`: ratio of max to min singular value.
- **Diagonalization**: `is_diagonalizable`, `jordan_form` (canonical Jordan/block-diagonal decomposition), `jordan_cells`.
- `diagonalize(reals_only, sort, normalize)`: returns (P, D) where D is diagonal and D = P⁻¹·M·P; optionally sorts eigenvalues (reverse `default_sort_key` order) and normalizes eigenvector columns to unit length.
- `_jordan_block_structure`: computes generalized eigenvector chain leaders per eigenvalue and block size; iterates block sizes largest-first, excluding vectors from smaller kernels and already-used chains.
- **Decompositions** (public wrappers that validate preconditions, then delegate to internal `_cholesky`/`_LDLdecomposition` in dense/sparse layers):
  - `cholesky`, `LDLdecomposition`: both enforce squareness (`NonSquareMatrixError`) and symmetry (`ValueError`) before delegating. LDL is the square-root-free variant (L·D·Lᵀ).
  - `QRdecomposition`: orthogonal-triangular via Gram-Schmidt; validates column rank via rref before factoring.
  - `LUdecomposition`.
- `LUdecomposition_Simple`: in-place LU factorization on a mutable copy; partial pivoting selects first non-zero candidate via `iszerofunc`; raises `ValueError` when all column pivots evaluate to zero. Returns combined L/U matrix + row-swap list.
- `LUdecompositionFF`: fraction-free LU returning PA=LD⁻¹U; keeps all entries in the original integral domain by dividing each update by the previous pivot; raises `ValueError("Matrix is not full rank")` when no nonzero pivot is found below a zero diagonal entry.
- **Solvers**: `solve`, `LUsolve`, `QRsolve`, `LDLsolve` (symmetric→direct LDL; overdetermined rows≥cols→normal equations A^T·A before decomposing; underdetermined→raises), `cholesky_solve` (symmetric→direct Cholesky; overdetermined rows≥cols→normal equations A^T·A then Cholesky; underdetermined→raises), `gauss_jordan_solve`, `solve_least_squares`, `pinv`, `pinv_solve`.
- **Triangular solvers** (public precondition-checking wrappers): `lower_triangular_solve(rhs)` validates squareness, row-count match, and `is_lower` before delegating; `upper_triangular_solve(rhs)` validates squareness, row-count match, and `is_upper` before delegating. Both delegate to `_lower/_upper_triangular_solve` in dense/sparse layers.
- **Calculus**: `jacobian(X)` — Jacobian matrix (derivative of vector function w.r.t. variables); requires self and X each be a row or column vector (raises `TypeError` if either has both dimensions > 1).
- **Determinant/inverse**: `det` (returns `S.One` for empty 0×0 matrix), `det_bareis` (fraction-free Gaussian elimination — searches below diagonal for non-zero pivot, swaps rows tracking sign; returns zero immediately when no pivot found in a column), `det_LU_decomposition`, `berkowitz_det` (division-free determinant via Berkowitz algorithm; extracts last coefficient of characteristic polynomial and applies `(-1)^(n-1)` sign correction), `inv`, `adjugate`, `cofactor`, `cofactorMatrix`.
- **Inversion strategies**:
  - `inverse_ADJ`: cofactor/adjugate — computes `berkowitz_det`, checks `d.equals(0)`; if indeterminate (None), falls back to `rref` diagonal-pivot check for singularity.
  - `inverse_LU`: LU decomposition via `LUsolve`; checks rref diagonal for singularity.
  - `inverse_GE`: Gaussian elimination — augments [self | I], row-reduces via `rref`, checks diagonal entries for singularity, returns right half.
- **Norms**: `norm` — vectors: p-norms (default 2-norm); non-vector matrices with default/Frobenius ord: reshapes to vector via `vec()` then computes 2-norm; ord=2/−2: max/min singular value.
- `normalized`: returns unit-length version of a vector; raises `ShapeError` if self is a non-vector matrix (both rows>1 and cols>1).
- `vec`: reshapes matrix into single-column by stacking columns. `vech`: extracts unique entries from a symmetric matrix into a single column (lower triangle); verifies symmetry by simplifying and comparing to transpose.
- **Block structure**: `get_diag_blocks` — decomposes a concrete square matrix into independent square sub-matrices along the main diagonal by verifying off-block regions are zero (recursive expansion).
- **Stacking**: `hstack(*args)` / `vstack(*args)` — class methods that horizontally/vertically concatenate matrices via `reduce` over `row_join`/`col_join`.
- **Structure / indexing**: `row_join` (horizontal concat; returns `type(self)(rhs)` when self is null/empty, enabling `reduce`-based stacking from an empty accumulator), `col_join` (vertical concat; same null-matrix guard), `row_insert`, `col_insert`, `extract` (submatrix by row/column index lists; also accepts boolean lists — True selects the corresponding row/column), `reshape`, `key2bounds`, `_setitem`.
- `key2ij`: converts indexing key to (row, col) — single integer→`divmod` by cols; sequence of length 2→per-axis index; slice→`.indices` on flattened length.
- **Element-wise symbolic operations**: `subs`, `xreplace`, `expand`, `simplify` — each delegates to `applyfunc`, applying the operation to every entry. `_eval_simplify` is aliased to `simplify`, so the core simplification framework's internal hook dispatches here.
- **Dynamic calculus dispatch** (`__getattr__`): lookups for `diff`, `integrate`, `limit` are intercepted and return a function that applies the operation element-wise via `applyfunc`.
- **Predicates**: `is_square`, `is_diagonal`, `is_upper`, `is_lower`, `is_zero` (three-valued), `is_nilpotent` (characteristic polynomial = x^n via `charpoly`).
- `is_hermitian`: three-valued (True/False/None) via `fuzzy_and`; checks diagonal entries are real and off-diagonal pairs satisfy conjugate symmetry. Returns None when assumptions are insufficient (e.g. symbolic diagonal with no real assumption).
- `is_symmetric`: computes self−transpose, simplifies, checks zero; `simplify=False` skips reduction → may yield false negatives.
- `is_anti_symmetric`: when `simplify` enabled, checks diagonal entries are zero then off-diagonal paired sums `M[i,j]+M[j,i]` are zero (separately); accepts custom simplify callable. `simplify=False` uses direct equality.
- **Construction**: `_handle_creation_inputs` — normalizes all constructor forms (nested list, flat list+dims, callable, NumPy array, MatrixBase) into (rows, cols, flat_list).
  - Validates uniform row lengths; skips 0×0 sub-matrices when tracking column widths.
- **Display**: `print_nonzero` (marks non-zero entries), `_format_str` (str representation; single-row matrices use inline `Matrix([...])` format; multi-row matrices insert a leading newline `Matrix([\n...])`; zero-dimension matrices embed explicit dimensions).
- `table`: tabular text formatter with per-column width alignment; returns `'[]'` for zero-row or zero-col matrices; maps alignment strings to Python justification methods.
- `DeferredVector`: lazily-evaluated symbolic vector; `__getitem__` creates named `Symbol` components on-the-fly. Rejects negative indices (raises `IndexError`); normalizes negative zero to 0.
- `classof(A, B)`: standalone function determining result type when combining matrices of different types; immutability is contagious (`_class_priority` comparison). Falls back to numpy detection: if either operand is a `numpy.ndarray`, the other operand's class wins. Raises `TypeError` if neither path resolves.
- `a2idx(j, n)`: standalone function converting an index to a validated non-negative integer; used by element/slice access throughout the module.
- `MatrixError`, `ShapeError`, `NonSquareMatrixError`: exception hierarchy.

### [`dense.py`](dense.py)
Dense matrix implementation — stores elements in a flat Python list (`_mat`).

- `DenseMatrix`: concrete dense storage; element access, `tolist`, `row`, `col`, `applyfunc`, `reshape`.
- `_eval_trace`: computes trace by summing diagonal entries via flat `_mat` list indexing (`_mat[i*cols + i]`), not two-dimensional access.
- `as_immutable`: converts to `ImmutableMatrix`; special-cases zero-row or zero-col matrices (passes shape+empty list instead of `tolist`). `as_mutable`: converts to mutable `Matrix`.
- `equals`: element-wise symbolic equivalence check using three-valued logic — returns True if all pairs proven equal, False if any pair provably unequal, None if indeterminate.
- `_eval_inverse`: dense matrix inversion dispatching to GE/LU/ADJ methods; supports `try_block_diag` flag to decompose into independent diagonal blocks via `get_diag_blocks()`, invert each block separately, and reassemble.
- DenseMatrix solver methods (operate on matrix objects, not raw lists): `_cholesky`, `_LDLdecomposition`, `_lower_triangular_solve` (forward substitution), `_upper_triangular_solve` (backward substitution), `_diagonal_solve`; each checks for zero diagonal and raises on singular matrices.
- `MutableDenseMatrix`: mutable variant with in-place mutation — `row_swap`, `col_swap`, `row_op`, `col_op`, `zip_row_op` (combines two rows element-wise via a binary functor), `fill`.
- `row_del`, `col_del`: delete a row/column; each validates and converts negative indices internally (no external `a2idx` call), then removes the corresponding slice from the flat `_mat` list.
- `copyin_matrix(key, value)`: copies a Matrix into the sub-region defined by `key`; raises `ShapeError` if source matrix dimensions don't match target slice dimensions.
- `copyin_list(key, value)`: copies elements from an iterable into the sub-region defined by `key`; raises `TypeError` if `value` is not an ordered iterable (e.g. a plain scalar).
- `__setitem__`: thin wrapper; actual assignment logic (including list→Matrix conversion) is `MatrixBase._setitem` in `matrices.py`.
- `_force_mutable(x)`: operand coercion helper used by all mutable arithmetic operators; converts matrices to mutable, sympifies 0-d numpy arrays (scalars), wraps other array-like objects as `Matrix`.
- `matrix_multiply_elementwise(A, B)`: concrete Hadamard (element-wise) product; raises `ShapeError` on dimension mismatch.
- **NumPy conversion utilities**: `matrix2numpy(m, dtype)` — converts a SymPy matrix to a NumPy array element-by-element; `list2numpy(l, dtype)` — converts a Python list of expressions to a NumPy array; `symarray(prefix, shape)` — creates a NumPy object array of named symbols.
- Factory functions: `zeros` (all-zero matrix), `eye` (identity), `ones` (all entries = `S.One`; omitting `c` returns square), `rot_axis1`, `rot_axis2`, `rot_axis3`.
- `hessian(f, varlist, constraints=[])`: (bordered) Hessian of `f` wrt `varlist`; optionally bordered by constraint gradients.
  - `varlist` accepts a sequence or row/column matrix (columns transposed; non-vector matrices raise `ShapeError`). Validates differentiability.
- `diag(*values)`: builds a concrete block-diagonal matrix from a mix of scalars, plain lists, and existing Matrix objects; auto-converts lists to Matrix, accumulates total rows/cols from each block, places blocks along the diagonal of a sparse intermediate, then converts to the target class.
- `wronskian(functions, var, method)`: computes the Wronskian determinant (derivatives matrix det) for testing linear independence of differential functions; returns 1 for an empty input list.
- `casoratian(seqs, n)`: computes the Casoratian determinant for testing linear independence of sequences (used in recurrence solving).
- `randMatrix(r, c, ...)`: generates a random matrix with optional symmetry and sparsity control.

### [`sparse.py`](sparse.py)
Sparse matrix implementation — stores entries in a dictionary-of-keys (`_smat`) mapping `(row, col)` to value.

- `SparseMatrix`: DOK-based sparse storage; `row_list`, `col_list`, `nnz`.
- `__eq__`: equality comparison — checks shape, then compares `_smat` dicts directly for sparse-vs-sparse; converts dense `MatrixBase` to sparse (via `MutableSparseMatrix`) before dict comparison; returns False on `AttributeError` for non-matrix operands.
- `row_join`: horizontal concatenation (`[A B]`); handles both sparse (dict iteration) and dense (flat-list `_mat` iteration) operands.
- `col_join`: vertical concatenation (`[A; B]`); similarly handles mixed sparse/dense operands.
- `MutableSparseMatrix`: mutable variant with in-place `row_swap`, `col_swap`, `row_del`, `col_del`, `row_op`, `col_op`.
- `liupc`: Liu's algorithm for elimination tree pre-determination; collects below-diagonal non-zero indices per row, iterates each row's off-diagonal entries (excludes diagonal via `[:-1]`) to build parent/virtual arrays. Returns (row index lists, parent list).
- `row_structure_symbolic_cholesky`: pre-computes non-zero row structure via elimination trees (calls `liupc`); used by sparse decompositions to skip zero entries.
- Sparse decompositions exploiting pre-computed non-zero pattern: `_cholesky_sparse`, `_LDL_sparse` (unit lower-triangular + diagonal factorization).
- Sparse triangular solvers exploiting sparsity: `_lower_triangular_solve` (forward substitution), `_upper_triangular_solve` (backward substitution; reverses each row's entries to process columns right-to-left), `_diagonal_solve`.
- `_eval_inverse`: sparse inversion dispatch (internal to sparse storage layer); symmetrizes via M^T·M when needed.
- Sparse composite solvers: `_cholesky_solve` (Cholesky factorization + triangular solves), `_LDL_solve` (L·D·L^T factorization then forward substitution → diagonal solve → backward substitution).
- `solve(rhs, method)`: solves self*soln = rhs for square systems; raises `ValueError` for under-determined (rows < cols) and over-determined (rows > cols) non-square systems.
- `solve_least_squares(rhs, method)`: least-squares fit via normal equations (A^T·A)⁻¹·A^T·rhs; handles over-determined sparse systems.

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
- **Indexing**: `__getitem__` — tuple key (i,j) returns `_entry(i,j)` after validation; slice key returns `MatrixSlice`; single integer (flat index) decomposes via `divmod` by cols but raises `IndexError` when shape is symbolic (non-concrete dimensions); symbolic single index also rejected.
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
- `BlockMatrix._eval_determinant`: determinant of 2×2 block partitions via Schur complement — tries upper-left invertibility first, falls back to lower-right; returns unevaluated `Determinant` otherwise.
- `BlockMatrix._entry`: resolves element access by locating the correct sub-block; uses `!= False` comparisons to handle symbolic (non-concrete) row/column indices.
- `BlockDiagMatrix`: unevaluated symbolic block-diagonal expression (does NOT detect/extract blocks from concrete matrices — see `get_diag_blocks` in `matrices.py`); `_eval_inverse` inverts each pre-declared diagonal block independently.
- Block arithmetic rules: `bc_matmul`, `bc_block_plus_ident`, `bc_dist`, `bc_transpose`.

### [`expressions/inverse.py`](expressions/inverse.py)
- `Inverse`: unevaluated symbolic matrix inverse M⁻¹.

### [`expressions/transpose.py`](expressions/transpose.py)
- `Transpose`: unevaluated symbolic transpose Mᵀ.
- `doit`: evaluates the transpose — recursively evaluates inner arg (if `deep=True`), then calls `arg._eval_transpose()`; if that returns `None`, wraps arg back in `Transpose` (never propagates None); catches `AttributeError` for objects lacking the hook.
- `refine_Transpose`: returns the original arg when symmetric assumptions hold.

### [`expressions/adjoint.py`](expressions/adjoint.py)
- `Adjoint`: unevaluated symbolic expression node for conjugate transpose M*; represents the operation lazily, does not verify self-adjoint properties.
- `_eval_trace`: returns `conjugate(Trace(arg))` — computes trace of the adjoint as the conjugate of the trace of the original argument, without iterating entries.

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
- `MatAdd`: unevaluated symbolic matrix sum A+B+C…; `doit()` evaluates via `canonicalize`.
- `validate`: checks all args are matrices with matching shapes.
- **Canonicalization rules**: `rm_id` (remove zeros), `unpack`, `flatten`, `glom` (combine like matrix terms by coefficient), `merge_explicit`, `sort`.
- `merge_explicit`: merges concrete `MatrixBase` summands into one; requires >1 explicit matrix to act, otherwise returns input unchanged.

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
Symbolic matrix factorization component nodes — each wraps a parent matrix expression and carries `predicates` for the assumption-query system.

- `Factorization(MatrixExpr)`: base; inherits shape from wrapped arg.
- LU: `LofLU` (lower_triangular), `UofLU` (upper_triangular). Cholesky: `LofCholesky`, `UofCholesky` (inherit from LU nodes).
- QR: `QofQR` (orthogonal), `RofQR` (upper_triangular). Eigen: `EigenValues` (diagonal), `EigenVectors` (orthogonal).
- SVD: `UofSVD` (orthogonal), `SofSVD` (diagonal), `VofSVD` (orthogonal).
- Factory functions: `lu(expr)` → (L, U), `qr(expr)` → (Q, R), `eig(expr)` → (vals, vecs), `svd(expr)` → (U, S, V).

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
- `LU`: raw list-of-lists LU decomposition without pivoting. For pivoted LU on matrix objects, see `LUdecomposition_Simple` in `matrices.py`.
- `LDL`: square-root-free L·D·Lᵀ factorization for hermitian/self-adjoint matrices on raw nested lists.
  - Returns unit lower-triangular L, diagonal D, and conjugate transpose of L — avoids square roots by separating the diagonal (unlike `cholesky`).
- `cholesky`: Hermitian decomposition on raw nested lists returning L and conjugate transpose; diagonal entries use `isqrt` (integer square root), restricting input to matrices where diagonal minus accumulated sum is a perfect square; off-diagonal entries use division by L[j][j].
- `rref_solve`, `cholesky_solve`, `LU_solve`: solver routines on raw nested-list data. Each deep-copies the coefficient matrix, decomposes it, allocates a fresh symbolic `y` vector for intermediate results, then performs forward substitution (mutating `y` in-place) followed by backward substitution (mutating the caller's `variable` list in-place).
- `cholesky_solve`: decomposes via `cholesky` into L and L*; `LU_solve`: decomposes via `LU` into L and U. Both rely on in-place mutation of passed vectors rather than returning new results.
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
