# sympy/ntheory — Catalog

> Part of [SymPy](../catalog.md). Number theory: primes, factorization, residues, continued fractions, and multiplicative functions.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package initializer that re-exports the public API from submodules (primes, factoring, residues, partitions, multinomials, continued fractions, Egyptian fractions). |
| `bbp_pi.py` | Implements the Bailey-Borwein-Plouffe (BBP) formula for computing hexadecimal digits of pi at an arbitrary starting position. |
| `continued_fraction.py` | Provides routines for periodic continued fraction expansion of quadratic irrationals, convergent computation, iterating over continued fraction terms, and reducing a continued fraction list back to a rational expression. |
| `egyptian_fraction.py` | Decomposes a positive rational number into a sum of distinct unit fractions (Egyptian fraction expansion) using selectable algorithms (Greedy, Graham-Jewett, Takenouchi, Golomb). |
| `factor_.py` | Integer factorization and divisor-related functions including `factorint`, `divisors`, `totient`, `multiplicity` (p-adic valuation), Pollard rho/p-1 methods, smoothness analysis, perfect power detection, and divisor-counting functions (`divisor_sigma`, `primenu`, `primeomega`). |
| `generate.py` | Prime and composite number generation via a dynamically growing Sieve of Eratosthenes, plus helpers like `nextprime`, `prevprime`, `primerange`, `primepi`, `randprime`, `primorial`, `composite`, `compositepi`, and `cycle_length`. |
| `modular.py` | Solves systems of linear congruences using the Chinese Remainder Theorem (`crt`, `crt1`, `crt2`) and provides a general `solve_congruence` interface with symmetric residue support. |
| `multinomial.py` | Computes binomial and multinomial coefficients, returning them as dictionaries or lists, with an efficient iterator variant for multinomial coefficients. |
| `partitions_.py` | Computes the integer partition function p(n) using the Hardy-Ramanujan-Rademacher series with mpmath arbitrary-precision arithmetic. |
| `primetest.py` | Primality testing routines including Miller-Rabin, Lucas probable-prime tests (standard, strong, extra-strong), a deterministic `isprime` dispatcher, and a fast perfect-square check. |
| `residue_ntheory.py` | Modular arithmetic and number-theoretic symbol functions: multiplicative order, primitive roots, quadratic and nth-power residues, square-root mod, Legendre/Jacobi symbols, the Möbius function (`mobius` class — maps positive integers to {-1, 0, 1} based on square-free prime factorization), and discrete logarithm algorithms (trial, baby-step giant-step, Pollard rho, Pohlig-Hellman). |
