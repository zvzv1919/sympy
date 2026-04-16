# sympy/combinatorics -- Catalog

> Part of [SymPy](../catalog.md). Combinatorics: permutations, partitions, polyhedra, group theory, and Graycode.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package initializer; imports and re-exports the main public classes and functions (Permutation, Prufer, Subset, Partition, Polyhedron, PermutationGroup, GrayCode, named groups, etc.). |
| `fp_groups.py` | Defines `FpGroup` (finitely presented groups), `CosetTable` class, and Todd-Coxeter coset enumeration algorithms (`coset_enumeration_r` relator-based, `coset_enumeration_c` coset-table-based), low-index subgroup enumeration, Reidemeister-Schreier presentations, and permutation group isomorphism. |
| `free_group.py` | Implements `FreeGroup` and `FreeGroupElement` for constructing and manipulating free groups and their elements, including word-level arithmetic, element equality, and short-lex total ordering of elements (comparison by length then lexicographic order of generators with inverse-generator handling). |
| `generators.py` | Provides generator functions that yield all permutation elements of standard groups: `symmetric`, `cyclic`, `alternating`, `dihedral`, and Rubik's cube generators. |
| `graycode.py` | Implements the `GrayCode` class for generating and manipulating Gray codes (reflected binary codes) over n-dimensional binary cubes. |
| `group_constructs.py` | Provides the `DirectProduct` function for computing the direct product of multiple (more than two) permutation groups by repeatedly calling `PermutationGroup.__mul__`. |
| `homomorphisms.py` | Implements `GroupHomomorphism` and a `homomorphism()` factory for creating and working with group homomorphisms between permutation groups and finitely presented groups. |
| `named_groups.py` | Factory functions for constructing standard named permutation groups: `SymmetricGroup`, `CyclicGroup`, `DihedralGroup`, `AlternatingGroup`, `AbelianGroup`, and `RubikGroup`. |
| `partitions.py` | Defines `Partition` (set partitions) and `IntegerPartition` classes, along with RGS (restricted growth string) ranking/unranking utilities. |
| `perm_groups.py` | Core `PermutationGroup` class implementing group-theoretic operations: direct product of two groups via `__mul__` (extends generators to combined index space), order, orbits, stabilizers, Schreier-Sims algorithm, coset decomposition with ranking/unranking (`coset_rank`, `coset_unrank`), subgroup testing, depth-first subgroup search with property predicates (`subgroup_search`), centralizer computation, derived series and derived subgroups, commutator of subgroups, and normal closure. |
| `permutations.py` | Core `Permutation` and `Cycle` classes for representing and manipulating individual permutations in array and cyclic notation, with support for composition, inversion, parity, lexicographic ranking (`rank`), and non-lexicographic linear-time ranking/unranking (`rank_nonlex`, `unrank_nonlex` — uses inverse permutation in a recursive algorithm). Also includes low-level array-form helpers (`_af_rmul`, `_af_commutes_with`, etc.). |
| `polyhedron.py` | Implements the `Polyhedron` class for representing polyhedral symmetry groups, with pre-built instances for the five Platonic solids (tetrahedron, cube, octahedron, dodecahedron, icosahedron). |
| `prufer.py` | Implements the `Prufer` class for the bijection between labeled trees and Prufer sequences, with ranking/unranking and tree-edge conversion methods. |
| `rewritingsystem.py` | Implements `RewritingSystem` for Knuth-Bendix completion on finitely presented groups, used for word reduction and confluence checking. |
| `subsets.py` | Implements the `Subset` class for enumerating subsets of a set via binary, lexicographic, and Gray code orderings with ranking/unranking support. |
| `tensor_can.py` | Algorithms for tensor canonicalization using permutation group double-coset representatives, including dummy-index symmetry handling. |
| `testutil.py` | Testing utilities for the combinatorics module: naive centralizer computation, BSGS verification, normal closure checks, and graph certificate comparison. |
| `util.py` | Internal utility functions for computational group theory: base ordering, cycle checks for alternating/symmetric detection, BSGS-related strip and distribution routines, lazy computation of missing BSGS structures (transversals, basic orbits, distributed strong generators) from precomputed data (`_handle_precomputed_bsgs`), and orbit/transversal calculation from base and strong generating sets. |
