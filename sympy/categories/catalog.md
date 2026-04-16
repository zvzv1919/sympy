# sympy/categories — Catalog

> Part of [SymPy](../catalog.md). Category theory: objects, morphisms, diagrams, and diagram drawing.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package init that re-exports core classes (Object, Morphism, Diagram, Category, etc.) and diagram-drawing utilities from the submodules. |
| `baseclasses.py` | Defines the fundamental category-theory data-model classes: Class, Object, Morphism (Identity, Named, Composite), Diagram, and Category. CompositeMorphism flattens/simplifies arrow chains at construction time. `Diagram.__new__` constructs diagrams from two arrow collections (premises and conclusions): it tracks objects from premises, then validates that conclusions only reference objects already in premises and skips adding identity morphisms and recursing composites for conclusions. Also includes subdiagram containment checks, hom-set queries, and subdiagram extraction. Pure data definitions — no layout or drawing logic. |
| `diagram_drawing.py` | Implements grid-based visual layout and rendering for commutative diagrams. `DiagramGrid._generic_layout` is the core node-placement algorithm: it handles special cases (single object → 1×1 cell; single arrow → horizontal pair), then builds a skeleton graph, enumerates triangles, computes triangle size metrics (via `_morphism_length` which dispatches on simple vs composite morphisms, and `_compute_triangle_min_sizes`), and iteratively welds/attaches triangles onto a growing fringe to place remaining nodes. Also includes layout-time morphism simplification (stripping property-less identities/composites before rendering, merging premises and conclusions for display), grouped/sequential/transpose layout variants, and Xy-pic rendering via `XypicDiagramDrawer`. |
