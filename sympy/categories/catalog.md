# sympy/categories — Catalog

> Part of [SymPy](../catalog.md). Category theory: objects, morphisms, diagrams, and diagram drawing.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package init that re-exports core classes (Object, Morphism, Diagram, Category, etc.) and diagram-drawing utilities from the submodules. |
| `baseclasses.py` | Defines the fundamental category-theory classes: Class, Object, Morphism (Identity, Named, Composite), Diagram, and Category. The **Diagram** class is the core data model for commutative diagrams—it stores premises (assumed arrows) and conclusions (derived arrows) as separate dictionaries, and is the sole place for **querying/retrieving arrows between nodes**: `hom(A, B)` filters all morphisms by domain/codomain and returns a 2-tuple of sets (premises arrows, conclusions arrows), separating assumed from derived. Also provides `objects`, `premises`, `conclusions` properties and `is_subdiagram` checks. Any question about how arrows are stored, retrieved, filtered, or separated between assumed and derived belongs here. |
| `diagram_drawing.py` | Implements grid-based **visual layout and rendering** of commutative diagrams: DiagramGrid arranges objects on a grid, and XypicDiagramDrawer produces Xy-pic LaTeX output. Consumes Diagram objects from `baseclasses.py` but does **not** store, query, or filter arrows—it only decides how to position and draw them. For retrieving or classifying arrows between nodes, see `baseclasses.py`. |
