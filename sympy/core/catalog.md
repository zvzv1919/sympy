# Core Module Catalog

## Architecture Overview

The `core` module is SymPy's foundation. `basic.py` defines the object model (`Basic`, `Atom`). Numeric concrete types live in `numbers.py` (`Integer`, `Rational`, `Float`), each with their own arithmetic and power-evaluation logic. Symbolic operators live in dedicated files: `add.py` (Add), `mul.py` (Mul), `power.py` (Pow). Numerical evaluation to arbitrary precision is in `evalf.py`. Symbols and variable identity are in `symbol.py`.

**Key distinction**: `power.py` owns the symbolic `Pow` class and simplification of `base**exp` expressions. Each numeric type in `numbers.py` owns its own `_eval_power` for concrete numeric exponentiation (e.g., Rational raised to Rational). `evalf.py` owns runtime numerical evaluation of any expression to a target precision.

---

## Object Model

### [`basic.py`](basic.py)
Root of the SymPy class hierarchy; every SymPy object inherits from `Basic`.

- `Basic` — base class: `__eq__`, `__hash__`, `compare()`, `atoms()`, `subs()`, `rewrite()`, canonical ordering
- `Atom` — parent for indivisible expressions (Symbol, Number); has no `.args`
- `_aresame(a, b)` — structural identity check (not mathematical equality); traverses both trees in preorder comparing type and value at each node; special-cases `UndefinedFunction`/`AppliedUndef` using `class_key()`
- `_atomic(e)` — returns atom-like quantities (Derivatives, Functions, Symbols) for substitution purposes
- `preorder_traversal` — generator yielding nodes in preorder

### [`core.py`](core.py)
Internal infrastructure: `ordering_of_classes` for canonical sort order, `BasicMeta` metaclass, `all_classes` registry.

### [`singleton.py`](singleton.py)
`S` registry and `Singleton` metaclass ensuring unique instances (S.Zero, S.One, S.NaN, S.Infinity, etc.).

---

## Numeric Types

### [`numbers.py`](numbers.py)
All concrete numeric types and their arithmetic operations.

- `Number` — abstract base for numerics; defines `__divmod__`, `__rdivmod__`, coercion logic
- `Float` — arbitrary-precision real via mpmath; `__new__` parses strings/ints/floats, auto-counts significant figures when precision is empty string (`''`), handles scientific notation significance rules (decimal point presence affects digit counting)
- `Rational` — exact p/q fractions; auto-reduces via GCD; `_eval_power` handles concrete rational exponentiation including negative-base sign separation for complex phase
- `Integer` — whole numbers (subclass of Rational); cached in `_intcache`; `__rdivmod__` converts non-int left operands via `Number()` with TypeError handling
- `igcd`, `ilcm` — integer GCD/LCM utilities
- `NumberSymbol` — base for named constants (pi, E, etc.)
- `_sympify` coercion and `SympifyError` handling throughout arithmetic methods

**Caveat**: Each numeric class implements its own `_eval_power`; Rational._eval_power handles negative-fraction-to-fractional-exponent by separating sign via `(-1)**(expt.p % expt.q / expt.q)`.

---

## Symbolic Operators

### [`add.py`](add.py)
`Add` class — commutative n-ary sum. `flatten()` collects coefficients, separates commutative/non-commutative terms.

### [`mul.py`](mul.py)
`Mul` class — commutative n-ary product. `flatten()` handles coefficient extraction and commutativity separation.

### [`power.py`](power.py)
`Pow` class — symbolic `base**exp` expression and simplification rules.

- `Pow.__new__` — evaluates special cases (x**0, x**1, 0**x, oo**x, etc.)
- `Pow._eval_power` — simplifies nested powers like `(x**a)**b`
- `integer_nthroot(y, n)` — exact integer nth root with boolean exactness flag

**Caveat**: `Pow` delegates to `base._eval_power(exp)` for type-specific evaluation; numeric power logic (Integer/Rational/Float raised to numeric exponents) lives in `numbers.py`, not here.

### [`mod.py`](mod.py)
`Mod` class for symbolic modulo; `eval()` simplifies for known operands.

---

## Symbols and Variables

### [`symbol.py`](symbol.py)
Named algebraic variables and temporary dummy symbols.

- `Symbol` — cached algebraic variable; `__new_stage2__` manages assumptions via `StdFactKB`, preserves copy of user-specified assumptions (vs defaults) in `_generator` so serialization can distinguish explicit from implicit commutativity
- `Symbol._sanitize()` — validates and coerces assumption values
- `Dummy` — unique uncached symbol (internal counter `_count`); identity by index, not name
- `Wild` — pattern-matching variable with optional `exclude`/`properties` constraints

### [`alphabets.py`](alphabets.py)
Precomputed Greek letter name collections for `symbols()` shorthand.

---

## Numerical Evaluation

### [`evalf.py`](evalf.py)
Adaptive arbitrary-precision numerical evaluation engine using mpmath.

- `evalf(x, prec, options)` — main dispatcher; routes to type-specific handlers
- `evalf_mul(v, prec, options)` — evaluates products; detects NaN/infinite factors by checking real parts before main multiply; separates pure-real, pure-imaginary, and complex factors with direction tracking
- `evalf_add(v, prec, options)` — sums terms with cumulative error tracking and iterative precision increase
- `evalf_pow(v, prec, options)` — power evaluation with special-case handling
- `evalf_log`, `evalf_atan`, `evalf_trig` — specialized transcendental evaluators
- `pure_complex(v)` — extracts a + b*I form
- `complex_accuracy`, `fastlog`, `bitcount` — precision/accuracy utilities
- `iszero`, `scaled_zero` — zero detection and underflow representation

**Caveat**: `evalf_mul` NaN/infinity check only inspects `arg[0]` (real part); a purely-imaginary infinity (arg[0] is None) is skipped by this check.

### [`evaluate.py`](evaluate.py)
Global evaluation toggle — context manager `evaluate(False)` suppresses automatic simplification. Not numerical evaluation; controls whether `Add`/`Mul`/`Pow` constructors simplify.

---

## Expressions and Manipulation

### [`expr.py`](expr.py)
`Expr` — base for algebraic expressions (inherits Basic + EvalfMixin). Arithmetic operators, `as_coeff_Mul()`, `as_coeff_Add()`, `sort_key()`, `is_constant()`.

### [`exprtools.py`](exprtools.py)
Expression manipulation utilities: `gcd_terms()`, `factor_terms()`, `collect_const()`, `_monotonic_sign()`.

### [`operations.py`](operations.py)
`AssocOp` — base for associative operations (Add, Mul). `_from_args()`, `flatten()`.

---

## Functions

### [`function.py`](function.py)
Function class hierarchy: `Function`, `AppliedUndef`, `UndefinedFunction`, `Lambda`, `Derivative`, `Subs`.

- `UndefinedFunction` — metaclass for user-created callable symbols (e.g., `f = Function('f')`)
- `AppliedUndef` — result of calling an UndefinedFunction on arguments

---

## Assumptions and Logic

### [`assumptions.py`](assumptions.py)
Property inference: `StdFactKB` knowledge base, `ManagedProperties` metaclass for assumption caching. Attributes: `is_positive`, `is_real`, `is_commutative`, etc.

### [`facts.py`](facts.py)
Rule-based deduction engine (RETE-style inference) for deriving assumption implications.

### [`logic.py`](logic.py)
Three-valued fuzzy logic: `fuzzy_and()`, `fuzzy_or()`, `fuzzy_not()`, `_fuzzy_group()` — True/False/None semantics.

---

## Type Conversion

### [`sympify.py`](sympify.py)
`sympify()` — converts Python objects to SymPy types. `converter` dict maps types to handlers. `SympifyError` for failures.

---

## Comparison

### [`relational.py`](relational.py)
`Eq`, `Ne`, `Lt`, `Le`, `Gt`, `Ge` — symbolic equations and inequalities. `Relational` base dispatches by operator string.

---

## Containers and Utilities

### [`containers.py`](containers.py)
`Tuple` — SymPy-aware immutable tuple; `Dict` — SymPy-aware immutable dictionary.

### [`rules.py`](rules.py)
`Transform` — immutable callable mapping (key→value with optional filter predicate).

### [`cache.py`](cache.py)
`cacheit` — LRU memoization decorator; integrates with fastcache.

### [`decorators.py`](decorators.py)
`_sympifyit` — auto-converts arguments to SymPy types; `deprecated` — deprecation warnings.

### [`compatibility.py`](compatibility.py)
Python 2/3 polyfills: `string_types`, `integer_types`, `with_metaclass()`, `iterable()`, `ordered()`.

---

## Errors

### [`coreerrors.py`](coreerrors.py)
Exception classes only: `BaseCoreError`, `NonCommutativeExpression`. No conversion logic or error handling routines.

---

## Specialized

### [`multidimensional.py`](multidimensional.py)
`vectorize` decorator — generalizes scalar functions to operate on nested iterables.

### [`trace.py`](trace.py)
`Tr` class — symbolic matrix/operator trace with cyclic permutation of arguments.
