# sympy/crypto — Catalog

> Part of [SymPy](../catalog.md). Classical cryptographic ciphers (Caesar, Vigenere, RSA, etc.).

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package initializer that re-exports all public cipher functions, key generators, LFSR utilities, and Morse code helpers from `crypto.py`. |
| `crypto.py` | Core implementation of classical ciphers (shift, affine, substitution, Vigenere, Hill, Bifid), public-key schemes (RSA, Kid-RSA, ElGamal), Diffie-Hellman key exchange, LFSR routines (`lfsr_sequence`, `lfsr_autocorrelation`, `lfsr_connection_polynomial`), and Morse code encoding/decoding. Contains all input validation and error-raising logic (e.g., TypeError for invalid arguments). |
| `tests/test_crypto.py` | Test suite exercising the cipher, LFSR, and utility functions defined in `crypto.py`. |
