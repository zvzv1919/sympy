# liealgebras — Module Catalog

Implements simple Lie algebras via Cartan classification: root systems, Cartan matrices, Dynkin diagrams, and Weyl groups for series A–G.

## Core Infrastructure

### [`__init__.py`](__init__.py)
Exports `CartanType` as the main entry point.

### [`cartan_type.py`](cartan_type.py)
Factory and base classes for Cartan type objects.
- `CartanType_generator` — parses string/list input (e.g. "A3", "D4") and instantiates the correct `Type*` class.
- `Standard_Cartan` — base class for all series; stores `series` letter and `rank`.

### [`cartan_matrix.py`](cartan_matrix.py)
Convenience function `CartanMatrix(ct)` — delegates to the appropriate type class to return its Cartan matrix.

### [`dynkin_diagram.py`](dynkin_diagram.py)
Convenience function `DynkinDiagram(t)` — delegates to the appropriate type class to return its Dynkin diagram string.

### [`root_system.py`](root_system.py)
`RootSystem` — facade over a Cartan type for root-level operations.
- `simple_roots()`, `all_roots()` — enumerates roots by delegating to the type class.
- `add_simple_roots()`, `add_as_roots()` — root arithmetic (sum only if result is a root).
- `root_space()` — string description of the root space as a span of simple roots.

## Weyl Groups

### [`weyl_group.py`](weyl_group.py)
`WeylGroup` — represents the Weyl group (reflection symmetry group) of a Lie algebra.
- `generators()` — lists generating reflections (r1, r2, …).
- `group_order()` — order of the full Weyl group.
- `group_name()` — descriptive name and geometric interpretation.
- `element_order(weylelt)` — order of a specific element given as a product of generators.
- `matrix_form(weylelt)` — converts a product-of-reflections string into its matrix representation; builds standard matrices for each generating reflection per series (A, B, C, D, F, G) with series-specific special cases (e.g. D-type last reflection differs from transpositions).
- `coxeter_diagram()` — undirected Coxeter diagram.

## Type Series — Root System Definitions

Each `type_*.py` file defines root-system properties (simple roots, positive roots, Cartan matrix, Dynkin diagram, dimension, basis count) for one Cartan series. They do **not** contain Weyl group matrix representations.

### [`type_a.py`](type_a.py)
`TypeA` — A_n series (first classical series). Lie algebra su(n+1). Dimension n+1. Roots: n(n+1).
- `basic_root(i, j)` — helper producing a vector with +1 at position i and −1 at position j.
- `positive_roots()` — enumerates all positive roots via nested loop over pairs i < j, each root a +1/−1 difference vector.

### [`type_b.py`](type_b.py)
`TypeB` — B_n series. Lie algebra so(2n+1). Dimension n. Roots: 2n².
- `simple_root(i)` — first n−1 roots are difference vectors (same as A_(n−1)); last root is a single unit vector [0,…,0,1].

### [`type_c.py`](type_c.py)
`TypeC` — C_n series. Lie algebra sp(2n). Dimension n. Roots: 2n².

### [`type_d.py`](type_d.py)
`TypeD` — D_n series. Lie algebra so(2n). Dimension n. Roots: 2n(n−1). Branching Dynkin diagram.
- `simple_root(i)` — defines simple roots; last root has two +1 entries (branching node).

### [`type_e.py`](type_e.py)
`TypeE` — E_6, E_7, E_8 exceptional algebras. Dimension 8 for all. Complex positive-root enumeration with Rationals.

### [`type_f.py`](type_f.py)
`TypeF` — F_4 exceptional algebra. Dimension 4. 48 roots. Explicit 4×4 Cartan matrix.

### [`type_g.py`](type_g.py)
`TypeG` — G_2 exceptional algebra. Dimension 3. 12 roots. Cartan matrix has −3 entry.
