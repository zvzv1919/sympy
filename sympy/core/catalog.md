# sympy/core — Core Symbolic Engine

## Glossary

- **Basic**: Root class of all SymPy objects. Every symbolic expression is a `Basic`.
- **Expr**: Subclass of `Basic` for mathematical expressions that support arithmetic.
- **Atom**: A `Basic` with no sub-expressions (e.g. `Symbol`, `Number`).
- **AssocOp**: Abstract base for associative operations (`Add`, `Mul`).
- **Singleton**: An object with exactly one instance, accessed via `S` (e.g. `S.One`, `S.Zero`).
- **sympify**: Convert Python objects into SymPy objects.
- **Assumptions**: Tri-valued (True/False/None) properties attached to symbolic objects (e.g. `is_real`, `is_positive`).

---

## 1. Expression System — Core Class Hierarchy

### core.py

The core's core — defines the canonical class ordering and the metaclass infrastructure.

- `ordering_of_classes` — list defining canonical sort order of SymPy types.
- `class Registry` — base class for singleton-style registries; all instances share class-level state.
- `class BasicMeta(type)` — metaclass for `Basic`; registers every class into `all_classes` and provides `__cmp__`/`__lt__`/`__gt__` based on `ordering_of_classes`.
- `all_classes` — global set of all SymPy class objects.

### basic.py

Base class for all SymPy objects. Defines the fundamental protocols: hashing, equality, tree traversal, substitution, matching, and rewriting.

- `class Basic`
  - Core slots: `_mhash`, `_args`, `_assumptions`.
  - Properties: `func`, `args`, `free_symbols`, `is_comparable`, `canonical_variables`.
  - Substitution/replacement: `subs()`, `_subs()`, `xreplace()`, `replace()`.
  - Pattern matching: `match()`, `matches()`, `has()`.
  - Tree inspection: `atoms()`, `find()`, `count()`, `count_ops()`.
  - Evaluation: `doit()`, `rewrite()`.
  - Comparison: `compare()`, `sort_key()`, `class_key()`.
- `class Atom(Basic)` — parent for indivisible expressions (Symbol, Number, etc.). Overrides `matches`, `xreplace`, `sort_key` for leaf-node behavior.
- `class preorder_traversal(Iterator)` — pre-order tree iterator with `skip()` support.
- `_aresame(a, b)` — structural identity check (stricter than `==`; e.g. `S(2.0)` vs `S(2)`).
- `_atomic(e)` — return atom-like quantities (Symbols, Functions, Derivatives) for substitution purposes.

### expr.py

Extends `Basic` with full algebraic expression support — arithmetic operators, calculus hooks, series expansion, and numeric evaluation.

- `class Expr(Basic, EvalfMixin)` (~3200 lines)
  - Arithmetic operators: `__add__`, `__mul__`, `__pow__`, `__neg__`, `__abs__`, etc., all dispatched via `@call_highest_priority`.
  - Algebraic decomposition: `as_coeff_add()`, `as_coeff_mul()`, `as_coeff_Mul()`, `as_coeff_exponent()`, `as_numer_denom()`, `as_real_imag()`, `as_powers_dict()`, `as_coefficients_dict()`, `as_base_exp()`, `as_ordered_terms()`, `as_ordered_factors()`, `as_content_primitive()`.
  - Series: `series()`, `nseries()`, `lseries()`, `taylor_term()`, `compute_leading_term()`, `as_leading_term()`, `leadterm()`.
  - Calculus: `diff()`, `integrate()`, `limit()`.
  - Numeric: `n()` / `evalf()`, `round()`, `equals()`.
  - Polynomial queries: `is_polynomial()`, `is_rational_function()`, `is_algebraic_expr()`.
  - Simplification: `normal()`, `together()`, `apart()`, `cancel()`, `factor()`, `simplify()`, `radsimp()`, `powsimp()`, `trigsimp()`, `nsimplify()`.
  - Extraction: `coeff()`, `extract_multiplicatively()`, `extract_additively()`, `extract_branch_factor()`.
  - `_op_priority` — numeric priority for dispatch of binary operations.
- `class AtomicExpr(Atom, Expr)` — atomic expressions that participate in arithmetic (e.g. `Symbol`, `Number`).

### singleton.py

Singleton mechanism — ensures classes like `Zero`, `One`, `Pi` have exactly one instance, accessible via `S`.

- `class SingletonRegistry(Registry)` — the global `S` object; maps class names to instances via lazy `__getattr__`. Also acts as `sympify` when called: `S(5)`.
- `class Singleton(ManagedProperties)` — metaclass that registers classes on `S` and caches unique instances.

---

## 2. Numbers

### numbers.py

All numeric types: integers, rationals, floats, special constants (pi, E, I, oo, nan), and algebraic numbers.

- **Utility functions**:
  - `igcd(*args)` — integer GCD.
  - `ilcm(*args)` — integer LCM.
  - `igcdex(a, b)` — extended GCD (returns `x, y, g` with `a*x + b*y = g`).
  - `mod_inverse(a, m)` — modular inverse.
  - `comp(z1, z2, tol=None)` — comparison with tolerance.
  - `seterr(divide=False)` — control division-by-zero behavior.
- **Core numeric classes**:
  - `class Number(AtomicExpr)` — abstract base for all numbers; provides `__add__`, `__mul__`, etc., sorting, precision handling.
  - `class Float(Number)` — arbitrary-precision floating point, backed by mpmath.
  - `class Rational(Number)` — exact rational `p/q`; auto-reduces. Properties: `p`, `q`.
  - `class Integer(Rational)` — exact integer; provides `__mod__`, `__floordiv__`, factorial, primality testing.
  - `class AlgebraicNumber(Expr)` — represents algebraic numbers via a minimal polynomial and a primitive element.
- **Singleton constants**:
  - `Zero`, `One`, `NegativeOne`, `Half` — integer/rational singletons with specialized arithmetic.
  - `Infinity` / `NegativeInfinity` — signed infinities (`oo`, `-oo`); arithmetic with infinities defined per mathematical convention.
  - `NaN` — not-a-number; absorbs in all operations.
  - `ComplexInfinity` — unsigned infinity (`zoo`).
- **Special number symbols** (`NumberSymbol` subclasses):
  - `Exp1` (Euler's number `E`), `Pi` (`pi`), `GoldenRatio`, `EulerGamma`, `Catalan`.
  - Each provides `_as_mpf_val(prec)` for arbitrary-precision evaluation.
  - `ImaginaryUnit` — the imaginary unit `I`, with power-cycling: `I**2 = -1`, `I**3 = -I`, `I**4 = 1`.

### alphabets.py

Single tuple `greeks` listing the 24 Greek letter names (alpha through omega). Used by printing/parsing.

---

## 3. Symbols

### symbol.py

Symbol classes — the named indeterminates of symbolic math.

- `class Symbol(AtomicExpr, Boolean)` — the primary symbolic variable. Identified by `name` + assumptions. Cached so `Symbol('x') is Symbol('x')`.
  - `_sanitize()` — validates and normalizes assumption kwargs.
  - `as_dummy()` — return a `Dummy` with same name/assumptions.
- `class Dummy(Symbol)` — unique symbol with internal counter (`dummy_index`). Two `Dummy('x')` instances are never equal. Used for temporary/internal variables.
- `class Wild(Symbol)` — pattern-matching wildcard. `exclude` and `properties` parameters constrain matches.
  - `matches(expr, ...)` — returns match dict if `expr` satisfies all constraints.
- `symbols(names, **args)` — batch symbol factory. Supports comma/space separation, range syntax (`x:10`, `a:z`), and `cls` kwarg for `Function`/`Wild`.
- `var(names, **args)` — like `symbols()` but injects into caller's global namespace (convenience for interactive use).

---

## 4. Arithmetic Operations

### add.py

The `Add` class — represents sums.

- `class Add(Expr, AssocOp)`
  - `flatten(seq)` — combines like terms, collects numeric coefficients, handles `Order` terms, sorts canonically. Returns `(commutative_part, noncommutative_part, order_symbols)`.
  - Coefficient extraction: `as_coeff_add()`, `as_coeff_Add()`, `as_coefficients_dict()`.
  - `as_two_terms()`, `as_numer_denom()`, `as_real_imag()`.
  - `primitive()` — extract rational GCD from all terms.
  - `as_content_primitive()` — recursive content/primitive factoring.
  - `extract_leading_order(symbols)` — returns leading terms and their orders.
  - Full assumption propagation: `_eval_is_real`, `_eval_is_positive`, `_eval_is_zero`, etc.
  - `_eval_subs()` — smart substitution recognizing sub-sums (e.g. `(a+b+c).subs(b+c, x)`).
- `_unevaluated_Add(*args)` — construct a well-formed unevaluated `Add` (numbers collected, args sorted).

### mul.py

The `Mul` class — represents products.

- `class Mul(Expr, AssocOp)`
  - `flatten(seq)` (~500 lines) — the heart of multiplication: separates commutative/non-commutative parts, combines powers of common bases, handles numeric exponent reduction, deals with `-1`, `I`, `oo`, `zoo`, and 2-arg special cases.
  - `_eval_power(e)` — handles `(a*b)**e` for commutative/non-commutative bases.
  - `as_coeff_mul()`, `as_coeff_Mul()`, `as_two_terms()`, `as_real_imag()`, `as_powers_dict()`, `as_numer_denom()`, `as_base_exp()`, `as_content_primitive()`.
  - `_expandsums(sums)` — recursive helper for expanding products of sums.
  - `_eval_expand_mul()` — distribute multiplication over addition.
  - Full assumption propagation including `_eval_pos_neg()` sign-tracking algorithm.
  - `_eval_subs()` — complex substitution with `breakup/rejoin/ndiv` for matching powers.
- `prod(a, start=1)` — product of iterable elements.
- `_keep_coeff(coeff, factors, clear=True, sign=False)` — keep `coeff*factors` unevaluated when appropriate.
- `_unevaluated_Mul(*args)` — construct a well-formed unevaluated `Mul`.
- `expand_2arg(e)` — expand `c*(a+b)` without full Mul machinery.

### power.py

The `Pow` class — represents exponentiation `x**y`.

- `isqrt(n)` — integer square root.
- `integer_nthroot(y, n)` — `(floor(y**(1/n)), exact?)` via Newton's method.
- `class Pow(Expr)`
  - `__new__(b, e)` — handles all singleton cases (x**0=1, x**1=x, 0**x, 1**x, oo**x, NaN, etc.) and recognizes `E` as base.
  - Properties: `base`, `exp`.
  - `_eval_power(other)` — computes `(b**e)**other` with careful branch-cut handling.
  - Expansion: `_eval_expand_power_exp()` (a**(n+m) → a**n·a**m), `_eval_expand_power_base()` ((a·b)**n → a**n·b**n), `_eval_expand_multinomial()` ((a+b)**n → multinomial expansion).
  - `as_base_exp()` — normalizes `(1/q)**e → (q, -e)`.
  - `as_real_imag()` — complex decomposition for integer, rational, and general exponents.
  - `_eval_nseries()` — Laurent/power series expansion (used by gruntz limit algorithm).
  - Full assumption propagation: positivity, sign, parity, rationality, algebraicity, finiteness.

### mod.py

Modular arithmetic as a symbolic function.

- `class Mod(Function)` — `Mod(p, q)` with Python's sign convention (remainder has sign of divisor).
  - `eval(p, q)` — simplifies via exact arithmetic, GCD extraction, and Add-term reduction.

### operations.py

Abstract base for associative operations.

- `class AssocOp(Basic)` — base for `Add` and `Mul`; handles flattening, identity element removal, `make_args()`, `_from_args()`, and commutative pattern matching (`_matches_commutative`).
- `class LatticeOp(AssocOp)` — for operations that are also idempotent and absorptive (e.g. `Max`, `Min`).

---

## 5. Functions & Calculus

### function.py

Infrastructure for defined, undefined, and anonymous functions, plus `Derivative`, `Subs`, and the `expand` family.

- `_coeff_isneg(a)` — True if leading numeric coefficient is negative.
- `class PoleError(Exception)` — raised when series expansion hits a pole.
- `class FunctionClass(ManagedProperties)` — metaclass for functions; manages `nargs`.
- `class Application(Basic)` — base for applied functions; dispatches `eval()`.
- `class Function(Application, Expr)` — base for mathematical functions (sin, cos, exp, ...).
  - Auto-evals to float when args are float.
  - `fdiff(argindex)` — symbolic first partial derivative.
  - `_eval_evalf(prec)` — delegates to mpmath by function name.
  - `_eval_nseries()` — general Taylor series expansion.
- `class AppliedUndef(Function)` — result of applying an undefined function `f(x)`.
- `class UndefinedFunction(FunctionClass)` — metaclass for `Function('f')`.
- `class WildFunction(Function, AtomicExpr)` — pattern-matching wildcard for functions.
- `class Derivative(Expr)` — unevaluated/evaluated derivatives.
  - Supports high-order, multi-variable, and derivatives wrt non-Symbols.
  - `_sort_variables()` — canonical variable ordering (symbols commute, non-symbols don't).
  - `doit()` — force evaluation.
  - `doit_numerically(z0)` — numerical derivative via mpmath.
- `class Lambda(Expr)` — anonymous function `Lambda(x, expr)`.
- `class Subs(Expr)` — unevaluated substitution `Subs(expr, vars, point)`.
- `diff(f, *symbols)` — top-level differentiation entry point.
- `expand(e, ...)` — master expand dispatcher with hints: `mul`, `multinomial`, `power_exp`, `power_base`, `log`, `complex`, `trig`, `func`, `basic`. Metahints: `deep`, `force`, `modulus`, `frac`, `numer`, `denom`.
- Specialized wrappers: `expand_mul`, `expand_multinomial`, `expand_log`, `expand_func`, `expand_trig`, `expand_complex`, `expand_power_base`, `expand_power_exp`.
- `count_ops(expr, visual=False)` — count operations in an expression.
- `nfloat(expr, n=15, exponent=False)` — replace Rationals with Floats.

---

## 6. Relational Expressions

### relational.py

Symbolic relational (comparison) operators.

- `class Relational(Boolean, Expr, EvalfMixin)` — base class; dispatch via `Relational(lhs, rhs, rop)`.
  - Properties: `lhs`, `rhs`, `reversed`, `reversedsign`, `negated`, `canonical`.
  - `equals(other)` — numerical equality check.
- `class Equality` / `Eq` — `lhs == rhs`, simplifies when possible.
- `class Unequality` / `Ne` — `lhs != rhs`.
- `class StrictGreaterThan` / `Gt`, `class GreaterThan` / `Ge`.
- `class StrictLessThan` / `Lt`, `class LessThan` / `Le`.
- Internal helpers: `_Inequality`, `_Greater`, `_Less`.

---

## 7. Assumptions & Logic

### assumptions.py

The assumptions framework — attaches mathematical properties (`is_real`, `is_positive`, `is_integer`, etc.) to symbolic objects via a three-valued knowledge base.

- `_assume_rules` — `FactRules` instance encoding all implication relationships between assumptions (e.g. integer → rational → real → complex).
- `class StdFactKB(FactKB)` — standard fact knowledge base; initialized from assumption generators.
- `class ManagedProperties(BasicMeta)` — metaclass that auto-generates `is_<property>` attributes and wires up the deduction engine.

### facts.py

Rule-based deduction engine (simplified RETE) used by the assumption system.

- `class FactRules` — compiles a rule set into alpha/beta networks.
  - `deduce_all_facts(facts)` — forward-chain all derivable facts from given facts.
- `class FactKB(dict)` — knowledge base that triggers deduction on every update.
- `class Prover` — rule compiler; builds `beta_rules`, `defined_facts`, `full_implications`.

### logic.py

Low-level three-valued logic primitives for the assumption/fact system.

- `_torf(args)` — True if all True, False if all False, else None.
- `_fuzzy_group(args, quick_exit=False)` — fuzzy-AND over an iterable.
- `fuzzy_bool(x)` — coerce to True/False/None.
- `fuzzy_and(args)`, `fuzzy_or(args)`, `fuzzy_not(v)` — three-valued logic connectives.
- `class Logic` / `And` / `Or` / `Not` — symbolic logic expressions (used during fact compilation, not at runtime).

---

## 8. Numeric Evaluation

### evalf.py

Adaptive arbitrary-precision numerical evaluation using mpmath.

- `class PrecisionExhausted(ArithmeticError)` — raised when desired precision cannot be achieved.
- `class EvalfMixin` — mixin providing `evalf(n, ...)` / `n(...)` for numeric evaluation.
  - Handles `subs` dict, `maxn` (max internal precision), `chop` (zero out tiny imaginary parts), `strict` mode.
- `N(x, n=15)` — top-level numeric evaluation function.
- Internal evaluation dispatch: `evalf_add`, `evalf_mul`, `evalf_pow`, `evalf_log`, `evalf_trig`, `evalf_atan`, `evalf_symbol`, `evalf_piecewise`, `evalf_sum`, `evalf_prod`, `evalf_integral`, `evalf_bernoulli`.
- Precision helpers: `pure_complex()`, `fastlog()`, `complex_accuracy()`, `check_target()`, `scaled_zero()`.
- `add_terms(terms, prec, target_prec)` — precision-aware multi-term addition.
- `hypsum(expr, n, start, prec)` — hypergeometric series summation.
- `check_convergence(numer, denom, n)` — check convergence of ratio test.

---

## 9. Expression Tools

### exprtools.py

Higher-level tools for manipulating commutative expressions: GCD, factoring, and non-commutative handling.

- `_monotonic_sign(self)` — determine the sign-definite value of a symbolic expression.
- `decompose_power(expr)` / `decompose_power_rat(expr)` — split `expr` into `(base, exponent)` pairs.
- `class Factors` — sparse representation of a product as `{base: exp}` dict. Supports `mul()`, `quo()`, `div()`, `gcd()`, `lcm()`, `pow()`, `normal()`.
- `class Term` — represents `coeff * numer / denom` as `Factors` objects. Supports `mul()`, `quo()`, `pow()`, `gcd()`, `lcm()`, `cancel()`.
- `gcd_terms(terms, ...)` — compute GCD of terms and factor it out.
- `factor_terms(expr, ...)` — pull out common factors from terms of an expression; handles radicals and signs.
- `_mask_nc(eq)` — replace non-commutative quantities with commutative Dummies for processing.
- `factor_nc(expr)` — factor a non-commutative expression.

---

## 10. Sympification & Type Conversion

### sympify.py

Conversion of arbitrary Python objects into SymPy internal form.

- `sympify(a, ...)` — main entry point. Handles: SymPy objects (passthrough), `converter` registry, string parsing, numeric types, iterables.
- `_sympify(a)` — strict/fast version; no string parsing.
- `class SympifyError(ValueError)` — raised on failed conversion.
- `class CantSympify` — mixin to disallow sympification of a class.
- `converter` — global dict mapping Python types to SymPy conversion callables.

---

## 11. Containers

### containers.py

SymPy-aware container types that participate in the expression tree.

- `class Tuple(Basic)` — immutable tuple subclass of `Basic`. Supports slicing, `subs`, iteration, concatenation, and is hashable.
- `class Dict(Basic)` — immutable dictionary subclass of `Basic`. Keys/values are sympified; supports standard dict interface (`items`, `keys`, `values`, `get`, `__contains__`).
- `tuple_wrapper(method)` — decorator converting plain tuples in function args to `Tuple`.

---

## 12. Caching

### cache.py

Memoization infrastructure for SymPy — wraps `functools.lru_cache` or `fastcache`.

- `CACHE` — global list of cached functions; supports `print_cache()` and `clear_cache()`.
- `cacheit` — the caching decorator. Behavior controlled by env vars:
  - `SYMPY_USE_CACHE=yes|no|debug`
  - `SYMPY_CACHE_SIZE=<int>|None`

---

## 13. Evaluation Control

### evaluate.py

Context manager for controlling automatic evaluation.

- `global_evaluate` — singleton list `[True]` controlling whether constructors auto-simplify.
- `evaluate(x)` — context manager: `with evaluate(False): ...` suppresses auto-evaluation.

---

## 14. Decorators

### decorators.py

Internal decorators for the core.

- `deprecated(**kwargs)` — marks a function as deprecated with `SymPyDeprecationWarning`.
- `_sympifyit(arg, retval)` — decorator that auto-sympifies a named argument before calling the function.
- `call_highest_priority(method_name)` — binary operator dispatch: calls the method of the operand with higher `_op_priority`.

---

## 15. Pattern Rules

### rules.py

Simple transformation rule container.

- `class Transform` — immutable mapping defined by a callable `transform` and optional `filter`. Supports `__contains__`, `__getitem__`, `get()`. Used by `xreplace()`.

---

## 16. Compatibility

### compatibility.py

Python 2/3 compatibility layer and general utility imports.

- String types: `string_types`, `unicode`, `unichr`.
- Integer types: `integer_types`, `long`, `SYMPY_INTS`.
- Itertools: `range`, `reduce`, `zip_longest`, `filterfalse`, `lru_cache`.
- Metaclass helpers: `with_metaclass`.
- Utility functions: `iterable()`, `is_sequence()`, `as_int()`, `ordered()`, `default_sort_key()`.
- Python 2/3 bridging for `exec_`, `StringIO`, `builtins`, `Iterator`, `Mapping`, etc.

### coreerrors.py

Exception definitions for the core module.

- `class BaseCoreError(Exception)` — base for core errors.
- `class NonCommutativeExpression(BaseCoreError)` — raised when commutativity is required but violated.

---

## 17. Multidimensional / Vectorization

### multidimensional.py

Decorator for lifting scalar functions to operate element-wise on iterables.

- `class vectorize` — decorator; specified positional/keyword args are treated as iterables, and the wrapped function is applied element-wise recursively.
- `apply_on_element(f, args, kwargs, n)` — recursive element-wise application helper.
- `iter_copy` / `structure_copy` — deep-copy helpers for iterables.

---

## 18. Trace

### trace.py

Symbolic trace operation for matrices and operators.

- `class Tr(Expr)` — symbolic trace `Tr(expr)`.
  - Handles cyclic permutation under the trace.
  - `doit()` — evaluates to `expr.trace()` for matrices, or distributes over Add.
  - Supports symbolic indices for partial traces.

---

## 19. Benchmarks

### benchmarks/

Micro-benchmarks for performance-critical operations (not library code).

- `bench_arit.py` — benchmarks for Add/Mul construction.
- `bench_assumptions.py` — benchmarks for assumption queries.
- `bench_basic.py` — benchmarks for Basic operations.
- `bench_expand.py` — benchmarks for expand.
- `bench_numbers.py` — benchmarks for numeric operations.
- `bench_sympify.py` — benchmarks for sympify.

---

## Appendix

**Caveats**:

- `subs()` vs `xreplace()`: `subs` performs algebraic matching (e.g. recognizing `x+y` inside `x+y+z`), while `xreplace` does exact node replacement only. Use `xreplace` when you want literal tree-node substitution.
- `evaluate=False`: Many constructors accept `evaluate=False`, but SymPy internals broadly assume evaluated forms. Using unevaluated expressions extensively may cause unexpected behavior.
- **Circular imports**: Several modules import from each other at the bottom of the file (e.g. `add.py` imports from `mul.py` and vice versa). This is intentional and necessary for the tightly coupled arithmetic system.
- `Pow` **branch cuts**: `(b**e)**other` simplification depends on assumptions about `b` and `e` (sign, real-ness). Without assumptions, many simplifications are blocked to preserve mathematical correctness.
- `Dummy` **symbols**: Never compare `Dummy` instances by name — each has a unique `dummy_index`. They are intentionally unequal even if names match.
