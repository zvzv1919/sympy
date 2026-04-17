# SymPy Root Catalog

## Architecture Overview

SymPy is organized into submodules by mathematical domain. Key routing rules:
- **Expression tree internals** (Pow, Add, Mul, Expr, Basic, Symbol, Number, _eval_subs, _eval_power) → `core/`
- **Polynomial algebra** (Gröbner bases, factorization, domains, modules, number fields) → `polys/`
- **Mathematical function classes** (trig, hyperbolic, piecewise, elliptic, combinatorial number sequences) → `functions/`
- **Expression transformation/simplification** (fu, trigsimp, hyperexpand, combsimp) → `simplify/`
- **Pretty/code/string printing** (LaTeX, Fortran, C, Python, MathML, lambda repr) → `printing/`
- **Equation solving** (algebraic, ODE, recurrence, diophantine, bivariate) → `solvers/`
- **Numeric code generation from expressions** (lambdify, autowrap, codegen, ufuncify) → `utilities/`
- **AST node definitions for code generation** (not expression printing) → `codegen/`
- **Property inference on symbols** (handlers, sathandlers, fact rules) → `assumptions/`
- **Propositional/boolean logic** (And, Or, satisfiability, DPLL) → `logic/`
- **Probability and statistics** (random variables, distributions, PDF/CDF) → `stats/`
- **New-style 3D vector algebra** (CoordSysCartesian, Point, divergence, curl) → `vector/`
- **Classical mechanics vectors** (ReferenceFrame, dynamicsymbols, mechanics printing) → `physics/vector/`

## Glossary

- **_eval_subs / _eval_power / _eval_is_***: internal hooks on expression classes. `Pow._eval_power` is in `core/power.py`.
  - Function classes (e.g. `Abs`) define their own `_eval_power` in `functions/`. `_eval_is_*` for core types in `core/`, NOT `assumptions/handlers/`.
- **handlers**: assumption inference callbacks in `assumptions/handlers/`, NOT in `logic/`.
- **lambdify / lambdastr / ufuncify / autowrap**: numeric callable generation in `utilities/`, NOT `codegen/` or `printing/`.
- **codegen (utilities/)**: generates Fortran/C/Julia **source files** with `Routine`, `FCodeGen`, `CCodeGen`.
- **codegen/ (submodule)**: defines AST node classes (`ast.py`). Does NOT generate source or callable code.
- **fcode / FCodePrinter**: Fortran expression printer in `printing/fcode.py`, NOT in `codegen/`.
- **ring_series**: formal power series via polynomial ring operations, in `polys/`, NOT `series/`.

## Top-Level Files

### [`abc.py`](abc.py)
Pre-defined single-letter Symbol instances (latin and greek) for interactive convenience.
- `clashing` — dict of multi-letter clashing names that conflict with SymPy objects.
- NOT where Symbol or Dummy classes are defined (those are in `core/symbol.py`).

### [`galgebra.py`](galgebra.py)
Deprecated stub redirecting to the external `galgebra` package.

## Submodules

### [`core/`](core/catalog.md)
Foundational expression tree: base classes, arithmetic operations, and evaluation hooks.
- `basic.py` — `Basic` base class: `subs` (simultaneous mode uses product of two Dummy placeholders to trigger `Subs` on bound variables), `_has`, `matches`, `atoms`, `xreplace`, structural traversal.
- `expr.py` — `Expr` class: `as_coefficient`, `leadterm`, `extract_multiplicatively`, `as_leading_term`; `is_rational_function` (checks if expression is ratio of polynomials; returns False for NaN/infinities), `is_algebraic_expr`; `series`.
- `power.py` — `Pow` class: `_eval_power`, `_eval_subs` (exponent splitting for substitution).
- `add.py` — `Add` class: `flatten` (infinity filtering, order processing), `_eval_as_leading_term`.
- `mul.py` — `Mul` class: `flatten`, `as_content_primitive`, `as_two_terms`.
- `numbers.py` — `Integer`, `Rational`, `Float`, `ImaginaryUnit`, `AlgebraicNumber`, `Exp1` (Euler's number `E`; rewrite methods express `e` via `sin`/`cos` of imaginary unit), `_eval_power`, `mod_inverse`; `igcd`.
- `function.py` — `Function`, `Derivative.__new__`, `expand()`, `_mexpand` (combined multinomial+mul expansion, optionally iterating to fixed point), `AppliedUndef`, `WildFunction`, `nfloat`; `_coeff_isneg`.
- `containers.py` — `Tuple`: immutable symbolic sequence wrapper; `tuple_count` (renamed from `count` to avoid `Basic.count` conflict); `Dict`: immutable symbolic dict wrapper.
- `cache.py` — `cacheit` memoization decorator; `__cacheit_debug` (debug mode: always calls both cached and uncached, verifies immutability via hash, raises RuntimeError on inconsistency); `_getenv` (reads `SYMPY_USE_CACHE` env var).
- `relational.py` — `Relational`, `Equality`, `GreaterThan`, `StrictLessThan`; `as_set` (univariate inequality → real set; raises NotImplementedError for multivariate).
- `exprtools.py` — `gcd_terms`, `_gcd_terms` (extracts shared divisor from additive components; returns zero/zero/one for empty input), `factor_terms`: GCD extraction with non-commutative masking.
- `evalf.py` — `hypsum`, `evalf_sum`: numerical evaluation, hypergeometric series summation, precision.
  - `do_integral`: numerical definite integration; optimizes constant-integrand case by simplifying bounds difference.
  - `get_integer_part`: floor/ceiling with near-integer handling.
- `logic.py` — fuzzy logic (`fuzzy_and`, `fuzzy_or`, `fuzzy_not`) AND internal propositional `Logic` class with `fromstring` (space-delimited boolean formula parser), `And`/`Or`/`AndOr_Base`.
  - `And.expand`: distributes conjunction over disjunction to convert to sum-of-products (disjunctive normal) form. NOT `logic/boolalg.py` `to_dnf`.
- `symbol.py` — `Symbol`, `Dummy`, `Wild` class definitions; `operations.py` — `AssocOp`, `LatticeOp.__new__`.
- `sympify.py` — `sympify()`: convert arbitrary objects to SymPy types; `kernS` (string-to-expression with placeholder insertion to prevent autosimplification of products with parenthesized sums; falls back to original string on parse error).
- `compatibility.py` — `ordered`: conservative tie-breaking sort with successive keys; fallback/warn on unresolved ties.
- `assumptions.py` — `_ask`: recursive property resolution with anti-recursion guard and shuffled prerequisites.
  - `make_property`: copy-on-write property factory (copies shared class-level assumptions dict before mutation).
  - `ManagedProperties` metaclass: protects subclasses from inheriting parent's cached static assumption values by replacing with dynamic descriptors.
- `facts.py` — `deduce_alpha_implications`: transitive closure of inference rules with contrapositive generation; `apply_beta_to_alpha_route` (extend single-premise forward-chaining tables with conjunctive AND rules via fixed-point iteration).
  - `rules_2prereq`: builds reverse dependency (prerequisite) table mapping each conclusion back to its source propositions, stripping negation wrappers from both sides.
- `multidimensional.py` — `vectorize`: decorator lifting scalar functions to element-wise nested collection operations.
- `trace.py` — `Tr`: generic trace for non-commutative products; extracts commutative prefactors from `Mul`.

### [`polys/`](polys/catalog.md)
Polynomial algebra, domains, Gröbner bases, factorization, root isolation, and module theory.
- `polytools.py` — `Poly` class, `degree`, `primitive`, `factor`, `gcd`, `groebner` (high-level API); `quo_ground`/`exquo_ground` (truncating vs exact ground division; exact variant raises ExactQuotientFailed).
- `polyclasses.py` — `GenericPoly` (base class for low-level polynomial representations): `ground_to_ring`, `ground_to_field`, `ground_to_exact` (convert approximate coefficient domain to exact); `DMP`, `DMF`, `ANP`.
- `polyroots.py` — `roots_quadratic` (inner `_sqrt` extracts perfect-square factors from discriminant), `roots_cubic`, `roots_quartic`: algebraic root formulas; `preprocess_roots` strips symbolic coefficients.
- `rootisolation.py` — `dup_isolate_real_roots_list` (VAS continued fractions); `_classify_point`, `dup_isolate_complex_roots_sqf`.
  - Collins-Krandick complex root isolation: `_vertical_bisection`/`_horizontal_bisection` (split rectangles, refine straddling boundary intervals).
  - `_traverse_quadrants`: convert quadrant sequences to winding-number rules with selective edge/corner exclusion via compass directions.
- `polyconfig.py` — `configure`: initializes polynomial algorithm settings from `SYMPY_`-prefixed OS environment variables at import time (uses `eval`, falls back to raw string on NameError); `setup`, `query`, `using` context manager.
- `polyoptions.py` — `Options._init_dependencies_order`: resolves processing order of polynomial options via topological sort of `before`/`after` declarations; raises RuntimeError on cycles.
- `orthopolys.py` — `hermite_poly`, `laguerre_poly`, `legendre_poly`: computational polynomial generators.
- `factortools.py` — low-level factorization: Hensel lifting, Zassenhaus algorithm.
- `numberfields.py` — `minimal_polynomial`, algebraic field extensions.
- `distributedmodules.py` — sparse distributed module elements (free modules over multivariate rings).
  - `sdm_add` (merge two sparse representations, deletes entries when coefficients cancel to zero), `sdm_strip`, `sdm_monomial_divides`, `sdm_nf_mora`.
- `densetools.py` — `dup_real_imag` (split univariate poly into real/imag bivariate parts via x→x+iy; uses k%4 cycle for i powers), `dup_mirror`, `dup_extract`.
  - `_dup_right_decompose`/`_dup_left_decompose`: recover inner/outer components of polynomial functional decomposition f=g(h).
- `ring_series.py` — `rs_tanh`, `rs_exp`, `rs_asin`, `rs_LambertW`, `rs_series_inversion`: formal power series via polynomial rings.
- `polytools.py` also has `terms_gcd` (extract shared monomial factor from addends, with `deep` flag for recursive traversal into function args).
- `agca/` — abstract algebra: `modules.py` (free modules), `homomorphisms.py` (module maps).
- `fields.py` — `FracElement` arithmetic: dispatch for nested quotient-of-quotient domain operations.
- `euclidtools.py` — `dmp_cancel`, `dup_cancel` (cancel common factors in rational functions; clears denominators for field domains before GCD), `dmp_content`, `dmp_primitive`, `dmp_gcd`, `dmp_inner_gcd`.
- `rationaltools.py` — `together`: combine fractional subexpressions into a single quotient (always recurses into Pow base, but only recurses into exponent when `deep=True`).
- `partfrac.py` — `apart`: partial fraction decomposition; handles non-commutative expressions by splitting commutative/non-commutative factors.
- `modulargcd.py` — modular GCD algorithms; `_integer_rational_reconstruction` (recover a/b from residue mod composite via extended Euclidean).
- `monomials.py` — `Monomial`: product-of-powers representation with `__mul__`, `__div__`, `gcd`, `lcm`; `MonomialOps` (code-generated fast operations).
- `domains/` — coefficient domains: `Domain` base class (`map`: recursively convert nested lists, distinguishing sublists from leaf values).
  - `ExpressionDomain` (auto-cancels via `.cancel()` after every arithmetic op), `IntegerRing`, `RationalField`.
  - `old_fractionfield.py` — `FractionField`: field of rational functions over indeterminates; sign predicates (`is_positive`, etc.) delegate to leading coefficient of numerator.
  - `mpelements.py` — `MPContext.to_rational`: continued-fraction rational approximation with bounded denominator.
  - `groundtypes.py` — fallback `python_sqrt`/`python_factorial` via mpmath when no gmpy available.
  - `modularinteger.py` — `ModularIntegerFactory`: cached type generation for ℤ/nℤ.

### [`functions/`](functions/catalog.md)
Mathematical function classes (symbolic, unevaluated). Defines the functions, does NOT simplify them.
- `elementary/trigonometric.py` — `sin`, `cos`, `tan`, `sec`, `csc`, `cot`, `acot`, `atan`, `asin`, `acos` and other inverse trig; `ReciprocalTrigonometricFunction` (base for sec/csc/cot with `taylor_term`); `_pi_coeff`, `_eval_aseries`.
- `elementary/piecewise.py` — `Piecewise`, `piecewise_fold` (distribute operations over branches; Boolean outer expr→Or/And/Not instead of Piecewise), `_sort_expr_cond`, `_eval_integral`.
- `elementary/hyperbolic.py` — `sinh`, `cosh`, `tanh` and inverses.
- `elementary/complexes.py` — `conjugate`, `Abs` (`_eval_power`: odd integer exponent reduces by 1; skips -1 to avoid circularity), `arg`, `re`, `im`, `sign`, `transpose`, `adjoint`.
- `elementary/miscellaneous.py` — `Min`, `Max` (`MinMaxBase._is_connected`: two-pass ordering check with factor_terms fallback; `_find_localzeros`), `root`, `real_root`.
- `elementary/integers.py` — `floor`, `ceiling`, `frac`: rounding functions with `_eval_nseries`.
- `special/tensor_functions.py` — `KroneckerDelta` (fermi-level index logic, second quantization), `LeviCivita`.
- `special/bessel.py` — Bessel functions (`besselj`, `bessely`, `besseli`, `besselk`), spherical Bessel (`jn`, `yn`), `jn_zeros` (spherical Bessel zeros; initial estimate n+π, spaced by π), Airy functions (`airyai`, `airybi`).
- `special/polynomials.py` — symbolic orthogonal polynomial classes: `laguerre`, `hermite`, `chebyshev`, etc.
- `special/elliptic_integrals.py` — `elliptic_f`, `elliptic_e`, `elliptic_k`, `elliptic_pi`.
- `combinatorial/numbers.py` — `fibonacci`, `bernoulli`, `catalan`, `harmonic` (`_eval_expand_func`: decomposes H(n±k) into partial terms), `euler`, `nP`, `nC` (multiset support).
  - `nT`: partition count (integer→identical items, sequence→multiset; detects n-th roots via repeated self-convolution).
- `combinatorial/factorials.py` — `factorial`, `subfactorial` (derangement count; `_eval_is_even` returns True for odd nonneg args), `binomial`, `RisingFactorial`, `FallingFactorial`, `factorial2`.
- `elementary/exponential.py` — `log` (eval handles `AccumBounds`, imaginary args, rationals), `exp`, `exp_polar`.
- `special/hyper.py` — `hyper` (generalized hypergeometric function), `meijerg` (Meijer G-function class: `fdiff`, `_diff_wrt_parameter`).
  - `HyperRep`: base for branched pFq representations; `_eval_rewrite_as_nonrep` converts to Piecewise (|x|<1 vs |x|>1) for analytic continuation.
  - `hyper.radius_of_convergence`: checks non-positive integer parameter cancellation between numerator/denominator lists; returns 0 if denominator entries can't be cancelled.
- `special/gamma_functions.py` — `gamma`, `loggamma`, `digamma`, `trigamma`, `polygamma` (iterated log-derivative of Γ), `uppergamma`, `lowergamma`.
  - `polygamma._eval_aseries`: asymptotic expansion at ∞ via Bernoulli numbers; intentionally returns extra terms for higher orders.
- `special/error_functions.py` — `TrigonometricIntegral` (base for `Si`, `Ci`, `Shi`, `Chi`), `FresnelIntegral` (base for `fresnels`/`fresnelc`), `Ei`, `li`, `Li`, `erf`; Fresnel integrals with `taylor_term` recurrence using previous terms.
- `special/zeta_functions.py` — `lerchphi` (Lerch transcendent Φ(z,s,a) with `_eval_expand_func`: reduces to polylog/zeta sums when a is rational), `polylog`, `zeta`, `dirichlet_eta`, `stieltjes`.
- Caveats: `special/polynomials.py` defines symbolic classes; `polys/orthopolys.py` has computational generators.

### [`simplify/`](simplify/catalog.md)
Expression transformation and simplification algorithms. Operates ON functions, does not define them.
- `fu.py` — trigonometric simplification rules: `TR10i` (product-to-sum with sqrt(3):1 ratio handling), `TR11` (double-angle to half-angle reduction, with optional base arg for targeting auto-simplified angles); `hyper_as_trig`.
- `hyperexpand.py` — `hyperexpand`, `ReduceOrder`, `add_formulae` (lookup table of hypergeometric identities), `add_meijerg_formulae` (Meijer G-function formula table with matcher functions), `_meijergexpand` (Slater expansion).
  - `Hyper_Function.build_invariants`: computes invariant vector by bucketing parameters mod 1; skips sorting when bucket keys are symbolic (`Mod` instances).
  - `Hyper_Function.difficulty`: estimates transformation steps between hypergeometric functions via residue-class bucket comparison; returns -1 if impossible.
  - Shift operators (`ShiftA`, `ShiftB`); unshift operators (`MeijerUnShiftA`–`MeijerUnShiftD`: raise ValueError when auxiliary polynomial constant term cancels).
- `trigsimp.py` — `trigsimp_groebner`: Gröbner-basis trig/hyperbolic simplification (substitutes I with Dummy, adds I²+1 to ideal).
- `ratsimp.py` — `ratsimpmodprime`: simplify rational expressions modulo a Gröbner basis ideal.
- `radsimp.py` — `collect`: group additive terms by pattern (supports derivatives, exact-match flag); `collect_sqrt` (group terms by shared half-integer power factors, treats I=sqrt(-1) as a radical), `collect_const`.
- `combsimp.py` — `combsimp`, `_rf` (rising factorial simplification for combinatorial expressions).
- `powsimp.py` — `powsimp`, `powdenest`: power/exponent simplification.
- `simplify.py` — `simplify()`: general-purpose dispatch; `logcombine`; `separatevars`; `bottom_up`; `nthroot`; `sum_add` (merge Sums: same-limits adds functions, same-function merges adjacent ranges).
  - `besselsimp` (Bessel simplification: half-integer order→trig via spherical rewrite, imaginary arg rewrites between J/I types).
- `cse_main.py` — `cse` (common subexpression elimination), `opt_cse` (pre-optimization: extracts shared args between Add/Mul pairs; asymmetric handling when first expr becomes empty).
- `epathtools.py` — `EPath`: XPath-like tool for selecting and transforming expression sub-trees.

### [`printing/`](printing/catalog.md)
String/code representation of SymPy expressions. Outputs text, NOT callable code or AST nodes.
- `ccode.py` — `CCodePrinter`, `ccode`: C expression printing; `_print_For` (only supports Range iterables, raises NotImplementedError for lists/tuples).
- `fcode.py` — `FCodePrinter`, `fcode`, `indent_code`: Fortran expression printing.
- `lambdarepr.py` — `LambdaPrinter`, `NumExprPrinter` (numexpr string backend; blacklists matrices/collections), `_print_Piecewise`.
- `python.py` — `python()`: generate executable Python code string for an expression.
- `latex.py` — `LatexPrinter`, `latex()`: LaTeX representation.
- `jscode.py` — `JavascriptCodePrinter`, `jscode`: JavaScript expression printing; `_print_MatrixElement` (row-major 2D→1D index flattening).
- `codeprinter.py` — `CodePrinter`: base class for all code printers; `_print_Mul` (splits numerator/denominator; `evaluate=False` for non-(-1) negative rational exponents to prevent simplification).
- `llvmjitcode.py` — `llvm_callable`: JIT-compile expressions to machine code via LLVM.
- `precedence.py` — bracket-necessity system; `precedence_PolyElement` (4-way dispatch: generator→Atom, ground→delegate, term→Mul, multi-term→Add), `precedence_FracElement`.
- `pretty/pretty.py` — `PrettyPrinter`: 2D human-readable output; `_print_meijerg` (4-parameter 2×2 grid with annotated G symbol), `_print_hyper`, `_print_Integral`, `_print_Matrix`.
- `defaults.py` — `DefaultPrinting` mixin: aliases `__repr__` to `__str__` so elements in Python lists/dicts display in human-readable form; forces default (lex) ordering regardless of global setting.
- `str.py` / `repr.py` — default `str()` / `repr()` printers; `_print_Pow` (uses identity `is` checks, not `==`, to avoid matching -0.5 as -S.Half); `_print_FiniteSet` truncates sets >10 elements.
- `octave.py` — `OctaveCodePrinter`: Octave/MATLAB code; rewrites spherical Bessel via cylindrical Bessel.
- `julia.py` — `JuliaCodePrinter`: Julia code; restructures `Piecewise` assignments in non-inline mode.
- `theanocode.py` — `TheanoPrinter`: Theano graph builder; `_print_Piecewise` uses `np.nan` fallback for single-branch.

### [`solvers/`](solvers/catalog.md)
Equation solving: algebraic, ODE, PDE, recurrence, diophantine, systems.
- `solvers.py` — `solve`, `_solve`, `unrad`, `solve_linear_system`, `solve_undetermined_coeffs` (match polynomial coefficients; returns None if residual system still contains the variable), `check_assumptions`: main algebraic solver.
- `solveset.py` — `solveset`, `linsolve`, `linear_eq_to_matrix`: new-style set-based solver API; `_invert_real` (invert power/trig/exp expressions; even-numerator rational exponents yield both ± roots).
- `recurr.py` — `rsolve`, `rsolve_poly`, `rsolve_hyper`: linear recurrence equation solvers.
- `ode.py` — ODE classification and solution: `classify_ode`, `dsolve`, `_frobenius`, `ode_2nd_power_series_ordinary` (truncated power series at non-singular points via recurrence construction), Lie group methods, `odesimp`.
- `diophantine.py` — `diophantine`, `sum_of_four_squares`, `PQa` (generator yielding 6-tuple sequences from periodic expansion of (P+√D)/Q for Pell equation solving), `_transformation_to_normal`: integer/diophantine equation solving.
- `bivariate.py` — `bivariate_type`: solve equations with two variables via back-substitution; `_filtered_gens` (extract symbol-dependent generators from a polynomial, deduplicating multiplicative inverses by preferring denominator-free form).
- `inequalities.py` — `solve_poly_inequality`, `solve_rational_inequalities`, `reduce_abs_inequality` (nested absolute-value decomposition into piecewise cases via Cartesian product of branches).
- `polysys.py` — `solve_generic` (Gröbner-based polynomial system solver), `solve_biquadratic`.
- `deutils.py` — `ode_order`: utility to determine the order of a differential equation.

### [`utilities/`](utilities/catalog.md)
Utility functions: numeric code generation, iterables, source inspection, multiset enumeration.
- `lambdify.py` — `lambdify`, `lambdastr`: convert expressions to callable Python/NumPy functions.
- `autowrap.py` — `autowrap`, `ufuncify`, `CythonCodeWrapper._partition_args`: compile to binary.
- `codegen.py` — `Routine`, `CCodeGen`, `OctaveCodeGen`: generate Fortran/C/Octave source files; `Variable` (typed variable with `get_datatype` for language-specific type lookup; raises error listing supported languages on unknown language).
- `iterables.py` — `partitions`, `generate_bell`, `_set_partitions`, `topological_sort`, `numbered_symbols`, `generate_derangements`, `minlex`, combinatoric generators.
  - `interactive_traversal`: user-guided step-by-step navigation through expression tree with re-prompt on invalid input.
- `randtest.py` — `verify_numerically` (test symbolic equivalence by substituting random complex values for all free symbols), `test_derivative_numerically`; `_randint`, `_randrange`: deterministic pseudo-random generators.
- `misc.py` — `replace` (simultaneous multi-pattern text substitution, longer keys matched first), `translate` (character-level replacement/deletion), `_replace` (regex-compiled helper).
- `decorator.py` — `threaded_factory` (decorator: maps function over iterables/matrices; silently returns input unchanged if container constructor rejects list), `threaded`, `xthreaded`.
- `enumerative.py` — `MultisetPartitionTraverser`: multiset partition enumeration (Knuth's algorithm); `factoring_visitor` (interpret partition state + prime bases to enumerate integer factorizations), `list_visitor`.
- `benchmarking.py` — `BenchSession`: py.test-based performance measurement; `print_bench_results` formats timing output with decimal-point alignment across time-unit columns (s/ms/μs/ns).
- `runtests.py` — `_doctest`, `SymPyOutputChecker`: test runner with float comparison, matplotlib backend management.

### [`codegen/`](codegen/catalog.md)
AST node definitions for code generation (abstract syntax tree). NOT source file generation (that's `utilities/codegen.py`).
- `ast.py` — `Assignment`, `CodeBlock`, `Variable`, `Declaration`, `FunctionPrototype`.

### [`assumptions/`](assumptions/catalog.md)
Property inference system for symbolic objects (is_positive, is_integer, etc.).
- `ask.py` — `Q` predicate definitions (`Q.transcendental`, `Q.algebraic`, etc.), `ask()` query function.
- `assume.py` — `Predicate.eval`: handler dispatch with contradiction detection; `assuming` context manager.
- `refine.py` — `refine`, `refine_atan2`, `refine_Pow`: simplify expressions under assumptions.
- `handlers/calculus.py` — `AskFiniteHandler`: boundedness inference for `Add`, `Mul`, `Pow` (|base|≥1 + unbounded exp → unbounded), `log`, `exp`.
- `handlers/sets.py` — `AskImaginaryHandler`, `AskRealHandler`, `AskIntegerHandler`: set-membership queries.
- `handlers/matrices.py` — `AskOrthogonalHandler`, `AskUnitaryHandler`, `AskDiagonalHandler`: matrix property inference.
- `sathandlers.py` — `register_fact`: register assumption rules for SAT-based inference.
- NOT propositional logic (that's `logic/`). This module infers numeric properties of expressions.

### [`logic/`](logic/catalog.md)
Propositional and boolean logic: representation, inference, satisfiability.
- `boolalg.py` — `And`, `Or`, `Not`, `Implies`, `Equivalent`: boolean algebra classes.
- `algorithms/dpll2.py` — `SATSolver`, `_vsids_calculate`: DPLL-based SAT solver.
- `inference.py` — `satisfiable`, `valid`, `entails`: logical inference functions.
- NOT assumption handlers (those are in `assumptions/`).

### [`physics/`](physics/catalog.md)
Physics subpackages: quantum mechanics, classical mechanics, optics, units, second quantization.
- `secondquant.py` — `Dagger` (Hermitian conjugate: reverses factor order over products; NOT matrix adjoint in `matrices/expressions/adjoint.py`).
  - `AntiSymmetricTensor`: antisymmetric two-electron integral; `_sortkey` (canonical index ordering: anonymous/dummy indices get higher sort priority than named indices).
  - Creation/annihilation operators, `Commutator`, Wick's theorem.
  - `evaluate_deltas`: simplify KroneckerDelta in products under Einstein summation; respects fermi-level index priority and equal-information checks.
  - `simplify_index_permutations`, `PermutationOperator`.
- `mechanics/functions.py` — `inertia` (construct inertia dyadic from frame + 6 components; TypeError if frame arg invalid), `msubs`, `angular_momentum`, `kinetic_energy`.
  - `_smart_subs`: intelligent substitution; selectively simplifies only subexpressions whose denominators become zero.
- `vector/frame.py` — `ReferenceFrame`: `__getitem__` (dual dispatch: numeric→coord var, string→basis vector), `orient` (Body/Space/Quaternion/Axis).
  - `set_ang_vel`/`set_ang_acc`: store bidirectionally with negated reverse; wraps scalar 0 to Vector(0).
  - `_w_diff_dcm`, `variable_map`.
- `vector/printing.py` — `VectorLatexPrinter`, `VectorPrettyPrinter`: Newton dot notation for time-derivatives.
  - `init_vprinting`: globally enables compact temporal derivative display (e.g. f') across all output formats.
- `quantum/cg.py` — `Wigner3j` (3j coupling coefficients with own `_pretty`/`_latex` methods for 2D grid layout), `CG` (Clebsch-Gordan coefficients, subclass of Wigner3j).
- `quantum/hilbert.py` — Hilbert space algebra: `DirectSumHilbertSpace`, `TensorProductHilbertSpace`, `TensorPowerHilbertSpace`, `ComplexSpace`, `L2` (square-integrable function space; validates Interval input type), `FockSpace`.
- `quantum/shor.py` — Shor's quantum factoring algorithm: `shor`, `period_find` (quantum order-finding via QFT).
- `quantum/qasm.py` — QASM circuit description parser: `Qasm`, `get_index` (reverses register declaration order to qubit indices via `flip_index`).
- `quantum/qexpr.py` — `QExpr`: base class for quantum expressions; `_eval_adjoint` (fallback creates Dagger wrapper, preserves Hilbert space attribute).
- `quantum/represent.py` — `represent`: convert symbolic quantum expressions to matrix form; fallback dispatch (state vectors → inner product, operators → expectation value) when direct `_represent` is not implemented.
- `quantum/` — states (`Ket`, `Bra`, `Wavefunction` with `normalize`/`norm`), spin eigenstates, operator sets.
- `quantum/commutator.py` — `Commutator`: Lie bracket [A,B]=AB-BA with scalar prefix extraction and canonical ordering.
- `quantum/density.py` — `Density`: statistical mixture / density operator; `doit` expands to weighted outer products (handles superposition cross-terms).
- `quantum/operator.py` — `Operator`, `IdentityOperator` (identity simplification in multiplication), `OuterProduct`.
- `quantum/matrixutils.py` — type-dispatch conversions between sympy Matrix, numpy ndarray, scipy sparse (stub classes when libraries unavailable).
- `quantum/tensorproduct.py` — quantum Kronecker products and simplification (NOT `tensor/`).
- `quantum/identitysearch.py` — gate identity BFS discovery; `GateIdentity` (precomputes equivalent permutations), `is_degenerate` (checks permutation membership in known identity sets); `lr_op`, `ll_op` rule operations.
- `quantum/circuitutils.py` — `kmp_table`, `find_subcircuit` (KMP pattern matching for gate sequences), `replace_subcircuit`, `random_reduce`.
- `quantum/circuitplot.py` — circuit diagram visualization (NOT `plotting/`).
- `vector/fieldfunctions.py` — `scalar_potential`, `scalar_potential_difference` (potential difference between points; uses scalar value directly if input is not a vector), `is_conservative`, `is_solenoidal`.
- `optics/utils.py` — `lens_formula` (thin-lens equation solver; handles infinite distances via symbolic Limit to avoid division-by-zero), `mirror_formula`, `hyperfocal_distance`.
- `hep/gamma_matrices.py` — Dirac gamma matrix traces and simplification for high-energy physics.
- `wigner.py` — Wigner 3j/6j/9j, Clebsch-Gordan, Gaunt coefficients (spherical harmonic integrals).
- `units.py` — legacy physical units; `Unit` class with `is_positive=True`.
- `unitsystems/units.py` — `UnitSystem`: `print_unit_base` (express derived unit in basis; sorts by decreasing power, normalizes scale factors), `get_unit`, `extend`.
- `unitsystems/dimensions.py` — `DimensionSystem`: `get_dim` (lookup by string name/symbol or Dimension object; returns None if not found), `extend`, `sort_dims`.
- `unitsystems/simplifiers.py` — `dim_simplify`: dimensional analysis simplification.

### [`matrices/`](matrices/catalog.md)
Matrix classes and matrix expression algebra.
- `matrices.py` — `DeferredVector` (symbolic vector for `lambdify`; `__getitem__` raises IndexError on negative index), `MatrixBase`: core matrix operations.
  - Linear system solvers: `gauss_jordan_solve` (row-reduction with parametric free-variable solutions), `LUsolve`, `cholesky_solve`, `QRsolve`, `pinv_solve`.
- `dense.py` — `MutableDenseMatrix`, `wronskian`, `hessian`, `casoratian`; `symarray` (create numpy ndarray of symbols with index-derived names; empty prefix yields `_0, _1, ...`; passes **kwargs to Symbol for assumptions).
- `sparse.py` — `SparseMatrix`: dictionary-backed sparse matrix; `add` (zero-entry cleanup on cancellation), `_LDL_solve` (L·D·L^T forward/diagonal/back-solve), `_cholesky_solve`.
- `immutable.py` — `ImmutableMatrix`, `_eval_Eq`: immutable matrix with equality support.
- `expressions/determinant.py` — `Determinant`, `det`; `refine_Determinant` (orthogonal→1, singular→0, unit_triangular→1).
- `expressions/inverse.py` — `Inverse` (symbolic matrix inverse, subclass of MatPow); `refine_Inverse` (orthogonal→transpose, unitary→conjugate, singular→error).
- `expressions/transpose.py` — `Transpose`, `refine_Transpose`: symbolic transpose operations.
- `expressions/matmul.py` — `MatMul`, `_eval_trace`: symbolic matrix multiplication and trace.

### [`vector/`](vector/catalog.md)
New-style 3D vector algebra module (sympy.vector). NOT physics.vector.
- `point.py` — `Point.position_wrt`: position vector between points in coordinate systems.
- `coordsysrect.py` — `CoordSysCartesian`: coordinate system definition; `scalar_map` (express one frame's coordinate variables in another's, combining rotation and translation).
- `orienters.py` — `QuaternionOrienter`, `BodyOrienter`, `SpaceOrienter`, `AxisOrienter`: coordinate system orientation transformers.
- Caveats: Separate from `physics/vector/` which uses `ReferenceFrame`-based classical mechanics vectors. `scalar_map` is here, NOT in `physics/vector/frame.py`.

### [`stats/`](stats/catalog.md)
Probability and statistics: random variables, distributions, expected values.
- `crv_types.py` — continuous distribution classes: `BetaDistribution`, `KumaraswamyDistribution`, etc.
- `crv.py` — `ContinuousPSpace` (`probability`: univariate via `where`; multivariate fallback computes density of lhs-rhs and reduces to univariate), `SingleContinuousDistribution`, `_inverse_cdf_expression`.
- `frv.py` — `FiniteDomain` (`as_boolean`: convert finite sample space to Or-of-And-of-Eq propositional logic), `SingleFiniteDomain`, `FinitePSpace`.
  - `ConditionalFiniteDomain`: restricted discrete event space; `_test` evaluates condition on outcome, falls back to equality LHS==RHS when substitution doesn't yield bool.
- `rv.py` — `RandomSymbol._eval_is_real`, `pspace`, `where`, `sample_iter_lambdify`/`sample_iter_subs` (Monte Carlo sampling).
- NOT special mathematical functions (those are `functions/special/`).

### [`integrals/`](integrals/catalog.md)
Symbolic integration: indefinite, definite, transforms, Risch algorithm, Meijer-G methods.
- `integrals.py` — `Integral` class, `Integral.transform`: variable substitution in definite integrals.
- `transforms.py` — `HankelTypeTransform`, Fourier/Laplace/Mellin/Hankel integral transforms.
- `meijerint.py` — Meijer-G integration helpers: `_get_coeff_exp`, `_check_antecedents_inversion`.
  - `_rewrite_single`: rewrite function as sum of C*x^s*G(a*x^b) forms using cached formula lookup tables; short-circuits for G-function input.
- `rationaltools.py` — `log_to_atan` (complex logarithms → real arctangents via extended GCD), `log_to_real`.
- `quadrature.py` — `gauss_hermite`, `gauss_legendre`, `gauss_laguerre`, `gauss_chebyshev_t`, `gauss_jacobi`: numerical integration nodes/weights; handles `RootOf` (implicit algebraic root) by converting to rational approximation.
- `risch.py` — Risch algorithm core: `risch_integrate`; `gcdex_diophantine` (extended Euclidean for polynomial ideal linear combinations), `frac_in`, `as_poly_1t`, `NonElementaryIntegralException`.
- `rde.py` — `order_at` (p-adic valuation; shortcut for p=t via lowest-degree term), `cancel_primitive`, `cancel_exp`, `no_cancel_equal`, `bound_degree`: Risch differential equation solvers.
- `prde.py` — `constant_system` (enforce constant-field solutions by differentiating non-constant RREF entries to generate constraint rows), `is_log_deriv_k_t_radical`: structure theorem test; `real_imag`.

### [`series/`](series/catalog.md)
Series expansions, limits, sequences, formal power series, Fourier series, and asymptotic analysis.
- `order.py` — `Order` (big-O notation): `__new__` (handles nested Order expressions; raises NotImplementedError if same variable has different limit point), `_eval_power` (nonneg exponent → raise inner expr; O(1) exponent → identity).
- `limits.py` — `Limit.doit`: compute limits using the Gruntz algorithm.
- `fourier.py` — `FourierSeries`, `fourier_series`: precomputed trigonometric series with `scale`, `shift`, `shiftx`, `scalex` (fast coefficient transforms without recomputing integrals).
- `formal.py` — `fps`, `solve_de`, `hyper_re`, `exp_re`: formal power series via DE-to-recurrence conversion.
- `limitseq.py` — `difference_delta`: sequence limits and difference operators.
- `sequences.py` — `SeqAdd`, `SeqMul`, `SeqFormula`, `SeqPer`: symbolic sequence algebra with pairwise reduction; `SeqBase.find_linear_recurrence` (discovers shortest recurrence from initial terms via matrix determinant; verifies against remaining terms).
- Caveats: `formal.py` derives series from DEs; `polys/ring_series.py` does series arithmetic on polynomial rings.

### [`combinatorics/`](combinatorics/catalog.md)
Combinatorics: permutation groups, polyhedra, partitions, free groups, tensor canonicalization.
- `perm_groups.py` — `PermutationGroup`, `derived_series`, `sylow_subgroup`, orbit/stabilizer.
- `named_groups.py` — `DihedralGroup`, `SymmetricGroup`, `CyclicGroup`, `AlternatingGroup` constructors.
- `free_group.py` — `FreeGroup`, `FreeGroupElement`: word reduction, cyclic reduction in free groups.
- `fp_groups.py` — finitely presented groups, coset enumeration (Todd-Coxeter, Felsch strategies).
- `permutations.py` — `Permutation`: finite bijective mappings; `__pow__` (rejects non-integer exponent with conjugate hint); `from_sequence` (derives rearrangement mapping from arbitrary comparable items relative to sorted order).
- `subsets.py` — `Subset`: `subset_indices` (locate elements of one list within another; returns [] for empty input or missing elements via for/else), `subset_from_bitlist`, `unrank_binary`, `unrank_gray`; `ksubsets`.
- `tensor_can.py` — `double_coset_can_rep`, `dummy_sgs` (symmetry generators for contracted index pairs): Butler-Portugal canonicalization.
- `util.py` — `_check_cycles_alt_sym` (detect prime-length cycles with n/2 < p < n-2 for alt/sym group recognition), `_base_ordering`, `_distribute_gens_by_base`.
- `partitions.py` — `Partition`, `IntegerPartition`: set and integer partition classes.

### [`concrete/`](concrete/catalog.md)
Concrete mathematics: sums, products, Gosper's algorithm, sequence guessing.
- `summations.py` — `Sum`, `eval_sum_hyper`: symbolic summation, hypergeometric closed forms; `is_convergent` (convergence tests including Dirichlet), `is_absolutely_convergent`; `reverse_order` (flip iteration bounds ±1 and negate body).
- `products.py` — `Product`: symbolic products; `is_convergent` (tests infinite product convergence via log→Sum transformation, falls back to |term-1| absolute convergence); `_eval_product` (closed-form evaluation).
- `gosper.py` — `gosper_normal` (rational factorization for hypergeometric summation), `gosper_term`, `gosper_sum`.
- `guess.py` — `guess_generating_function`: guess a closed-form generating function from terms.

### [`ntheory/`](ntheory/catalog.md)
Number theory: primes, residues, continued fractions, factorization, partitions, multinomial coefficients.
- `egyptian_fraction.py` — `egyptian_fraction` (unit fraction decomposition: Greedy/Graham-Jewett/Takenouchi/Golomb algorithms; harmonic prefix extraction with early return when remainder is zero).
- `generate.py` — `prime`, `primerange`, `primorial`, `cycle_length`: prime generation and cycle detection.
- `factor_.py` — `factorint` (integer factorization; trial division, Pollard rho, Pollard p-1), `divisors`, `primefactors`, `smoothness`.
  - `perfect_power`: test if n=b^e; finds small divisor, checks exact root, recursively strips factors via GCD of exponents.
- `partitions_.py` — `npartitions`: exact partition count via Hardy-Ramanujan-Rademacher series; `_a` (inner exponential sum with special-case branching for primes 2, 3, and general primes).
- `continued_fraction.py` — `continued_fraction_periodic` (periodic CF expansion of quadratic irrationals (p+√d)/q; normalizes by scaling when (d−p²) % q ≠ 0), `continued_fraction_reduce`, `continued_fraction_iterator`.
- `residue_ntheory.py` — `is_nthpow_residue`, `nthroot_mod`, `primitive_root`, `is_quad_residue`, `discrete_log`; power residue checks for prime powers (odd prime with odd exponent mod 2^k returns True immediately).
- `multinomial.py` — `multinomial_coefficients_iterator`: iterate over multinomial coefficients.

### [`plotting/`](plotting/catalog.md)
Plotting backends for 2D/3D mathematical visualization.
- `plot.py` — `plot`, `plot3d_parametric_line`, `Plot` class: matplotlib-based plotting.
- `plot_implicit.py` — `plot_implicit`, `ImplicitSeries`: implicit equation/inequality rendering (adaptive interval and uniform grid with inequality sign handling).
- `intervalmath/` — `interval`: bounded numeric range with three-valued validity flag. NOT symbolic interval arithmetic (that's `calculus/util.py`).
  - `lib_interval.py` — `sin`, `cos`, `cosh` (lower bound=1 when range crosses zero), `sinh`, `tanh`, `asin`, `acos`, `exp`, `log`, `atan`.
- `experimental_lambdify.py` — `Lambdifier.translate_func`: expression-to-string with float/complex wrapping for plotting.
- `pygletplot/plot.py` — `PygletPlot.show`: alternative pyglet-based 3D plotting backend.

### [`geometry/`](geometry/catalog.md)
Euclidean geometry: points, lines, polygons, circles, ellipses, planes.
- `ellipse.py` — `Ellipse`, `Circle`: tangent lines, intersection, `encloses_point`.
- `plane.py` — `Plane`: `projection_line` (project line onto plane; returns Point3D if line is along normal), `angle_between`, `distance`, `intersection`, `are_concurrent`.
- `point.py` — `Point` (2D/3D Euclidean): `Point3D.__new__` (auto-pads 2-coord input with zero for third component), `intersection` (delegates to other entity if not a Point). NOT `vector/point.py` (coordinate system points).
- `polygon.py` — `Polygon`, `Triangle` (medial, nine-point circle, Euler line), `RegularPolygon`; `_sss` (side-side-side construction: returns None if side lengths violate triangle inequality), `_sas`, `_asa`.

### [`holonomic/`](holonomic/catalog.md)
Holonomic function representation via differential equations.
- `holonomic.py` — `HolonomicFunction`: `diff` (differentiation; shifts ODE when zeroth coefficient is zero), `integrate` (definite: tries `to_expr` then `evalf` fallback).
  - Series expansion, `_indicial`, `_extend_y0`, `evalf` (numerical evaluation via RK4/Euler).
- `recurrence.py` — `RecurrenceOperator` (shift operator algebra with `__eq__`), `HolonomicSequence`: recurrence-based holonomic function algebra.
- NOT recurrence solvers (those are `solvers/recurr.py`). NOT general ODE solving (that's `solvers/ode.py`).

### [`tensor/`](tensor/catalog.md)
Abstract index notation for tensors: `TensorHead`, `TensorIndex`, `TIDS`, Einstein summation.
- `tensor.py` — `TensMul.canon_bp` (canonicalization), `TensAdd`, `TensorIndexType`, `TIDS.from_components_and_indices`.
  - `tensor_indices`: create named indices from comma-separated string; returns bare object for single name, list for multiple.
- `indexed.py` — `Idx` (integer subscript for array access; single dimension arg→lower=0, upper=dim-1; numeric label short-circuits to number), `Indexed`, `IndexedBase`.
- `array/` — N-dimensional arrays (dense/sparse, mutable/immutable): `Array`, `tensorproduct`, `tensorcontraction` (index summation).
  - `derive_by_array`: element-wise symbolic derivatives w.r.t. basis variables; combine with contraction for divergence-like operations.
- NOT quantum tensor products (those are `physics/quantum/tensorproduct.py`).
- NOT second quantization index ordering (that's `physics/secondquant.py`).

### [`sets/`](sets/catalog.md)
Set theory: intervals, finite sets, unions, complements, images.
- `sets.py` — `Interval` (`_eval_imageset`: piecewise function image via progressive domain removal; continuous function image via critical points), `FiniteSet`, `Union` (with `reduce`: merge FiniteSets then iterative pairwise simplification), `Complement`.
- `contains.py` — `Contains`: element-membership predicate (delegates to set's `contains`; returns None to break mutual recursion).
- `fancysets.py` — `Range`, `Naturals`, `Integers`, `Reals`; `ComplexRegion` (polar/rectangular complex sets); `normalize_theta_set` (angular interval normalization to [0,2π), splits wrap-around).

### [`calculus/`](calculus/catalog.md)
Calculus utilities: finite differences, Euler equations, singularities, accumulation bounds.
- `util.py` — `AccumBounds`: symbolic interval arithmetic for limit computation (`__pow__`, `__add__`, `__sub__` (∞−∞ returns full real line), `__contains__` with ±∞ pairing semantics); `not_empty_in`. NOT plotting interval math (that's `plotting/intervalmath/`).
- `singularities.py` — `singularities`: find singularities of a function.
  - `is_increasing`, `is_decreasing`, `is_strictly_increasing`, `is_strictly_decreasing`, `is_monotonic`: monotonicity tests via derivative sign analysis.
  - Constants (no free symbols): non-strict variants return True, strict variants return False.
- `euler.py` — `euler_equations`: derive Euler-Lagrange stationary-condition equations from a Lagrangian.
- NOT ODE solving (that's `solvers/ode.py`). NOT limits (that's `series/limits.py`).

### [`diffgeom/`](diffgeom/catalog.md)
Differential geometry: manifolds, forms, connections, tensor products.
- `diffgeom.py` — `CoordSystem` (coordinate charts with auto-generated labels), `Manifold`, `Patch`, `BaseScalarField`; `TensorProduct` (differential form tensor product; filters scalars from forms, returns scalar*form if only one form), `WedgeProduct`, `LieDerivative`.
- `rn.py` — predefined Euclidean spaces `R2`, `R3` with coordinate systems (rectangular, polar/cylindrical/spherical); all pairwise transition maps explicitly specified with `inverse=False, fill_in_gaps=False`.

### [`liealgebras/`](liealgebras/catalog.md)
Lie algebra representations, root systems, and Weyl groups.
- `weyl_group.py` — `WeylGroup`, `element_order` (matrix exponentiation for most types; string-reduction for G2), `delete_doubles`.
- NOT permutation groups (those are `combinatorics/perm_groups.py`).

### [`categories/`](categories/catalog.md)
Category theory: objects, morphisms, diagrams.
- `baseclasses.py` — `Morphism` (base arrow: `compose`/`__mul__` for sequential composition in mathematical order g∘f), `IdentityMorphism`, `NamedMorphism`, `CompositeMorphism`, `Object`, `Class`.
- `diagram_drawing.py` — `DiagramGrid` (2D lattice layout with disconnected-component handling), `XypicDiagramDrawer`, `ArrowStringDescription`.

### [`parsing/`](parsing/catalog.md)
Expression parsing: string-to-SymPy conversion, Mathematica/Maxima translators.
- `sympy_parser.py` — `parse_expr`, `implicit_multiplication`, `implicit_application`.
  - `split_symbols_custom`: break multi-char names into chars for implicit multiplication; known names emitted as direct refs, unknown wrapped in Symbol().
  - `convert_equals_signs`: nested `=` to `Eq()` via recursive parenthesis grouping.

### [`interactive/`](interactive/catalog.md)
Interactive session setup: `init_session`, `init_printing`.
- `printing.py` — `_init_ipython_printing`: configures IPython display hooks.
  - `_can_print_latex`: type-gate for LaTeX rendering; explicitly excludes `bool` (even though bool subclasses int); recurses into containers.
  - NOT `printing/latex.py` (which formats LaTeX strings).

### [`unify/`](unify/catalog.md)
Unification algorithms for expression pattern matching.
- `core.py` — generic tree unification (AIMA-based): `Variable` (unconstrained wildcard), `CondVariable` (wildcard with predicate filter: `valid(x)` must return True for match), `Compound`, `unify`/`unify_var`.
- `usympy.py` — `unify` (structural unification with commutative/associative matching), `is_commutative` (for Mul: checks all args, not hardcoded), `deconstruct`/`construct` (SymPy↔Compound conversion).
- NOT `core/basic.py` `_has`/`matches` (those are tree-structural matching).

### [`crypto/`](crypto/catalog.md)
Classical cryptographic ciphers and key exchange protocols (educational).
- `crypto.py` — ciphers, `lfsr_sequence`, `lfsr_autocorrelation` (raises TypeError if input is not a list), `lfsr_connection_polynomial` (Berlekamp-Massey).
  - `dh_private_key`/`dh_public_key` (Diffie-Hellman), `elgamal_private_key` (ElGamal).

### [`external/`](external/catalog.md)
Utilities for importing optional third-party packages.
- `importtools.py` — `import_module`: safe optional dependency loader with version checking; supports callable version attributes via `module_version_attr_call_args`.
  - `__sympy_debug`: reads `SYMPY_DEBUG` env var; raises RuntimeError on values other than 'True'/'False'.

### [`sandbox/`](sandbox/catalog.md)
Experimental/sandbox code.
