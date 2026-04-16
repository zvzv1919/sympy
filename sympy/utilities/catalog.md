# sympy/utilities -- Catalog

> Part of [SymPy](../catalog.md). General utilities: iterables, decorators, code generation helpers, lambdify, memoization, testing.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package init that re-exports key public utilities such as `flatten`, `lambdify`, `source`, `threaded`, `test`, and `timed`. |
| `autowrap.py` | High-level module that compiles symbolic expressions into callable Python binaries via f2py, Cython, or numpy-ufunc backends. The main `autowrap` function handles error recovery when the user-supplied argument list is incomplete (appends missing output-only args and retries). The ufunc wrapper partitions arguments by category and raises ValueError for bidirectional (in-out) arguments. Also provides `binary_function` and `ufuncify`. |
| `benchmarking.py` | Provides a py.test-based benchmarking framework with custom Timer, Function, and TerminalSession classes for timing SymPy functions. |
| `codegen.py` | Lower-level code generation: produces complete compilable source files in C, C++, Fortran, Julia, Rust, and Octave/Matlab from SymPy expressions. Defines `Routine`, `CodeGen`, `CCodeGen`, `FCodeGen`, `JuliaCodeGen`, `OctaveCodeGen`, `codegen`, and `make_routine`. Raises `CodeGenArgumentListError` when required arguments are missing, but does not itself recover—callers (e.g. autowrap) handle recovery. |
| `decorator.py` | Utility decorators including `threaded`/`xthreaded` (apply functions elementwise), `conserve_mpmath_dps`, `doctest_depends_on`, `public`, and `memoize_property`. |
| `enumerative.py` | Multiset partition enumeration and counting following Knuth's TAOCP algorithm 7.1.2.5M. `MultisetPartitionTraverser` enumerates partitions and provides `count_partitions`, which uses dynamic-programming memoization with a persistent cross-call cache for efficient tallying. |
| `exceptions.py` | Defines `SymPyDeprecationWarning`, a structured deprecation warning class that includes version, issue tracker link, and migration guidance. |
| `iterables.py` | Extensive collection of iterable utilities: `flatten`, `group`, `subsets`, `variations`, `topological_sort`, `sift`, `ordered`, integer `partitions`, and many more combinatorial helpers for sequences and sets (not multiset partition enumeration). |
| `lambdify.py` | Converts SymPy expressions into fast numerical lambda functions targeting math, mpmath, NumPy, TensorFlow, and other numeric backends. |
| `magic.py` | Contains the `pollute` function that injects name-object mappings into a caller's global namespace via frame introspection. |
| `memoization.py` | Memoization decorators for recurrence-defined sequences (`recurrence_memo`) and associated sequences (`assoc_recurrence_memo`). |
| `misc.py` | Miscellaneous helpers including `filldedent` (text formatting), `rawlines` (pasteable string repr), `translate`, `replace`, `find_executable`, and the `Undecidable` exception. |
| `pkgdata.py` | Provides `get_resource` for acquiring data files from a package, supporting both filesystem and custom `__loader__` access. |
| `pytest.py` | Compatibility layer for py.test functionality (`raises`, `skip`, `XFAIL`, `slow`) that works both with and without py.test installed. |
| `randtest.py` | Helpers for randomized testing: `random_complex_number`, `verify_numerically` (compare expressions at random points), and `test_derivative_numerically`. |
| `runtests.py` | SymPy's built-in test runner framework, compatible with py.test but requiring no external dependencies. Supports doctest, split-based parallelism, and timeout handling. Includes an output checker that performs approximate floating-point comparison of decimal numbers in doctest results (handling ellipsis-truncated expected values), and an AST-based assertion rewriter that transforms assert statements to display actual compared values on failure. |
| `source.py` | Interactive source inspection utilities: `source` (print an object's source code), `get_class`, and `get_mod_func` (resolve dotted class paths). |
| `timeutils.py` | Simple adaptive timing tools (`timed`) for measuring function execution time, plus recursive timing instrumentation via `timethis`. |
| `mathml/__init__.py` | MathML utilities for transforming MathML content markup into MathML presentation markup using XSL transforms (requires lxml). |
