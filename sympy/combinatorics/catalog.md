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
  - Properties: `is_even`, `is_odd`, `parity`, `order`, `inversions`, `cycle_structure`, `support`.
  - `commutes_with` — checks if two individual permutations commute (boolean, no search).
  - Distance metrics: `get_precedence_distance`, `get_adjacency_distance`, `get_positional_distance`.
- Low-level array-form helpers: `_af_rmul`, `_af_rmuln`, `_af_parity`, `_af_invert`, `_af_pow`, `_af_commutes_with`.

### [`perm_groups.py`](perm_groups.py)
Permutation group (set of permutations) with group-theoretic algorithms.
- `PermutationGroup` — the main group class, constructed from generating permutations.
  - **BSGS framework**: `schreier_sims`, `schreier_sims_incremental`, `schreier_sims_random`, `schreier_vector`.
  - Properties: `base`, `strong_gens`, `basic_orbits`, `basic_transversals`, `basic_stabilizers`.
  - **Coset-based ranking/unranking** (group-level, via Schreier-Sims): `coset_rank`, `coset_unrank`, `coset_factor`.
    - `coset_unrank` returns `None` when rank is negative or ≥ group order.
  - **Subgroup search**: `subgroup_search` — depth-first search for all elements satisfying a boolean predicate, with tree-pruning tests and base-change strategy.
  - **Centralizer**: `centralizer` — finds the subgroup of elements commuting with a given permutation, list, or subgroup. Type-dispatches: wraps single permutation or list into a group before searching.
  - Stabilizers: `stabilizer`, `pointwise_stabilizer`.
  - Subgroup/normality: `is_subgroup`, `is_normal`, `normal_closure`, `commutator`, `derived_subgroup`, `derived_series`.
  - Classification: `is_abelian`, `is_transitive`, `is_primitive`, `is_solvable`, `is_nilpotent`, `is_alt_sym`, `is_trivial`.
  - `center` — subgroup of elements commuting with all group elements.
  - Orbits: `orbit`, `orbits`, `orbit_rep`, `orbit_transversal`, `transitivity_degree`.
  - Membership: `contains(g, strict=True)` — tests if permutation belongs to the group; when `strict=False`, resizes `g` to match group degree before testing.
  - Element generation: `generate`, `generate_dimino`, `generate_schreier_sims`, `elements`, `order`, `random`, `random_pr`.
  - `__mul__` — direct product of two groups; extends each group's generators to act on disjoint point sets (shifts indices).
  - `minimal_block`, `max_div`, `lower_central_series`, `baseswap`.

---

## Named Groups and Construction

### [`named_groups.py`](named_groups.py)
Factory functions for standard finite groups. Each factory pre-sets algebraic properties (solvability, nilpotency, abelianness, transitivity) based on group-theoretic thresholds (e.g., `SymmetricGroup` marks solvable only for degree < 5).
- `SymmetricGroup`, `AlternatingGroup`, `CyclicGroup`, `DihedralGroup`, `AbelianGroup`, `RubikGroup`.

### [`group_constructs.py`](group_constructs.py)
Composite group construction.
- `DirectProduct` — N-ary direct product of permutation groups (optimized batch version of `PermutationGroup.__mul__`).

### [`generators.py`](generators.py)
Generates standard permutation group generators (symmetric, cyclic, alternating, dihedral, Rubik's cube).

---

## Finitely Presented and Free Groups

### [`fp_groups.py`](fp_groups.py)
Finitely presented groups and coset enumeration.
- `FpGroup`, `CosetTable`.
- Coset enumeration: `coset_enumeration_r`, `coset_enumeration_c`.
- `low_index_subgroups`, `reidemeister_presentation`.

### [`free_group.py`](free_group.py)
Free groups with symbolic generators.
- `FreeGroup` — finitely generated free group; generators are ordered by creation order.
- `FreeGroupElement` — word (element) in a free group, stored as tuple of (generator, exponent) pairs.
  - Comparison: `__lt__` implements short-lex total ordering — shorter words first, then lexicographic by generator index; each inverse is ordered between its positive generator and the next smaller generator.

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
- `Polyhedron` — polyhedron with face-permutation symmetry group.

---

## Tensor and Canonicalization

### [`tensor_can.py`](tensor_can.py)
Tensor canonicalization using double-coset representatives.
- `canonicalize`, `double_coset_can_rep`, `canonical_free`.
- `get_symmetric_group_sgs`, `tensor_gens`, `perm_af_direct_product`, `dummy_sgs`.

---

## Utilities

### [`util.py`](util.py)
Low-level algorithms for computational group theory.
- `_handle_precomputed_bsgs` — lazily fills missing BSGS structures (transversals, basic orbits, distributed strong gens) from whichever are already available; derives orbits from transversal keys when transversals are known but orbits are not.
- `_base_ordering`, `_distribute_gens_by_base`, `_orbits_transversals_from_bsgs`.
- `_strip`, `_strip_af` — strip an element through a BSGS.
- `_strong_gens_from_distr`, `_remove_gens`, `_check_cycles_alt_sym`.

### [`testutil.py`](testutil.py)
Testing utilities for permutation groups (excluded from localization targets).
