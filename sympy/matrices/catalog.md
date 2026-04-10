# sympy/matrices — Catalog

> Part of [SymPy](../catalog.md). Symbolic and numeric matrix algebra: dense, sparse, immutable, and block matrices; eigenvalues, decompositions.

## Python Files

| File | Summary |
|------|----------|
| `__init__.py` | Package init that re-exports all public matrix classes (Matrix, SparseMatrix, ImmutableMatrix) and matrix expression types. |
| `common.py` | Base classes and shared infrastructure for all matrix types, including `MatrixRequired`, `MatrixShaping`, `MatrixProperties`, `MatrixOperations`, `MatrixArithmetic`, `MatrixSpecial`, error classes, and the `MatrixCommon` composite class. |
| `matrices.py` | Core `MatrixBase` class providing determinant computation (Berkowitz, LU), row reduction (RREF), subspace methods (nullspace, columnspace), eigenvalue/eigenvector routines, and calculus operations (diff, integrate, limit). |
| `dense.py` | `DenseMatrix` and `MutableDenseMatrix` (aliased as `Matrix`) implementations, plus top-level helpers such as `eye`, `zeros`, `ones`, `diag`, `randMatrix`, rotation matrices, `GramSchmidt`, `wronskian`, `hessian`, and `casoratian`. |
| `sparse.py` | `SparseMatrix` (`MutableSparseMatrix`) backed by a dictionary-of-keys storage, with sparse-specific LIL/row-list access, Cholesky and LDL decomposition, triangular solve methods (forward/backward substitution for lower/upper triangular systems), diagonal solve, and applyfunc. |
| `immutable.py` | `ImmutableDenseMatrix` and `ImmutableSparseMatrix` — hashable, read-only matrix variants that can be used as keys and within SymPy expressions. |
| `normalforms.py` | Smith Normal Form computation (`smith_normal_form`) and abelian invariant factors (`invariant_factors`) for matrices over a principal ideal domain. |
| `densearith.py` | Deprecated list-of-lists dense arithmetic helpers (`add`, `sub`, `mulmatmat`, `mulmatscaler`, `negatemat`). |
| `densesolve.py` | Deprecated list-of-lists dense solvers: row echelon, RREF, LU solve, Cholesky solve, forward/backward substitution. |
| `densetools.py` | Deprecated list-of-lists dense utilities: trace, transpose, conjugate, eye construction, row operations, and augmentation. |
| `sparsetools.py` | Conversion between dictionary-of-keys (DOK) and Compressed Sparse Row (CSR) formats (`_doktocsr`, `_csrtodok`). |
| `benchmarks/bench_matrix.py` | Micro-benchmarks for `Matrix.__getitem__` and `zeros` construction. |
| `expressions/__init__.py` | Package init for the matrix-expressions sub-package; re-exports all symbolic matrix expression classes. |
| `expressions/matexpr.py` | `MatrixExpr` base class for symbolic matrix expressions, plus `MatrixSymbol`, `Identity`, `ZeroMatrix`, `MatrixElement`, and generic expression infrastructure (shape, transpose, inverse, indexing). |
| `expressions/matmul.py` | `MatMul` — symbolic product of matrix expressions with canonicalization rules (flatten, combine scalars, identity removal, zero detection). |
| `expressions/matadd.py` | `MatAdd` — symbolic sum of matrix expressions with canonicalization rules (merge, flatten, sort, zero removal). |
| `expressions/matpow.py` | `MatPow` — symbolic matrix power expression with evaluation for integer exponents and identity/zero special cases. |
| `expressions/inverse.py` | `Inverse` — symbolic multiplicative inverse of a matrix expression (subclass of `MatPow` with `exp = -1`). |
| `expressions/transpose.py` | `Transpose` — symbolic transpose of a matrix expression, with `doit` evaluation and canonicalization rules. |
| `expressions/adjoint.py` | `Adjoint` — symbolic Hermitian adjoint (conjugate transpose) of a matrix expression. |
| `expressions/trace.py` | `Trace` — symbolic matrix trace expression with rewrite-as-Sum support and a convenience `trace()` function. |
| `expressions/determinant.py` | `Determinant` — symbolic matrix determinant expression, a convenience `det()` function, and assumption-based refinement. |
| `expressions/slice.py` | `MatrixSlice` — symbolic slicing of a matrix expression by row and column ranges. |
| `expressions/blockmatrix.py` | `BlockMatrix` and `BlockDiagMatrix` — symbolic block-structured matrices with `block_collapse`, `blockcut`, transpose, trace, and determinant rules. For 2×2 block matrices, determinant evaluation uses Schur complement formulas: if the top-left sub-block is invertible, uses `det(A)*det(D - C*A⁻¹*B)`; if not but the bottom-right is invertible, uses `det(D)*det(A - B*D⁻¹*C)`. |
| `expressions/funcmatrix.py` | `FunctionMatrix` — a matrix expression defined by a Lambda function applied to index pairs, enabling lazy evaluation. |
| `expressions/diagonal.py` | `DiagonalMatrix` and `DiagonalOf` — symbolic wrappers that treat a matrix as diagonal or extract its diagonal, using Kronecker deltas for element access. |
| `expressions/hadamard.py` | `HadamardProduct` and `hadamard_product()` — symbolic elementwise (Hadamard) product of matrix expressions. |
| `expressions/dotproduct.py` | `DotProduct` — symbolic dot product of two vector matrices, evaluated by delegating to MatMul with appropriate transposes. |
| `expressions/fourier.py` | `DFT` and `IDFT` — symbolic Discrete Fourier Transform and its inverse as matrix expressions. |
| `expressions/factorizations.py` | Symbolic matrix factorization placeholders (LU, Cholesky, QR, SVD, Eigen) with associated assumption predicates. |
