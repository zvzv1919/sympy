# Combinatorics Module Catalog

## Glossary

- **BSGS**: Base and Strong Generating Set — a compact representation of a permutation group as a stabilizer chain, enabling efficient membership testing (sifting/stripping) and enumeration.
- **Coset decomposition**: Factoring a single group element via Schreier-Sims transversals; `coset_rank`/`coset_unrank`/`coset_factor` in `perm_groups.py` operate on individual elements. Bulk computation of all orbit/transversal data from a BSGS lives in `util.py`.
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
  - `__mul__` (`*` operator) — composes two permutations; pads the shorter array with identity-mapped indices to match the longer one before computing the product.
  - Ranking/unranking of individual permutations in lexicographic order: `rank`, `unrank_lex`, `next_lex`.
  - Non-lex ranking: `rank_nonlex`, `unrank_nonlex`, `rank_trotterjohnson`, `unrank_trotterjohnson`.
  - Properties: `is_even`, `is_odd`, `parity`, `inversions`, `support`, `ascents`, `descents` (positions where consecutive array-form values increase/decrease).
  - `order()` — smallest positive exponent k such that p^k = identity; computed as LCM of disjoint cycle lengths.
  - `cycle_structure` — dict mapping each cycle length to its multiplicity; fixed points (self-mapping elements) counted as length-1 cycles.
  - `__xor__` (`^` operator) — conjugation: `p^q` computes `~q*p*q`; raises `ValueError` if the two permutations have different sizes. When left operand is an int `i`, returns `q(i)` instead.
  - `commutator(x)` — group commutator of two individual permutations (~x·~self·x·self); raises `ValueError` if sizes differ.
  - `commutes_with` — checks if two individual permutations commute (boolean, no search).
  - Distance metrics: `get_precedence_distance`, `get_adjacency_distance`, `get_positional_distance`.
  - `josephus(m, n, s=1)` — classmethod; simulates circular elimination (Josephus problem) where every m-th item is removed from range(n); parameter `s` switches to sequential (step-1) selection when s items remain.
- `_af_new(perm)` — static fast-path factory; constructs a `Permutation` directly from a raw int list, skipping all input validation (no duplicate/range checks). Internal use only.
- Low-level array-form helpers: `_af_rmul`, `_af_parity`, `_af_invert`, `_af_pow`, `_af_commutes_with`.
- `_af_rmuln(*perms)` — composes N permutations in array form (right-to-left); unrolled special cases for ≤8 operands, recursive divide-and-conquer fallback for >8.

### [`perm_groups.py`](perm_groups.py)
Permutation group (set of permutations) with group-theoretic algorithms.
- `PermutationGroup` — the main group class, constructed from generating permutations.
  - **BSGS framework**: `schreier_sims`, `schreier_sims_incremental`, `schreier_vector`.
  - `schreier_sims_incremental` — deterministic BSGS construction: computes Schreier generators, sifts each through the chain via `_strip`/`_strip_af`, and handles failures — extends the base when a non-identity residual survives all levels, or adds a new strong generator at the level where sifting failed.
  - `schreier_sims_random` — randomized BSGS computation: orchestrates a sifting loop that samples random elements, decides when to extend the base sequence (new anchor points), and amends stabilizer chains/orbits when sifting fails. Uses `_strip` from `util.py` as a subroutine.
  - Properties: `base`, `strong_gens`, `basic_orbits`, `basic_transversals`.
  - `basic_stabilizers` — returns the descending chain of point-fixing subgroups from BSGS; partitions strong generators across base levels via `_distribute_gens_by_base` and wraps each level as a `PermutationGroup`.
  - **Coset-based ranking/unranking** (group-level, via Schreier-Sims): `coset_rank`, `coset_unrank`, `coset_factor` — these decompose/rank a single permutation element against the stabilizer chain; they do not compute bulk orbit or transversal data (see `util.py`).
    - `coset_unrank` returns `None` when rank is negative or ≥ group order.
  - **Subgroup search**: `subgroup_search` — depth-first search for all elements satisfying a boolean predicate, with tree-pruning tests and base-change strategy.
  - **Centralizer**: `centralizer` — finds the subgroup of elements commuting with a given permutation, list, or subgroup. Type-dispatches: wraps single permutation or list into a group before searching.
  - Stabilizers: `stabilizer`, `pointwise_stabilizer`.
  - Subgroup/normality: `is_subgroup`, `is_normal`, `normal_closure`, `commutator` (group-level: returns subgroup [G,H]), `derived_subgroup`, `derived_series`.
  - Classification: `is_abelian`, `is_transitive`, `is_primitive`, `is_solvable`, `is_nilpotent`, `is_trivial`.
  - `is_alt_sym(eps)` — one-sided Monte Carlo test for symmetric/alternating group; returns False immediately for degree < 8; samples random elements looking for prime-length cycles via `_check_cycles_alt_sym` in `util.py`.
  - `center` — subgroup of permutation group elements commuting with all group elements; computed via subgroup search.
  - Orbits: `orbit`, `orbits`, `orbit_transversal`, `transitivity_degree`.
  - `orbit_rep(alpha, beta)` — finds a single group element mapping point `alpha` to point `beta` via the Schreier vector; returns `False` if `beta` is not in the orbit of `alpha`.
  - Membership: `contains(g, strict=True)` — tests if permutation belongs to the group; when `strict=False`, resizes `g` to match group degree before testing.
  - Element generation: `generate`, `generate_dimino`, `generate_schreier_sims`, `elements`, `order` (group cardinality), `random`, `random_pr`.
  - `__mul__` — pairwise direct product of exactly two groups; shifts generators to act on disjoint point sets. For N-ary products, use `DirectProduct` in `group_constructs.py`.
  - `max_div` — property; largest proper divisor of the group's degree (degree / smallest prime factor); caches result for degree > 1, returns 1 without caching for degree 1.
  - `minimal_block` — finds block system generated by a set of points for a transitive group.
  - `lower_central_series` — descending series of iterated commutator subgroups.
  - `baseswap` — low-level primitive that swaps two consecutive base points in an existing BSGS.
    - Initializes working generator set T from distributed strong generators at the next-deeper stabilizer level; when swapping the last two base points (no deeper level exists), T starts empty.
    - Deterministic branch iterates transversal elements; randomized branch samples random stabilizer elements.
    - Not used by `get_minimal_bsgs` in `tensor_can.py`.
- Standalone orbit functions (module-level, not `PermutationGroup` methods):
  - `_orbit(degree, generators, alpha)` — computes the set of elements reachable from a starting point under group generators.
  - `_orbits(degree, generators)` — partitions all domain elements into disjoint equivalence classes by iterating single-point orbits.
  - `_orbit_transversal` — finds representative group elements mapping a point to each member of its orbit.

---

## Named Groups and Construction

### [`named_groups.py`](named_groups.py)
Factory functions returning `PermutationGroup` objects for standard finite groups, with pre-set algebraic properties (e.g. `_is_nilpotent`, `_is_solvable`, `_is_abelian`, `_is_transitive`). Contrast with `generators.py`, which yields individual permutation elements.
- `SymmetricGroup(n)` — constructs Sn (full bijection group on n elements); pre-sets `_is_solvable = True` iff n < 5 (reflecting that An is simple for n ≥ 5).
- `CyclicGroup`, `AbelianGroup`, `RubikGroup`.
- `DihedralGroup(n)` — constructs Dn as a `PermutationGroup` from rotation + reflection generators (does not enumerate elements; for full element enumeration see `generators.py::dihedral`). Pre-sets `_is_nilpotent = True` iff n is a power of 2.
- `AlternatingGroup(n)` — constructs An with explicit generators: uses different generators for odd n vs even n (full n-cycle vs (n−1)-cycle fixing 0). Deduplicates generators when both coincide (e.g. n=3), keeping only one.

### [`group_constructs.py`](group_constructs.py)
Composite group construction.
- `DirectProduct(*groups)` — N-ary direct product of arbitrarily many permutation groups in a single pass. Faster than repeated pairwise `__mul__` calls.
  - Shifts each group's generators onto disjoint index slices of a combined identity mapping.
  - Deduplicates embedded generators (via `uniq`) before constructing the result group with `dups=False`.

### [`generators.py`](generators.py)
Yields individual `Permutation` elements (not `PermutationGroup` objects) for standard groups. Contrast with `named_groups.py`, which returns constructed `PermutationGroup` objects with pre-set properties.
- `symmetric(n)`, `cyclic(n)`, `alternating(n)` — yield all permutations of Sn, Cn, An respectively. `alternating` filters by `is_even`.
- `dihedral(n)` — yields all 2n individual `Permutation` elements of Dn.
  - Contains the actual embedding logic for degenerate cases: n=1 yields 2 perms on 2 positions (S2), n=2 yields 4 perms on 4 positions (Klein 4-group in S4).
  - These embeddings are needed because Dn cannot be a subgroup of Sn for n<3.
- `rubik_cube_generators()` — standard 3×3 Rubik's cube face-turn permutations.
- `rubik(n)` — NxN Rubik's cube permutation generator; raises `ValueError` for n < 2.
  - Represents each face as an n×n numbered matrix; simulates single-face rotations, slice-plane moves, and whole-cube rotations.
  - Records each resulting state as a `Permutation`.
  - Uses a dual-purpose `perm(show)` inner function: `show=0` appends the current state as a `Permutation` to the group list; `show=1` returns the flat state list for identity verification.
  - After each set of slice rotations (front, right, bottom), asserts `perm(1) == I` to verify the cube is restored to its original configuration.

---

## Finitely Presented and Free Groups

### [`fp_groups.py`](fp_groups.py)
Finitely presented groups, coset enumeration, and subgroup presentations.
- `FpGroup` — finitely presented group defined by generators and relators.
- `CosetTable` — tabular structure for coset enumeration; tracks coset–generator mappings.
  - `scan(alpha, word)` — traces a relator word through the table from coset α; calls the `coincidence` routine when a completed scan is inconsistent.
  - `scan_check(alpha, word)` — boolean-returning variant of `scan`; returns False on inconsistent completion instead of calling `coincidence`. Used in `low_index_subgroups`.
  - `coincidence(alpha, beta)` / `coincidence_f` — merges two cosets found to be equivalent.
- Coset enumeration: `coset_enumeration_r`, `coset_enumeration_c`.
- `low_index_subgroups`, `reidemeister_presentation`.
- `_simplification_technique_1(rels)` — reduces multi-syllable relators using single-generator relators of the form g^n; bounds exponents to (-n/2, n/2] via modular arithmetic and deletes syllables whose exponent is an exact multiple of n.
- Subgroup presentation pipeline:
  - `define_schreier_generators(C)` — builds Schreier generators for a subgroup from a coset table.
  - `reidemeister_relators(C)` — computes defining relators for the subgroup; simplifies by eliminating trivial (order-1) generators and removing cyclic-conjugate duplicates (Tietze transformation TT_1).
  - `rewrite(C, α, w)` — rewrites a word in the original generators into the Schreier generator set for a given coset.

### [`free_group.py`](free_group.py)
Free groups with symbolic generators.
- Constructor entry points: `free_group(symbols)`, `xfree_group`, `vfree_group` — create a `FreeGroup` from a string, Symbol/Expr, or sequence thereof.
  - `_parse_symbols` — normalizes the `symbols` argument; accepts str, Expr, sequence of str, or sequence of Expr. Mixed-type sequences (e.g. str and Symbol together) raise `ValueError`.
- `FreeGroup` — finitely generated free group (generators only, no defining relations); generators are ordered by creation order.
  - `is_abelian` — True only when the group has rank 0 or 1 (at most one generator); rank ≥ 2 free groups are always non-abelian.
  - `center` — returns the center of the free group (always `{identity}`, since free groups of rank ≥ 2 are non-abelian).
  - `contains(g)` — tests word membership; rejects elements not of `FreeGroupElement` type, and also rejects words built from a different `FreeGroup` instance (compares group identity, not generator names).
  - `is_subgroup`, `identity`.
- `FreeGroupElement` — word (element) in a free group, stored as tuple of (generator, exponent) pairs.
  - Comparison: `__lt__` implements short-lex total ordering — shorter words first, then lexicographic by generator index; each inverse is ordered between its positive generator and the next smaller generator.
  - `is_cyclic_conjugate` — checks if two words are cyclic conjugates (rotational rearrangements) after cyclic reduction; uses string-doubling rotation detection.
  - `identity_cyclic_reduction` — returns the unique cyclically reduced form of a word: combines exponents of first and last syllables, stripping them if they cancel completely.
  - `is_dependent(word)` / `is_independent(word)` — checks whether one word appears as a substring of another (or its inverse) in letter form; `is_independent` is the boolean negation.
  - Word operations: `cyclic_reduction`, `number_syllables`, `sub_syllables`, `substituted_word`, `eliminate_word`, `letter_form`.
  - `eliminate_word(gen, by)` — replaces every occurrence of a generator in a word with another associative word; expands powers by repeated concatenation.

---

## Combinatorial Objects

### [`partitions.py`](partitions.py)
Set and integer partitions.
- `Partition` — set partition (disjoint-set decomposition) with RGS (restricted growth string) representation.
  - `__new__` validates inputs: raises `ValueError` if any element appears in more than one subset (checks after flattening all subsets).
  - `rank` — property; lazily computes and caches the ordinal position of the partition by converting to RGS and calling `RGS_rank`.
  - `__add__`/`__sub__` — integer arithmetic on partitions: advances/retreats rank modularly (rank % total RGS count), wrapping cyclically past boundaries.
  - `from_rgs(rgs, elements)` — reconstructs a set partition from a sequence of block indices paired with items; validates that no block is left empty.
  - `RGS` — property returning the restricted growth string encoding which block each element belongs to.
- `IntegerPartition` — partition of an integer.
- `RGS_enum(m)` — total count of restricted growth strings for superset size m.
- `RGS_unrank(rank, m)` — converts a rank to a restricted growth string; raises `ValueError` if rank < 0 or rank ≥ `RGS_enum(m)`.
- `RGS_rank`, `RGS_generalized`, `random_integer_partition`.

### [`graycode.py`](graycode.py)
Gray code (Hamiltonian walk on an n-dimensional binary hypercube) representation and bit-level operations (rank/unrank, conversion).
- `GrayCode` — n-bit Gray code object; generates all codes, supports `rank`, `unrank` (returns a bitstring, not a subset), `next`, `current`.
  - Constructor accepts optional `start` (binary string) or `rank` (integer starting position); rank wraps via modulo (rank % 2^n) when it exceeds total selections.
- `gray_to_bin`, `bin_to_gray` — convert between Gray and binary encodings.
- `get_subset_from_bitstring(super_set, bitstring)` — selects elements from a collection using a binary string mask; raises `ValueError` if lengths differ.
- `graycode_subsets(gray_code_set)` — yields all subsets of a collection in Gray code order via `get_subset_from_bitstring`.
- **Note**: For traversing *subsets* in Gray code order, see `subsets.py::Subset.iterate_graycode`.

### [`subsets.py`](subsets.py)
Subset generation and manipulation via binary, lexicographic, and Gray code enumeration.
- `Subset` — subset of a superset with ranking/unranking in binary, lexicographic, and Gray code orders.
  - Ranking properties: `rank_binary`, `rank_lex`, `rank_gray` — compute the position of a subset relative to its parent set in each ordering.
  - Unranking class methods: `unrank_binary(rank, superset)`, `unrank_gray(rank, superset)` — retrieve the subset at a given ordinal position in binary or Gray code order; `unrank_gray` delegates to `GrayCode.unrank` then maps the bitlist to subset elements.
  - Gray code traversal: `iterate_graycode`, `next_gray`, `prev_gray` — step through subsets in reflected binary code order with modular wraparound.
  - `subset_from_bitlist` — constructs a `Subset` object from a superset and bitlist (returns object, not raw list; contrast `graycode.py::get_subset_from_bitstring`).
- `ksubsets` — k-element subsets of a set.

### [`prufer.py`](prufer.py)
Prufer sequence correspondence for labeled trees.
- `Prufer` — bijection between labeled trees (edge lists) and Prufer codes (length n−2 integer sequences).
  - Constructor accepts an edge list or a Prufer sequence. When given edges, validates vertex labels are contiguous 0..max; raises `ValueError` listing absent vertices if gaps exist. An optional second arg `n` bypasses validation.
  - `to_prufer(tree, n)` — static; converts edge list to Prufer sequence by iteratively finding the smallest leaf node, recording its neighbor, and removing the edge.
  - `to_tree(prufer)` — static; reconstructs edge list from a Prufer sequence.
  - `prufer_repr` / `tree_repr` — lazy properties returning the Prufer sequence or edge list.
  - `prufer_rank` / `unrank(rank, n)` — ranking and unranking of Prufer sequences.
  - `edges(*runs)` — static; builds deduplicated edges from sequences of node-index runs; validates all node labels in min..max range are present and raises `ValueError` listing missing nodes if gaps exist; normalizes labels to start from 0.
  - `next(delta)`, `prev(delta)` — navigate to adjacent Prufer sequences by rank.

### [`polyhedron.py`](polyhedron.py)
Polyhedral symmetry groups (tetrahedron, cube/octahedron, dodecahedron/icosahedron).
- `Polyhedron` — 3D solid defined by named corners, faces, and a permutation group (`pgroup`).
  - `__new__` docstring contains tutorial on constructing permutation groups from geometric intuition.
    - Camera-based method: uses fixed-position cameras at each vertex to derive index-based permutations from spatial rotations.
    - Shows that two generators (diagonal flip + central rotation) suffice for all orientations of a polygon.
  - `rotate(perm)` — apply a permutation to vertices in place. Accepts `Permutation` or int index into `pgroup`. Validates permutation size matches vertex count; raises `ValueError` on mismatch.
  - Properties: `corners`, `faces`, `edges`, `pgroup`, `size`, `array_form`, `cyclic_form`.
- `_pgroup_calcs()` — constructs all five Platonic solids (tetrahedron, cube, octahedron, dodecahedron, icosahedron) with face definitions and rotation groups.
  - Rotation groups are **not** algebraic groups in the strict sense; they are curated so powers of individual elements alone generate all valid orientations (no mixed products like a*b needed).
  - Dual solids (octahedron from cube, icosahedron from dodecahedron) derive their rotation groups from the enclosing solid via `_pgroup_of_double`.
  - Dodecahedron rotations use `_string_to_perm` to decode compact shorthand strings into composed permutations.

---

## Tensor and Canonicalization

### [`tensor_can.py`](tensor_can.py)
Tensor canonicalization using double-coset representatives.
- `canonicalize`, `double_coset_can_rep`, `canonical_free`.
- `transversal2coset(size, base, transversal)` — converts BSGS transversals to coset representation; fills identity for positions not in base, then trims trailing identity entries so the returned list may be shorter than `size`.
- `get_symmetric_group_sgs(n, antisym)` — returns minimal BSGS for a rank-n (anti)symmetric tensor; encodes sign via trailing positions: symmetric appends `[n, n+1]`, antisymmetric appends `[n+1, n]`.
- `perm_af_direct_product` — direct product of two generator lists in array form; embeds each set into the combined index space.
- `bsgs_direct_product` — direct product of two BSGS structures; merges bases and generators, filters out identity permutations, and falls back to a single identity generator if all combined generators are trivial.
- `tensor_gens`, `dummy_sgs`.
- `get_minimal_bsgs(base, gens)` — attempts to compute a lexicographically minimal BSGS via `schreier_sims_incremental`; returns `None` if the result is not minimal (does not fall back to baseswap or other strategies on failure).
- `_is_minimal_bsgs` — verifies a BSGS has the lexicographically smallest base by reconstructing the base from generators and comparing to the given one.
- `get_transversals` — returns transversals for a group given its BSGS.

---

## Utilities

### [`util.py`](util.py)
Low-level algorithms for computational group theory.
- `_handle_precomputed_bsgs` — lazily fills missing BSGS structures (transversals, basic orbits, distributed strong gens) from whichever are already available; derives orbits from transversal keys when transversals are known but orbits are not.
- `_distribute_gens_by_base(base, gens)` — partitions generators into basic stabilizer levels; each level i collects gens fixing the first i base points; empty levels receive the identity element.
- `_base_ordering` — reorders `{0..n-1}` so that base points appear first; produces an index mapping, does not verify or compute minimal bases.
- `_orbits_transversals_from_bsgs` — computes basic orbits and transversal dicts from distributed strong generators; `transversals_only=True` skips orbit lists and returns only the coset-representative mappings.
- `_strip` — decomposes (sifts) one permutation element through the stabilizer chain of an existing BSGS; returns residual and level. Does not modify the BSGS (caller decides how to react to failure).
- `_strip_af` — array-form optimized variant of `_strip`; skips levels already known to be fixed (parameter `j`), and detects identity early when the residual equals a coset representative mid-chain, returning `(False, base_len + 1)` instead of computing further products.
- `_remove_gens(base, strong_gens)` — prunes redundant generators from a strong generating set; iterates stabilizer levels in reverse, skipping removal when it would leave zero generators at a level.
- `_strong_gens_from_distr` — reconstructs strong generating set from distributed generators.
- `_check_cycles_alt_sym(perm)` — checks if a permutation contains a cycle of prime length p with n/2 < p < n-2; early-terminates when remaining unvisited elements can't form a long enough cycle. Helper for `is_alt_sym` in `perm_groups.py`.

### [`testutil.py`](testutil.py)
Testing/verification helpers for permutation groups, plus brute-force tensor canonicalization.
- `_cmp_perm_lists`, `_naive_list_centralizer` — test comparison helpers.
- `_verify_bsgs`, `_verify_centralizer`, `_verify_normal_closure` — correctness checks for BSGS, centralizers, and normal closures.
- `canonicalize_naive(g, dummies, sym, *v)` — brute-force tensor canonicalization; reference implementation for `tensor_can.canonicalize`.
  - Generates full orbit under symmetry and dummy-index groups, sorts it, detects zero tensors via sign contradictions in consecutive orbit elements.
- `graph_certificate(gr)` — computes a canonical certificate for an unoriented graph by encoding it as a tensor problem and calling `tensor_can.canonicalize`.
