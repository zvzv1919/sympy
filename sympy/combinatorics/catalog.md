# sympy/combinatorics — Combinatorics & Group Theory

## Glossary

- **Array form**: a permutation represented as a list where position `i` holds `p(i)`.
- **Cyclic form**: a permutation represented as a list of disjoint cycles.
- **BSGS**: Base and Strong Generating Set — a compact representation of a permutation group enabling efficient algorithms (Schreier-Sims).
- **RGS**: Restricted Growth String — an encoding of set partitions.
- **Coset table**: a data structure used in Todd-Coxeter coset enumeration for finitely presented groups.

---

## Permutations

### permutations.py

Core permutation infrastructure: the `Permutation` and `Cycle` classes and low-level array-form helpers.

- **`Cycle(dict)`** — Wrapper around dict representing a disjoint cycle. Supports composing cycles via `__call__`, converting to explicit list form, and pretty-printing in cycle notation.
- **`Permutation(Basic)`** — The central permutation class. Constructed from array form, cyclic form, or `Cycle` objects.
  - Supports both array-form and cyclic-form representations (controlled by `print_cyclic` class flag).
  - Arithmetic: `*` (composition), `**` (power), `~` (inverse), `^` (conjugation / element application).
  - Ranking/unranking: `rank()`, `unrank_lex()`, `next_lex()`, lexicographic and Trotter-Johnson orderings, non-lex ranking.
  - Properties: `parity`, `is_even`, `is_odd`, `is_Identity`, `order`, `cycles`, `cycle_structure`, `signature`, `length`, `cardinality`.
  - Combinatorial statistics: `ascents`, `descents`, `inversions`, `inversion_vector`, `runs`, `index`.
  - Distance metrics: `get_precedence_distance`, `get_adjacency_distance`, `get_positional_distance` (with corresponding matrix methods).
  - Utilities: `transpositions`, `commutator`, `support`, `from_sequence`, `from_inversion_vector`, `josephus`, `random`.
- **Array-form helpers** (module-level):
  - `_af_rmul(a, b)` / `_af_rmuln(*abc)` — fast right-multiplication in array form.
  - `_af_parity(pi)` — parity via cycle decomposition.
  - `_af_invert(a)` — inverse in array form.
  - `_af_pow(a, n)` — exponentiation with binary method.
  - `_af_commutes_with(a, b)` — commutativity check.
- `_merge(arr, temp, left, mid, right)` — merge-sort helper for inversion counting.

**Caveats**: `p * q` means "apply p then q" (i.e., `(p*q)(i) = q(p(i))`), which is the *opposite* of the mathematical function-composition convention. The `^` operator is overloaded: `i^p` applies `p` to integer `i`, while `p^q` computes the conjugate `~q*p*q`.

### perm_groups.py

`PermutationGroup` — the main permutation group class (~3300 lines). Implements the Schreier-Sims algorithm and many group-theoretic operations.

- **Construction & basics**: `__new__`, `generators`, `degree`, `order()`, `elements`, `is_trivial`, `__contains__`.
- **BSGS machinery**:
  - `schreier_sims()` / `schreier_sims_incremental()` / `schreier_sims_random()` — compute base & strong generating set.
  - `base`, `strong_gens`, `basic_orbits`, `basic_stabilizers`, `basic_transversals`.
  - `baseswap(...)` — swap adjacent base points.
- **Orbits & stabilizers**:
  - `orbit(alpha)`, `orbits()`, `orbit_rep(alpha, beta)`, `orbit_transversal(alpha)`.
  - `stabilizer(alpha)`, `pointwise_stabilizer(points)`.
  - `schreier_vector(alpha)`.
- **Coset operations**: `coset_factor(g)`, `coset_rank(g)`, `coset_unrank(rank)`.
- **Group properties**: `is_abelian`, `is_transitive`, `is_primitive`, `is_nilpotent`, `is_solvable`, `is_alt_sym`.
- **Subgroup structure**:
  - `center()`, `centralizer(other)`, `commutator(G, H)`, `derived_subgroup()`, `derived_series()`, `lower_central_series()`.
  - `normal_closure(other)`, `is_normal(gr)`, `is_subgroup(G)`.
  - `subgroup_search(prop)` — backtrack search for subgroups satisfying a property.
- **Generation**: `generate(method)`, `generate_dimino()`, `generate_schreier_sims()`.
- **Miscellaneous**: `minimal_block(points)`, `max_div`, `transitivity_degree`, `make_perm(n)`, `random()`, `random_pr()`.
- **Module-level helpers**: `_orbit()`, `_orbits()`, `_orbit_transversal()`, `_stabilizer()`.

---

## Groups — Constructs & Named Groups

### named_groups.py

Factory functions that return `PermutationGroup` instances for well-known groups, with pre-set properties.

- `SymmetricGroup(n)` — S_n, generators: n-cycle + transposition (0 1).
- `AlternatingGroup(n)` — A_n, generators: (0 1 2) + an n-cycle or (n-1)-cycle.
- `CyclicGroup(n)` — C_n, generator: the n-cycle.
- `DihedralGroup(n)` — D_n (order 2n), generators: n-cycle + reversal.
- `AbelianGroup(*cyclic_orders)` — direct product of cyclic groups.
- `RubikGroup(n)` — permutation group for an n×n Rubik's cube.

### group_constructs.py

- `DirectProduct(*groups)` — direct product of an arbitrary number of `PermutationGroup`s. Faster than chaining `__mul__` because generators are shifted in a single pass.

### generators.py

Generator functions that yield `Permutation` objects for classical groups and Rubik's cube.

- `symmetric(n)` / `cyclic(n)` / `alternating(n)` / `dihedral(n)` — yield all elements of S_n, C_n, A_n, D_n respectively.
- `rubik_cube_generators()` — 6 generators for the standard 3×3 Rubik's cube (48 facelets).
- `rubik(n)` — generators for an n×n Rubik's cube (returns `3*(n-1)` permutations for front/right/bottom slices).

---

## Free Groups & Finitely Presented Groups

### free_group.py

Free groups and their elements (associative words).

- **Constructors**: `free_group(symbols)`, `xfree_group(symbols)`, `vfree_group(symbols)` — return `(FreeGroup, generators)` in slightly different unpacking styles.
- **`FreeGroup(DefaultPrinting)`** — cached free group object.
  - `generators`, `rank`, `order()`, `identity`, `is_abelian`, `center()`, `contains(g)`.
- **`FreeGroupElement(CantSympify, DefaultPrinting, tuple)`** — an associative word.
  - Representations: `array_form` (tuples of (symbol, exponent)), `letter_form`, `ext_rep`.
  - Arithmetic: `*`, `**`, `/`, `inverse()`.
  - Word operations: `eliminate_word(gen, by)`, `subword`, `substituted_word`, `cyclic_subword`.
  - Syllable access: `number_syllables`, `exponent_syllable(i)`, `generator_syllable(i)`, `sub_syllables`.
  - Analysis: `exponent_sum(gen)`, `generator_count(gen)`, `contains_generators()`, `is_dependent(word)`.
  - Cyclic operations: `cyclic_conjugates()`, `is_cyclic_conjugate(w)`, `is_cyclically_reduced()`, `identity_cyclic_reduction()`.
  - Ordering: short-lex (`<`, `>`, `<=`, `>=`).
- `letter_form_to_array_form(array_form, group)` — convert letter-form list to array-form tuples.
- `zero_mul_simp(l, index)` — in-place simplification of adjacent inverse syllables.

### fp_groups.py

Finitely presented groups, coset enumeration (Todd-Coxeter), low-index subgroups, and Reidemeister-Schreier subgroup presentations.

- **`FpGroup(DefaultPrinting)`** — finitely presented group `<X | R>`. Returns the underlying `FreeGroup` if no relators given.
- **`CosetTable(DefaultPrinting)`** — the coset table data structure.
  - Core operations: `define`, `scan`, `scan_and_fill`, `coincidence`, `merge`, `rep` (union-find representative).
  - Variants with deduction stack: `define_f`, `scan_f`, `scan_and_fill_f`, `coincidence_f`.
  - `scan_check` — scanning without coincidence (returns bool), used in low-index algorithm.
  - Bookkeeping: `compress()`, `standardize()`, `switch(beta, gamma)`, `is_complete()`, `omega`, `n`.
  - `look_ahead()`, `process_deductions(...)`, `process_deductions_check(...)`.
  - `conjugates(R)` — partition cyclic conjugates of relators by leading generator.
- **Coset enumeration**:
  - `coset_enumeration_r(fp_grp, Y)` — relator-based (HLT) method.
  - `coset_enumeration_c(fp_grp, Y)` — coset-table-based (Felsch) method.
- **Low-index subgroups**:
  - `low_index_subgroups(G, N, Y=[])` — find all subgroups up to index N.
  - `descendant_subgroups(...)`, `try_descendant(...)`, `first_in_class(C)` — backtrack helpers.
- **Subgroup presentations (Reidemeister-Schreier)**:
  - `define_schreier_generators(C)` — compute Schreier generators from a coset table.
  - `reidemeister_relators(C)` — compute relators for the subgroup presentation.
  - `rewrite(C, alpha, w)` — rewrite a word as a product of Schreier generators.
  - `elimination_technique_1(C)` / `elimination_technique_2(C)` — Tietze transformations to simplify presentations.
  - `simplify_presentation(C)` / `_simplification_technique_1(rels)` — bound exponents using single-syllable relators.
  - `reidemeister_presentation(fp_grp, H)` — full pipeline: enumerate cosets → Schreier generators → relators → simplify.

---

## Combinatorial Objects

### partitions.py

Set partitions and integer partitions.

- **`Partition(FiniteSet)`** — a set partition (disjoint union of subsets).
  - `partition`, `rank`, `RGS`, `from_rgs(rgs, elements)`, `sort_key`.
  - Arithmetic: `+` / `-` to step through partitions by rank.
- **`IntegerPartition(Basic)`** — a partition of a positive integer.
  - `partition`, `integer`, `conjugate`, `as_dict()`, `as_ferrers(char)`.
  - Traversal: `next_lex()`, `prev_lex()`.
- `random_integer_partition(n, seed=None)`
- **RGS utilities**: `RGS_generalized(m)`, `RGS_enum(m)`, `RGS_unrank(rank, m)`, `RGS_rank(rgs)`.

### subsets.py

Subset enumeration with multiple orderings.

- **`Subset(Basic)`** — a subset of a given superset.
  - Binary ordering: `next_binary()`, `prev_binary()`, `iterate_binary(k)`, `rank_binary`, `unrank_binary`.
  - Lexicographic ordering: `next_lexicographic()`, `prev_lexicographic()`, `rank_lexicographic`.
  - Gray code ordering: `next_gray()`, `prev_gray()`, `iterate_graycode(k)`, `rank_gray`, `unrank_gray`.
  - Conversions: `subset_from_bitlist(superset, bitlist)`, `bitlist_from_subset(subset, superset)`, `subset_indices`.
  - Properties: `subset`, `superset`, `size`, `superset_size`, `cardinality`.
- `ksubsets(superset, k)` — generate all k-element subsets (wraps `itertools.combinations`).

### graycode.py

Gray code generation and utilities.

- **`GrayCode(Basic)`** — n-dimensional binary reflected Gray code.
  - `generate_gray(**hints)` — yields all 2^n bit strings in Gray code order.
  - `skip()` — skip the next generated code (for early-exit iteration patterns).
  - `rank`, `unrank(n, rank)`, `current`, `selections`, `n`, `next(delta)`.
- `random_bitstring(n)`
- `gray_to_bin(bin_list)` / `bin_to_gray(bin_list)` — conversions between Gray and binary encoding.
- `get_subset_from_bitstring(super_set, bitstring)` — extract subset given a bitstring mask.
- `graycode_subsets(gray_code_set)` — yield all subsets in Gray code order.

### prufer.py

Prüfer sequences and labeled trees.

- **`Prufer(Basic)`** — bijection between labeled trees and Prüfer codes.
  - `prufer_repr` / `tree_repr` — lazy-computed Prüfer sequence and edge-list representations.
  - `nodes`, `rank`, `size`.
  - Static: `to_prufer(tree, n)`, `to_tree(prufer)`, `edges(*runs)`.
  - Navigation: `next(delta)`, `prev(delta)`, `unrank(rank, n)`, `prufer_rank()`.

---

## Polyhedra

### polyhedron.py

Polyhedral symmetry groups and the five Platonic solids.

- **`Polyhedron(Basic)`** — a decorated `PermutationGroup` representing a polyhedron.
  - `corners` / `vertices`, `faces`, `edges`, `pgroup`, `size`.
  - `array_form`, `cyclic_form` — track current orientation.
  - `rotate(perm)` — apply a permutation in-place.
  - `reset()` — restore to original orientation.
- **Pre-built solids** (module-level constants):
  - `tetrahedron`, `cube`, `octahedron`, `dodecahedron`, `icosahedron`.
  - Corresponding `*_faces` tuples.
- `_pgroup_calcs()` — internal function that computes all pgroups and face definitions. The octahedron and icosahedron pgroups are derived as duals of the cube and dodecahedron respectively.

---

## Tensor Canonicalization

### tensor_can.py

Butler-Portugal algorithm for canonicalizing tensors with slot and dummy index symmetries.

- **`canonicalize(g, dummies, msym, *v)`** — main entry point. Takes a permutation representing a tensor, dummy index information, metric symmetries, and tensor type specifications (BSGS, count, commutation symmetry). Returns 0 (tensor is zero) or canonical array form.
- **`double_coset_can_rep(dummies, sym, b_S, sgens, S_transversals, g)`** — core algorithm: finds the canonical representative of the double coset D*g*S using stabilizer chains.
- **`canonical_free(base, gens, g, num_free)`** — canonicalize with respect to free indices only (lexicographic minimum).
- **BSGS utilities**:
  - `get_symmetric_group_sgs(n, antisym=False)` — minimal BSGS for (anti)symmetric tensors.
  - `bsgs_direct_product(base1, gens1, base2, gens2)` — BSGS of a direct product.
  - `get_transversals(base, gens)`, `get_minimal_bsgs(base, gens)`, `_is_minimal_bsgs(base, gens)`.
  - `tensor_gens(base, gens, list_free_indices, sym)` — BSGS for n tensors of the same type.
  - `gens_products(*v)` — BSGS for tensors of different types.
- `dummy_sgs(dummies, sym, n)` — strong generators for dummy index symmetry.
- `perm_af_direct_product(gens1, gens2)` — direct product of generator lists in array form.
- `riemann_bsgs` — pre-computed BSGS for the Riemann tensor.
- Internal helpers: `_min_dummies`, `_trace_S`, `_trace_D`, `_dumx_remove`, `transversal2coset`, `_get_map_slots`, `_lift_sgens`.

---

## Utilities

### util.py

Low-level BSGS and group-theory helpers used by `perm_groups.py` and other modules.

- `_base_ordering(base, degree)` — order points so base points come first.
- `_check_cycles_alt_sym(perm)` — check for prime-length cycles (used in `is_alt_sym`).
- `_distribute_gens_by_base(base, gens)` — distribute generators into basic stabilizer levels.
- `_handle_precomputed_bsgs(base, strong_gens, ...)` — fill in missing BSGS structures.
- `_orbits_transversals_from_bsgs(base, strong_gens_distr)` — compute basic orbits and transversals.
- `_remove_gens(base, strong_gens, ...)` — remove redundant strong generators.
- `_strip(g, base, orbits, transversals)` — sift a permutation through a BSGS (Schreier-Sims core).
- `_strip_af(h, base, orbits, transversals, j)` — optimized `_strip` in array form.
- `_strong_gens_from_distr(strong_gens_distr)` — reconstruct strong generating set from distributed form.

### testutil.py

Testing utilities for verifying group-theoretic computations.

- `_cmp_perm_lists(first, second)` — compare two permutation lists as sets.
- `_naive_list_centralizer(self, other)` — brute-force centralizer computation.
- `_verify_bsgs(group, base, gens)` — verify a BSGS is correct.
- `_verify_centralizer(group, arg, centr)` — verify centralizer against naive computation.
- `_verify_normal_closure(group, arg, closure)` — verify normal closure.
- `canonicalize_naive(g, dummies, sym, *v)` — brute-force tensor canonicalization (reference implementation).
- `graph_certificate(gr)` — compute a canonical certificate for an undirected graph via tensor canonicalization.

---

## Package Init

### \_\_init\_\_.py

Re-exports the public API: `Permutation`, `Cycle`, `Prufer`, `Subset`, `Partition`, `IntegerPartition`, `RGS_rank`, `RGS_unrank`, `RGS_enum`, `Polyhedron` (+ five solids), `PermutationGroup`, `DirectProduct`, `GrayCode`, and named group constructors (`SymmetricGroup`, `DihedralGroup`, `CyclicGroup`, `AlternatingGroup`, `AbelianGroup`, `RubikGroup`). Also re-exports generator functions `cyclic`, `alternating`, `symmetric`, `dihedral`.
