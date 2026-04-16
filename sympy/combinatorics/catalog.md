# sympy/combinatorics -- Catalog

> Part of [SymPy](../catalog.md). Combinatorics: permutations, partitions, polyhedra, group theory, and Graycode.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package initializer; imports and re-exports the main public classes and functions (Permutation, Prufer, Subset, Partition, Polyhedron, PermutationGroup, GrayCode, named groups, etc.). |
| `fp_groups.py` | Defines `FpGroup` (finitely presented groups), `CosetTable` class, and Todd-Coxeter coset enumeration algorithms (`coset_enumeration_r` relator-based, `coset_enumeration_c` coset-table-based), low-index subgroup enumeration, Reidemeister-Schreier presentations, and permutation group isomorphism. |
| `free_group.py` | Implements `FreeGroup` and `FreeGroupElement` for constructing and manipulating free groups and their elements. Includes `_parse_symbols` for parsing and validating generator inputs (strings, Expr, sequences, raising on mixed-type sequences). Also provides word-level arithmetic, element equality, and short-lex total ordering of elements. |
| `generators.py` | Generator functions that **yield every individual permutation element** (not group objects) of standard groups: `symmetric`, `cyclic`, `alternating`, `dihedral` (with special-case embeddings for small n=1,2), and Rubik's cube generators. Returns iterables of `Permutation` instances, not `PermutationGroup`. |
| `graycode.py` | Implements the `GrayCode` class for generating and manipulating Gray codes (reflected binary codes) over n-dimensional binary cubes. |
| `group_constructs.py` | Provides the `DirectProduct` function for computing the direct product of **three or more** permutation groups in a single optimized call (shifts all generator index sets at once). For the product of exactly **two** groups, use `PermutationGroup.__mul__` in `perm_groups.py` instead. |
| `homomorphisms.py` | Implements `GroupHomomorphism` and a `homomorphism()` factory for creating and working with group homomorphisms between permutation groups and finitely presented groups. |
| `named_groups.py` | Factory functions that **construct `PermutationGroup` objects** (not element iterators) for standard named groups: `SymmetricGroup`, `CyclicGroup`, `DihedralGroup`, `AlternatingGroup` (with odd/even-n generator selection), `AbelianGroup`, and `RubikGroup`. Each function chooses minimal generating sets and pre-caches group properties (abelian, solvable, transitive, etc.). |
| `partitions.py` | Defines `Partition` (set partitions) and `IntegerPartition` classes, along with RGS (restricted growth string) ranking/unranking utilities. |
| `perm_groups.py` | Core `PermutationGroup` class implementing group-theoretic operations: `__mul__` computes the direct product of exactly **two** groups by shifting the second group's indices (for 3+ groups use `group_constructs.py`), order, orbits, stabilizers, Schreier-Sims algorithm, coset decomposition with ranking/unranking (`coset_rank`, `coset_unrank`), subgroup testing, depth-first subgroup search with property predicates (`subgroup_search`), centralizer computation, derived series and derived subgroups, commutator of subgroups, and normal closure. |
| `permutations.py` | Core `Permutation` and `Cycle` classes for representing and manipulating **individual** permutations in array and cyclic notation. `__mul__` composes two permutations, padding the shorter array to reconcile different domain sizes. Also supports inversion, parity, lexicographic ranking (`rank`), non-lexicographic linear-time ranking/unranking (`rank_nonlex`, `unrank_nonlex`), and low-level array-form helpers (`_af_rmul`, `_af_commutes_with`, etc.). |
| `polyhedron.py` | Implements the `Polyhedron` class for representing polyhedral symmetry groups, with pre-built instances for the five Platonic solids (tetrahedron, cube, octahedron, dodecahedron, icosahedron). |
| `prufer.py` | Implements the `Prufer` class for the bijection between labeled trees and Prufer sequences, with ranking/unranking and tree-edge conversion methods. |
| `rewritingsystem.py` | Implements `RewritingSystem` for Knuth-Bendix completion on finitely presented groups, used for word reduction and confluence checking. |
| `subsets.py` | Implements the `Subset` class for enumerating subsets of a set via binary, lexicographic, and Gray code orderings with ranking/unranking support. |
| `tensor_can.py` | Algorithms for tensor canonicalization using permutation group double-coset representatives, including dummy-index symmetry handling. |
| `testutil.py` | Testing utilities for the combinatorics module: naive centralizer computation, BSGS verification, normal closure checks, and graph certificate comparison. |
| `util.py` | Internal utility functions for computational group theory: base ordering, cycle checks for alternating/symmetric detection, BSGS-related strip and distribution routines, lazy computation of missing BSGS structures (transversals, basic orbits, distributed strong generators) from precomputed data (`_handle_precomputed_bsgs`), and orbit/transversal calculation from base and strong generating sets. |
