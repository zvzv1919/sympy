# sympy/polys — Catalog

> Part of [SymPy](../catalog.md). Polynomial algebra: factorization, GCD, Groebner bases, algebraic number fields, polynomial rings and domains.

## Python Files

| File | Summary |
|------|----------|
| `__init__.py` | Package init that re-exports the public API from submodules (polytools, polyfuncs, rationaltools, numberfields, monomials, orderings, rootoftools, polyroots, domains, constructor, specialpolys, orthopolys, partfrac, polyerrors, dispersion, ring_series). |
| `polytools.py` | User-friendly public interface to polynomial functions; defines the `Poly` class, `Groebner` basis wrapper, and top-level helpers like `factor`, `gcd`, `resultant`, `discriminant`, etc. |
| `polyclasses.py` | OO layer for low-level polynomial representations: `DMP` (dense multivariate), `DMF` (dense multivariate fractions), and `ANP` (algebraic number field elements). |
| `polyoptions.py` | Options manager for `Poly` and public API functions; defines `Options`, `Flag`, and various option classes (Domain, Order, Extension, etc.) with validation and preprocessing. |
| `polyutils.py` | Utility functions for higher-level polynomial classes: generator sorting, expression-to-dict conversion, `PicklableWithSlots` mixin, and numerical root sorting. |
| `polyroots.py` | Algorithms for computing symbolic roots of polynomials: `roots_linear`, `roots_quadratic`, `roots_cubic`, `roots_quartic`, `roots_quintic` solvers for degrees 1–5. The quintic solver (`roots_quintic`) handles solvable quintics via Dummit's algorithm, requiring depressed form (no x⁴ term) and normalizing by the leading coefficient only if all resulting coefficients are rational; otherwise it bails out. |
| `polyerrors.py` | Definitions of exception classes for the polys module (e.g., `PolynomialError`, `DomainError`, `CoercionFailed`, `ExactQuotientFailed`). |
| `polyconfig.py` | Configuration utilities for polynomial algorithms; manages settings such as GCD method, factorization method, and Groebner basis algorithm via `setup`/`query`/`using`. |
| `polyfuncs.py` | High-level polynomial manipulation functions: `symmetrize`, `horner`, `interpolate`, `rational_interpolate`, and `viete`. |
| `polymatrix.py` | `PolyMatrix` (alias `MutablePolyDenseMatrix`), a mutable matrix class whose entries are `Poly` objects, supporting polynomial-aware matrix arithmetic. |
| `polyquinticconst.py` | Constants and resolvent polynomial computations for solving solvable quintics, implementing Dummit's algorithm. |
| `rings.py` | Sparse polynomial rings: `PolyRing` and `PolyElement` classes with constructors `ring`, `xring`, `vring`, `sring` for building polynomial ring objects over a given domain and ordering. |
| `fields.py` | Sparse rational function fields: `FracField` and `FracElement` classes with constructors `field`, `xfield`, `vfield`, `sfield` for building fraction field objects. |
| `ring_series.py` | Power series evaluation and manipulation using sparse polynomials; provides functions for series arithmetic, composition, inversion, and elementary function expansions (exp, log, sin, cos, etc.). |
| `monomials.py` | Tools and arithmetics for monomials of distributed polynomials: `itermonomials`, `monomial_count`, `Monomial` class, and monomial arithmetic operations (mul, div, gcd, lcm). |
| `orderings.py` | Definitions of monomial orderings: `lex`, `grlex`, `grevlex` and their inverses, plus `ProductOrder` for block orderings and the `monomial_key` helper. |
| `groebnertools.py` | Groebner basis algorithms: improved Buchberger and F5B implementations operating on sparse polynomial ring elements. |
| `fglmtools.py` | Implementation of the matrix FGLM algorithm for converting a reduced Groebner basis from one monomial ordering to another. |
| `factortools.py` | Polynomial factorization routines in characteristic zero: Zassenhaus, Wang's EEZ, and multivariate factorization over ZZ, QQ, and GF(p). |
| `euclidtools.py` | Euclidean algorithms for GCD, LCM, extended GCD, resultants, subresultants, and polynomial remainder sequences over dense recursive representations. |
| `sqfreetools.py` | Square-free decomposition algorithms: `dup_sqf_p`, `dup_sqf_list`, `dup_sqf_part`, and their multivariate (`dmp_`) counterparts. |
| `rootoftools.py` | Implementation of `CRootOf` and `RootSum` classes for representing and evaluating individual roots of irreducible polynomials. |
| `rootisolation.py` | Real and complex root isolation and refinement algorithms based on Sturm sequences, Descartes' rule, and interval arithmetic. |
| `solvers.py` | Low-level linear systems solver using polynomial ring elements; converts equations to matrix form and solves via row reduction. |
| `constructor.py` | Tools for constructing coefficient domains from expressions; `construct_domain` auto-detects whether ZZ, QQ, RR, algebraic, or composite domains are appropriate. |
| `partfrac.py` | Algorithms for partial fraction decomposition of rational functions: undetermined coefficients method and Bronstein's full partial fraction algorithm. |
| `numberfields.py` | Computational algebraic field theory: minimal polynomials, primitive elements, field isomorphisms, algebraic number recognition, and `to_number_field`. |
| `specialpolys.py` | Functions for generating special and benchmark polynomials: Swinnerton-Dyer, cyclotomic, symmetric, random, and the Fateman benchmark polynomials. |
| `orthopolys.py` | Efficient generation of classical orthogonal polynomials: Jacobi, Gegenbauer, Chebyshev (T and U), Hermite, Legendre, Laguerre, and Spherical Bessel. |
| `rationaltools.py` | Tools for manipulation of rational expressions; provides `together` for combining rational subexpressions by denesting. |
| `galoistools.py` | Dense univariate polynomials with coefficients in Galois fields (GF(p)): arithmetic, GCD, factorization (Berlekamp, Zassenhaus, Shoup), irreducibility testing, and CRT. |
| `densebasic.py` | Basic tools for dense recursive polynomial representations in K[x] or K[X]: degree, leading/trailing coefficients, conversion, inflation/deflation, and list manipulations. |
| `densearith.py` | Arithmetic operations for dense recursive polynomials: addition, subtraction, multiplication, division, power, pseudo-division, and norm computations. |
| `densetools.py` | Advanced tools for dense recursive polynomials: integration, differentiation, evaluation, composition, truncation, monic/primitive normalization, and real/imaginary part extraction. |
| `heuristicgcd.py` | Heuristic polynomial GCD algorithm (HEUGCD) for sparse polynomials in Z[X], based on evaluation-interpolation. |
| `modulargcd.py` | Modular GCD algorithms for polynomials over ZZ and algebraic number fields, using Chinese Remainder Theorem and evaluation/interpolation. |
| `distributedmodules.py` | Sparse distributed elements of free modules over polynomial rings; implements Mora's algorithm for standard bases and Groebner basis computations for modules. |
| `compatibility.py` | Compatibility interface between dense and sparse polynomial representations; re-exports all dense operations through an `IPolys` wrapper class. |
| `dispersion.py` | Computation of the dispersion set and dispersion of two polynomials, measuring how far apart integer-shifted roots can be shared. |
| `subresultants_qq_zz.py` | Functions for computing Euclidean, Sturmian, and (modified) subresultant polynomial remainder sequences, including Sylvester and Bezout matrix methods. |
| `agca/__init__.py` | Package init for algebraic geometry and commutative algebra (AGCA); re-exports the `homomorphism` function. |
| `agca/modules.py` | Computations with modules over polynomial rings: free modules, submodules, quotient modules, and sub-quotient modules with Groebner basis support. |
| `agca/ideals.py` | Computations with ideals of polynomial rings: containment, intersection, union, product, quotient, and ideal properties (prime, maximal, principal). |
| `agca/homomorphisms.py` | Computations with homomorphisms of modules and rings: kernel, image, restriction, quotient, and composition of module homomorphisms. |
| `domains/__init__.py` | Package init for mathematical domains; re-exports all domain classes and sets up aliases (ZZ, QQ, RR, CC, FF, EX, etc.). |
| `domains/domain.py` | Implementation of the abstract `Domain` base class with methods for conversion, unification, injection, and algebraic field construction. |
| `domains/field.py` | Implementation of the `Field` base class, a `Ring` subclass providing exact quotient, GCD, LCM, and revert for field domains. |
| `domains/ring.py` | Implementation of the `Ring` base class with exact quotient, modular inversion, `ideal`/`free_module`/`quotient_ring` constructors. |
| `domains/simpledomain.py` | Base class `SimpleDomain` for simple (non-composite) domains like ZZ and QQ. |
| `domains/compositedomain.py` | Base class `CompositeDomain` for composite domains like ZZ[x] and ZZ(x), with generator injection. |
| `domains/characteristiczero.py` | Mixin class `CharacteristicZero` for domains with infinite elements (characteristic zero). |
| `domains/domainelement.py` | `DomainElement` trait for objects recognized as elements of a domain, providing a `parent()` method. |
| `domains/integerring.py` | Abstract `IntegerRing` class (ZZ) with algebraic field construction and logarithm support. |
| `domains/rationalfield.py` | Abstract `RationalField` class (QQ) with algebraic field construction. |
| `domains/realfield.py` | `RealField` class (RR) for arbitrary-precision real number arithmetic using mpmath contexts. |
| `domains/complexfield.py` | `ComplexField` class (CC) for arbitrary-precision complex number arithmetic using mpmath contexts. |
| `domains/finitefield.py` | `FiniteField` class (FF/GF) for finite fields GF(p) built on modular integer arithmetic. |
| `domains/algebraicfield.py` | `AlgebraicField` class for algebraic number fields Q(alpha), using `ANP` elements and minimal polynomials. |
| `domains/expressiondomain.py` | `ExpressionDomain` class (EX) for the domain of arbitrary SymPy expressions, acting as a fallback domain. |
| `domains/polynomialring.py` | `PolynomialRing` domain class for multivariate polynomial rings backed by sparse `PolyRing` objects. |
| `domains/fractionfield.py` | `FractionField` domain class for multivariate rational function fields backed by sparse `FracField` objects. |
| `domains/quotientring.py` | `QuotientRing` domain class for quotient rings R/I with `QuotientRingElement` arithmetic. |
| `domains/old_polynomialring.py` | Legacy `PolynomialRingBase` and `GlobalPolynomialRing` implementations using dense `DMP` representations; supports global, local, and graded orderings. |
| `domains/old_fractionfield.py` | Legacy `FractionField` domain class using dense `DMF` representations for rational function fields. |
| `domains/groundtypes.py` | Ground types for mathematical domains: imports and selects Python or GMPY integer, rational, and helper functions (gcd, lcm, sqrt, factorial). |
| `domains/mpelements.py` | `RealElement` and `ComplexElement` classes wrapping mpmath floats/complexes as domain elements, plus `MPContext` for precision management. |
| `domains/modularinteger.py` | `ModularInteger` class and `ModularIntegerFactory` for constructing modular integer types used by finite fields. |
| `domains/pythonrational.py` | `PythonRational` type: a rational number implementation based on pure Python integers, used as the dtype for the Python-based QQ domain. |
| `domains/pythonrationalfield.py` | `PythonRationalField`: the rational field (QQ) implementation backed by `PythonRational`. |
| `domains/pythonintegerring.py` | `PythonIntegerRing`: the integer ring (ZZ) implementation backed by Python's native `int` type. |
| `domains/pythonfinitefield.py` | `PythonFiniteField`: finite field (FF) implementation backed by Python integers. |
| `domains/gmpyintegerring.py` | `GMPYIntegerRing`: the integer ring (ZZ) implementation backed by GMPY's `mpz` type for faster arithmetic. |
| `domains/gmpyrationalfield.py` | `GMPYRationalField`: the rational field (QQ) implementation backed by GMPY's `mpq` type for faster arithmetic. |
| `domains/gmpyfinitefield.py` | `GMPYFiniteField`: finite field (FF) implementation backed by GMPY integers. |
| `benchmarks/bench_solvers.py` | Benchmarks for the low-level linear systems solver with large polynomial systems. |
| `benchmarks/bench_groebnertools.py` | Benchmarks for Groebner basis algorithms (graph coloring problems). |
| `benchmarks/bench_galoispolys.py` | Benchmarks for polynomial factorization over Galois fields using Gathen and Shoup polynomials. |
