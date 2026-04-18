# Utilities Module Catalog

## Code Generation & Compilation

### [`autowrap.py`](autowrap.py)
Compiles SymPy expressions into binary-callable functions via Fortran (f2py), Cython, or Ufuncify backends.
- `autowrap(expr)` — compile an expression to a binary callable; catches `CodeGenArgumentListError` from `make_routine`: silently appends missing output-only args and retries, but re-raises if any missing arg is not an `OutputArgument`.
- `binary_function(symfunc, expr)` — returns a symbolic `UndefinedFunction` whose `.evalf()` delegates to a compiled native binary (autowrap + implemented_function).
- `ufuncify(args, expr)` — top-level entry for creating NumPy ufunc-compatible C extensions; numpy backend enforces maxargs=32 limit on total (inputs+outputs), raises `ValueError` if exceeded.
- `CodeWrapper` — base class; subclasses handle compilation and module import; `_get_wrapped_function(mod, name)` resolves the callable from the compiled module.
  - `wrap_code(routine)` — compiles a routine: creates a temp directory if no filepath given, generates/compiles/imports the module, then cleans up; silently swallows `OSError` on temp directory removal (Windows file-locking edge case).
- `CythonCodeWrapper` — Cython backend; `_get_wrapped_function` appends `'_c'` suffix to the routine name when retrieving the callable from the built extension module.
  - `dump_pyx(routines, f, prefix)` — writes the `.pyx` bridge file: emits `cdef extern` headers and Python wrapper functions; constructs the function body differently for void routines (call then return output args) vs value-returning routines (return call result).
  - `_partition_args(args)` — categorizes routine arguments into py_args, py_returns, py_locals, and py_inferred; infers array dimension parameters from InputArgument/InOutArgument shapes so they need not be passed explicitly.
  - `_call_arg` / `_prototype_arg` / `_declare_arg` — format each argument for the C call, typed declaration, and initialization respectively; arrays→C pointer cast via `.data`, ResultBase outputs→address-of `&`, scalars→pass by name.
- `F2PyCodeWrapper` — Fortran (f2py) backend; resolves callable by original routine name (no suffix).
- `DummyWrapper` — backend-independent mock wrapper for testing; `_generate_code` writes a Python module whose function metadata labels each return value as `'nameless'` (unnamed `Result`) or by variable name (`OutputArgument`).
- `UfuncifyCodeWrapper` — generates C extension code wrapping routines as NumPy ufuncs.
  - `wrap_code(routines)` — compiles multiple expression routines into a single binary; generates a unique exported function name (not derived from routine names) via `id()`.
  - `dump_c(routines, f, prefix, funcname)` — writes C extension source wrapping routines as broadcasting-capable NumPy ufuncs; raises `ValueError` if multiple output routines are given without an explicit `funcname`; all types hardcoded to `NPY_DOUBLE`; assumes all routines share the same input arguments.
- `_validate_backend_language(backend, language)` — raises `ValueError` if a recognized backend is paired with an unsupported language.
- `_infer_language(backend)` — returns the default language for a given backend; raises `ValueError` for unrecognized backends.

### [`codegen.py`](codegen.py)
Builds procedural routine representations (`Routine`) from SymPy expressions and optionally writes source code files (C, C++, Fortran, Julia, Octave/Matlab); does not compile or import modules (see `autowrap.py` for compilation).
- `Routine` — represents a callable routine with inputs/outputs.
- `CodeGen`, `CCodeGen`, `FCodeGen`, `JuliaCodeGen`, `OctaveCodeGen` — language-specific code generators.
- `OctaveCodeGen.dump_m` — writes `.m` file; raises `ValueError` if the first routine's name doesn't match the output file prefix (Octave/Matlab requires function name = filename).
- `JuliaCodeGen._get_routine_opening` — raises `CodeGenError` if any `OutputArgument` appears in the routine; Julia supports multiple return values natively, so pure output-only parameters are invalid in its function signatures.
- `CodeGen.routine()` — builds a `Routine` from an expression; partitions free symbols: `Idx` labels → local iteration variables, free symbols of `Idx` bound/range → function parameters; validates and reorders a user-supplied `argument_sequence`, silently adding unused symbols as extra inputs.
- `make_routine(name, expr)` — simplified factory that creates a single `Routine` object from expressions without generating any source files; raises `CodeGenArgumentListError` (with `.missing_args`) if user-supplied argument list is incomplete — does not recover; callers (e.g. `autowrap`) handle recovery.
  - Classifies `Equality` LHS as `OutputArgument` (or `InOutArgument`); non-equality expressions become return values.
- `codegen(name_expr, language)` — top-level convenience function; delegates to `make_routine` internally.

### [`lambdify.py`](lambdify.py)
Transforms SymPy expressions into interpreted Python lambda functions using math/numpy/mpmath backends (no compilation, no argument-count limits).
- `lambdify(args, expr, modules)` — main entry point for expression-to-function conversion.

## Decorators & Memoization

### [`decorator.py`](decorator.py)
Utility decorators for function manipulation and threading.
- `threaded` / `xthreaded` — apply a function element-wise over Add/Mul/Matrix arguments.
- `public` — mark a function for inclusion in `__all__`.
- `memoize_property` — cached property descriptor.

### [`memoization.py`](memoization.py)
Memoization decorators optimized for recurrence relations.
- `recurrence_memo` / `assoc_recurrence_memo`

## Iterables & Combinatorics

### [`iterables.py`](iterables.py)
Large collection of iterable/container utility functions.
- `flatten`, `unflatten` — recursive/structured flattening of nested iterables.
- `group`, `take`, `dict_merge`, `postorder_traversal`
- `partitions(n, m, k)` — generator of unordered integer partitions of n; yields mutable dicts {part: count}; terminates when decomposition capacity is exhausted.
- `ordered_partitions(n, m)` — generator of sorted integer compositions of n into m additive parts (lists); reuses the same list object across iterations when m is given; recursive strategy iterates a base value and decomposes the remainder.
- `_set_partitions(n)` — generator enumerating all ways to assign n distinct elements into non-overlapping groups (restricted growth strings / Knuth 7.2.1.5H); uses a constrained n-digit counter where each digit ≤ 1 + max of all digits to its left; yields (num_groups, mutable assignment vector).
- `multiset_partitions(multiset, m)` — high-level dispatcher for splitting a collection (possibly with repeats) into non-overlapping groups; sorts input to guarantee canonical output order regardless of input ordering; special-cases all-identical elements (integer `partitions`), pure sets (`_set_partitions`), and general multisets (delegates to `enumerative.py`).
- `kbins(l, k, ordered)` — partition a list into k bins; `ordered` is a 2-digit flag (00/01/10/11) controlling whether bin order and item order matter; raises `ValueError` for unsupported values.
- `subsets`, `variations`, `cartes` — combinatoric generators (set-level, not multiset partition counting).
- `numbered_symbols` — infinite generator of Symbol objects.
- `topological_sort` — Kahn's algorithm for DAG ordering.
- `generate_oriented_forest(n)` — enumerates all acyclic rooted tree (oriented forest) structures on n vertices using a parent-pointer array representation (Beyer–Hedetniemi algorithm).
- `necklaces`, `bracelets` — circular sequence enumeration under rotation (necklaces) or rotation+reversal (bracelets).
- `has_dups`, `has_variety` — duplicate/uniqueness checks.

### [`enumerative.py`](enumerative.py)
Low-level algorithms for enumerative combinatorics (multiset partition traversal/counting); operates on multisets, not plain integer partitions. Called by `iterables.multiset_partitions` for the general multiset case.
- `multiset_partitions_taocp` — Knuth's Algorithm M for partitioning a multiset given as a multiplicity vector (not plain sets of n distinct elements).
- `MultisetPartitionTraverser` — stateful traverser for multiset partition enumeration with size/count constraints.
  - `count_partitions(multiplicities)` — fast partition counting via dynamic programming with a persistent cross-call cache.
  - `enum_all` — enumerate all partitions (no size constraint).
  - `enum_small(multiplicities, ub)` — enumerate partitions with at most `ub` parts.
  - `enum_large(multiplicities, lb)` — enumerate partitions with more than `lb` parts.
  - `enum_range(multiplicities, lb, ub)` — enumerate partitions where `lb < num_parts <= ub`; combines upper- and lower-bound pruning during traversal.

## Inspection & Source

### [`source.py`](source.py)
Interactive source code inspection and dotted-path class resolution.
- `source(object)` — print the source code and file location of any Python object.
- `get_class(lookup_view)` — resolve a dotted string path (e.g. `'sympy.core.Basic'`) to the actual class object; raises `AttributeError` if the resolved attribute is not callable.
- `get_mod_func(callback)` — split a dotted path into (module_path, attribute_name).

## Testing

### [`runtests.py`](runtests.py)
SymPy's built-in testing framework (py.test-compatible, no external dependencies).
- `convert_to_native_paths(lst)` — converts forward-slash-delimited paths to OS-native paths; on Windows, re-inserts the backslash after a drive letter colon when `os.path.join` drops it.
- `test(*paths)` — run tests; supports `split='a/b'` to partition test files into segments for parallel CI.
- `_test()` — internal runner; when `slow=True`, deterministically shuffles tests (fixed seed) before splitting to ensure even workload distribution across segments.
- `split_list(l, split)` — partition a list into segment `a` of `b` (e.g. `'2/3'`); used by `_test` and `_doctest` for CI splitting.
- `doctest(*paths)` — run doctests in a subprocess with hash randomization; supports `rerun=N` to retry failed runs, exiting on first failure; falls back to in-process execution (same retry logic) when hash randomization is unavailable.
- `SymPyDocTests.get_test_files(dir)` — collects `.py` source files for doctest verification; determines importability by checking for `__init__.py` in the file's immediate parent directory only (does not verify ancestor dirs).
- `SymPyDocTests.test_file` — executes docstring examples; in default (non-normal) mode, clears each function's global namespace so all imports must be explicit within docstrings.
- `SymPyDocTestFinder` — recursive doctest discovery; filters classes/functions by module ownership; for properties, checks `val.fget.__module__` instead of `val.__module__`.
  - `_get_test` — extracts doctest from an object; when obj is a raw string (compiled polys modules), parses line number from the name via regex `"line \d+"`; for property descriptors, resolves source line via `obj.fget` and skips if `obj.fget.__doc__` is None.
- `SymPyOutputChecker` — custom output checker that supports approximate float comparison in doctest output, including handling of trailing-dot ellipsis in expected values.
- `SymPyTests.test_file` — discovers and runs unit tests from a file; collects `test_`-prefixed functions, unpacks generator-based tests (yields tuples of (func, *args)) into individual callable sub-tests, and supports keyword filtering and slow-test selection.
- `SymPyTests._enhance_asserts(source)` — rewrites assertion statements via AST transformation (`NodeTransformer`); replaces comparison-based `assert` with temporary variable assignments and a formatted message displaying actual compared values on failure.
- `sympytestfile(filename)` — runs embedded code examples from a standalone file; assembles a globals namespace from `globs`/`extraglobs` (copied, not shared), defaults `__name__` to `'__main__'` if absent.
- `SymPyDocTestRunner` — custom runner that patches stdout/pdb/linecache during doctest execution.

### [`pytest.py`](pytest.py)
Custom test-assertion utilities and py.test integration helpers; provides its own implementations when py.test is unavailable.
- `raises(expectedException, code)` — exception-assertion utility; dispatches on the second argument: `None` → context manager (`RaisesContext`), callable → invoke and assert, `str` → raises `TypeError` with migration message guiding users to lambdas/`with` blocks, other types → `TypeError`.
- `XFAIL` — decorator marking expected-failure tests; catches exceptions as `XFail`, raises `XPass` on unexpected success, converts timeouts to `Skipped`.
- `skip`, `SKIP` — skip a test with a reason; `SKIP` is a decorator form.
- `slow` — decorator marking slow tests.

### [`randtest.py`](randtest.py)
Randomized numerical verification of symbolic expression equivalence (not doctest output checking).
- `random_complex_number`, `verify_numerically`, `test_derivative_numerically`

### [`benchmarking.py`](benchmarking.py)
Py.test-integrated benchmarking framework for comparative performance reporting; does not select display units, perform log-scale conversion, or handle zero-time edge cases.
- `Timer(timeit.Timer)` — subclass that compiles/executes timing code using caller-provided globals instead of `timeit`'s default isolated namespace.
- `Function` — py.test item that extracts function source, runs calibration targeting ~0.2 s measurement windows for benchmark reporting.

## Miscellaneous

### [`misc.py`](misc.py)
Miscellaneous text, path, and debugging utilities.
- `filldedent(s)` — dedent + fill a string for clean error messages.
- `rawlines(s)` — convert a string to a rawstring-safe representation.
- `debug_decorator(func)` — decorator that prints function arguments and return values to stderr when `SYMPY_DEBUG` is enabled; does not measure time or build duration trees.
- `debug(*args)` — conditional stderr print when `SYMPY_DEBUG` is True.
- `find_executable(name)` — locate an executable on `PATH`.

### [`exceptions.py`](exceptions.py)
SymPy-specific exception and warning classes.
- `SymPyDeprecationWarning`

### [`timeutils.py`](timeutils.py)
Timing and profiling utilities independent of IPython.
- `timed(func)` — adaptively measure execution time of a callable, auto-scaling iteration count; auto-selects display unit (s/ms/μs/ns) via log10; guards against zero-time by defaulting to ns (order=3) to avoid math domain error.
- `timethis(name)` — decorator factory that profiles recursive/nested calls by maintaining a global stack; builds a hierarchical tree of durations. Selectively enabled via `SYMPY_TIMINGS` env var (comma-separated function names); no-op for unlisted names.

### [`magic.py`](magic.py)
Interactive convenience for polluting the user's session namespace (not related to test/doctest execution).
- `pollute` — inject SymPy names into the caller's global namespace.

### [`pkgdata.py`](pkgdata.py)
Acquires bundled data files from within a package using module-level loader introspection.
- `get_resource(identifier)` — locates a data file relative to the calling package's `__file__` path; if the module has a `__loader__`, delegates to its `get_data` and returns an in-memory StringIO; otherwise falls back to direct filesystem open. Raises `IOError` if the module has no `__file__`.
