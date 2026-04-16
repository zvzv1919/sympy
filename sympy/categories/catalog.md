# sympy/categories — Catalog

> Part of [SymPy](../catalog.md). Category theory: objects, morphisms, diagrams, and diagram drawing.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package init that re-exports core classes (Object, Morphism, Diagram, Category, etc.) and diagram-drawing utilities from the submodules. |
| `baseclasses.py` | Defines the fundamental category-theory classes: Class, Object, Morphism (Identity, Named, Composite), Diagram, and Category. Diagram includes operations for subdiagram containment checks (verifying arrow inclusion and property matching), hom-set queries, and extracting subdiagrams from object subsets. |
| `diagram_drawing.py` | Implements grid-based visual layout algorithms for positioning diagram nodes on a grid and rendering them via Xy-pic back-end (DiagramGrid and XypicDiagramDrawer). Handles only spatial arrangement and drawing, not logical diagram operations. |
