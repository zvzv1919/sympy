# ntheory Module Catalog

## Prime Generation and Testing

### [`generate.py`](generate.py)
Generating, counting, and enumerating primes and composites.
- `Sieve` — infinite dynamically-growing sieve of Eratosthenes (prime cache); `search(n)` returns bounding prime indices; `__contains__` membership test short-circuits even numbers without cache lookup
- `prime(nth)`, `primepi(n)` — nth prime / count of primes ≤ n
- `nextprime`, `prevprime` — adjacent prime lookup
- `primerange(a, b)` — generate primes in half-open range
- `randprime(a, b)` — random prime in range
- `primorial(n)` — product of first n primes
- `composite(nth)`, `compositepi(n)` — nth composite / count composites ≤ n
- `cycle_length(f, x0)` — detect cycle length in iterated integer sequence

### [`primetest.py`](primetest.py)
Primality testing algorithms.
- `isprime(n)` — main primality test; layered strategy: trial division, Fermat test with pseudoprime exclusion for small n, sieve bisection, deterministic Miller-Rabin for n < 2^64, strong BPSW for larger
- `mr(n, bases)` — Miller-Rabin strong pseudoprime test
- `_lucas_sequence(n, P, Q, k)` — modular Lucas sequence (U_k, V_k, Q^k mod n) via binary doubling ladder; three optimized branches for Q=1, P=1∧Q=−1, and general case
- `is_lucas_prp`, `is_strong_lucas_prp`, `is_extra_strong_lucas_prp` — Lucas probable-prime tests (use `_lucas_sequence` internally)
- `is_square(n)` — fast perfect-square test; uses two-stage bitwise modular filtering (mod 128 and mod 63 with magic constants) to quickly reject non-squares before expensive exact verification via `perfect_power`

## Factorization and Divisor Analysis

### [`factor_.py`](factor_.py)
Integer factorization, divisor enumeration, and digit/base decomposition.
- `factorint(n)` — complete prime factorization of an integer
- `factorrat(rat)` — prime factorization of a rational number
- `primefactors(n)` — sorted list of distinct prime factors
- `pollard_rho`, `pollard_pm1` — probabilistic factorization algorithms
- `perfect_power(n)` — detect if n = b^e for some integers b, e with e ≥ 2 (general perfect powers, not limited to squares)
- `multiplicity(p, n)` — exponent of prime p in n
- `smoothness(n)`, `smoothness_p(n)` — B-smooth and B-power-smooth analysis
- `trailing(n)` — count trailing zero bits (factors of 2)
- `divisors(n)`, `divisor_count(n)` — all divisors / count of divisors
- `udivisors(n)`, `udivisor_count(n)` — unitary divisors and count
- `antidivisors(n, generator=False)` — antidivisors of n; `generator=True` returns lazy iterator instead of sorted list
- `antidivisor_count(n)` — number of antidivisors
- `core(n, t)` — t-free core of n
- `digits(n, b=10)` — decompose integer n into list of digits in base b; encodes sign via negated base prefix
- `totient` — Euler's totient function (symbolic Function class)
- `reduced_totient` — Carmichael's λ(n): smallest m > 0 such that a^m ≡ 1 mod n for **all** a coprime to n; special-cases prime factor 2 with multiplicity > 2
- `divisor_sigma`, `udivisor_sigma` — (unitary) divisor sigma functions
- `primenu`, `primeomega` — count of distinct / total prime factors

## Modular Arithmetic and Residues

### [`residue_ntheory.py`](residue_ntheory.py)
Modular roots, residues, and multiplicative-order computations.
- `n_order(a, n)` — multiplicative order of a modulo n; raises ValueError if a and n are not coprime
- `primitive_root(p)`, `is_primitive_root(a, p)` — primitive root lookup/test
- `sqrt_mod(a, p)`, `sqrt_mod_iter(a, p)` — modular square root (single / iterator)
- `nthroot_mod(a, n, p)` — modular nth root; raises `NotImplementedError` when p has no primitive root (e.g. most composite moduli)
- `is_quad_residue(a, p)`, `quadratic_residues(p)` — quadratic residue test / enumeration
- `is_nthpow_residue(a, n, m)` — nth-power residue test
- `legendre_symbol(a, p)`, `jacobi_symbol(m, n)` — Legendre and Jacobi symbols
- `mobius` — Möbius function: maps n → {-1, 0, 1}; returns 0 if n has squared prime factors, (-1)^k if square-free with k primes, 1 if n=1 (symbolic Function class)

### [`modular.py`](modular.py)
Chinese Remainder Theorem and systems of linear congruences (not power-residue/nth-root equations).
- `crt(m, v)` — Chinese Remainder Theorem solver
- `crt1`, `crt2` — two-phase CRT for repeated use with same moduli
- `symmetric_residue(a, m)` — symmetric residue with |r| ≤ m/2
- `solve_congruence(*remainder_modulus_pairs)` — solve systems of linear congruences

## Combinatorics and Partitions

### [`multinomial.py`](multinomial.py)
Binomial and multinomial coefficient computation.
- `binomial_coefficients(n)` — dict of C(n, k) values
- `binomial_coefficients_list(n)` — Pascal's triangle row as list
- `multinomial_coefficients(m, n)` — multinomial coefficient dict
- `multinomial_coefficients_iterator(m, n)` — lazy iterator over multinomial coefficients

### [`partitions_.py`](partitions_.py)
Integer partition counting via Hardy-Ramanujan-Rademacher formula.
- `npartitions(n)` — exact number of partitions of n

## Fraction Expansions

### [`continued_fraction.py`](continued_fraction.py)
Continued fraction operations for rationals and quadratic irrationals.
- `continued_fraction_periodic(p, q, d)` — periodic CF expansion of (p + √d) / q
- `continued_fraction_iterator(x)` — lazy CF expansion of a real number
- `continued_fraction_reduce(cf)` — reconstruct rational from CF terms
- `continued_fraction_convergents(cf)` — sequence of convergent fractions

### [`egyptian_fraction.py`](egyptian_fraction.py)
Egyptian fraction (unit-fraction sum) decomposition.
- `egyptian_fraction(r, algorithm)` — decompose rational r into sum of distinct unit fractions
- Algorithms: Greedy (default), Graham-Jewett, Takenouchi, Golomb
- Each algorithm differs in how duplicate denominators are resolved during expansion
- Takenouchi: resolves duplicates via even/odd parity check on the repeated denominator
- Graham-Jewett: resolves duplicates by incrementing one copy and appending a product term

## Special Computations

### [`bbp_pi.py`](bbp_pi.py)
Hexadecimal digits of π via the Bailey–Borwein–Plouffe formula.
- `pi_hex_digits(n, prec)` — return hex digits of π starting at position n
