# sympy/core — Catalog

> Part of [SymPy](../catalog.md). Core symbolic engine: basic objects (Symbol, Number, Expr, Add, Mul, Pow), caching, evaluation, and compatibility.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package entry point; re-exports all public core classes (Symbol, Expr, Add, Mul, Pow, Number, etc.) and exposes singleton constants like Catalan, EulerGamma, and GoldenRatio. |
| `add.py` | Implements the `Add` class for symbolic addition, including canonical ordering, flattening of nested sums, and the `_unevaluated_Add` helper. |
| `alphabets.py` | Defines the `greeks` tuple containing the names of the 24 Greek letters used for symbol rendering. |
| `assumptions.py` | Assumption system machinery: `StdFactKB`, `ManagedProperties` metaclass, and the `BasicMeta` metaclass that attach `.is_*` assumption attributes (e.g. `is_real`, `is_integer`) to symbolic objects. |
| `backend.py` | Backend selector that imports core symbols from either SymEngine (if `USE_SYMENGINE=1`) or SymPy itself, providing a unified interface for downstream modules. |
| `basic.py` | Defines `Basic`, the root base class for all SymPy objects, with core infrastructure for args storage, substitution, traversal, pattern matching, and canonical comparison. |
| `cache.py` | Caching facility providing the `cacheit` decorator and a global `CACHE` registry; uses `fastcache` when available, falling back to `lru_cache`. |
| `compatibility.py` | Python 2/3 compatibility layer providing unified imports for `string_types`, `integer_types`, `range`, `reduce`, `exec_`, `with_metaclass`, `lru_cache`, `ordered`, and other cross-version utilities. |
| `containers.py` | Symbolic container classes `Tuple` and `Dict` that subclass `Basic`, enabling SymPy-aware tuples and dictionaries with support for substitution and sympification. |
| `core.py` | The core's core: defines `ordering_of_classes` for canonical ordering of symbolic types and the `Registry` base class used by the singleton mechanism. |
| `coreerrors.py` | Defines core exception classes: `BaseCoreError` and `NonCommutativeExpression`. |
| `decorators.py` | Core decorators including `deprecated` (deprecation warnings), `_sympifyit` (auto-sympification of method arguments), and `call_highest_priority` (operator dispatch by `_op_priority`). |
| `evaluate.py` | Provides the `evaluate` and `distribute` context managers and their backing `global_evaluate`/`global_distribute` flags to control automatic expression evaluation. |
| `evalf.py` | Adaptive numerical evaluation engine using mpmath; implements `EvalfMixin`, the `N()` function, and internal helpers for arbitrary-precision floating-point arithmetic. |
| `expr.py` | Defines `Expr`, the base class for algebraic expressions, providing arithmetic operators, series expansion, differentiation hooks, `as_numer_denom`, `as_coeff_Add/Mul`, canonical sign selection (`could_extract_minus_sign`), `extract_additively`, `extract_multiplicatively` (factors out a constant from an expression — for sums, all terms must be divisible or it fails), sign-determination methods `_eval_is_positive`/`_eval_is_negative` (uses low-precision numerical evaluation with fallback to minimal polynomial when floating-point yields no significant digits), and many other expression-analysis methods. Also defines `AtomicExpr` and `UnevaluatedExpr`. |
| `exprtools.py` | Tools for manipulating large commutative expressions: `gcd_terms`, `factor_terms`, `factor_nc`, `decompose_power` (splits an exponentiation into symbolic base and integer exponent, returning `(expr, 1)` for irrational exponents), `Factors`/`Term` decomposition, and monotonic-sign analysis. |
| `facts.py` | Rule-based deduction system: compiles alpha/beta implication rules and performs runtime inference via `FactRules` and `FactKB` for the assumption system. |
| `function.py` | Implements `Function`, `FunctionClass`, `Lambda`, `Derivative`, `Subs`, and all `expand_*` helpers. Defines the framework for both defined and undefined symbolic functions. |
| `logic.py` | Internal propositional logic primitives (`And`, `Or`, `Not`, `Logic`) and fuzzy logic helpers (`fuzzy_and`, `fuzzy_or`, `fuzzy_not`, `_fuzzy_group`), primarily used by the facts/assumption engine. |
| `mod.py` | Implements the `Mod` function for symbolic modulo operations, following Python's sign convention for the remainder. |
| `mul.py` | Implements the `Mul` class for symbolic multiplication, including flattening, canonical term ordering within products, coefficient extraction, and the `_unevaluated_Mul` and `_keep_coeff` helpers. Does not handle sign canonicalization between `{e, -e}` (see `Expr.could_extract_minus_sign` in `expr.py`). |
| `multidimensional.py` | Provides the `vectorize` decorator that enables scalar functions to operate element-wise on nested iterables/arrays. |
| `numbers.py` | Numeric types hierarchy: `Number`, `Integer`, `Rational`, `Float`, `AlgebraicNumber`, and singleton constants (`Zero`, `One`, `Half`, `NegativeOne`, `Infinity`, `NegativeInfinity`, `ComplexInfinity`, `NaN`, `ImaginaryUnit`, `Pi`, `Exp1`, `GoldenRatio`, `Catalan`, `EulerGamma`). Each numeric type implements its own `_eval_power` for type-specific exponentiation (e.g., `ComplexInfinity` returns NaN for zero/itself exponents, zoo for positive, zero for negative; `Infinity`/`NegativeInfinity` handle sign-dependent power results; `ImaginaryUnit` uses mod-4; `Rational` separates sign from magnitude for negative-base fractional exponents). Infinity types (`Infinity`, `NegativeInfinity`) also implement their own ordering comparison operators (`__lt__`, `__le__`, `__gt__`, `__ge__`) with special-case logic for comparing against other infinite quantities. `Float` provides `epsilon_eq` for approximate equality comparison with a configurable tolerance. Also provides per-type arithmetic dunders (`__divmod__`, `__rdivmod__`, etc.) with error handling for non-convertible operands, plus `igcd`, `ilcm`, `comp`, and `mod_inverse`. |
| `operations.py` | Defines `AssocOp` (abstract base for associative operations like Add and Mul) and `LatticeOp` (base for lattice operations like Max and Min). |
| `power.py` | Implements the generic `Pow` class for symbolic exponentiation (construction, canonicalization, and evaluation dispatch), but type-specific power logic for concrete numeric types lives in `numbers.py`. Also provides `integer_nthroot` and `isqrt` integer root utilities. |
| `relational.py` | Relational comparison classes: `Relational` base, `Equality` (Eq), `Unequality` (Ne), `StrictLessThan` (Lt), `LessThan` (Le), `StrictGreaterThan` (Gt), `GreaterThan` (Ge). |
| `rules.py` | Defines the `Transform` class, an immutable mapping used as a generic transformation/replacement rule. |
| `singleton.py` | Singleton mechanism: `SingletonRegistry` (accessed as `S`) and the `Singleton` metaclass that ensure unique instances of frequently used objects like `S.Zero`, `S.One`, `S.Half`. |
| `symbol.py` | Defines `Symbol`, `Wild`, and `Dummy` symbolic variable classes, plus the `symbols` and `var` convenience functions for bulk symbol creation. |
| `sympify.py` | The `sympify` function that converts Python objects (ints, floats, strings, etc.) into SymPy objects, along with `SympifyError`, `CantSympify`, `kernS`, and the `converter` registry. |
| `trace.py` | Implements the `Tr` class for symbolic trace operations on non-commutative expressions, with support for cyclic permutation of arguments. |
| `benchmarks/bench_arit.py` | Benchmarks for basic arithmetic operations (negation, Add, Mul) on symbolic expressions. |
| `benchmarks/bench_assumptions.py` | Benchmarks for assumption property lookups (`is_integer`, `is_irrational`). |
| `benchmarks/bench_basic.py` | Benchmarks for Basic operations: method lookup, singleton access, and Symbol equality. |
| `benchmarks/bench_expand.py` | Benchmarks for `expand()` including polynomial expansion and complex number expansion. |
| `benchmarks/bench_numbers.py` | Benchmarks for numeric operations: Integer/Rational creation, arithmetic, `integer_nthroot`, `igcd`, and evalf. |
| `benchmarks/bench_sympify.py` | Benchmarks for `sympify` conversion of integers and Symbol objects. |
