# sympy/ntheory — Number Theory

## Primality & Prime Generation

### primetest.py
Primality testing via trial division, Miller-Rabin, and Lucas probable-prime tests.

- `isprime(n)` — Main entry point. Deterministic for n < 2^64 (cascading M-R witness sets); strong BPSW for larger n.
- `mr(n, bases)` — Miller-Rabin strong pseudoprime test against explicit witness list.
- `is_square(n)` — Fast negative-filter using bit tricks, falls back to `perfect_power`.
- Lucas probable-prime tests (all use `_lucas_sequence` internally):
  - `is_lucas_prp(n)` — Standard Lucas test with Selfridge parameters.
  - `is_strong_lucas_prp(n)` — Strong variant; combined with M-R base 2 forms the strong BPSW test.
  - `is_extra_strong_lucas_prp(n)` — 20-50% faster than strong variant, different parameter selection (P=3, Q=1, incrementing P).
- `_lucas_sequence(n, P, Q, k)` — Modular Lucas sequence (U_k, V_k, Q^k) via binary ladder; handles arbitrary P, Q with special-case optimizations for Q=1 and (P=1, Q=-1).
- `_lucas_selfridge_params(n)`, `_lucas_extrastrong_params(n)`

### generate.py
Generation, enumeration, and counting of primes and composites.

- **`Sieve`** class — Dynamically growing Sieve of Eratosthenes backed by `array('l', ...)`.
  - `extend(n)` — Grow sieve to cover all primes ≤ n.
  - `extend_to_no(i)` — Ensure at least i primes are sieved (grows by 50%).
  - `primerange(a, b)` — Yield primes in [a, b) from the sieve.
  - `search(n)` — Binary search returning bounding indices (i, j); i == j iff n is prime.
  - Supports `__contains__`, `__getitem__` (1-indexed), slicing.
- `sieve` — Global `Sieve` singleton.
- `prime(nth)` — Return the nth prime (1-indexed). Uses `li(x)` binary search for large n, then walks forward.
- `primepi(n)` — Prime counting function π(n) via a DP-based Legendre-type sieve in O(n^{2/3}) time/space.
- `nextprime(n, ith=1)` / `prevprime(n)` — Navigation using 6k±1 wheel.
- `primerange(a, b)` — Module-level generator; delegates to sieve or walks via `nextprime`.
- `randprime(a, b)` — Random prime in [a, b) via Bertrand's postulate.
- `primorial(n, nth=True)` — Product of first n primes (or primes ≤ n).
- `composite(nth)` / `compositepi(n)` — Composite analogs of `prime` / `primepi`.
- `cycle_length(f, x0, nmax=None, values=False)` — Brent's cycle-detection algorithm; returns (λ, μ) or iterated values.

## Factorization & Divisors

### factor_.py
Integer factorization, divisor enumeration, and multiplicative arithmetic functions.

- **Factorization core:**
  - `factorint(n, limit=None, use_trial=True, use_rho=True, use_pm1=True, verbose=False, visual=None)` — Main factorizer returning `{prime: exponent}`. Combines trial division (up to 2^15), Fermat's method, Pollard rho, and Pollard p-1. Accepts dicts and `Mul` objects for re-factoring; `visual=True` returns unevaluated `Mul`.
  - `factorrat(rat, ...)` — Factor a `Rational`; negative exponents for denominator primes.
  - `pollard_rho(n, s, a, retries, seed, max_steps, F)` — Pollard's rho with Brent's cycle detection.
  - `pollard_pm1(n, B, a, retries, seed)` — Pollard's p-1 method; effective when p-1 is B-power-smooth.
  - `_factorint_small(factors, n, limit, fail_max)` — Trial division using 6k±1 wheel with fail-count cutoff.
  - `_trial`, `_check_termination` — Internal helpers.
- **Factorization analysis:**
  - `primefactors(n, limit=None)` — Sorted list of distinct prime factors.
  - `multiplicity(p, n)` — Largest m s.t. p^m | n; works on rationals too.
  - `perfect_power(n, candidates=None, big=True, factor=True)` — Returns `(b, e)` or `False`.
  - `smoothness(n)` — Returns (B-smooth, B-power-smooth) pair.
  - `smoothness_p(n, m=-1, power=0, visual=None)` — Per-factor smoothness analysis; useful for estimating p±1 method effectiveness.
  - `trailing(n)` — Count trailing zero bits (= v_2(n)); uses lookup table + doubling.
- **Divisor functions:**
  - `divisors(n, generator=False)` / `_divisors(n)` — All divisors, sorted or as generator.
  - `divisor_count(n, modulus=1)` — Number of divisors (optionally filtered by modulus).
  - `udivisors(n)` / `udivisor_count(n)` — Unitary divisors (d | n with gcd(d, n/d) = 1).
  - `antidivisors(n)` / `antidivisor_count(n)` — Numbers that "almost don't divide" n.
  - `divisor_sigma(n, k=1)` (class) — σ_k(n), the sum-of-kth-powers-of-divisors function.
  - `udivisor_sigma(n, k=1)` (class) — Unitary analog of σ_k.
- **Arithmetic functions (as SymPy `Function` subclasses):**
  - `totient(n)` — Euler's φ(n).
  - `reduced_totient(n)` — Carmichael's λ(n).
  - `primenu(n)` — ω(n), number of distinct prime factors.
  - `primeomega(n)` — Ω(n), number of prime factors with multiplicity.
- **Misc:**
  - `core(n, t=2)` — t-th power free part of n (squarefree part when t=2).
  - `digits(n, b=10)` — Digit list in base b (first element is the base).

## Modular Arithmetic & Residues

### residue_ntheory.py
Quadratic/nth-power residues, modular square/nth roots, primitive roots, Legendre/Jacobi symbols, and the Möbius function.

- **Primitive roots & order:**
  - `n_order(a, n)` — Multiplicative order of a mod n.
  - `primitive_root(p)` — Smallest primitive root of p (or None if none exists); handles p, 2p, p^k, 2·p^k.
  - `is_primitive_root(a, p)` — Test via `n_order(a, p) == totient(p)`.
- **Quadratic residues & symbols:**
  - `is_quad_residue(a, p)` — True if a is a QR mod p; uses Euler criterion for odd primes, falls back to `sqrt_mod` for composites.
  - `legendre_symbol(a, p)` — Returns 0, 1, or -1 for odd prime p.
  - `jacobi_symbol(m, n)` — Generalized Legendre symbol for odd n; O(log^2) via quadratic reciprocity reduction.
- **Modular square roots:**
  - `sqrt_mod(a, p, all_roots=False)` — Find x with x² ≡ a (mod p). Returns smallest root ≤ p//2, or sorted list of all roots.
  - `sqrt_mod_iter(a, p, domain=int)` — Iterator over all solutions; uses CRT to combine prime-power solutions.
  - `_sqrt_mod_prime_power(a, p, k)` — Core solver for x² ≡ a (mod p^k) when p ∤ a. Uses direct formulas for p ≡ 3 (mod 4), p ≡ 5 (mod 8), Tonelli-Shanks otherwise; Hensel lifts for k > 1.
  - `_sqrt_mod1(a, p, n)` — Handles x² ≡ a (mod p^n) when p | a.
  - `_sqrt_mod_tonelli_shanks(a, p)` — Tonelli-Shanks for p ≡ 1 (mod 8).
- **Nth-power residues & roots:**
  - `is_nthpow_residue(a, n, m)` — True if x^n ≡ a (mod m) has solutions.
  - `nthroot_mod(a, n, p, all_roots=False)` — Solve x^n ≡ a (mod p) via Johnston's generalized root algorithm; requires primitive root to exist.
- **Möbius function:**
  - `mobius(n)` (class) — μ(n): 1 if n=1, 0 if n has squared factor, (-1)^k if squarefree with k prime factors.
- `quadratic_residues(p)` — Exhaustive list of QRs mod p.

### modular.py
Chinese Remainder Theorem and systems of linear congruences.

- `crt(m, v, symmetric=False, check=True)` — CRT for pairwise coprime moduli; returns (result, lcm). Falls back to `solve_congruence` when moduli are not coprime.
- `crt1(m)` / `crt2(m, v, mm, e, s)` — Two-phase CRT for repeated use with the same moduli (precomputation + evaluation).
- `solve_congruence(*remainder_modulus_pairs, symmetric=False, check=True)` — General system solver via successive substitution; handles non-coprime moduli.
- `symmetric_residue(a, m)` — Map residue to [-m/2, m/2].

## Continued & Egyptian Fractions

### continued_fraction.py
Continued fraction expansion, reduction, and convergent computation.

- `continued_fraction_periodic(p, q, d=0)` — CF expansion of (p + √d)/q; detects and returns repeating block as a nested list.
- `continued_fraction_iterator(x)` — Lazy CF expansion of any SymPy number (Rational, pi, etc.).
- `continued_fraction_reduce(cf)` — Reconstruct rational or quadratic irrational from (possibly periodic) CF.
- `continued_fraction_convergents(cf)` — Iterator of successive convergents p_n/q_n.

### egyptian_fraction.py
Decompose a positive rational into a sum of distinct unit fractions.

- `egyptian_fraction(r, algorithm="Greedy")` — Returns list of denominators. Supports four algorithms:
  - **Greedy** (Fibonacci-Sylvester): at most p terms for p/q, doubly-exponential worst-case denominator growth.
  - **Graham Jewett**: always 2^(x/gcd(x,y)) - 1 terms; tends to blow up.
  - **Takenouchi**: similar to Graham Jewett but merges duplicates differently.
  - **Golomb**: uses modular inverses; equivalent to Bleicher's CF method.
- `egypt_greedy`, `egypt_graham_jewett`, `egypt_takenouchi`, `egypt_golomb` — Algorithm implementations.
- `egypt_harmonic(r)` — Prefix extraction via harmonic series for r ≥ 1.

## Combinatorial Number Theory

### multinomial.py
Binomial and multinomial coefficient computation.

- `binomial_coefficients(n)` — Dict `{(k1, k2): C(n, k1)}` with k1 + k2 = n.
- `binomial_coefficients_list(n)` — Row of Pascal's triangle as a list.
- `multinomial_coefficients(m, n)` — Dict `{(k1,...,km): C}` for all compositions of n into m parts. Uses co-lex enumeration; delegates to iterator when m ≥ 2n.
- `multinomial_coefficients_iterator(m, n)` — Memory-efficient iterator exploiting zero-stripping symmetry for large m.

### partitions_.py
Integer partition counting via the Hardy-Ramanujan-Rademacher formula.

- `npartitions(n, verbose=False)` — Exact P(n) using the HRR series with dynamically reduced precision. Validated through 10^10; raises for n > ~1.7×10^11.
- `_a(n, k, prec)` — Inner (Kloosterman-like) sum of the HRR formula with prime-power CRT decomposition.
- `_d(n, j, prec)` — Sinh/cosh term in the outer HRR sum.
- `_pre()` — Precompute smallest-prime-factor and totient arrays up to 10^5 (lazy, on first call).

## Special Computations

### bbp_pi.py
Compute hexadecimal digits of π at arbitrary positions using the Bailey-Borwein-Plouffe formula.

- `pi_hex_digits(n, prec=14)` — Return `prec` hex digits of π starting at position n (0-indexed, 0 → "3..."). Working precision is auto-calculated.
- `_series(j, n, prec)` — Left + right partial sums of the BBP series.
- `_dn(n, prec)` — Working precision controller.

## Package Init

### \_\_init\_\_.py
Re-exports the public API from all submodules. No logic beyond imports.
