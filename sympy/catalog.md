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

- **_eval_subs / _eval_power**: internal hooks on core expression classes, always in `core/`.
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
- `basic.py` — `Basic` base class: `_has`, `matches`, `atoms`, `xreplace`, structural traversal.
- `expr.py` — `Expr` class: `as_coefficient`, `leadterm`, `extract_multiplicatively`, `as_leading_term`.
- `power.py` — `Pow` class: `_eval_power`, `_eval_subs` (exponent splitting for substitution).
- `add.py` — `Add` class: `flatten` (infinity filtering, order processing), `_eval_as_leading_term`.
- `mul.py` — `Mul` class: `flatten`, `as_content_primitive`, `as_two_terms`.
- `numbers.py` — `Integer`, `Rational`, `Float`, `ImaginaryUnit`, `AlgebraicNumber`, `_eval_power`, `mod_inverse`.
- `function.py` — `Function`, `Derivative.__new__`, `expand()`, `AppliedUndef`, `WildFunction`.
- `exprtools.py` — `gcd_terms`, `factor_terms`: GCD extraction with non-commutative masking for Add terms.
- `evalf.py` — `hypsum`, `evalf_sum`: numerical evaluation, hypergeometric series summation, precision.
- `logic.py` — fuzzy logic (`fuzzy_and`, `fuzzy_or`, `fuzzy_not`) AND internal propositional `And`/`Or`/`AndOr_Base` classes.
- `operations.py` — `AssocOp`, `LatticeOp.__new__`: base classes for associative/lattice operations.
- `symbol.py` — `Symbol`, `Dummy`, `Wild` class definitions.
- `sympify.py` — `sympify()`: convert arbitrary objects to SymPy types.
- `compatibility.py` — `ordered`: conservative tie-breaking sort with successive keys; fallback/warn on unresolved ties.
- `assumptions.py` — `_ask`: recursive property resolution with anti-recursion guard and shuffled prerequisites.
- `facts.py` — `deduce_alpha_implications`: transitive closure of inference rules with contrapositive generation.
- `multidimensional.py` — `vectorize`: decorator lifting scalar functions to element-wise nested collection operations.
- `trace.py` — `Tr`: generic trace for non-commutative products; extracts commutative prefactors from `Mul`.

### [`polys/`](polys/catalog.md)
Polynomial algebra, domains, Gröbner bases, factorization, root isolation, and module theory.
- `polytools.py` — `Poly` class, `degree`, `primitive`, `factor`, `gcd`, `groebner` (high-level API).
- `polyroots.py` — `roots_quadratic`, `roots_cubic`, `roots_quartic`: algebraic root formulas; `preprocess_roots` strips symbolic coefficients.
- `rootisolation.py` — `dup_isolate_real_roots_list`: real root isolation via continued fractions (VAS).
- `orthopolys.py` — `hermite_poly`, `laguerre_poly`, `legendre_poly`: computational polynomial generators.
- `factortools.py` — low-level factorization: Hensel lifting, Zassenhaus algorithm.
- `numberfields.py` — `minimal_polynomial`, algebraic field extensions.
- `ring_series.py` — `rs_tanh`, `rs_exp`, `rs_series_inversion`: formal power series via polynomial rings.
- `agca/` — abstract algebra: `modules.py` (free modules), `homomorphisms.py` (module maps).
- `fields.py` — `FracElement` arithmetic: dispatch for nested quotient-of-quotient domain operations.
- `domains/` — coefficient domains: `ExpressionDomain`, `IntegerRing`, `RationalField`, `mpelements.py` (multi-precision context, tolerance, rational conversion).

### [`functions/`](functions/catalog.md)
Mathematical function classes (symbolic, unevaluated). Defines the functions, does NOT simplify them.
- `elementary/trigonometric.py` — `sin`, `cos`, `tan`, `acot`, `atan`, `asin`, `acos` and other inverse trig; `_pi_coeff`, `_eval_aseries`.
- `elementary/piecewise.py` — `Piecewise`, `_sort_expr_cond`, `_eval_integral`.
- `elementary/hyperbolic.py` — `sinh`, `cosh`, `tanh` and inverses.
- `elementary/complexes.py` — `conjugate`, `Abs`, `arg`, `re`, `im`, `sign`, `transpose`, `adjoint`.
- `elementary/integers.py` — `floor`, `ceiling`, `frac`: rounding functions with `_eval_nseries`.
- `special/tensor_functions.py` — `KroneckerDelta` (fermi-level index logic, second quantization), `LeviCivita`.
- `special/bessel.py` — Bessel functions (`besselj`, `bessely`) and Airy functions (`airyai`, `airybi`).
- `special/polynomials.py` — symbolic orthogonal polynomial classes: `laguerre`, `hermite`, `chebyshev`, etc.
- `special/elliptic_integrals.py` — `elliptic_f`, `elliptic_e`, `elliptic_k`, `elliptic_pi`.
- `combinatorial/numbers.py` — `fibonacci`, `bernoulli`, `catalan`, `harmonic`, `nP` permutation counting.
- `combinatorial/factorials.py` — `factorial`, `binomial`, `RisingFactorial`, `FallingFactorial`.
- `elementary/exponential.py` — `log` (eval handles `AccumBounds`, imaginary args, rationals), `exp`, `exp_polar`.
- `special/error_functions.py` — `TrigonometricIntegral` (base for `Si`, `Ci`, `Shi`, `Chi`), `Ei`, `li`, `Li`, `erf`.
- Caveats: `special/polynomials.py` defines symbolic classes; `polys/orthopolys.py` has computational generators.

### [`simplify/`](simplify/catalog.md)
Expression transformation and simplification algorithms. Operates ON functions, does not define them.
- `fu.py` — trigonometric simplification rules; `hyper_as_trig` (rewrite hypergeometric via trig).
- `hyperexpand.py` — `hyperexpand`, `ReduceOrder`: reduce hypergeometric/Meijer-G to closed form.
- `combsimp.py` — `combsimp`, `_rf` (rising factorial simplification for combinatorial expressions).
- `powsimp.py` — `powsimp`, `powdenest`: power/exponent simplification.
- `simplify.py` — `simplify()`: general-purpose dispatch; `logcombine`: combine/split logarithmic terms.
- `epathtools.py` — `EPath`: XPath-like tool for selecting and transforming expression sub-trees.

### [`printing/`](printing/catalog.md)
String/code representation of SymPy expressions. Outputs text, NOT callable code or AST nodes.
- `fcode.py` — `FCodePrinter`, `fcode`, `indent_code`: Fortran expression printing.
- `lambdarepr.py` — `LambdaPrinter`, `NumExprPrinter` (numexpr string backend; blacklists matrices/collections), `_print_Piecewise`.
- `python.py` — `python()`: generate executable Python code string for an expression.
- `latex.py` — `LatexPrinter`, `latex()`: LaTeX representation.
- `llvmjitcode.py` — `llvm_callable`: JIT-compile expressions to machine code via LLVM.
- `str.py` / `repr.py` — default `str()` / `repr()` printers.
- `octave.py` — `OctaveCodePrinter`: Octave/MATLAB code; rewrites spherical Bessel via cylindrical Bessel.
- `julia.py` — `JuliaCodePrinter`: Julia code; restructures `Piecewise` assignments in non-inline mode.
- `theanocode.py` — `TheanoPrinter`: Theano graph builder; `_print_Piecewise` uses `np.nan` fallback for single-branch.

### [`solvers/`](solvers/catalog.md)
Equation solving: algebraic, ODE, PDE, recurrence, diophantine, systems.
- `solvers.py` — `solve`, `_solve`, `unrad` (radical elimination/denesting), `solve_linear_system`: main algebraic solver.
- `solveset.py` — `solveset`, `linsolve`, `linear_eq_to_matrix`: new-style set-based solver API.
- `recurr.py` — `rsolve`, `rsolve_poly`, `rsolve_hyper`: linear recurrence equation solvers.
- `ode.py` — ODE classification and solution: `dsolve`, `_frobenius`, Lie group methods, `odesimp`.
- `diophantine.py` — `diophantine`, `sum_of_four_squares`: integer/diophantine equation solving.
- `bivariate.py` — `bivariate_type`: solve equations with two variables via back-substitution.
- `inequalities.py` — `solve_poly_inequality`, `solve_rational_inequalities`: polynomial/rational inequality solving.
- `polysys.py` — `solve_generic` (Gröbner-based polynomial system solver), `solve_biquadratic`.
- `deutils.py` — `ode_order`: utility to determine the order of a differential equation.

### [`utilities/`](utilities/catalog.md)
Utility functions: numeric code generation, iterables, source inspection, multiset enumeration.
- `lambdify.py` — `lambdify`, `lambdastr`: convert expressions to callable Python/NumPy functions.
- `autowrap.py` — `autowrap`, `ufuncify`, `CythonCodeWrapper._partition_args`: compile to binary.
- `codegen.py` — `Routine`, `CCodeGen`, `OctaveCodeGen`: generate Fortran/C/Octave source files.
- `iterables.py` — `partitions`, `_set_partitions` (set partition enumeration), `topological_sort`, `numbered_symbols`, combinatoric generators.
- `enumerative.py` — `MultisetPartitionTraverser`: multiset partition enumeration (Knuth's algorithm).
- `runtests.py` — `_doctest`, `SymPyOutputChecker`: test runner with float comparison, matplotlib backend management.

### [`codegen/`](codegen/catalog.md)
AST node definitions for code generation (abstract syntax tree). NOT source file generation (that's `utilities/codegen.py`).
- `ast.py` — `Assignment`, `CodeBlock`, `Variable`, `Declaration`, `FunctionPrototype`.

### [`assumptions/`](assumptions/catalog.md)
Property inference system for symbolic objects (is_positive, is_integer, etc.).
- `ask.py` — `Q` predicate definitions (`Q.transcendental`, `Q.algebraic`, etc.), `ask()` query function.
- `assume.py` — `Predicate.eval`: handler dispatch with contradiction detection; `assuming` context manager.
- `refine.py` — `refine`, `refine_atan2`, `refine_Pow`: simplify expressions under assumptions.
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
- `secondquant.py` — creation/annihilation operators, `Commutator` (Lie bracket with scalar extraction), Wick's theorem.
- `mechanics/functions.py` — `_smart_subs`: intelligent substitution for mechanics expressions.
- `vector/printing.py` — `VectorLatexPrinter`, `VectorPrettyPrinter`: Newton dot notation for time-derivatives.
- `quantum/` — states (`Ket`, `Bra`, `QExpr`), `Dagger` (Hermitian conjugate), spin eigenstates, operator sets, representations.
- `quantum/tensorproduct.py` — quantum Kronecker products and simplification (NOT `tensor/`).
- `quantum/identitysearch.py` — gate identity BFS discovery; `lr_op`, `ll_op` rule operations.
- `quantum/circuitplot.py` — circuit diagram visualization (NOT `plotting/`).
- `hep/gamma_matrices.py` — Dirac gamma matrix traces and simplification for high-energy physics.
- `wigner.py` — Wigner 3j/6j/9j, Clebsch-Gordan, Gaunt coefficients (spherical harmonic integrals).
- `units.py` — legacy physical units; `Unit` class with `is_positive=True`.
- `unitsystems/simplifiers.py` — `dim_simplify`: dimensional analysis simplification.

### [`matrices/`](matrices/catalog.md)
Matrix classes and matrix expression algebra.
- `immutable.py` — `ImmutableMatrix`, `_eval_Eq`: immutable matrix with equality support.
- `expressions/transpose.py` — `Transpose`, `refine_Transpose`: symbolic transpose operations.
- `expressions/matmul.py` — `MatMul`, `_eval_trace`: symbolic matrix multiplication and trace.

### [`vector/`](vector/catalog.md)
New-style 3D vector algebra module (sympy.vector). NOT physics.vector.
- `point.py` — `Point.position_wrt`: position vector between points in coordinate systems.
- `coordsysrect.py` — `CoordSysCartesian.__new__`: Cartesian coordinate system definition.
- Caveats: Separate from `physics/vector/` which uses `ReferenceFrame`-based classical mechanics vectors.

### [`stats/`](stats/catalog.md)
Probability and statistics: random variables, distributions, expected values.
- `crv_types.py` — continuous distribution classes: `BetaDistribution`, `KumaraswamyDistribution`, etc.
- `crv.py` — `SingleContinuousDistribution`, `_inverse_cdf_expression`: continuous RV framework.
- `rv.py` — `RandomSymbol._eval_is_real`, `pspace`, `where`, `sample_iter_lambdify`/`sample_iter_subs` (Monte Carlo sampling).
- NOT special mathematical functions (those are `functions/special/`).

### [`integrals/`](integrals/catalog.md)
Symbolic integration: indefinite, definite, transforms, Risch algorithm, Meijer-G methods.
- `integrals.py` — `Integral` class, `Integral.transform`: variable substitution in definite integrals.
- `transforms.py` — `HankelTypeTransform`, Fourier/Laplace/Mellin/Hankel integral transforms.
- `meijerint.py` — `_get_coeff_exp`, `_check_antecedents_inversion`: Meijer-G integration helpers.
- `rde.py` — `cancel_primitive`, `cancel_exp`: Risch differential equation solvers (cancellation cases).
- `prde.py` — `is_log_deriv_k_t_radical`: structure theorem test for logarithmic derivatives in extensions.

### [`series/`](series/catalog.md)
Series expansions, limits, sequences, formal power series, and asymptotic analysis.
- `limits.py` — `Limit.doit`: compute limits using the Gruntz algorithm.
- `formal.py` — `fps`, `solve_de`, `hyper_re`, `exp_re`: formal power series via DE-to-recurrence conversion.
- `limitseq.py` — `difference_delta`: sequence limits and difference operators.
- `sequences.py` — `SeqAdd`, `SeqMul`, `SeqFormula`, `SeqPer`: symbolic sequence algebra with pairwise reduction.
- Caveats: `formal.py` derives series from DEs; `polys/ring_series.py` does series arithmetic on polynomial rings.

### [`combinatorics/`](combinatorics/catalog.md)
Combinatorics: permutation groups, polyhedra, partitions, free groups, tensor canonicalization.
- `perm_groups.py` — `PermutationGroup`, `derived_series`, `sylow_subgroup`, orbit/stabilizer.
- `named_groups.py` — `DihedralGroup`, `SymmetricGroup`, `CyclicGroup`, `AlternatingGroup` constructors.
- `free_group.py` — `FreeGroup`, `FreeGroupElement`: word reduction, cyclic reduction in free groups.
- `fp_groups.py` — finitely presented groups, coset enumeration (Todd-Coxeter, Felsch strategies).
- `permutations.py` — `Permutation`: finite bijective mappings; `__pow__` (rejects non-integer exponent with conjugate hint).
- `tensor_can.py` — `double_coset_can_rep`, `dummy_sgs` (symmetry generators for contracted index pairs): Butler-Portugal canonicalization.
- `partitions.py` — `Partition`, `IntegerPartition`: set and integer partition classes.

### [`concrete/`](concrete/catalog.md)
Concrete mathematics: sums, products, Gosper's algorithm, sequence guessing.
- `summations.py` — `Sum`, `eval_sum_hyper`: symbolic summation, hypergeometric closed forms.
- `gosper.py` — `gosper_normal` (rational factorization for hypergeometric summation), `gosper_term`, `gosper_sum`.
- `guess.py` — `guess_generating_function`: guess a closed-form generating function from terms.

### [`ntheory/`](ntheory/catalog.md)
Number theory: primes, residues, continued fractions, multinomial coefficients.
- `generate.py` — `prime`, `primerange`, `primorial`, `cycle_length`: prime generation and cycle detection.
- `multinomial.py` — `multinomial_coefficients_iterator`: iterate over multinomial coefficients.

### [`plotting/`](plotting/catalog.md)
Plotting backends for 2D/3D mathematical visualization.
- `plot.py` — `plot`, `plot3d_parametric_line`, `Plot` class: matplotlib-based plotting.
- `experimental_lambdify.py` — `Lambdifier.translate_func`: expression-to-string with float/complex wrapping for plotting.
- `pygletplot/plot.py` — `PygletPlot.show`: alternative pyglet-based 3D plotting backend.

### [`geometry/`](geometry/catalog.md)
Euclidean geometry: points, lines, polygons, circles, ellipses.
- `ellipse.py` — `Ellipse`, `Circle`: tangent lines, intersection, `encloses_point`.
- `point.py` — `Point` (2D/3D Euclidean). NOT `vector/point.py` (coordinate system points).

### [`holonomic/`](holonomic/catalog.md)
Holonomic function representation via differential equations.
- `recurrence.py` — `RecurrenceOperator`, `RecurrenceOperators`: recurrence-based holonomic function algebra.
- NOT recurrence solvers (those are `solvers/recurr.py`).

### [`tensor/`](tensor/catalog.md)
Abstract index notation for tensors: `TensorHead`, `TensorIndex`, `TIDS`, Einstein summation.
- `tensor.py` — `TensMul.canon_bp` (canonicalization entry point), `TIDS.from_components_and_indices`: index data, contraction.
- NOT quantum tensor products (those are `physics/quantum/tensorproduct.py`).
- NOT second quantization index ordering (that's `physics/secondquant.py`).

### [`sets/`](sets/catalog.md)
Set theory: intervals, finite sets, unions, complements, images.
- `sets.py` — `Interval`, `FiniteSet` (`as_relational` converts membership to equality/Or), `Union`, `Complement`.
- `fancysets.py` — `Range` (integer sequences with slicing), `Naturals`, `Integers`, `Reals`.

### [`calculus/`](calculus/catalog.md)
Calculus utilities: finite differences, Euler equations, singularities, accumulation bounds.
- `util.py` — `AccumBounds`: interval arithmetic for limit computation (`__pow__`, `__add__`).
- `euler.py` — `euler_equations`: derive Euler-Lagrange stationary-condition equations from a Lagrangian.
- NOT ODE solving (that's `solvers/ode.py`). NOT limits (that's `series/limits.py`).

### [`diffgeom/`](diffgeom/catalog.md)
Differential geometry: manifolds, forms, connections.
- `diffgeom.py` — `CoordSystem` (coordinate charts with auto-generated labels), `Manifold`, `Patch`, `BaseScalarField`.

### [`liealgebras/`](liealgebras/catalog.md)
Lie algebra representations and root systems.
- NOT permutation groups (those are `combinatorics/perm_groups.py`).

### [`categories/`](categories/catalog.md)
Category theory: objects, morphisms, diagrams.
- `diagram_drawing.py` — `DiagramGrid` (2D lattice layout with disconnected-component handling), `XypicDiagramDrawer`, `ArrowStringDescription`.

### [`parsing/`](parsing/catalog.md)
Expression parsing: string-to-SymPy conversion, Mathematica/Maxima translators.

### [`interactive/`](interactive/catalog.md)
Interactive session setup: `init_session`, `init_printing`.

### [`unify/`](unify/catalog.md)
Unification algorithms for expression pattern matching.
- NOT `core/basic.py` `_has`/`matches` (those are tree-structural matching).

### [`crypto/`](crypto/catalog.md)
Classical cryptographic ciphers and key exchange protocols (educational).
- `crypto.py` — ciphers, `dh_private_key`/`dh_public_key` (Diffie-Hellman), `elgamal_private_key` (ElGamal).

### [`sandbox/`](sandbox/catalog.md)
Experimental/sandbox code.
