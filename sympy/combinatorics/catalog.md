# Combinatorics Module Catalog

## Glossary

- **BSGS**: Base and Strong Generating Set — a compact representation of a permutation group enabling efficient membership testing and enumeration.
- **Coset decomposition**: Factoring a group element via Schreier-Sims transversals; used for group-level ranking/unranking (`coset_rank`/`coset_unrank` in `perm_groups.py`).
- **Lexicographic rank/unrank**: Converting between a single permutation and its position in lex order; lives in `permutations.py` (`rank`, `unrank_lex`).
- **Non-lex rank/unrank**: Linear-time ranking that does not enforce lexicographic order; also in `permutations.py` (`rank_nonlex`, `unrank_nonlex`).

## Notes

- **`perm_groups.py` vs `permutations.py`**: `permutations.py` defines individual `Permutation` objects and their properties (cycles, parity, inversions, lex ranking). `perm_groups.py` defines `PermutationGroup` — group-theoretic operations on *sets* of permutations (stabilizers, centralizers, coset ranking, subgroup search, orbits, normality).
- **Centralizer**: The set of group elements commuting with a given element or subgroup. Computed in `perm_groups.py` via `PermutationGroup.centralizer`, which type-dispatches on input (single permutation, list, or group). This is distinct from `Permutation.commutes_with` in `permutations.py`, which only checks if two individual permutations commute.

---

## Permutation and Group Core

### [`permutations.py`](permutations.py)
Individual permutation representation, construction, and properties.
- `Cycle` — wrapper around dict for disjoint cycle notation.
- `Permutation` — core permutation class (array form, cyclic form, composition, inversion).
  - Ranking/unranking of individual permutations in lexicographic order: `rank`, `unrank_lex`, `next_lex`.
  - Non-lex ranking: `rank_nonlex`, `unrank_nonlex`, `rank_trotterjohnson`, `unrank_trotterjohnson`.
  - Properties: `is_even`, `is_odd`, `parity`, `order`, `inversions`, `support`.
  - `cycle_structure` — dict mapping each cycle length to its multiplicity; fixed points (self-mapping elements) counted as length-1 cycles.
  - `commutes_with` — checks if two individual permutations commute (boolean, no search).
  - Distance metrics: `get_precedence_distance`, `get_adjacency_distance`, `get_positional_distance`.
- Low-level array-form helpers: `_af_rmul`, `_af_rmuln`, `_af_parity`, `_af_invert`, `_af_pow`, `_af_commutes_with`.

### [`perm_groups.py`](perm_groups.py)
Permutation group (set of permutations) with group-theoretic algorithms.
- `PermutationGroup` — the main group class, constructed from generating permutations.
  - **BSGS framework**: `schreier_sims`, `schreier_sims_incremental`, `schreier_vector`.
  - `schreier_sims_random` — randomized BSGS computation: orchestrates a sifting loop that samples random elements, decides when to extend the base sequence (new anchor points), and amends stabilizer chains/orbits when sifting fails. Uses `_strip` from `util.py` as a subroutine.
  - Properties: `base`, `strong_gens`, `basic_orbits`, `basic_transversals`, `basic_stabilizers`.
  - **Coset-based ranking/unranking** (group-level, via Schreier-Sims): `coset_rank`, `coset_unrank`, `coset_factor`.
    - `coset_unrank` returns `None` when rank is negative or ≥ group order.
  - **Subgroup search**: `subgroup_search` — depth-first search for all elements satisfying a boolean predicate, with tree-pruning tests and base-change strategy.
  - **Centralizer**: `centralizer` — finds the subgroup of elements commuting with a given permutation, list, or subgroup. Type-dispatches: wraps single permutation or list into a group before searching.
  - Stabilizers: `stabilizer`, `pointwise_stabilizer`.
  - Subgroup/normality: `is_subgroup`, `is_normal`, `normal_closure`, `commutator`, `derived_subgroup`, `derived_series`.
  - Classification: `is_abelian`, `is_transitive`, `is_primitive`, `is_solvable`, `is_nilpotent`, `is_alt_sym`, `is_trivial`.
  - `center` — subgroup of permutation group elements commuting with all group elements; computed via subgroup search.
  - Orbits: `orbit`, `orbits`, `orbit_rep`, `orbit_transversal`, `transitivity_degree`.
  - Membership: `contains(g, strict=True)` — tests if permutation belongs to the group; when `strict=False`, resizes `g` to match group degree before testing.
  - Element generation: `generate`, `generate_dimino`, `generate_schreier_sims`, `elements`, `order`, `random`, `random_pr`.
  - `__mul__` — pairwise direct product of exactly two groups; shifts generators to act on disjoint point sets. For N-ary products, use `DirectProduct` in `group_constructs.py`.
  - `minimal_block`, `max_div`, `lower_central_series`, `baseswap`.

---

## Named Groups and Construction

### [`named_groups.py`](named_groups.py)
Factory functions returning `PermutationGroup` objects for standard finite groups, with pre-set algebraic properties (e.g. `_is_nilpotent`, `_is_solvable`, `_is_abelian`, `_is_transitive`). Contrast with `generators.py`, which yields individual permutation elements.
- `SymmetricGroup(n)` — constructs Sn (full bijection group on n elements); pre-sets `_is_solvable = True` iff n < 5 (reflecting that An is simple for n ≥ 5).
- `CyclicGroup`, `AbelianGroup`, `RubikGroup`.
- `DihedralGroup(n)` — constructs Dn with rotation + reflection generators; special-case construction for n=1 (single transposition in S2) and n=2 (three generators on 4 elements, Klein 4-group embedding in S4). Pre-sets `_is_nilpotent = True` iff n is a power of 2.
- `AlternatingGroup(n)` — constructs An with explicit generators: uses different generators for odd n vs even n (full n-cycle vs (n−1)-cycle fixing 0).

### [`group_constructs.py`](group_constructs.py)
Composite group construction.
- `DirectProduct(*groups)` — N-ary direct product of arbitrarily many permutation groups in a single pass; shifts each group's generators onto disjoint index slices of a combined identity mapping. Faster than repeated pairwise `__mul__` calls.

### [`generators.py`](generators.py)
Yields individual `Permutation` elements (not `PermutationGroup` objects) for standard groups. Contrast with `named_groups.py`, which returns constructed `PermutationGroup` objects with pre-set properties.
- `symmetric(n)`, `cyclic(n)`, `alternating(n)` — yield all permutations of Sn, Cn, An respectively. `alternating` filters by `is_even`.
- `dihedral(n)` — yields all 2n elements of Dn. Special-case embeddings for n=1 (in S2) and n=2 (Klein 4-group in S4) where Dn is not a subgroup of Sn.
- `rubik_cube_generators()` — standard 3×3 Rubik's cube face-turn permutations.
- `rubik(n)` — NxN Rubik's cube permutation generator; builds all plane-rotation permutations for each face direction.

---

## Finitely Presented and Free Groups

### [`fp_groups.py`](fp_groups.py)
Finitely presented groups and coset enumeration.
- `FpGroup`, `CosetTable`.
- Coset enumeration: `coset_enumeration_r`, `coset_enumeration_c`.
- `low_index_subgroups`, `reidemeister_presentation`.

### [`free_group.py`](free_group.py)
Free groups with symbolic generators.
- Constructor entry points: `free_group(symbols)`, `xfree_group`, `vfree_group` — create a `FreeGroup` from a string, Symbol/Expr, or sequence thereof.
  - `_parse_symbols` — normalizes the `symbols` argument; accepts str, Expr, sequence of str, or sequence of Expr. Mixed-type sequences (e.g. str and Symbol together) raise `ValueError`.
- `FreeGroup` — finitely generated free group; generators are ordered by creation order.
  - `center` — returns the center of the free group (always `{identity}`, since free groups of rank ≥ 2 are non-abelian).
  - `contains`, `is_subgroup`, `identity`.
- `FreeGroupElement` — word (element) in a free group, stored as tuple of (generator, exponent) pairs.
  - Comparison: `__lt__` implements short-lex total ordering — shorter words first, then lexicographic by generator index; each inverse is ordered between its positive generator and the next smaller generator.
  - `is_cyclic_conjugate` — checks if two words are cyclic conjugates (rotational rearrangements) after cyclic reduction; uses string-doubling rotation detection.
  - `identity_cyclic_reduction` — returns the unique cyclically reduced form of a word: combines exponents of first and last syllables, stripping them if they cancel completely.
  - Word operations: `cyclic_reduction`, `number_syllables`, `sub_syllables`, `substituted_word`, `letter_form`.

---

## Combinatorial Objects

### [`partitions.py`](partitions.py)
Set and integer partitions.
- `Partition` — set partition with RGS (restricted growth string) representation.
- `IntegerPartition` — partition of an integer.
- `RGS_enum`, `RGS_unrank`, `RGS_rank`, `RGS_generalized`, `random_integer_partition`.

### [`graycode.py`](graycode.py)
Gray code representation and bit-level operations (rank/unrank, conversion).
- `GrayCode` — n-bit Gray code object; generates all codes, supports `rank`, `unrank`, `next`, `current`.
- `gray_to_bin`, `bin_to_gray`, `get_subset_from_bitstring`, `graycode_subsets`.
- **Note**: For traversing *subsets* in Gray code order, see `subsets.py::Subset.iterate_graycode`.

### [`subsets.py`](subsets.py)
Subset generation and manipulation via binary, lexicographic, and Gray code enumeration.
- `Subset` — subset ranking/unranking in binary, lexicographic, and Gray code orders.
  - Gray code traversal: `iterate_graycode`, `next_gray`, `prev_gray` — step through subsets in reflected binary code order with modular wraparound.
- `ksubsets` — k-element subsets of a set.

### [`prufer.py`](prufer.py)
Prufer sequence correspondence for labeled trees.
- `Prufer` — bijection between labeled trees and Prufer codes.

### [`polyhedron.py`](polyhedron.py)
Polyhedral symmetry groups (tetrahedron, cube/octahedron, dodecahedron/icosahedron).
- `Polyhedron` — 3D solid defined by named corners, faces, and a permutation group (`pgroup`).
  - `rotate(perm)` — apply a permutation to vertices in place. Accepts `Permutation` or int index into `pgroup`. Validates permutation size matches vertex count; raises `ValueError` on mismatch.
  - Properties: `corners`, `faces`, `edges`, `pgroup`, `size`, `array_form`, `cyclic_form`.

---

## Tensor and Canonicalization

### [`tensor_can.py`](tensor_can.py)
Tensor canonicalization using double-coset representatives.
- `canonicalize`, `double_coset_can_rep`, `canonical_free`.
- `get_symmetric_group_sgs`, `tensor_gens`, `perm_af_direct_product`, `dummy_sgs`.
- `get_minimal_bsgs(base, gens)` — attempts to compute a lexicographically minimal BSGS via `schreier_sims_incremental`; returns `None` if the result is not minimal (no baseswap fallback).
- `_is_minimal_bsgs` — checks if a BSGS has the lexicographically smallest base.
- `get_transversals` — returns transversals for a group given its BSGS.

---

## Utilities

### [`util.py`](util.py)
Low-level algorithms for computational group theory.
- `_handle_precomputed_bsgs` — lazily fills missing BSGS structures (transversals, basic orbits, distributed strong gens) from whichever are already available; derives orbits from transversal keys when transversals are known but orbits are not.
- `_distribute_gens_by_base(base, gens)` — partitions generators into basic stabilizer levels; each level i collects gens fixing the first i base points; empty levels receive the identity element.
- `_base_ordering`.
- `_orbits_transversals_from_bsgs` — computes basic orbits and transversal dicts from distributed strong generators; `transversals_only=True` skips orbit lists and returns only the coset-representative mappings.
- `_strip` — single-pass sift of one permutation through an existing BSGS; returns residual and level. Does not modify the BSGS (caller decides how to react to failure).
- `_strip_af` — optimized array-form variant of `_strip`; returns `False` (instead of identity) when element is fully sifted.
- `_remove_gens(base, strong_gens)` — prunes redundant generators from a strong generating set; iterates stabilizer levels in reverse, skipping removal when it would leave zero generators at a level.
- `_strong_gens_from_distr`, `_check_cycles_alt_sym`.

### [`testutil.py`](testutil.py)
Testing utilities for permutation groups (excluded from localization targets).
