# liealgebras — Module Catalog

Implements simple Lie algebras via Cartan classification: root systems, Cartan matrices, Dynkin diagrams, and Weyl groups for series A–G.

## Core Infrastructure

### [`__init__.py`](__init__.py)
Exports `CartanType` as the main entry point.

### [`cartan_type.py`](cartan_type.py)
Factory and base classes for Cartan type objects.
- `CartanType_generator` — parses string/list input (e.g. "A3", "D4") and instantiates the correct `Type*` class.
- Caveat: if series letter is valid but rank is out of range (e.g. E5, F3, G4), silently returns `None` instead of raising an error.
- `Standard_Cartan` — base class for all series; stores `series` letter and `rank`.

### [`cartan_matrix.py`](cartan_matrix.py)
Convenience function `CartanMatrix(ct)` — delegates to the appropriate type class to return its Cartan matrix.

### [`dynkin_diagram.py`](dynkin_diagram.py)
Convenience function `DynkinDiagram(t)` — delegates to the appropriate type class to return its Dynkin diagram string.

### [`root_system.py`](root_system.py)
`RootSystem` — facade over a Cartan type for root-level operations.
- `simple_roots()` — collects simple roots from the type class.
- `all_roots()` — builds complete root set: fetches positive roots dict from the type class, then negates each to produce negative roots in the same dict. Negative root keys start at max(positive_key)+1 and increment (snapshots keys first to avoid mutation during iteration).
- `add_simple_roots()`, `add_as_roots()` — root arithmetic (sum only if result is a root).
- `root_space()` — string description of the root space as a span of simple roots.

## Weyl Groups

### [`weyl_group.py`](weyl_group.py)
`WeylGroup` — represents the Weyl group (reflection symmetry group) of a Lie algebra.
- `generators()` — lists generating reflections (r1, r2, …).
- `group_order()` — order of the full Weyl group; dispatches per series: factorial formulas for A/B/C/D, hardcoded constants for E (ranks 6/7/8 only), F, G.
- `group_name()` — descriptive name and geometric interpretation.
- `element_order(weylelt)` — order of a specific element given as a product of generators.
- `matrix_form(weylelt)` — converts a product-of-reflections string into its reflection matrix representation per series (A, B/C, D, E, F, G).
  - Exceptional types (G, F, E) use hardcoded per-generator matrices with Rational entries (e.g. G2 in 3D, F4 in 4D, E in 8D).
- `coxeter_diagram()` — undirected Coxeter diagram.

## Type Series — Root System Definitions

Each `type_*.py` file defines root-system properties (simple roots, positive roots, Cartan matrix, Dynkin diagram, dimension, basis count) for one Cartan series. They contain **no** Weyl group logic — group order, generators, element operations, and reflection matrices all live in `weyl_group.py`.

### [`type_a.py`](type_a.py)
`TypeA` — A_n series (first classical series). Lie algebra su(n+1). Dimension n+1. Roots: n(n+1).
- `basic_root(i, j)` — helper producing a vector with +1 at position i and −1 at position j.
- `positive_roots()` — enumerates all positive roots via nested loop over pairs i < j, each root a +1/−1 difference vector.

### [`type_b.py`](type_b.py)
`TypeB` — B_n series (odd-dimensional orthogonal). Dimension n. Roots: 2n². Rank ≥ 2.
- `simple_root(i)` — first n−1 roots are difference vectors (same as A_(n−1)); nth root is a single unit vector [0,…,0,1] (no −1 component).
- `positive_roots()` — generates three kinds of positive roots: difference vectors (e_i−e_j), sum vectors (e_i+e_j), and unit vectors (e_i).
- Cartan matrix has asymmetric boundary entry (−2 in [n−2,n−1] vs −1 in [n−1,n−2]).
- Caveat: `lie_algebra()` returns "so(2n)" (same string as D_n); mathematically B_n corresponds to so(2n+1).

### [`type_c.py`](type_c.py)
`TypeC` — C_n series. Lie algebra sp(2n). Dimension n. Roots: 2n².

### [`type_d.py`](type_d.py)
`TypeD` — D_n series (even-dimensional orthogonal, so(2n)). Dimension n. Roots: 2n(n−1). Branching Dynkin diagram. Rank ≥ 3.
- `simple_root(i)` — constructs fundamental roots: first n−1 are difference vectors like A_(n−1); the nth (last) root breaks the pattern with two +1 entries [0,…,0,1,1] instead of a +1/−1 pair (branching node).
- `positive_roots()` — generates two kinds of positive roots: difference vectors (e_i−e_j) and sum vectors (e_i+e_j). No unit-vector roots (unlike B_n).

### [`type_e.py`](type_e.py)
`TypeE` — E_6, E_7, E_8 exceptional algebras. Dimension 8 for all ranks. Branching Dynkin diagram (trivalent node).
- `cartan_matrix()` — builds Cartan matrix with hardcoded off-diagonal entries for the branching node (indices 0–3), then a loop for the linear chain portion (index 3 onward).
- `positive_roots()` — rank-dependent enumeration (branches on n=6/7/8); uses Rational(±1/2) vectors with even-count sign constraint on 8-dimensional coordinates.

### [`type_f.py`](type_f.py)
`TypeF` — F_4 exceptional algebra. Rank 4, dimension 4. 48 roots. Explicit 4×4 Cartan matrix.
- `basic_root(i, j)` — helper producing a vector with +1 at position i and −1 at position j.
- `simple_root(i)` — returns the ith simple root of F_4; first two roots use `basic_root`, third is a unit vector, fourth is all −1/2.
- `positive_roots()` — constructs 24 positive roots in three groups: difference/sum vectors (e_i±e_j), unit vectors (e_i), and half-integer vectors (±1/2 coordinate 4-vectors) enumerated by explicit sign combinations.

### [`type_g.py`](type_g.py)
`TypeG` — G_2 exceptional algebra. Dimension 3. 12 roots. Cartan matrix has −3 entry.
