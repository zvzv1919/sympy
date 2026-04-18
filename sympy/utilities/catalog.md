# Utilities Module Catalog

## Code Generation & Compilation

### [`autowrap.py`](autowrap.py)
Compiles SymPy expressions into binary-callable functions via Fortran (f2py), Cython, or Ufuncify backends.
- `autowrap(expr)` — compile an expression to a binary callable; auto-recovers from incomplete argument lists by appending missing output-only arguments.
- `binary_function(symfunc, expr)` — attach compiled numerics to a SymPy Function.
- `ufuncify(args, expr)` — top-level entry for creating NumPy ufunc-compatible C extensions; numpy backend enforces maxargs=32 limit on total (inputs+outputs), raises `ValueError` if exceeded.
- `CodeWrapper` — base class; subclasses handle compilation and module import; `_get_wrapped_function(mod, name)` resolves the callable from the compiled module.
- `CythonCodeWrapper` — Cython backend; `_get_wrapped_function` appends `'_c'` suffix to the routine name when retrieving the callable from the built extension module.
- `F2PyCodeWrapper`, `DummyWrapper` — Fortran/dummy backends; resolve callable by original routine name (no suffix).
- `UfuncifyCodeWrapper` — generates C extension code wrapping routines as NumPy ufuncs.
  - `wrap_code(routines)` — compiles multiple expression routines into a single binary; generates a unique exported function name (not derived from routine names) via `id()`.
  - `dump_c(routines, f)` — writes C source for ufunc; n_out = len(routines), assumes all routines share the same input arguments (partitioned from routines[0]).
- `_validate_backend_language(backend, language)` — raises `ValueError` if a recognized backend is paired with an unsupported language.
- `_infer_language(backend)` — returns the default language for a given backend; raises `ValueError` for unrecognized backends.

### [`codegen.py`](codegen.py)
Generates source code files (C, C++, Fortran, Julia, Octave/Matlab) from SymPy expressions; does not compile or import modules (see `autowrap.py` for compilation and callable resolution).
- `Routine` — represents a callable routine with inputs/outputs.
- `CodeGen`, `CCodeGen`, `FCodeGen` — language-specific code generators.
- `CodeGen.routine()` — builds a `Routine` from an expression; validates and reorders a user-supplied `argument_sequence`, silently adding unused symbols as extra inputs.
- `codegen(name_expr, language)` — top-level convenience function.

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
- `_set_partitions(n)` — generator enumerating all ways to assign n elements into non-overlapping groups via a constrained n-digit counter; each digit ≤ 1 + max of all digits to its left; yields (num_groups, mutable assignment vector).
- `multiset_partitions(multiset, m)` — high-level dispatcher for splitting a collection into groups; special-cases all-identical elements (reduces to integer `partitions`), pure sets (`_set_partitions`), and general multisets (delegates to `enumerative.py`).
- `kbins(l, k, ordered)` — partition a list into k bins; `ordered` is a 2-digit flag (00/01/10/11) controlling whether bin order and item order matter; raises `ValueError` for unsupported values.
- `subsets`, `variations`, `cartes` — combinatoric generators (set-level, not multiset partition counting).
- `numbered_symbols` — infinite generator of Symbol objects.
- `topological_sort` — Kahn's algorithm for DAG ordering.
- `has_dups`, `has_variety` — duplicate/uniqueness checks.

### [`enumerative.py`](enumerative.py)
Low-level algorithms for enumerative combinatorics (multiset partition traversal/counting); called by `iterables.multiset_partitions` for the general multiset case.
- `multiset_partitions_taocp` — Knuth's algorithm for multiset partitions.
- `MultisetPartitionTraverser` — stateful traverser for multiset partition enumeration with size/count constraints.
  - `count_partitions(multiplicities)` — fast partition counting via dynamic programming with a persistent cross-call cache.
  - `enum_all`, `enum_small`, `enum_range` — generate partitions with optional size bounds.

## Inspection & Source

### [`source.py`](source.py)
Interactive source code inspection and dotted-path class resolution.
- `source(object)` — print the source code and file location of any Python object.
- `get_class(lookup_view)` — resolve a dotted string path (e.g. `'sympy.core.Basic'`) to the actual class object; raises `AttributeError` if the resolved attribute is not callable.
- `get_mod_func(callback)` — split a dotted path into (module_path, attribute_name).

## Testing

### [`runtests.py`](runtests.py)
SymPy's built-in testing framework (py.test-compatible, no external dependencies).
- `test(*paths)` — run tests; supports `split='a/b'` to partition test files into segments for parallel CI.
- `_test()` — internal runner; when `slow=True`, deterministically shuffles tests (fixed seed) before splitting to ensure even workload distribution across segments.
- `split_list(l, split)` — partition a list into segment `a` of `b` (e.g. `'2/3'`); used by `_test` and `_doctest` for CI splitting.
- `doctest(*paths)` — run doctests.
- `SymPyDocTests.get_test_files(dir)` — collects `.py` source files for doctest verification; determines importability by checking for `__init__.py` in the file's immediate parent directory only (does not verify ancestor dirs).
- `SymPyDocTests.test_file` — executes docstring examples; in default (non-normal) mode, clears each function's global namespace so all imports must be explicit within docstrings.
- `SymPyDocTestFinder` — recursive doctest discovery; filters classes/functions by module ownership; for properties, checks `val.fget.__module__` instead of `val.__module__`.
  - `_get_test` — extracts doctest from an object; for property descriptors, resolves source line number via `obj.fget` and skips the property entirely if `obj.fget.__doc__` is None.
- `SymPyOutputChecker` — custom output checker that supports approximate float comparison in doctest output, including handling of trailing-dot ellipsis in expected values.
- `SymPyDocTestRunner` — custom runner that patches stdout/pdb/linecache during doctest execution.

### [`pytest.py`](pytest.py)
py.test integration helpers.
- `raises`, `XFAIL`, `skip`

### [`randtest.py`](randtest.py)
Randomized numerical verification of symbolic expression equivalence (not doctest output checking).
- `random_complex_number`, `verify_numerically`, `test_derivative_numerically`

### [`benchmarking.py`](benchmarking.py)
Benchmarking framework via py.test.
- `Timer(timeit.Timer)` — subclass that compiles/executes timing code using caller-provided globals instead of `timeit`'s default isolated namespace.
- `Function` — py.test item that extracts function source, runs calibration targeting ~0.2 s measurement windows for benchmark reporting (no unit selection or standalone timing).

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
Resource acquisition for package data files.
- `get_resource(identifier)` — retrieve a file from package data.
