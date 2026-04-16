# sympy/combinatorics -- Catalog

> Part of [SymPy](../catalog.md). Combinatorics: permutations, partitions, polyhedra, group theory, and Graycode.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package initializer; imports and re-exports the main public classes and functions (Permutation, Prufer, Subset, Partition, Polyhedron, PermutationGroup, GrayCode, named groups, etc.). |
| `coset_table.py` | Implements the `CosetTable` class and Todd-Coxeter coset enumeration algorithms (relator-based and coset-table-based) for finitely presented groups. |
| `fp_groups.py` | Defines `FpGroup` (finitely presented groups) and related algorithms including low-index subgroup enumeration, Reidemeister-Schreier presentations, and permutation group isomorphism. |
| `free_groups.py` | Implements `FreeGroup` and `FreeGroupElement` for constructing and manipulating free groups and their elements with word-level arithmetic. |
| `generators.py` | Provides generator functions that yield all permutation elements of standard groups: `symmetric`, `cyclic`, `alternating`, `dihedral`, and Rubik's cube generators. |
| `graycode.py` | Implements the `GrayCode` class for generating and manipulating Gray codes (reflected binary codes) over n-dimensional binary cubes. |
| `group_constructs.py` | Provides the `DirectProduct` function for computing the direct product of multiple permutation groups. |
| `homomorphisms.py` | Implements `GroupHomomorphism` and a `homomorphism()` factory for creating and working with group homomorphisms between permutation groups and finitely presented groups. |
| `named_groups.py` | Factory functions for constructing standard named permutation groups: `SymmetricGroup`, `CyclicGroup`, `DihedralGroup`, `AlternatingGroup`, `AbelianGroup`, and `RubikGroup`. |
| `partitions.py` | Defines `Partition` (set partitions) and `IntegerPartition` classes, along with RGS (restricted growth string) ranking/unranking utilities. |
| `perm_groups.py` | Core `PermutationGroup` class implementing group-theoretic operations: order, orbits, stabilizers, Schreier-Sims algorithm, coset decomposition with ranking/unranking (`coset_rank`, `coset_unrank`), subgroup testing, centralizer computation (handling single permutations, lists, or groups as input), derived series and derived (commutator) subgroups, commutator of subgroups, and normal closure. |
| `permutations.py` | Core `Permutation` and `Cycle` classes for representing and manipulating individual permutations in array and cyclic notation, with support for composition, inversion, parity, and lexicographic ranking of individual permutations (not group coset ranking). Also includes low-level array-form helpers (`_af_rmul`, `_af_commutes_with`, etc.). |
| `polyhedron.py` | Implements the `Polyhedron` class for representing polyhedral symmetry groups, with pre-built instances for the five Platonic solids (tetrahedron, cube, octahedron, dodecahedron, icosahedron). |
| `prufer.py` | Implements the `Prufer` class for the bijection between labeled trees and Prufer sequences, with ranking/unranking and tree-edge conversion methods. |
| `rewritingsystem.py` | Implements `RewritingSystem` for Knuth-Bendix completion on finitely presented groups, used for word reduction and confluence checking. |
| `subsets.py` | Implements the `Subset` class for enumerating subsets of a set via binary, lexicographic, and Gray code orderings with ranking/unranking support. |
| `tensor_can.py` | Algorithms for tensor canonicalization using permutation group double-coset representatives, including dummy-index symmetry handling. |
| `testutil.py` | Testing utilities for the combinatorics module: naive centralizer computation, BSGS verification, normal closure checks, and graph certificate comparison. |
| `util.py` | Internal utility functions for computational group theory: base ordering, cycle checks for alternating/symmetric detection, BSGS-related strip and distribution routines. |
