# sympy/categories — Catalog

> Part of [SymPy](../catalog.md). Category theory: objects, morphisms, diagrams, and diagram drawing.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package init that re-exports core classes (Object, Morphism, Diagram, Category, etc.) and diagram-drawing utilities from the submodules. |
| `baseclasses.py` | Defines the fundamental category-theory data-model classes: Class, Object, Morphism (Identity, Named, Composite), Diagram, and Category. CompositeMorphism flattens/simplifies arrow chains at construction time. Diagram includes operations for subdiagram containment checks (verifying arrow inclusion and property matching), hom-set queries, and extracting subdiagrams from object subsets. Pure data definitions — no layout or drawing logic. |
| `diagram_drawing.py` | Implements grid-based visual layout and rendering for commutative diagrams. `DiagramGrid._generic_layout` is the core node-placement algorithm: it handles special cases (single object → 1×1 cell; single arrow → horizontal pair), then builds a skeleton graph, enumerates triangles, and iteratively welds/attaches triangles onto a growing fringe to place remaining nodes. Also includes morphism pre-processing (dropping identities, merging premises/conclusions), grouped/sequential/transpose layout variants, and Xy-pic rendering via `XypicDiagramDrawer`. |
