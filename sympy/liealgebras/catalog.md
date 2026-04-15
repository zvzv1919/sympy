# sympy/liealgebras -- Catalog

> Part of [SymPy](../catalog.md). Lie algebras and root systems for classical groups (A, B, C, D, E, F, G).

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package init; imports `CartanType` to make it the primary public entry point. |
| `cartan_type.py` | Defines `CartanType_generator` (the factory callable `CartanType`) that dispatches to series-specific classes based on letter and rank. Enforces rank constraints for exceptional types (E: 6–8, F: 4, G: 2) and classical types (non-negative) **inside the dispatcher itself** — if a rank is valid but outside the allowed range for its series (e.g., E5), the dispatcher silently returns `None` instead of raising an error. Also defines `Standard_Cartan`, the concrete base class for all Cartan types. |
| `cartan_matrix.py` | Provides the `CartanMatrix` convenience function that returns the Cartan matrix for a given Lie algebra type string. |
| `dynkin_diagram.py` | Provides the `DynkinDiagram` convenience function that returns the ASCII Dynkin diagram for a given Lie algebra type string. |
| `root_system.py` | Implements the `RootSystem` class representing the root system of a simple Lie algebra, with methods for simple roots, all roots, root addition, Cartan matrix, and Dynkin diagram delegation. |
| `type_a.py` | `TypeA(Standard_Cartan)` -- series A (sl/su) Lie algebras: simple/positive roots, Cartan matrix, Dynkin diagram, and basis count for A_n. |
| `type_b.py` | `TypeB(Standard_Cartan)` -- series B (so, odd-dimensional) Lie algebras: root system data, Cartan matrix, and Dynkin diagram for B_n. |
| `type_c.py` | `TypeC(Standard_Cartan)` -- series C (sp) Lie algebras: root system data, Cartan matrix, and Dynkin diagram for C_n. |
| `type_d.py` | `TypeD(Standard_Cartan)` -- series D (so, even-dimensional) Lie algebras: simple/positive root vectors, Cartan matrix, and branched Dynkin diagram for D_n. Does NOT contain reflection matrices or Weyl group logic. |
| `type_e.py` | `TypeE(Standard_Cartan)` -- exceptional series E (E6, E7, E8): root system data, Cartan matrix, and Dynkin diagram. The `positive_roots` method enumerates roots in two stages: ±e_i ± e_j integer roots via `basic_root`, then half-integer (±1/2) coordinate vectors with an even-parity sign constraint using nested binary loops that mutate a root list in-place. |
| `type_f.py` | `TypeF(Standard_Cartan)` -- exceptional Lie algebra F4 (4-dimensional): root system data including 4-dimensional half-integer roots, Cartan matrix, and Dynkin diagram. See `type_e.py` for 8-dimensional half-integer root enumeration in E6/E7/E8. |
| `type_g.py` | `TypeG(Standard_Cartan)` -- exceptional Lie algebra G2: root system data, Cartan matrix, and Dynkin diagram. |
| `weyl_group.py` | `WeylGroup` class for computing Weyl group properties: generating reflections, group order, element order, and Coxeter diagrams. Contains `matrix_form` which builds explicit matrix representations of generating reflections with per-series branching (A, B, C, D, E, F, G) — each series has its own special-case matrix construction, including edge cases for the highest-numbered generator (e.g., D-type last generator uses negations instead of transpositions). |
