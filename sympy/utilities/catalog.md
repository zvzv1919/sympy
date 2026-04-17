# Utilities Module Catalog

## Code Generation & Compilation

### [`autowrap.py`](autowrap.py)
Compiles SymPy expressions into binary-callable functions via Fortran (f2py), Cython, or Ufuncify backends.
- `autowrap(expr)` — compile an expression to a binary callable.
- `binary_function(symfunc, expr)` — attach compiled numerics to a SymPy Function.

### [`codegen.py`](codegen.py)
Generates source code (C, C++, Fortran, Julia, Octave/Matlab) from SymPy expressions.
- `Routine` — represents a callable routine with inputs/outputs.
- `CodeGen`, `CCodeGen`, `FCodeGen` — language-specific code generators.
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
- `subsets`, `variations`, `cartes` — combinatoric generators.
- `numbered_symbols` — infinite generator of Symbol objects.
- `topological_sort` — Kahn's algorithm for DAG ordering.
- `has_dups`, `has_variety` — duplicate/uniqueness checks.

### [`enumerative.py`](enumerative.py)
Algorithms for enumerative combinatorics (multiset partitions).
- `multiset_partitions_taocp` — Knuth's algorithm for multiset partitions.
- `MultisetPartitionTraverser` — breadth-first enumeration with size/count constraints.

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

### [`pytest.py`](pytest.py)
py.test integration helpers.
- `raises`, `XFAIL`, `skip`

### [`randtest.py`](randtest.py)
Randomized testing helpers.
- `random_complex_number`, `verify_numerically`, `test_derivative_numerically`

### [`benchmarking.py`](benchmarking.py)
Benchmarking framework via py.test.

## Miscellaneous

### [`misc.py`](misc.py)
Miscellaneous text and path utilities.
- `filldedent(s)` — dedent + fill a string for clean error messages.
- `rawlines(s)` — convert a string to a rawstring-safe representation.

### [`exceptions.py`](exceptions.py)
SymPy-specific exception and warning classes.
- `SymPyDeprecationWarning`

### [`timeutils.py`](timeutils.py)
Timing utilities independent of IPython.
- `timed(func)` — decorator that prints execution time.

### [`magic.py`](magic.py)
Global namespace manipulation.
- `pollute` — inject SymPy names into the global namespace.

### [`pkgdata.py`](pkgdata.py)
Resource acquisition for package data files.
- `get_resource(identifier)` — retrieve a file from package data.
