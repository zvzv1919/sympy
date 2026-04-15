# sympy/categories — Catalog

> Part of [SymPy](../catalog.md). Category theory: objects, morphisms, diagrams, and diagram drawing.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package init that re-exports core classes (Object, Morphism, Diagram, Category, etc.) and diagram-drawing utilities from the submodules. |
| `baseclasses.py` | Defines the fundamental category-theory classes: Class, Object, Morphism (Identity, Named, Composite), Diagram, and Category. The Diagram class stores premises (assumed arrows) and conclusions (derived arrows) separately, and provides methods to query arrows between two nodes (`hom` returns a pair of sets—one from premises, one from conclusions), list all objects, and check subdiagram relationships. |
| `diagram_drawing.py` | Implements grid-based **layout and rendering** algorithms for commutative diagrams: DiagramGrid arranges nodes on an abstract grid, and XypicDiagramDrawer produces Xy-pic output. Does **not** define diagram data structures or arrow-querying logic (see `baseclasses.py` for that). |
