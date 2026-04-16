# sympy/liealgebras -- Catalog

> Part of [SymPy](../catalog.md). Lie algebras and root systems for classical groups (A, B, C, D, E, F, G).

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package init; imports `CartanType` to make it the primary public entry point. |
| `cartan_type.py` | Defines `CartanType_generator` (the factory callable `CartanType`) that dispatches to series-specific classes, and `Standard_Cartan`, the concrete base class for all Cartan types. |
| `cartan_matrix.py` | Provides the `CartanMatrix` convenience function that returns the Cartan matrix for a given Lie algebra type string. |
| `dynkin_diagram.py` | Provides the `DynkinDiagram` convenience function that returns the ASCII Dynkin diagram for a given Lie algebra type string. |
| `root_system.py` | Implements the `RootSystem` class representing the root system of a simple Lie algebra, with methods for simple roots, all roots, root addition, Cartan matrix, and Dynkin diagram delegation. |
| `type_a.py` | `TypeA(Standard_Cartan)` -- series A (sl/su) Lie algebras: simple/positive roots, Cartan matrix, Dynkin diagram, and basis count for A_n. |
| `type_b.py` | `TypeB(Standard_Cartan)` -- series B (so(2n+1), odd-dimensional orthogonal) Lie algebras: simple roots where the first n-1 are difference vectors (1,-1) like A-series but the last simple root is a single unit vector [0,…,0,1]; positive roots, Cartan matrix, and Dynkin diagram for B_n. |
| `type_c.py` | `TypeC(Standard_Cartan)` -- series C (sp) Lie algebras: root system data, Cartan matrix, and Dynkin diagram for C_n. |
| `type_d.py` | `TypeD(Standard_Cartan)` -- series D (so(2n), even-dimensional orthogonal) Lie algebras: simple roots where the first n-1 are difference vectors (1,-1) like A-series but the last simple root is a sum vector [0,…,0,1,1] with 1s in both final positions (branching); positive roots, Cartan matrix, and branched Dynkin diagram for D_n. |
| `type_e.py` | `TypeE(Standard_Cartan)` -- exceptional series E (E6, E7, E8): root system data, Cartan matrix, and Dynkin diagram. |
| `type_f.py` | `TypeF(Standard_Cartan)` -- exceptional Lie algebra F4: root system data including half-integer roots, Cartan matrix, and Dynkin diagram. |
| `type_g.py` | `TypeG(Standard_Cartan)` -- exceptional Lie algebra G2: root system data, Cartan matrix, and Dynkin diagram. |
| `weyl_group.py` | `WeylGroup` class for computing Weyl group properties: group order, element order, Coxeter diagrams, and — crucially — `matrix_form`, which builds the standard matrix representation for each generating reflection with series-specific construction logic (e.g., D-type's last reflection uses a distinct negative-entry matrix unlike the transposition matrices of other reflections). |
