# sympy/utilities — Shared Utilities & Helpers

General-purpose utilities consumed across SymPy: iterables, decorators, code generation, numerical evaluation helpers, testing infrastructure, and miscellaneous support.

---

## Iterables & Combinatorics

### iterables.py

Comprehensive library of iterator/sequence utilities and combinatorial generators used throughout SymPy.

**Sequence manipulation**
- `flatten(iterable, levels, cls)` — recursively denest containers, optionally limiting depth or restricting to a class.
- `unflatten(iter, n)` — group flat sequence into tuples of length `n`.
- `reshape(seq, how)` — reshape a flat sequence according to a nested template of lists/tuples/sets.
- `group(seq, multiple)` — split sequence into runs of equal adjacent elements.
- `common_prefix(*seqs)` / `common_suffix(*seqs)`
- `rotate_left(x, y)` / `rotate_right(x, y)`
- `minlex(seq, directed, is_set, small)` — canonical rotation (smallest-first) of a cyclic sequence.
- `runs(seq, op)` — split into runs where successive elements satisfy a comparison operator.
- `sift(seq, keyfunc)` — partition into a `defaultdict(list)` by key function.
- `topological_sort(graph, key)` — Kahn-style topological sort; raises on cycles.
- `has_dups(seq)` / `has_variety(seq)` / `uniq(seq)`

**Combinatorial generators**
- `variations(seq, n, repetition)` — permutations with optional repetition.
- `subsets(seq, k, repetition)` — combinations with optional repetition.
- `multiset(seq)` — frequency dict from a sequence.
- `multiset_combinations(m, n)` / `multiset_permutations(m, size)` — unique combinations/permutations of multisets.
- `multiset_partitions(multiset, m)` — unique partitions of a multiset, dispatching to optimised backends.
- `partitions(n, m, k, size)` — integer partitions as dicts (constant-time per partition).
- `ordered_partitions(n, m, sort)` — integer partitions as sorted lists.
- `binary_partitions(n)` — partitions into powers of two.
- `kbins(l, k, ordered)` — partition a list into `k` bins with configurable ordering semantics.
- `necklaces(n, k, free)` / `bracelets(n, k)` — equivalence classes under cyclic (and optionally reflective) symmetry.
- `generate_bell(n)` — Steinhaus–Johnson–Trotter permutations (adjacent transpositions).
- `generate_involutions(n)` / `generate_derangements(perm)`
- `generate_oriented_forest(n)` — oriented forests via Beyer–Hedetniemi algorithm.
- `ibin(n, bits, str)` — binary representation as list or string.
- `permute_signs(t)` / `signed_permutations(t)`

**Tree traversal**
- `postorder_traversal(node, keys)` — recursive postorder generator over SymPy expression trees.
- `interactive_traversal(expr)` — interactive (terminal) expression explorer.

**Simple helpers**: `take`, `dict_merge`, `prefixes`, `postfixes`, `capture`, `filter_symbols`, `numbered_symbols`.

**Caveats**: `partitions()` reuses the same dict object for speed; copy each yielded value if collecting results. `ordered_partitions()` with `m` set also mutates its list in-place.

---

### enumerative.py

Multiset partition enumeration algorithms based on Knuth's Algorithm 7.1.2.5M (TAOCP Vol 4A).

- `PartComponent` — internal struct holding component number (`c`), unallocated multiplicity (`u`), and allocated multiplicity (`v`).
- `multiset_partitions_taocp(multiplicities)` — generator yielding internal state `(f, lpart, pstack)` for each partition.
- `list_visitor(state, components)` / `factoring_visitor(state, primes)` — convert internal state into human-readable partitions or integer factorings.
- **`MultisetPartitionTraverser`** — refactored OOP version with bounded enumeration:
  - `enum_all(multiplicities)` — all partitions.
  - `enum_small(multiplicities, ub)` / `enum_large(multiplicities, lb)` / `enum_range(multiplicities, lb, ub)` — bounded part-count enumeration.
  - `count_partitions(multiplicities)` — fast memoized counting via dynamic programming.
  - Internal: `decrement_part`, `decrement_part_small`, `decrement_part_large`, `decrement_part_range`, `spread_part_multiplicity`.
- `part_key(part)` — builds a DP cache key from a part.

---

## Code Generation & Numerical Evaluation

### codegen.py

Framework for generating compilable C, Fortran 95, Julia, and Octave/Matlab source code from SymPy expressions.

**Data model**
- `Routine(name, arguments, results, local_vars, global_vars)` — language-agnostic description of a callable routine.
- `DataType` / `Variable` / `Argument` / `InputArgument` — typed variable hierarchy.
- `OutputArgument` / `InOutArgument` / `Result` / `ResultBase` — represent return values and pass-by-reference outputs.

**Code generators** (all inherit `CodeGen`):
- `CCodeGen` — emits `.c` + `.h` files.
- `FCodeGen` — emits `.f90` + `.h` (Fortran 95).
- `JuliaCodeGen` — emits `.jl` files.
- `OctaveCodeGen` — emits `.m` files.

**Friendly API**
- `codegen(name_expr, language, ...)` — one-call interface: takes `(name, expr)` pairs, returns generated source as strings or writes to files.
- `make_routine(name, expr, ...)` — factory that creates a `Routine` from an expression.
- `get_code_generator(language, project)`

---

### autowrap.py

Compile SymPy expressions into callable Python binaries via external toolchains (f2py, Cython).

- **`CodeWrapper`** — base class handling the compile-link-import cycle in a temp directory.
  - `CythonCodeWrapper` — generates `.pyx` + `setup.py`, builds via Cython.
  - `F2PyCodeWrapper` — delegates to numpy.f2py.
  - `DummyWrapper` — testing stub.
  - `UfuncifyCodeWrapper` — generates C extension implementing a NumPy ufunc.
- `autowrap(expr, language, backend, ...)` — compile an expression and return a binary callable.
- `binary_function(symfunc, expr, ...)` — autowrap + `implemented_function` in one step.
- `ufuncify(args, expr, ...)` — create a NumPy ufunc supporting broadcasting.

---

### lambdify.py

Convert SymPy expressions into fast numerical lambda functions backed by math/mpmath/numpy/numexpr.

- `lambdify(args, expr, modules, ...)` — the primary entry point; builds a namespace from the chosen module(s), generates a lambda string, and `eval`s it.
- `lambdastr(args, expr, ...)` — returns the lambda source string without evaluating it.
- `implemented_function(symfunc, implementation)` — attach a numerical implementation (`_imp_`) to a symbolic `UndefinedFunction`.
- Module translation dicts: `MATH_TRANSLATIONS`, `MPMATH_TRANSLATIONS`, `NUMPY_TRANSLATIONS`, `NUMEXPR_TRANSLATIONS`.
- `_import(module)` — lazily populate the global namespace dicts for each backend.
- `_imp_namespace(expr)` — collect `_imp_` implementations from sub-expressions.

**Caveats**: Default module priority is math → numpy (if available) → mpmath → sympy; this can produce different behaviour on different systems.

---

## Decorators & Memoization

### decorator.py

Utility decorators used across SymPy.

- `threaded(func)` / `xthreaded(func)` — apply `func` element-wise over matrices, iterables, and (optionally) `Add` terms.
- `conserve_mpmath_dps(func)` — restore `mpmath.mp.dps` after function execution.
- `no_attrs_in_subclass(cls, f)` — descriptor that hides an attribute from subclasses.
- `doctest_depends_on(exe, modules, disable_viewers)` — attach dependency metadata for selective doctest execution.
- `public(obj)` — append `obj.__name__` to the caller's `__all__`.
- `memoize_property(storage)` — create a lazily-cached property backed by an external dict.

---

### memoization.py

Decorator-based memoization for recurrence sequences.

- `recurrence_memo(initial)` — memoize a 1-index recurrence `f(n, cache)` with a growing list cache.
- `assoc_recurrence_memo(base_seq)` — same but for associated (2-index) recurrences `f(n, m, cache)`.

---

## Testing Infrastructure

### runtests.py

SymPy's built-in test runner (~2200 lines). Provides py.test-compatible testing without external dependencies.

- `test(*paths, **kwargs)` — run unit tests; supports subprocess isolation, hash randomization, timeout, split modes, and regex filtering.
- `doctest(*paths, **kwargs)` — run doctests across SymPy modules.
- `_test` / `_doctest` — internal drivers.
- **`SymPyTests`** — orchestrates test discovery, execution, timing, and reporting.
- **`SymPyDocTests`** — orchestrates doctest discovery and execution.
- **`SymPyDocTestFinder`** — customised `DocTestFinder` that respects `_doctest_depends_on` metadata and skips deprecated/dependent tests.
- **`SymPyDocTestRunner`** — customised `DocTestRunner` with timeout support.
- **`SymPyOutputChecker`** — relaxed output comparison (whitespace, dict ordering, floating-point tolerance).
- **`PyTestReporter`** — terminal reporter with colour, timing, and TB formatting.
- `run_in_subprocess_with_hash_randomization(...)` — re-launch tests in a subprocess with `PYTHONHASHSEED` set.
- `split_list(l, split)` — split a test list for parallel sharding.

---

### pytest.py

Thin compatibility shim providing test markers and helpers regardless of whether py.test is installed.

- `raises(expectedException, code)` — assert-raises as callable or context manager.
- `RaisesContext` — context manager backing `raises`.
- `XFAIL` / `XPass` / `XFail` / `Skipped` — expected-failure decorator and exceptions.
- `SKIP(reason)` / `skip(str)` / `slow(func)` — skip and slow-test decorators.

---

### randtest.py

Helpers for randomised numerical verification of symbolic results.

- `random_complex_number(a, b, c, d, rational)` — sample a complex number in a bounding box.
- `verify_numerically(f, g, z, tol)` — check `f ≈ g` at a random point.
- `test_derivative_numerically(f, z, tol)` — check `f.diff(z)` against numerical differentiation.
- `_randrange(seed)` / `_randint(seed)` — seeded or list-driven random generators for reproducible tests.

---

### benchmarking.py

Benchmarking framework built on top of (legacy) py.test internals.

- `Directory` / `Module` — py.test collectors scanning for `bench_*.py` files and `bench_`/`timeit_` functions.
- `Timer` — subclass of `timeit.Timer` that uses caller-provided globals.
- `Function` — py.test `Function` item extended with `benchtime` / `benchtitle`.
- `BenchSession` — terminal session that prints formatted benchmark results with unit-aligned columns.
- `main(args)` — entry point hooking the custom collectors and session into py.test.

---

## Exceptions & Warnings

### exceptions.py

SymPy-specific warning class for deprecations.

- **`SymPyDeprecationWarning`** — rich deprecation warning supporting `feature`, `useinstead`, `issue`, `deprecated_since_version`, and `last_supported_version` fields. Rendered as a filled paragraph via `filldedent`. Provides a `.warn()` method with configurable `stacklevel`.

Auto-configures `warnings.simplefilter("once", SymPyDeprecationWarning)` on import.

---

## Miscellaneous

### misc.py

Grab-bag of small utilities.

- `filldedent(s, w=70)` — strip, dedent, and word-wrap a string (used heavily for error messages).
- `rawlines(s)` — format a multi-line string as a copy-pasteable Python literal.
- `debug(*args)` / `debug_decorator(func)` — conditional stderr printing gated on `SYMPY_DEBUG`.
- `find_executable(executable, path)` — search `PATH` for a binary (cross-platform).
- `func_name(x)` — return `x.func.__name__` or `type(x)`.
- `replace(string, *reps)` / `_replace(reps)` — simultaneous multi-pattern string replacement.
- `translate(s, a, b, c)` — character-level translate/delete with Unicode and multichar support.
- Module-level constants: `ARCH` ("32-bit"/"64-bit"), `HASH_RANDOMIZATION`.

---

### source.py

Interactive source-code inspection helpers.

- `source(object)` — print the source file path and source code of any Python object.
- `get_class(lookup_view)` — resolve a dotted string like `'sympy.core.Basic'` to the actual class.
- `get_mod_func(callback)` — split `'mod.path.ClassName'` into `('mod.path', 'ClassName')`.

---

### magic.py

Namespace manipulation utility.

- `pollute(names, objects)` — inject name→object pairs into the caller's caller's global namespace.

---

### pkgdata.py

Package-relative resource loading.

- `get_resource(identifier, pkgname)` — return a file-like object for a data file relative to the given package, using `__loader__.get_data` when available.

---

### timeutils.py

Lightweight function timing when IPython is not available.

- `timed(func, setup, limit)` — adaptively measure execution time, returning `(number, time, scaled_time, unit)`.
- `timethis(name)` — decorator for hierarchical timing of recursive algorithms, gated by the `SYMPY_TIMINGS` env var.

---

## Sub-packages

### mathml/ (sub-package)

MathML content-to-presentation transformation (requires `lxml`).

- `add_mathml_headers(s)` — wrap a MathML fragment in `<math>` tags with namespace declarations.
- `apply_xsl(mml, xsl)` — apply an XSLT stylesheet to a MathML string.
- `c2p(mml, simple)` — convert MathML Content to MathML Presentation using bundled XSL stylesheets.

---

## Package Initialisation

### \_\_init\_\_.py

Re-exports the most commonly used utilities: `flatten`, `group`, `take`, `subsets`, `variations`, `numbered_symbols`, `cartes`, `capture`, `dict_merge`, `postorder_traversal`, `interactive_traversal`, `prefixes`, `postfixes`, `sift`, `topological_sort`, `unflatten`, `has_dups`, `has_variety`, `reshape`, `default_sort_key`, `ordered`, `filldedent`, `lambdify`, `source`, `threaded`, `xthreaded`, `public`, `memoize_property`, `test`, `doctest`, `timed`.
