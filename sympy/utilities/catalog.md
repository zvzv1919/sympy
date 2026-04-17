# Utilities Module Catalog

## Code Generation & Compilation

### [`autowrap.py`](autowrap.py)
Compiles SymPy expressions into binary-callable functions via Fortran (f2py), Cython, or Ufuncify backends.
- `autowrap(expr)` — compile an expression to a binary callable; auto-recovers from incomplete argument lists by appending missing output-only arguments, re-raises if any missing arg is not output-only.
- `binary_function(symfunc, expr)` — attach compiled numerics to a SymPy Function.

### [`codegen.py`](codegen.py)
Generates source code (C, C++, Fortran, Julia, Octave/Matlab) from SymPy expressions.
- `Routine` — represents a callable routine with inputs/outputs.
- `CodeGen`, `CCodeGen`, `FCodeGen` — language-specific code generators.
- `CodeGen.routine()` — builds a `Routine` from an expression; validates and reorders a user-supplied `argument_sequence`, silently adding unused symbols as extra inputs.
- `codegen(name_expr, language)` — top-level convenience function.

### [`lambdify.py`](lambdify.py)
Transforms SymPy expressions into fast numerical lambda functions using math/numpy/mpmath backends.
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
- `test(*paths)` — run tests.
- `doctest(*paths)` — run doctests.
- `SymPyDocTests.test_file` — executes docstring examples; in default (non-normal) mode, clears each function's global namespace so all imports must be explicit within docstrings.
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
- `Function` — py.test item that extracts function source, runs adaptive calibration targeting ~0.2 s measurement windows.

## Miscellaneous

### [`misc.py`](misc.py)
Miscellaneous text, path, and debugging utilities.
- `filldedent(s)` — dedent + fill a string for clean error messages.
- `rawlines(s)` — convert a string to a rawstring-safe representation.
- `debug_decorator(func)` — decorator that prints a visual call-tree trace (nested args and return values) when `SYMPY_DEBUG` is enabled.
- `debug(*args)` — conditional stderr print when `SYMPY_DEBUG` is True.
- `find_executable(name)` — locate an executable on `PATH`.

### [`exceptions.py`](exceptions.py)
SymPy-specific exception and warning classes.
- `SymPyDeprecationWarning`

### [`timeutils.py`](timeutils.py)
Timing utilities independent of IPython.
- `timed(func)` — decorator that prints execution time.

### [`magic.py`](magic.py)
Interactive convenience for polluting the user's session namespace (not related to test/doctest execution).
- `pollute` — inject SymPy names into the caller's global namespace.

### [`pkgdata.py`](pkgdata.py)
Resource acquisition for package data files.
- `get_resource(identifier)` — retrieve a file from package data.
