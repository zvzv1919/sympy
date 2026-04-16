# SymPy — Catalog

SymPy is a pure-Python library for symbolic mathematics. It aims to be a full-featured computer algebra system (CAS) while keeping its code simple, comprehensible, and easily extensible. Its only hard dependency is mpmath (arbitrary-precision arithmetic). This version is **1.1.2.dev**.

## Subdirectories

| Directory | Description |
|-----------|-------------|
| [algebras/](algebras/catalog.md) | Abstract algebra structures (e.g. quaternions) |
| [assumptions/](assumptions/catalog.md) | Assumption system: declaring and querying properties of symbols (positive, integer, etc.), SAT-based inference rule registration (ask_generated), and specific handlers for determining properties of compound expressions (e.g., log/exp/pow real/imaginary checks) |
| [benchmarks/](benchmarks/catalog.md) | Performance benchmarks for SymPy internals |
| [calculus/](calculus/catalog.md) | Calculus operations: finite differences (differentiate_finite, apply_finite_diff), singularities, and continuity utilities — for symbolic integration see `integrals/`, for series/limits see `series/` |
| [categories/](categories/catalog.md) | Category theory: objects, morphisms, diagrams, and diagram drawing |
| [codegen/](codegen/catalog.md) | Code generation infrastructure: AST node definitions, routine/argument management, algorithm-to-code translation, and language-specific code wrappers — for actual code string rendering and pretty-printing see `printing/` |
| [combinatorics/](combinatorics/catalog.md) | Combinatorics: permutations, partitions, polyhedra, group theory (named groups, symmetry groups), Graycode, prufer sequences, and subsets |
| [concrete/](concrete/catalog.md) | Concrete mathematics: symbolic sums (Sum), products (Product), gosper and hypergeometric summation algorithms, and delta operators |
| [core/](core/catalog.md) | Core symbolic engine: expression tree nodes (Symbol, Number, Expr, Add, Mul, Pow), basic arithmetic dispatch, evaluation protocol, caching, and Python compatibility shims — does NOT contain specialized simplification, polynomial algorithms, code generation, or domain-specific logic |
| [crypto/](crypto/catalog.md) | Classical cryptographic ciphers (Caesar, Vigenere, RSA, etc.) |
| [deprecated/](deprecated/catalog.md) | Deprecated modules with import-time warnings pointing to replacements |
| [diffgeom/](diffgeom/catalog.md) | Differential geometry: manifolds, coordinate systems, differential forms |
| [external/](external/catalog.md) | Utilities for importing and probing optional external dependencies |
| [functions/](functions/catalog.md) | Elementary and special mathematical functions: trig (with npi_key pi-coefficient extraction), exponential, Bessel, combinatorial, elliptic integrals, tensor functions (Levi-Civita, Kronecker delta), delta/Heaviside distributions, and Piecewise expressions with interval sorting |
| [geometry/](geometry/catalog.md) | Computational geometry: points, lines, polygons, circles, ellipses, and curves |
| [holonomic/](holonomic/catalog.md) | Holonomic functions represented via linear differential equations with polynomial coefficients |
| [integrals/](integrals/catalog.md) | Symbolic integration: definite/indefinite integrals, transforms (Laplace, Fourier, Mellin), the Risch algorithm, Meijer G-function integration, trigonometric integration, and manualintegrate (step-by-step) |
| [interactive/](interactive/catalog.md) | Interactive session helpers (`isympy` startup, printing configuration, IPython integration) |
| [liealgebras/](liealgebras/catalog.md) | Lie algebras and root systems for classical groups (A, B, C, D, E, F, G) |
| [logic/](logic/catalog.md) | Boolean algebra, propositional logic, satisfiability (DPLL), and inference — handles symbolic boolean expressions only; for mathematical property inference on symbolic expressions see `assumptions/` |
| [matrices/](matrices/catalog.md) | Symbolic and numeric matrix algebra: dense, sparse, immutable, and block matrices; eigenvalues, decompositions, matrix expressions (Trace, MatAdd, MatMul, MatPow), and common matrix constructors |
| [ntheory/](ntheory/catalog.md) | Number theory: primes, factorization, residues, continued fractions, multiplicative functions, Egyptian fractions, and partitions |
| [parsing/](parsing/catalog.md) | Parsing mathematical expressions from strings, LaTeX, Mathematica, and other formats |
| [physics/](physics/catalog.md) | Physics subpackages: quantum mechanics (spin, Wigner 3j/Clebsch-Gordan symbols, Hilbert spaces), second quantization (fermion/boson operators, canonical ordering of dummy indices), optics, classical mechanics (smart_subs for nan avoidance, Lagrangian/Hamiltonian), units and dimensional analysis, hydrogen wave functions, vector algebra (curl, divergence, gradient) |
| [plotting/](plotting/catalog.md) | 2-D and 3-D plotting with backends for matplotlib, pyglet, and text-based output |
| [polys/](polys/catalog.md) | Polynomial algebra: factorization, GCD, Groebner bases, algebraic number fields, polynomial rings and domains, dense polynomial tools (densetools), polynomial fraction classes (DMF/DMP), ring-based power series (ring_series), expression domain wrapping, AGCA module homomorphisms, bivariate solving utilities, and root isolation |
| [printing/](printing/catalog.md) | Output formatting and code string rendering: LaTeX, MathML, pretty-print, C/Fortran/JS/Julia/Rust/GLSL code printers, Python code printer, lambda/anonymous function printing, LLVM JIT code emission, Fortran-specific type handling and line wrapping, vector/physics-aware LaTeX printers, and str/repr printers |
| [sandbox/](sandbox/catalog.md) | Experimental / sandbox modules (indexed objects, tensors prototype) |
| [series/](series/catalog.md) | Series expansions, limits (Gruntz algorithm), formal power series, Fourier series, sequences, and order term (O) handling — for ring-based power series see `polys/ring_series` |
| [sets/](sets/catalog.md) | Set theory: intervals, unions, intersections, finite sets, conditional sets, image sets, and complex region sets |
| [simplify/](simplify/catalog.md) | Expression simplification: trigsimp, radsimp, powsimp, collect, CSE, hypergeometric expansion (hyperexpand), sign canonicalization (signsimp), combinatorial simplification (combsimp), Fu trigonometric identity engine, square-root denesting, and EPath-based expression selection |
| [solvers/](solvers/catalog.md) | Equation solving: algebraic, ODE, PDE, recurrence, inequality, and Diophantine solvers, undetermined coefficient matching, bivariate type classification, Gaussian elimination, and rounding/divisibility utilities for integer solutions |
| [stats/](stats/catalog.md) | Symbolic probability and statistics: random variables, distributions, and expectation |
| [strategies/](strategies/catalog.md) | Rule-based expression rewriting strategies (tree traversal, branching, and composition) |
| [tensor/](tensor/catalog.md) | Tensor algebra: indexed objects (Idx, IndexedBase), index conventions, multi-dimensional array operations (NDimArray, dense/sparse), and tensor contraction/products |
| [unify/](unify/catalog.md) | Unification algorithms (structural and expression-level pattern matching) |
| [utilities/](utilities/catalog.md) | General utilities: iterables (Bell permutations, partitioning, deduplication), decorators, autowrap/Cython code generation, lambdify, memoization, test runner and reporter, source code inspection and class resolution, executable path finding, and timing helpers |
| [vector/](vector/catalog.md) | Vector algebra and calculus in curvilinear coordinate systems |

## Root Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package entry point; checks for mpmath, imports all public submodules, initializes the evalf table, and sets `SYMPY_DEBUG` from the environment |
| `abc.py` | Exports all single Latin and Greek letters as pre-defined `Symbol` objects for convenient interactive use (`from sympy.abc import x, y`); also provides `clashing()` to detect namespace collisions between letter symbols and other SymPy exports |
| `conftest.py` | Pytest configuration: test-splitting (`--split`), cache clearing between modules, architecture/ground-type reporting, and disabled-module skipping |
| `galgebra.py` | Deprecated stub that raises `ImportError` directing users to the standalone `galgebra` package |
| `release.py` | Defines the version string `__version__ = "1.1.2.dev"` |
| `this.py` | Easter egg — prints "The Zen of SymPy" (ROT-13 encoded aphorisms about the project's philosophy) |
