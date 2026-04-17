# crypto module catalog

## [`crypto.py`](crypto.py)
Classical ciphers, encoding schemes, and key-exchange protocols.

### Utilities
- `AZ(s)` — uppercase letter extraction.
- `padded_key(key, symbols)` — pads/filters a key to fill an alphabet.
- `check_and_join(phrase, symbols)` — validates and joins phrase characters.
- `cycle_list(k, n)` — cyclic permutation helper.

### Shift Cipher
- `encipher_shift` / `decipher_shift` — additive (Caesar-style) shift cipher.

### Affine Cipher
- `encipher_affine` / `decipher_affine` — affine cipher (multiply + shift mod alphabet size).

### Substitution Cipher
- `encipher_substitution(msg, old, new)` — arbitrary character substitution.

### Vigenère Cipher
- `encipher_vigenere` / `decipher_vigenere` — polyalphabetic Vigenère cipher.

### Hill Cipher
- `encipher_hill` / `decipher_hill` — matrix-based Hill cipher.

### Bifid Ciphers
- `encipher_bifid` / `decipher_bifid` — general Bifid cipher on a Polybius square.
- `bifid_square(key)` — generates an n×n Bifid square.
- `encipher_bifid5` / `decipher_bifid5` — Bifid cipher on 5×5 grid (merges J→I).
- `encipher_bifid6` / `decipher_bifid6` — Bifid cipher on 6×6 grid (alphanumeric).

### RSA
- `rsa_public_key` / `rsa_private_key` — RSA key generation from primes p, q, e.
- `encipher_rsa` / `decipher_rsa` — RSA encryption/decryption.

### Kid RSA
- `kid_rsa_public_key` / `kid_rsa_private_key` — simplified RSA key generation.
- `encipher_kid_rsa` / `decipher_kid_rsa` — Kid RSA encryption/decryption.

### Morse Code
- `morse_char` / `char_morse` — bidirectional mapping dicts between Morse sequences and characters.
- `encode_morse(msg, sep, mapping)` — encodes plaintext to Morse code; unmapped characters are silently omitted.
- `decode_morse(msg, sep, mapping)` — decodes Morse code back to plaintext.

### LFSR (Linear-Feedback Shift Register)
- `lfsr_sequence(key, fill, n)` — generates n terms of an LFSR sequence.
- `lfsr_autocorrelation(L, P, k)` — autocorrelation of an LFSR sequence.
- `lfsr_connection_polynomial(s)` — finds minimal connection polynomial for a sequence.

### ElGamal
- `elgamal_private_key` / `elgamal_public_key` — ElGamal key generation.
- `encipher_elgamal` / `decipher_elgamal` — ElGamal encryption/decryption.

### Diffie-Hellman Key Exchange
- `dh_private_key` / `dh_public_key` — Diffie-Hellman key generation.
- `dh_shared_key(key, b)` — computes shared secret from private key and other party's public key.
