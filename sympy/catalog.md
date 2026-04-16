# SymPy — Project Catalog

SymPy is a Python library for symbolic mathematics. It aims to become a full-featured computer algebra system (CAS) while keeping the code as simple as possible in order to be comprehensible and easily extensible. SymPy is written entirely in Python and depends on mpmath for arbitrary-precision arithmetic.

---

## Root Files

### `__init__.py`
Package entry point. Verifies `mpmath` is installed, imports the version string, enables deprecation warnings in dev builds, and re-exports the public API from all submodules.

### `abc.py`
Exports all single-letter latin and greek letters as pre-defined `Symbol` instances for convenience (e.g. `from sympy.abc import x, y`).

**Caveats**: Names `C`, `O`, `S`, `I`, `N`, `E`, `Q` collide with SymPy builtins; whichever import comes second wins. Does not support on-demand symbol creation — `from sympy.abc import foo` will fail.

### `conftest.py`
Pytest configuration: cache clearing between tests, test splitting (`--split a/b`), seed reporting, and conditional skipping for slow/tooslow tests.

### `galgebra.py`
Stub that raises `ImportError` directing users to the separately maintained [galgebra](https://github.com/brombo/galgebra) package (split out as of SymPy 1.0).

### `release.py`
Single-line module defining `__version__`.

---

## Submodules

### [assumptions](assumptions/catalog.md)
The assumption system: query symbol/expression properties (`Q.positive`, `Q.integer`, etc.) via `ask()`, refine expressions with `refine()`, and register deduction handlers per predicate. Also includes the older `old_assumptions` mechanism on `Symbol` kwargs.

### [benchmarks](benchmarks/catalog.md)
Performance benchmarks for SymPy internals — micro-benchmarks on core arithmetic operations and symbolic expansion.

### [calculus](calculus/catalog.md)
Calculus-related algorithms: Euler-Lagrange equations, finite difference approximations, singularity detection, monotonicity tests, continuous-domain analysis, and interval arithmetic via accumulation bounds.

### [categories](categories/catalog.md)
Fundamental category-theory classes (objects, morphisms, diagrams, categories) and Xy-pic diagram rendering. Follows *Abstract and Concrete Categories* (Adamek, Herrlich, Strecker).

### [codegen](codegen/catalog.md)
AST types for representing code structures (assignments, code blocks, loops) used by SymPy's code printers and code generation utilities.

### [combinatorics](combinatorics/catalog.md)
Combinatorics and group theory: permutations (array and cyclic notation), permutation groups, named groups, coset tables, rewriting systems, polyhedra, prufer sequences, subsets, Gray codes, and group homomorphisms.

### [concrete](concrete/catalog.md)
Concrete mathematics: symbolic summation, product evaluation, Gosper's hypergeometric algorithm, Kronecker delta handling, and sequence-guessing heuristics.

### [core](core/catalog.md)
The core symbolic engine: expression tree (`Basic`, `Expr`), number types (`Integer`, `Rational`, `Float`), symbols, arithmetic operations (`Add`, `Mul`, `Pow`), functions, relations, pattern matching, caching, evaluation, and compatibility infrastructure.

### [crypto](crypto/catalog.md)
Classical and modern ciphers: shift, affine, substitution, Hill, bifid, RSA, Elgamal, Vigenere, kid RSA, and related encoding/decoding utilities.

### [deprecated](deprecated/catalog.md)
Houses deprecations that cannot remain in their original modules (e.g. removed modules or import-cycle issues). Imported last, after all other submodules.

### [diffgeom](diffgeom/catalog.md)
Differential geometry: manifolds, coordinate patches, coordinate systems, scalar/vector/form fields, covariant derivatives, Lie derivatives, metric tensors, and related differential-geometric operations.

### [external](external/catalog.md)
Unified place for safely importing optional external dependencies (`numpy`, `Cython`, `matplotlib`, etc.) with version checking and warning control.

### [functions](functions/catalog.md)
Central library of symbolic mathematical functions: elementary (trig, exp, log, hyperbolic, piecewise), combinatorial (factorials, binomials, Fibonacci, Stirling), and special functions (Bessel, gamma, error, orthogonal polynomials, elliptic integrals, etc.).

### [geometry](geometry/catalog.md)
2D and 3D Euclidean geometry: points, lines, rays, segments, polygons, ellipses, circles, parabolas, curves, and planes. All entities are symbolic and integrate with SymPy's CAS.

### [holonomic](holonomic/catalog.md)
Holonomic functions: representation as linear homogeneous ODEs with polynomial coefficients, conversion between holonomic and symbolic forms, series expansion, numerical evaluation, and recurrence generation.

### [integrals](integrals/catalog.md)
Symbolic integration engine: indefinite/definite integration, integral transforms (Laplace, Fourier, Mellin, Hankel), and underlying decision procedures (Risch algorithm, heuristic Risch, Meijer G-functions, trigonometric integrals, rational functions).

### [interactive](interactive/catalog.md)
Tools for initializing and configuring interactive SymPy sessions (used by `isympy`): pretty-printing setup, IPython integration, and session initialization.

### [liealgebras](liealgebras/catalog.md)
Classification of simple Lie algebras via Cartan types (A-G), their root systems, Cartan matrices, Dynkin diagrams, and Weyl groups.

### [logic](logic/catalog.md)
Boolean logic and inference: boolean algebra classes (`And`, `Or`, `Not`, `Implies`, etc.), CNF/DNF normal forms, SAT-based satisfiability, and truth table inference.

### [matrices](matrices/catalog.md)
Matrix algebra: dense and sparse matrix implementations, matrix expressions (symbolic lazy evaluation), decompositions, solvers, eigenvalue computation, and special matrix constructors.

### [ntheory](ntheory/catalog.md)
Number theory: primality testing, prime generation/counting, integer factorization, modular arithmetic, quadratic residues, continued fractions, Egyptian fractions, and multiplicative functions.

### [parsing](parsing/catalog.md)
Expression parsing: `sympify` string input, LaTeX-to-SymPy conversion, Mathematica/Maxima expression import, and configurable token transformations.

### [physics](physics/catalog.md)
Symbolic physics: quantum mechanics (states, operators, commutators, angular momentum, spin), classical mechanics (Lagrangian/Hamiltonian, Kane's method, rigid bodies), optics (Gaussian beams, geometric optics, wave propagation), unit/dimension systems, Pauli/Dirac matrices, second quantization, and physical constants.

### [plotting](plotting/catalog.md)
Symbolic expression plotting with multiple backends (matplotlib, text, pyglet). Supports 2D/3D line plots, parametric plots, surface plots, implicit equation plots, and interactive 3D visualization.

### [polys](polys/catalog.md)
Polynomial algebra: polynomial rings, dense/sparse representations, GCD, factorization, Groebner bases, resultants, algebraic number fields, polynomial domains (ZZ, QQ, RR, etc.), and the AGCA sub-package for modules, ideals, and homomorphisms over polynomial rings.

### [printing](printing/catalog.md)
Output and code generation: converts SymPy expression trees into human-readable or machine-consumable strings. Printers include pretty-print (ASCII/Unicode), LaTeX, MathML, C/Fortran/Julia/Rust code generators, dot graphs, and more. All dispatched via `_print_<ClassName>` MRO-based methods.

### [sandbox](sandbox/catalog.md)
Experimental/unstable code with no stability guarantees. Currently contains `indexed_integrals` for integration over indexed/tensor variables.

### [series](series/catalog.md)
Series and limits: Taylor/Laurent/Puiseux series expansions (`Order`), the Gruntz limit algorithm, formal power series (FPS), Fourier series, sequences, residue computation, and asymptotic analysis.

### [sets](sets/catalog.md)
Core set-theoretic types and operations: `FiniteSet`, `Interval`, `Union`, `Intersection`, `Complement`, `ProductSet`, `ConditionSet`, `ImageSet`, and membership predicates used throughout SymPy.

### [simplify](simplify/catalog.md)
Expression simplification: the main `simplify()` heuristic, trigonometric simplification, radical denesting, CSE (common subexpression elimination), hypergeometric term rewriting, power simplification, `combsimp`, `sqrtdenest`, and traversal-based rewrite utilities.

### [solvers](solvers/catalog.md)
Equation solvers: algebraic equation solving (`solve`, `solveset`), systems of equations, ordinary differential equations (ODE), partial differential equations (PDE), Diophantine equations, inequalities, and decompilation of polynomial systems.

### [stats](stats/catalog.md)
Statistics and probability: random variable types (continuous, discrete, finite, joint, compound, matrix, stochastic processes) created via convenience constructors and queried with `P`, `E`, `density`, `variance`. Backed by probability spaces for integration, sampling, and conditioning.

### [strategies](strategies/catalog.md)
Rule-based transformation strategies: higher-order combinators (`do_one`, `exhaust`, `chain`, `minimize`) that control how `Expr -> Expr` rewrite rules are applied to expression trees. Includes a branching sub-package for non-deterministic strategies. *Experimental module.*

### [tensor](tensor/catalog.md)
Tensor algebra: indexed objects (`Idx`, `IndexedBase`, `Indexed`), abstract tensor index notation with automatic contraction and symmetry handling, and N-dimensional array operations (dense, sparse, immutable variants).

### [unify](unify/catalog.md)
Unification: a generic unification engine (associative-commutative aware) with a SymPy-specific frontend for structural pattern matching on expression trees.

### [utilities](utilities/catalog.md)
Shared utilities: iterable helpers, decorators (`threaded`, `deprecated`), code generation wrappers (`lambdify`, `autowrap`), numerical evaluation (`nfloat`), source inspection, testing infrastructure, and miscellaneous support consumed across all of SymPy.

### [vector](vector/catalog.md)
Symbolic 3D vector algebra and calculus: Cartesian coordinate systems, vectors, dyadics, differential operators (nabla), and standard vector-calculus operations (gradient, divergence, curl, scalar/vector potential).
