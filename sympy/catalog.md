# SymPy — Catalog

SymPy is a pure-Python library for symbolic mathematics. It aims to be a full-featured computer algebra system (CAS) while keeping its code simple, comprehensible, and easily extensible. Its only hard dependency is mpmath (arbitrary-precision arithmetic). This version is **1.1.2.dev**.

## Subdirectories

| Directory | Description |
|-----------|-------------|
| [algebras/](algebras/catalog.md) | Abstract algebra structures (e.g. quaternions) |
| [assumptions/](assumptions/catalog.md) | Assumption system for declaring and querying properties of symbols (positive, integer, etc.) |
| [benchmarks/](benchmarks/catalog.md) | Performance benchmarks for SymPy internals |
| [calculus/](calculus/catalog.md) | Calculus operations: finite differences, singularities, and continuity utilities |
| [categories/](categories/catalog.md) | Category theory: objects, morphisms, diagrams, and diagram drawing |
| [codegen/](codegen/catalog.md) | Code generation for C, Fortran, Julia, Rust, and other languages |
| [combinatorics/](combinatorics/catalog.md) | Combinatorics: permutations, partitions, polyhedra, group theory, and Graycode |
| [concrete/](concrete/catalog.md) | Concrete mathematics: symbolic sums, products, and related algorithms |
| [core/](core/catalog.md) | Core symbolic engine: basic objects (Symbol, Number, Expr, Add, Mul, Pow), caching, evaluation, and compatibility |
| [crypto/](crypto/catalog.md) | Classical cryptographic ciphers (Caesar, Vigenere, RSA, etc.) |
| [deprecated/](deprecated/catalog.md) | Deprecated modules with import-time warnings pointing to replacements |
| [diffgeom/](diffgeom/catalog.md) | Differential geometry: manifolds, coordinate systems, differential forms |
| [external/](external/catalog.md) | Utilities for importing and probing optional external dependencies |
| [functions/](functions/catalog.md) | Elementary and special mathematical functions (trig, exponential, Bessel, combinatorial, etc.) |
| [geometry/](geometry/catalog.md) | Computational geometry: points, lines, polygons, circles, ellipses, and curves |
| [holonomic/](holonomic/catalog.md) | Holonomic functions represented via linear differential equations with polynomial coefficients |
| [integrals/](integrals/catalog.md) | Symbolic integration: definite/indefinite integrals, transforms (Laplace, Fourier, Mellin), and the Risch algorithm |
| [interactive/](interactive/catalog.md) | Interactive session helpers (`isympy` startup, printing configuration, IPython integration) |
| [liealgebras/](liealgebras/catalog.md) | Lie algebras and root systems for classical groups (A, B, C, D, E, F, G) |
| [logic/](logic/catalog.md) | Boolean algebra, propositional logic, satisfiability (DPLL), and inference |
| [matrices/](matrices/catalog.md) | Symbolic and numeric matrix algebra: dense, sparse, immutable, and block matrices; eigenvalues, decompositions |
| [ntheory/](ntheory/catalog.md) | Number theory: primes, factorization, residues, continued fractions, and multiplicative functions |
| [parsing/](parsing/catalog.md) | Parsing mathematical expressions from strings, LaTeX, Mathematica, and other formats |
| [physics/](physics/catalog.md) | Physics subpackages: quantum mechanics, optics, classical mechanics, units, hydrogen wave functions, vector algebra |
| [plotting/](plotting/catalog.md) | 2-D and 3-D plotting with backends for matplotlib, pyglet, and text-based output |
| [polys/](polys/catalog.md) | Polynomial algebra: factorization, GCD, Groebner bases, algebraic number fields, polynomial rings and domains |
| [printing/](printing/catalog.md) | Output formatting: LaTeX, MathML, pretty-print, C/Fortran/JS/Julia/Rust/GLSL code generation, and more |
| [sandbox/](sandbox/catalog.md) | Experimental / sandbox modules (indexed objects, tensors prototype) |
| [series/](series/catalog.md) | Series expansions, limits (Gruntz algorithm), formal power series, Fourier series, and sequences |
| [sets/](sets/catalog.md) | Set theory: intervals, unions, intersections, finite sets, and conditional sets |
| [simplify/](simplify/catalog.md) | Expression simplification: trigsimp, radsimp, powsimp, collect, CSE, hypergeometric simplification |
| [solvers/](solvers/catalog.md) | Equation solving: algebraic, ODE, PDE, recurrence, inequality, and Diophantine solvers |
| [stats/](stats/catalog.md) | Symbolic probability and statistics: random variables, distributions, and expectation |
| [strategies/](strategies/catalog.md) | Rule-based expression rewriting strategies (tree traversal, branching, and composition) |
| [tensor/](tensor/catalog.md) | Tensor algebra: indexed objects, index conventions, and multi-dimensional array operations |
| [unify/](unify/catalog.md) | Unification algorithms (structural and expression-level pattern matching) |
| [utilities/](utilities/catalog.md) | General utilities: iterables, decorators, code generation helpers, lambdify, memoization, testing |
| [vector/](vector/catalog.md) | Vector algebra and calculus in curvilinear coordinate systems |

## Root Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package entry point; checks for mpmath, imports all public submodules, initializes the evalf table, and sets `SYMPY_DEBUG` from the environment |
| `abc.py` | Exports all single Latin and Greek letters as pre-defined `Symbol` objects for convenient interactive use (`from sympy.abc import x, y`) |
| `conftest.py` | Pytest configuration: test-splitting (`--split`), cache clearing between modules, architecture/ground-type reporting, and disabled-module skipping |
| `galgebra.py` | Deprecated stub that raises `ImportError` directing users to the standalone `galgebra` package |
| `release.py` | Defines the version string `__version__ = "1.1.2.dev"` |
| `this.py` | Easter egg — prints "The Zen of SymPy" (ROT-13 encoded aphorisms about the project's philosophy) |
