# sympy/crypto — Cryptography

All functionality lives in a single file; `__init__.py` re-exports the public API.

## Utilities

### crypto.py — helpers & alphabet constants

Internal helpers for message preparation and alphabet manipulation.

- `AZ(s=None)` — filter string to uppercase A-Z; returns the full alphabet when called with no args.
- `padded_key(key, symbols, filter=True)` — deduplicate `key` and pad with remaining `symbols` to form a full permutation.
- `check_and_join(phrase, symbols=None, filter=None)` — join phrase parts into a string; optionally validate against allowed `symbols`.
- `_prep(msg, key, alp, default=None)` — normalize message, key, and alphabet for cipher functions.
- `cycle_list(k, n)` — return `range(n)` rotated left by `k`.
- Module-level constants: `bifid5` (A-Z minus J), `bifid6` (A-Z + 0-9), `bifid10` (`string.printable`).

## Classical Ciphers

### crypto.py — Shift (Caesar) cipher

Monoalphabetic substitution by fixed offset.

- `encipher_shift(msg, key, symbols=None)` — encrypt by shifting each character right by `key`.
- `decipher_shift(msg, key, symbols=None)` — decrypt (delegates to `encipher_shift` with negated key).

### crypto.py — Affine cipher

Monoalphabetic substitution via the map x → ax + b (mod N).

- `encipher_affine(msg, key, symbols=None, _inverse=False)` — encrypt with key `(a, b)`; requires gcd(a, N) = 1.
- `decipher_affine(msg, key, symbols=None)` — decrypt by computing the modular inverse mapping.

### crypto.py — Substitution cipher

- `encipher_substitution(msg, old, new=None)` — general character-for-character replacement; `old` may be a mapping dict.

### crypto.py — Vigenère cipher

Polyalphabetic substitution with a repeating keyword.

- `encipher_vigenere(msg, key, symbols=None)` — encrypt by cycling key offsets over the message.
- `decipher_vigenere(msg, key, symbols=None)` — decrypt by subtracting key offsets.

### crypto.py — Hill cipher

Polygraphic cipher using matrix multiplication over Z_N.

- `encipher_hill(msg, key, symbols=None, pad="Q")` — encrypt blocks of size k with a k×k invertible matrix key; pads the message to a multiple of k.
- `decipher_hill(msg, key, symbols=None)` — decrypt using the modular inverse of the key matrix.

### crypto.py — Bifid cipher

Fractional substitution cipher using an n×n Polybius square.

- `encipher_bifid(msg, key, symbols=None)` — general n×n Bifid encryption (default 10×10 with `string.printable`).
- `decipher_bifid(msg, key, symbols=None)` — corresponding decryption.
- `bifid_square(key)` — display the Polybius square as a SymPy `Matrix` of `Symbol`s.
- `encipher_bifid5(msg, key)` / `decipher_bifid5(msg, key)` — 5×5 variant (A-Z, J omitted).
- `bifid5_square(key=None)` — build/display the 5×5 square.
- `encipher_bifid6(msg, key)` / `decipher_bifid6(msg, key)` — 6×6 variant (A-Z + digits).
- `bifid6_square(key=None)` — build/display the 6×6 square.

## Public-Key Cryptography

### crypto.py — RSA

Textbook RSA (no padding, operates on single integers).

- `rsa_public_key(p, q, e)` — return `(n, e)` or `False` if inputs violate primality / coprimality.
- `rsa_private_key(p, q, e)` — return `(n, d)` where d = e⁻¹ mod φ(n), or `False`.
- `encipher_rsa(i, key)` / `decipher_rsa(i, key)` — modular exponentiation encrypt / decrypt.

### crypto.py — Kid RSA

Simplified RSA variant (multiplication only, no exponentiation) suitable for teaching.

- `kid_rsa_public_key(a, b, A, B)` / `kid_rsa_private_key(a, b, A, B)` — generate `(n, e)` and `(n, d)`.
- `encipher_kid_rsa(msg, key)` / `decipher_kid_rsa(msg, key)` — encrypt / decrypt as `msg * e mod n`.

### crypto.py — ElGamal

ElGamal encryption based on the Discrete Logarithm Problem.

- `elgamal_private_key(digit=10, seed=None)` — generate `(p, r, d)`: prime, primitive root, random exponent.
- `elgamal_public_key(key)` — derive `(p, r, r^d mod p)` from the private key.
- `encipher_elgamal(i, key, seed=None)` — encrypt integer `i` into a two-element tuple `(c1, c2)`.
- `decipher_elgamal(msg, key)` — decrypt `(c1, c2)` back to the original integer.

### crypto.py — Diffie-Hellman key exchange

Key-agreement protocol (not a cipher); both parties derive a shared secret.

- `dh_private_key(digit=10, seed=None)` — generate `(p, g, a)`.
- `dh_public_key(key)` — compute `(p, g, g^a mod p)`.
- `dh_shared_key(key, b)` — compute the shared secret `x^b mod p`.

## Encoding

### crypto.py — Morse code

- `encode_morse(msg, sep='|', mapping=None)` — plaintext → Morse string; letters separated by `sep`, words by double `sep`.
- `decode_morse(msg, sep='|', mapping=None)` — Morse string → plaintext.
- `morse_char` / `char_morse` — bidirectional lookup dicts.

## Linear-Feedback Shift Registers (LFSR)

### crypto.py — LFSR utilities

Routines for generating and analyzing binary sequences over finite fields.

- `lfsr_sequence(key, fill, n)` — generate `n` terms of the LFSR defined by connection coefficients `key` and initial state `fill` (elements of `FF(p)`).
- `lfsr_autocorrelation(L, P, k)` — compute the k-th autocorrelation value of a periodic LFSR sequence.
- `lfsr_connection_polynomial(s)` — recover the minimal connection polynomial from a sequence using the Berlekamp–Massey algorithm.
