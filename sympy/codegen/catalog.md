# sympy/codegen -- Catalog

> Part of [SymPy](../catalog.md). Code generation for C, Fortran, Julia, Rust, and other languages.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package init that re-exports `Assignment`, `aug_assign`, `CodeBlock`, and `For` from the `ast` submodule. |
| `ast.py` | Defines Abstract Syntax Tree node types for code generation, including `Assignment`, `AugmentedAssignment`, `CodeBlock`, `For`, `Token`, `Type`, `Variable`, `Pointer`, and `Declaration`, along with predefined numeric types (`float32`, `float64`, `integer`, `real`, etc.). |
| `cfunctions.py` | Provides SymPy function classes corresponding to C math functions (`expm1`, `log1p`, `exp2`, `log2`, `fma`, `log10`, `Sqrt`, `Cbrt`, `hypot`) for symbolic manipulation and numerically stable code generation. Also defines C-specific variable attributes (`restrict`, `volatile`, `static`). |
| `ffunctions.py` | Provides SymPy function classes corresponding to Fortran intrinsics (`isign`, `dsign`, `cmplx`, `kind`, `merge`) and literal types (`literal_sp`, `literal_dp`) for Fortran code generation. |
| `rewriting.py` | Implements an expression-rewriting framework (`Optimization`, `ReplaceOptim`, `FuncMinusOneOptim`, `optimize`) that rewrites SymPy expressions into forms using specialized C99 math functions (e.g., replacing `exp(x)-1` with `expm1(x)`) for better precision or performance. |
