# Core Module Catalog

## Architecture Overview

The `core` module is SymPy's foundation. `basic.py` defines the object model (`Basic`, `Atom`). Numeric concrete types live in `numbers.py` (`Integer`, `Rational`, `Float`), each with their own arithmetic and power-evaluation logic. Symbolic operators live in dedicated files: `add.py` (Add), `mul.py` (Mul), `power.py` (Pow). Numerical evaluation to arbitrary precision is in `evalf.py`. Symbols and variable identity are in `symbol.py`.

**Key distinction**: `power.py` owns the symbolic `Pow` class and simplification of `base**exp` expressions. Each numeric type in `numbers.py` owns its own `_eval_power` for concrete numeric exponentiation (e.g., Rational raised to Rational). `evalf.py` owns runtime numerical evaluation of any expression to a target precision.

---

## Module Exports

### [`__init__.py`](__init__.py)
Public API surface of the `core` module. Re-exports all fundamental types and utilities. Exposes well-known mathematical constants (Catalan, EulerGamma, GoldenRatio) as top-level importable names by aliasing from the `S` singleton registry.

---

## Object Model

### [`basic.py`](basic.py)
Root of the SymPy class hierarchy; every SymPy object inherits from `Basic`.

- `Basic` — base class: `__eq__`, `__hash__`, `compare()`, `atoms()`, `subs()`, `replace()`, `rewrite()`, `dummy_eq()`, canonical ordering
  - `compare(other)` — pairwise comparison of `_hashable_content` elements; wraps `frozenset` elements into `Basic(*)` before recursive comparison (handles set-like content in ordering)
  - `atoms(*types)` — collects all leaf (atomic) subexpressions; when types given, filters by isinstance; instance args are converted via `type()` (e.g., `S(1)` filters by `One`, not `Integer`)
  - `__eq__` — structural equality; special-cases `Pow` with exponent equal to 1 (e.g., `a**1.0 == a`) by comparing base to other operand
  - `subs()` — substitution; silently drops pairs where old/new cannot be sympified (non-string, non-symbolic objects)
  - `_subs()` — internal recursive substitution; fallback traverses args and reconstructs via `self.func(*args)`; in simultaneous mode, prevents type-collapse when a Mul reconstruction loses its Mul type by manually separating numeric coefficients
  - `replace(query, value, simultaneous)` — wildcard-capable replacement; in simultaneous mode, creates Dummy placeholders defaulting commutativity to True when replacement's `is_commutative` is None
  - `dummy_eq(other, symbol)` — structural comparison tolerant of anonymous placeholder variables; raises ValueError if left side has more than one Dummy; also raises ValueError if `symbol` is None and right side has multiple free symbols
- `is_comparable` — property; True if expression evaluates to a real number with meaningful precision; decomposes into real/imag parts via `as_real_imag()`, evaluates numerically, returns False if imaginary part is nonzero or real part has precision=1 (indeterminate/no significant digits)
- `Atom` — parent for indivisible expressions (Symbol, Number); has no `.args`
- `rcall(*args)` / `_recursive_call(expr, on_args)` — applies args through compound expression trees; calls `__call__` on callable sub-expressions but explicitly skips bare Symbols to prevent conversion into UndefinedFunction objects
- `_aresame(a, b)` — structural identity check (not mathematical equality); traverses both trees in preorder comparing type and value at each node; special-cases `UndefinedFunction`/`AppliedUndef` using `class_key()`
- `_atomic(e)` — returns atom-like quantities (Derivatives, Functions, Symbols) for substitution purposes
- `preorder_traversal(node, keys)` — generator yielding nodes in preorder; when `keys` is None and a node stores children as a set (e.g., lattice-style ops), uses the internal set directly to avoid unnecessary sorting; when `keys` is provided, delegates to `ordered()` for deterministic traversal

### [`core.py`](core.py)
Internal infrastructure: `ordering_of_classes` for canonical sort order, `BasicMeta` metaclass, `all_classes` registry.

### [`singleton.py`](singleton.py)
`S` registry and `Singleton` metaclass ensuring unique instances (S.Zero, S.One, S.NaN, S.Infinity, etc.).

---

## Numeric Types

### [`numbers.py`](numbers.py)
All concrete numeric types and their arithmetic operations.

- `comp(z1, z2, tol)` — module-level numerical comparison; with nonzero tol uses relative error (`diff/|z1|`) when z2 is nonzero and `|z1| > 1`, otherwise absolute error; with tol=None uses precision-based significance test; with tol='' uses exact string comparison
- `Number` — abstract base for numerics; defines `__divmod__`, `__rdivmod__`, coercion logic; `__mul__`/`__add__`/`__sub__` handle Infinity/NegativeInfinity directly (e.g., zero × infinity → NaN, positive × infinity → Infinity)
  - `__mul__` returns `NotImplemented` (not parent delegation) when other is a `Tuple`, deferring to the container's own multiplication
  - `_eval_subs(old, new)` — if `old` equals the negation of self, returns `-new`; otherwise returns self unchanged (handles e.g., substituting `-3` when atom is `3`)
  - `as_coeff_Mul(rational)` — coefficient extraction; returns `(self, S.One)` for nonzero values but `(S.One, self)` when self is zero (zero goes into the "rest" term, not the coefficient)
- `Float` — arbitrary-precision real via mpmath; `__new__` parses strings/ints/floats; normalizes string inputs before parsing (prepends '0' to '.5', converts '-.5' to '-0.5'); auto-counts significant figures when precision is empty string (`''`), handles scientific notation significance rules (decimal point presence affects digit counting)
  - `_new(cls, _mpf_, _prec)` — internal classmethod constructing Float from raw mpf tuple; returns `S.Zero` (exact integer) for zero input instead of `Float(0.0)` — differs from `__new__` which preserves floating-point zero
  - `_eval_power` — negative Float base with rational exponent p/q where p=1 and q is odd: factors out `(-1)**(1/q)` and recurses on positive base, avoiding spurious complex result
  - `__eq__` — equality comparison; short-circuits to False when other is an irrational `NumberSymbol` (e.g., pi, E) without numerical comparison
  - `__gt__`/`__ge__` — ordering comparisons; checks `other.is_comparable` to decide whether to numerically evaluate the other operand before mpf comparison
  - `__mod__` — modulo operator; when divisor is a non-integer Rational (q≠1), converts self to exact Rational first, computes mod in exact arithmetic, then rounds result back to Float precision; when divisor is Float and `self/other` is exact integer, short-circuits to `Float(0)` at max precision of both operands
  - `__lt__`/`__le__` — ordering comparisons; checks `other.is_real and other.is_number` (different predicate from `__gt__`/`__ge__`) to decide whether to evalf the other operand
- `Rational` — exact p/q fractions; auto-reduces via GCD; `_eval_power` handles concrete rational exponentiation including negative-base sign separation for complex phase
  - `gcd(other)` — greatest common divisor of two Rationals: `igcd(numerators) / ilcm(denominators)`
  - `lcm(other)` — least common multiple of two Rationals: `lcm(numerators) / gcd(denominators)`
  - `as_content_primitive()` — returns `(|self|, sign)` for nonzero; returns `(1, self)` when self is zero
- `Rational` comparison operators (`__gt__`, `__ge__`, `__lt__`, `__le__`) — cross-multiplies `self.p*other.q` vs `self.q*other.p` for Rational-vs-Rational
  - For symbolic real operands, transforms `p/q > expr` into `Integer(p) > q*expr` to clear denominator
- `int_trace` — profiling decorator for `Integer.__new__`; optimistically increments hit counter before cache lookup, then on KeyError decrements hit and increments miss; registered via `atexit` to print stats
- `Integer` — whole numbers (subclass of Rational); cached in `_intcache`; `__rdivmod__` converts non-int left operands via `Number()` with TypeError handling
  - `_eval_power` — negative-base sign branching differs for integer vs fractional exponents
  - For fractional exponents, factors base into primes and extracts perfect roots via divmod; reduces remaining radicals by shared GCD
- `NegativeOne` — singleton `-1`; `_eval_power`: odd exp → -1, even → 1; rational exp with denominator 2 → `I**p` (imaginary unit); general rational exponents decomposed via `divmod` into integer and fractional parts
- `Zero` — additive identity singleton; `_eval_power`: positive exp → 0, negative exp → ComplexInfinity, non-real (complex) exp → NaN
  - Fallback strips leading numeric coefficient from product exponents (negative coeff → zoo**terms, non-unity coeff → 0**remaining_terms)
- `igcd`, `ilcm` — integer GCD/LCM utilities
- `NumberSymbol` — base for named constants (pi, E, etc.)
- `Infinity` / `NegativeInfinity` — signed unbounded sentinels; each implements `_eval_power`
  - `Infinity.__add__`/`__sub__`/`__mul__` — arithmetic operators; when operand is Float, returns `Float('inf')`/`Float('-inf')` (preserving float type) except Float zero × infinity → NaN; when operand is exact zero (S.Zero), also returns NaN; when operand is non-Float Number, returns symbolic `S.Infinity`/`S.NegativeInfinity`
  - `Infinity._eval_power` — positive exp → oo, negative → 0, NaN/zoo exp → NaN; complex (non-real) numeric exponent: extracts real part — positive real part → ComplexInfinity, negative → 0, zero → NaN
  - `NegativeInfinity.__add__`/`__sub__`/`__mul__`/`__div__` — same float-vs-symbolic branching as Infinity; Float operands yield Float results, exact operands yield symbolic singletons; zero × -oo → NaN for both exact and Float zero
  - `NegativeInfinity._eval_power` — integer exponents: odd → -oo, even → oo; non-integer numeric exponents: decomposes as `(-1)**expt * oo**expt` instead of returning a direct result
  - Own `__lt__`, `__le__`, `__gt__`, `__ge__` with special-case branches for finite, nonnegative, and infinite-negative operands
- `ImaginaryUnit` — the imaginary unit `I = sqrt(-1)`; `_eval_power`: integer exponents use mod-4 cycle; non-integer numeric exponents delegate to `(-1)**(expt/2)`; symbolic exponents return None
- `NaN` — indeterminate placeholder; structurally equal to itself (`__eq__`) but mathematically unequal to everything (`_eval_Eq` returns false)
- `ComplexInfinity` — unsigned (undirected) infinite quantity; `_eval_power`: zero exp → NaN, positive exp → zoo, negative exp → 0, zoo exp → NaN
- `_sympify` coercion and `SympifyError` handling throughout arithmetic methods

**Caveat**: Each numeric class (Integer, Rational, Float, NegativeOne, ImaginaryUnit, Infinity, NegativeInfinity) implements its own `_eval_power`. Rational._eval_power separates sign via `(-1)**(expt.p % expt.q / expt.q)`.

---

## Symbolic Operators

### [`add.py`](add.py)
`Add` class — commutative n-ary sum. `flatten()` collects coefficients, separates commutative/non-commutative terms.

- `as_coeff_add(*deps)` — without deps, returns `(leading_number, remaining_terms)`; with deps, partitions terms into symbol-independent sum and symbol-dependent tuple (not numeric extraction)
- `as_coeff_Add(rational=False)` — efficiently extracts leading numeric coefficient; when `rational=True`, only extracts Rational coefficients (irrational Numbers like pi are not separated); when False (default), any Number is extracted
- `as_numer_denom()` — converts sum to (numerator, denominator) form; collects per-term numerators/denominators; special-cases zero-denominator terms (infinity) by moving them into the numerator under denominator 1
- `primitive()` — extracts rational GCD of leading coefficients; returns `(R, self/R)`; special-cases `ComplexInfinity` terms by skipping zero-denominator entries in GCD/LCM computation
- `as_content_primitive(radical, clear)` — recursive content extraction; when `clear=False`, avoids distributing denominators unless doing so yields integer coefficients in the result
- Assumption handlers: `_eval_is_real`, `_eval_is_complex`, `_eval_is_integer`, `_eval_is_rational`, `_eval_is_finite`, etc. — fuzzy-group over all args
- `_eval_subs(old, new)` — Add-specific substitution; handles replacing sub-sums and negated sub-sums within a larger sum (e.g., `(a+b+c+d).subs(-b-c, x)` → `a-x+d`); uses set-subset matching on term args after coefficient separation
- `_eval_is_imaginary` — classifies each term as real-nonzero, imaginary, or "becomes real when multiplied by I"; returns True only if all real parts cancel to zero and imaginary parts are nonzero
- `_eval_is_zero` — separates terms into real/imaginary/unknown; returns True if all args are zero; returns False if real nonzero terms don't cancel or if imaginary terms coexist

### [`mul.py`](mul.py)
`Mul` class — commutative n-ary product. `flatten()` collects powers, coefficients, and separates commutative/non-commutative factors.
- `flatten()` canonicalizes negative numeric bases with non-integer rational exponents by extracting the sign into a running `(-1)**e` accumulator and storing the positive base separately for later combination

- `_eval_is_zero` — determines if product vanishes; returns None (indeterminate) when a zero factor coexists with a non-finite factor (0×∞ scenario)
- `_eval_is_real` / `_eval_real_imag` — real/imaginary inference for products; tracks sign flips from imaginary factors
- `as_coeff_mul(*deps, rational=True)` — splits leading numeric coefficient from remaining factors; with `rational=True`, non-rational negative leading numbers return `(-1, (abs_num, ...))` instead of the number itself
- `_eval_power(b, e)` — raising a product to a power; separates commutative from non-commutative factors; NC factors stay grouped (not distributed) to preserve ordering
- `_eval_evalf(prec)` — numerical evaluation; when coefficient is -1 and remainder is non-Mul, individually evaluates remainder (falls back to original if None); otherwise delegates to AssocOp
- `as_real_imag()` — decomposes product into real/imaginary parts; classifies factors as real, imaginary, or complex; detects complex conjugate pairs among commutative factors and replaces them with `|x|²` (real coefficient); Add factors accumulated separately and expanded last
- `_eval_conjugate` — conjugate of product preserves factor order: `conjugate(a*b) = conjugate(a)*conjugate(b)`
- `_eval_transpose` — transpose of product reverses factor order: `transpose(a*b) = transpose(b)*transpose(a)` (non-commutative algebra rule)
- `_eval_adjoint` — adjoint of product reverses factor order (like transpose)
- `_eval_is_rational`, `_eval_is_algebraic` — assumption handlers with zero-fallback for mixed cases

### [`power.py`](power.py)
`Pow` class — symbolic `base**exp` expression and simplification rules.

- `Pow.__new__` — evaluates special cases (x**0, x**1, oo**x, etc.); delegates `0**x` to `Zero._eval_power` in `numbers.py`
- `Pow._eval_power` — simplifies nested powers like `(x**a)**b`
- `Pow._eval_evalf(prec)` — numerical evaluation of `base**exp`; when exponent is negative and base is non-real, rewrites as `conjugate(base)/|base|²` with negated exponent to avoid complex-power precision issues
- `Pow.as_content_primitive(radical, clear)` — extracts positive Rational from `base**exp`; when base is rational, decomposes exponent into integer + fractional parts via `divmod` and splits the power accordingly; when base is Mul, recursively extracts content from base
- `integer_nthroot(y, n)` — exact integer nth root with boolean exactness flag

**Caveat**: `Pow` delegates to `base._eval_power(exp)` for type-specific evaluation; numeric power logic (Integer/Rational/Float raised to numeric exponents) lives in `numbers.py`, not here.

### [`mod.py`](mod.py)
`Mod` class — symbolic modulo AST node (unevaluated `x % y`); `eval()` simplifies structurally. Concrete numeric `%` operators (Float.__mod__, Integer.__mod__) live in `numbers.py`.

---

## Symbols and Variables

### [`symbol.py`](symbol.py)
Named algebraic variables and temporary dummy symbols.

- `Symbol` — cached algebraic variable; `__new_stage2__` manages assumptions via `StdFactKB`, preserves copy of user-specified assumptions (vs defaults) in `_generator` so serialization can distinguish explicit from implicit commutativity
- `Symbol._sanitize()` — validates and coerces assumption values
- `Symbol.as_real_imag(deep, **hints)` — decomposes into `(re(self), im(self))`; returns `None` (not a tuple) when `hints.get('ignore')` equals the symbol itself
- `Dummy` — unique uncached symbol (internal counter `_count`); identity by index, not name
- `Wild` — pattern-matching variable with optional `exclude`/`properties` constraints

### [`alphabets.py`](alphabets.py)
Precomputed Greek letter name collections for `symbols()` shorthand.

---

## Numerical Evaluation

### [`evalf.py`](evalf.py)
Adaptive arbitrary-precision numerical evaluation engine using mpmath.

- `EvalfMixin` — mixin class adding `.evalf()` method to Expr; when n=1 and self is a Number, recurses at n=2 then rounds by magnitude (special case for Sage compatibility); otherwise dispatches to module-level `evalf()`
- `evalf(x, prec, options)` — main dispatcher; routes to type-specific handlers
- `evalf_mul(v, prec, options)` — evaluates products; detects NaN/infinite factors by checking real parts before main multiply; separates pure-real, pure-imaginary, and complex factors with direction tracking
- `evalf_add(v, prec, options)` — sums terms with cumulative error tracking and iterative precision increase
- `evalf_pow(v, prec, options)` — numerical power evaluation with special-case branches:
  - Integer exponent: real base via `mpf_pow_int`; purely-imaginary base uses `p % 4` cycle to classify result as real/imag/negated
  - Half exponent: square-root fast path; general case: precision-adjusted base/exp with complex-power fallback
- `evalf_log`, `evalf_atan`, `evalf_trig` — specialized transcendental evaluators
- `pure_complex(v)` — extracts a + b*I form
- `fastlog(x)` — bit-level log2 approximation from mpf exponent+mantissa; returns approximate magnitude, not a fallback for overflow
- `complex_accuracy`, `bitcount` — precision/accuracy utilities
- `iszero(mpf)` — tests whether an mpf tuple represents zero
- `scaled_zero(mag, sign)` — constructs a power-of-two mpf placeholder for zero with given magnitude; validates sign is exactly +1 or -1 (raises ValueError otherwise); also unwraps previously created scaled-zero tuples

**Caveat**: `evalf_mul` NaN/infinity check only inspects `arg[0]` (real part); a purely-imaginary infinity (arg[0] is None) is skipped by this check.

### [`evaluate.py`](evaluate.py)
Global evaluation toggle — context manager `evaluate(False)` suppresses automatic simplification. Not numerical evaluation; controls whether `Add`/`Mul`/`Pow` constructors simplify.

---

## Expressions and Manipulation

### [`expr.py`](expr.py)
`Expr` — base for algebraic expressions (inherits Basic + EvalfMixin). Arithmetic operators (`+`, `-`, `*`, `/`), ordering comparisons, `as_coeff_Mul()`, `as_coeff_Add()`, `sort_key()`, `is_constant()`.

- `as_independent(*deps, as_Add=None)` — general-purpose split into (independent, dependent) parts w.r.t. given symbols; works on any Expr
  - Distinct from `Add.as_coeff_add(*deps)` which partitions an Add's own args by symbol dependency or extracts leading numeric coefficient
  - `as_Add` hint forces Add or Mul mode; when forced mode mismatches actual type (e.g., Add forced as Mul), returns `(identity, self)` (1 for Mul, 0 for Add)
  - For Mul, non-commutative factors after the first dependent one are all grouped as dependent
- `equals(other, failing_expression)` — determines symbolic equality when `simplify(self - other)` fails to produce zero; multi-stage: numerical probing, surd self-consistency, then minimal polynomial of the difference (if `mp.is_Symbol`, difference is zero → True; otherwise False)
- `extract_multiplicatively(c)` / `extract_additively(c)` — returns self/c (or self-c) if operation moves value toward zero, else None
  - When `c` is an Add, extracts its rational primitive before attempting factoring; when `c` is a Mul, recursively splits into two-term factors
  - Numeric `extract_additively`: requires diff has same sign as self AND smaller magnitude (overshooting zero returns None)
  - Add `extract_multiplicatively`: requires all terms individually divisible
- `as_coefficient(expr)` — returns scalar multiplier `r` such that `self == r*expr`, or None if self is not a pure scalar multiple of expr
  - Calls `extract_multiplicatively` then rejects if result still `.has(expr)` (e.g., `2*sin(E)*E` w.r.t. `E` → None because `sin(E)` contains `E`)
- `args_cnc(cset, warn, split_1)` — separates factors into commutative and non-commutative lists; when `cset=True`, returns commutative part as a set and raises ValueError if duplicate commutative factors exist (e.g., from unevaluated Mul)
- `coeff(x, n)` — extracts coefficient of `x**n` from a sum; short-circuits to `S.Zero` when self is commutative but x is non-commutative; when x is the multiplicative identity (1), returns only additive terms whose leading numeric factor is 1; for noncommutative expressions, tries common prefix/suffix matching first
- `could_extract_minus_sign()` — canonical choice between `{e, -e}`; first checks `extract_multiplicatively(-1)` asymmetry; then type-specific: Add counts negative-vs-positive terms (majority wins); Mul checks parity of negative factors across numer/denom (odd count → True); final tiebreaker uses `sort_key()` comparison
- `sort_key()` — canonical ordering key; decomposes expression via `as_coeff_Mul` then splits Pow nodes into (base, exp), non-Pow defaults to exp=S.One; Dummy atoms use recursive sort_key (identity-based), other atoms use string representation
- `_random(n, re_min, im_min, re_max, im_max)` — evaluates self with random complex substitutions for free symbols; escalates precision from 2 up to `DEFAULT_MAXPREC` via `giant_steps` when initial evaluation yields no significant digits; returns None if no significance achieved
- `is_constant(*wrt)` — checks if expression is constant w.r.t. given symbols; uses numerical probing (substitutes 0, 1, random values)
- `is_polynomial(*syms)` — returns True only if expression is an exact finite-degree polynomial; rejects symbolic exponents (e.g., `x**n` where n is a symbol, even if integer/nonneg); delegates to `_eval_is_polynomial`
- `is_rational_function(*syms)` — tests if expression is a ratio of polynomials in given symbols; no simplification attempted (unsimplified forms may return False even if simplifiable to rational); delegates to `_eval_is_rational_function`
- `is_algebraic_expr(*syms)` — tests if expression is constructible from field operations and rational exponentiation (extends `is_rational_function` to include fractional powers); no simplification attempted; base `_eval_is_algebraic_expr` returns False when free symbols overlap, relying on subclass overrides
- `as_terms()` — decomposes a sum into structured term list: each term becomes `(coeff, monom, ncpart)` where coeff is `(real, imag)`, monom is a tuple of commutative base exponents indexed by sorted generators, ncpart is non-commutative factors
- `leadterm(x)` — returns leading term as `(coeff, exponent)` tuple; temporarily replaces `log(x)` with a Dummy before decomposition to avoid variable leaking into the coefficient
- `extract_branch_factor(allow_half)` — decomposes products of `exp_polar` into `(residual, n)` where n is the integer winding number; collects `pi*I` multiples and rounds down to nearest even integer via `ceiling`
- `primitive()` — extracts positive Rational from expression non-recursively (treats self as Add); returns `(S.One, S.Zero)` for zero-valued expressions (content is 1, not 0); if `as_coeff_Mul(rational=True)` yields negative coefficient, negates both parts to guarantee positive result
- `__int__` — converts symbolic expression to Python int; rounds to 2 decimal places, applies off-by-one correction when rounded value equals truncated int
  - Uses Dummy-substitution `evalf(2, subs={x: i})` (not direct `(self - i).evalf(2)`, which doesn't always work) to determine difference sign
- `_expand_hint(expr, hint, deep, **hints)` — static recursive helper for `expand()`; walks expression tree applying named `_eval_expand_<hint>()` methods to each subnode; returns `(new_expr, hit)` where `hit` indicates whether any subnode was actually modified
- `_eval_expand_complex` — expansion hint for complex decomposition via `as_real_imag()`
- `_eval_lseries` / `taylor_term` / `lseries()` / `nseries()` — series expansion infrastructure
- `Expr.round(p)` — rounds numeric expression to `p` decimal places; complex inputs are split into real and imaginary parts, each rounded independently; for negative values, detects when adding the rounding half-unit flips sign of the scaled intermediate and reverses direction; uses `_mag` for digit counting
- `__ge__` / `__le__` / `__gt__` / `__lt__` — raises TypeError for complex non-real operands, operands containing ComplexInfinity (via `.has()`), or NaN; otherwise computes difference and checks sign or returns unevaluated relational
- `invert(g)` — multiplicative inverse of self mod g; dispatches to numeric `mod_inverse` if both are numbers, otherwise to polynomial `invert`
- `_eval_is_positive` / `_eval_is_negative` — sign determination; uses low-precision evalf, falls back to minimal polynomial when no significant digits
- `_eval_interval` — definite evaluation over an interval with limit fallback for singular values
- `AtomicExpr` — parent class for objects that are both Atom and Expr (Symbol, Number, etc.)
- `_mag(x)` — module-level helper returning base-10 order of magnitude (`i` such that `.1 <= x/10**i < 1`); uses `math.log10` with fallback to multi-precision `mpf_log` on overflow

### [`exprtools.py`](exprtools.py)
Expression manipulation utilities: `gcd_terms()`, `factor_terms()`, `collect_const()`, `_monotonic_sign()`, `factor_nc()`.

- `_monotonic_sign(expr)` — returns closest-to-zero bound if expression has uniform sign; for non-Add with numeric denominator: prime+odd→3, prime+even→2, positive+even→2, positive+integer→1, positive→_eps, negative mirrors; for multivariate signed linear expressions, substitutes each free symbol's monotonic sign using epsilon placeholder

- `factor_nc(expr)` — factors expressions with non-commutative symbols; extracts common NC prefixes/suffixes, then tries permutations of NC factors to find correct ordering

- `decompose_power(expr)` — splits exponentiation into symbolic base and integer exponent; absorbs rational denominator into base; returns `(expr, 1)` for irrational exponents
- `decompose_power_rat(expr)` — variant preserving rational exponents
- `Factors` — efficient multiplicative representation `f_1*f_2*...*f_n` as a dict mapping bases to exponents
  - Init from Number: negative → stores `-1` as separate key; Rational `p/q` → numerator `p` with exponent 1, denominator `q` with exponent -1
  - `as_expr()` — converts dict back to symbolic Mul; dispatches on exponent type: Python int → wraps in Integer, Rational → keeps as-is, symbolic → multiplies into existing base exponent
  - `normal()` — cancels shared base-power pairs; optimized for few overlaps; for symbolic exponent diffs, tries `extract_additively` first, then falls back to `as_coeff_Add` to partially cancel numeric coefficient parts
  - `div()` — cancels shared base-power pairs optimized for many common factors; for non-numeric exponents, tries `extract_additively` first, then decomposes exponents via `as_coeff_Add` to partially cancel symbolic exponent remainders

### [`operations.py`](operations.py)
`AssocOp` — base for associative operations (Add, Mul). `_from_args()`, `flatten()`.

- `AssocOp.__new__` — constructor: sympifies args, filters identity elements; when `evaluate=False` (or `global_evaluate[0]` is off), returns unevaluated via `_from_args`; when evaluating, calls `flatten()` then determines `is_commutative` based on whether the non-commutative partition is empty
- `_from_args(args, is_commutative)` — creates instance from pre-processed args; when `is_commutative` is None, infers via `fuzzy_and` over args
- `_new_rawargs(*args, reeval=True)` — fast instance creation for rebuilding Add/Mul from a subset of original args
  - When self is non-commutative and `reeval` is True (default), forces recomputation of commutativity from new args as a safety mechanism
  - When self is commutative, inherits commutativity from self without recomputation

- `_eval_evalf(prec)` — numerical evaluation for Add/Mul; splits into numeric-independent and dependent parts; guards against infinite recursion when the independent part is itself an AssocOp Function
- `_matches_commutative` — pattern matching for Add/Mul; after removing exact (non-wild) parts, rejects match if inverse-combined expression has more ops than original (count_ops guard)
  - On first-pass failure, decomposes to retry: Mul rewrites `x**n` (integer n) as `x * x**(n-1)` for positive n, or `1/x * x**(n+1)` for negative n; Add rewrites `c*x` as `x + (c-1)*x`; also tries `collect` on non-Wild symbols

---

## Functions

### [`function.py`](function.py)
Function class hierarchy: `Function`, `AppliedUndef`, `UndefinedFunction`, `Lambda`, `Derivative`, `Subs`.

- `Function.__new__` — after evaluation, checks `_should_evalf` on all args; auto-calls `evalf` only if **every** arg is floating-point (min precision > 0); mixed float/symbolic args remain unevaluated
- `Function._eval_evalf(prec)` — numerical evaluation of symbolic functions; looks up matching mpmath function by name, falling back to `MPMATH_TRANSLATIONS` table; if no mpmath match, tries user-provided `_imp_` numerical implementation; returns None if all fallbacks fail
- `Function.fdiff(argindex)` — first derivative w.r.t. the given argument position; if target arg is a plain Symbol that also appears free in another argument, falls through to Dummy-substitution path (returns `Subs(Derivative(...))`) to avoid incorrect results
- `Function._eval_nseries` — series expansion for symbolic functions; handles infinite-argument cases via leading-term substitution; general algorithm uses repeated differentiation at zero with NaN→limit fallback and PoleError on infinite results
- `Function._should_evalf(arg)` — returns precision (or -1) for auto-evalf decision; detects Float args directly; for Add args, pattern-matches `a + b*I` form to detect complex floats and returns max component precision
- `Lambda` — anonymous function expression `Lambda(x, expr)`; `__eq__` performs alpha-equivalence (renames bound variables before comparing bodies, so `Lambda(x, x**2) == Lambda(y, y**2)`)
- `UndefinedFunction` — metaclass for user-created callable symbols (e.g., `f = Function('f')`)
- `AppliedUndef` — result of calling an UndefinedFunction on arguments; `_eval_as_leading_term` returns self unchanged (no series computation possible for unknown functions)
- `Derivative._sort_variables` — sorts differentiation variables into canonical order; sorts symbols among themselves and non-symbols among themselves, but preserves boundaries between groups (symbol/non-symbol derivatives don't commute)
- `Derivative` uses structural-substitution semantics for diff w.r.t. composed expressions (e.g., `f(x)`): replaces expression with placeholder, differentiates, substitutes back; disallows diff w.r.t. products like `x*y`
- `Subs.__new__` — validates substitution variables are distinct (raises ValueError for duplicates); checks variable/point list length match
  - Generates underscore-prefixed placeholder symbols for variable-independent form; loops to add more underscores when placeholders clash with free symbols mapped to different point values
- `Subs._eval_subs` — guards bound variables: if the substitution target is one of the Subs' bound placeholder variables, returns self unchanged
- `diff(f, *symbols)` — top-level convenience function for symbolic differentiation; dispatches to `f._eval_diff()` if available, falls back to constructing `Derivative(f, ...)` on `AttributeError`; sets `evaluate=True` by default
- `expand(expr, deep=True, **hints)` — main expansion entry point; dispatches named hints (e.g., `mul`, `log`, `trig`, `power_base`) to per-object `_eval_expand_<hint>()` methods
  - When `deep=True` (default), recursively applies hints to subexpressions; when `deep=False`, only top-level expression is expanded
  - Custom classes define `_eval_expand_hint(**hints)` to implement their own expansion behavior; the method should only expand the top level — `expand()` handles recursion
  - Metahints (e.g., `force`, `modulus`) are passed through to `_eval_expand_hint` methods; `deep` is handled by `expand()` itself and not forwarded
- `expand_power_base(expr, deep, force)` — wrapper around `expand(power_base=True)`; splits `(a*b)**e` into `a**e * b**e`; `deep=False` applies only at top level (does not descend into subexpressions like `sin(...)`)
- `expand_power_exp`, `expand_complex`, `expand_trig`, `expand_log`, `expand_func`, `expand_mul` — similar single-hint expand wrappers
- `_coeff_isneg(a)` — returns True only if the leading numeric factor is a negative Number; a symbol with `negative=True` assumption returns False (coeff is implicitly 1)
- `count_ops(expr, visual)` — tallies arithmetic operations in an expression; skips S.One entirely (0 ops); classifies other rationals by sign (NEG) and denominator (DIV); handles Add terms by classifying each as ADD or SUB; corrects count when leading term is negative (e.g., `-x + y`)
- `nfloat(expr, n, exponent)` — converts all Rationals in an expression to Floats; by default protects exponents via Dummy replacement
  - When `exponent=True`, requires separate pass to float-ify Integer exponents because `Pow._eval_evalf` special-cases them

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

- `CantSympify` — mixin trait; classes inheriting this are blocked from sympification even if their base type (e.g., `dict`) would normally be convertible

---

## Comparison

### [`relational.py`](relational.py)
`Eq`, `Ne`, `Lt`, `Le`, `Gt`, `Ge` — symbolic relational expression nodes (unevaluated comparison objects). `Relational` base dispatches by operator string.

- `_Greater` / `_Less` — internal base classes providing `.gts` (greater-than side) and `.lts` (less-than side) properties; `_Greater` maps gts→arg[0], lts→arg[1]; `_Less` swaps them (gts→arg[1], lts→arg[0])
- These are the AST nodes returned when `Expr.__ge__`/`__lt__`/etc. in `expr.py` cannot resolve a comparison to True/False
- `Equality.__new__` — multi-stage evaluation: (1) delegates to `_eval_Eq` hooks on either side; (2) structural equality check; (3) finiteness check — if both sides are non-finite (infinite), returns True; if one finite and one not, returns False; (4) difference-based zero test with non-commutative guard; (5) ratio-based numerator/denominator analysis
- `Unequality.__new__` — delegates to `Equality`; if result is a `BooleanAtom` (True/False), returns its negation; if equality is indeterminate, falls through to create an unevaluated `Relational` node

---

## Containers and Utilities

### [`containers.py`](containers.py)
`Tuple` — SymPy-aware immutable tuple wrapping `Basic`; `Dict` — SymPy-aware immutable dictionary.

- `Tuple.tuple_count(value)` — counts occurrences of value; named `tuple_count` (not `count`) because `Basic.count` already defines expression-tree traversal counting, which would conflict
- `Tuple.index(value)` — returns first index of value; custom implementation handles None start/stop arguments that Python's built-in tuple.index rejects

### [`rules.py`](rules.py)
`Transform` — immutable callable mapping (key→value with optional filter predicate).

### [`cache.py`](cache.py)
`cacheit` — SymPy-specific memoization wrapper; selects between `fastcache.clru_cache` (if installed) or the backported `lru_cache` from `compatibility.py` as the underlying cache engine.

- `__cacheit` — fallback decorator (used when fastcache unavailable); wraps `compatibility.lru_cache`; catches `TypeError` on unhashable args and silently falls back to calling the original uncached function
- `CACHE` — global registry (`_cache` list) with `print_cache()` and `clear_cache()` helpers

### [`decorators.py`](decorators.py)
`_sympifyit` / `__sympifyit` — auto-converts second argument to SymPy type before calling wrapped function; when a fallback return value is specified and the operand has `_op_priority`, skips conversion (assumes external class handles SymPy interop)
- `call_highest_priority(method_name)` — decorator for binary special methods; delegates to the other operand's reflected method if it has higher `_op_priority`; silently falls back if reflected method missing
- `deprecated` — deprecation warning decorator

### [`compatibility.py`](compatibility.py)
Python 2/3 polyfills and backported utilities: `string_types`, `integer_types`, `with_metaclass()`, `iterable()`, `ordered()`, `as_int()`.

- `default_sort_key(item, order)` — canonical ordering key for arbitrary objects (not just SymPy types); more robust than `sort_key()` method
  - Handles plain ints/floats by attempting sympification, strings by wrapping in a tuple key, dicts/sets by recursively sorting keys
  - When sympification fails (e.g., lambda functions), catches `SympifyError` and falls through to string-based key with class index 0
- `as_int(n)` — converts argument to Python `int` with strict equality validation; raises `ValueError` if `int(n) != n` (e.g., `sqrt(10)` → 3 but not equal)
  - Catches `TypeError` from the equality check (e.g., objects where `int(x) != x` is undefined) and re-raises as `ValueError`
- `iterable(i, exclude=(str, dict, NotIterable))` — checks if object is iterable in the SymPy sense; if object has `_iterable` boolean attribute, returns it directly (bypasses both `iter()` check and type exclusion); otherwise calls `iter()` then applies `exclude` filter

- `lru_cache` — backported LRU memoization decorator (used when `functools.lru_cache` unavailable); three branches by maxsize: 0 (no cache), None (unbounded), else size-limited with doubly-linked-list eviction
  - Size-limited branch uses thread lock; releases lock during user_function call, then re-acquires; if another thread inserted same key during release, skips link update and only increments miss counter
  - Size-limited branch catches `TypeError` on unhashable args (e.g., lists) and falls through to uncached call
- `_make_key` — builds hashable cache key from args/kwds; fast-path: single positional arg of primitive type (int, str, frozenset, NoneType) with no kwds returns the raw arg directly, avoiding wrapper allocation

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

- `_cycle_permute(l)` — computes canonical rotation of a cyclic sequence; finds all positions of the minimum element, builds sublists between consecutive minima, picks the lexicographically smallest rotation; used in `_hashable_content` so cyclically equivalent products hash identically
- `_is_scalar(e)` — helper classifying scalars (Integer, Float, Rational, Number, commutative Symbol)
