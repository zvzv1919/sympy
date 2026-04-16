# sympy/categories — Category Theory

Provides fundamental category-theory classes (objects, morphisms, diagrams, categories) and Xy-pic diagram rendering. Follows *Abstract and Concrete Categories: The Joy of Cats* (Adamek, Herrlich, Strecker). Functors are not yet implemented.

## Core Classes

### `baseclasses.py`

Defines the foundational category-theory objects used throughout the module.

- `Class(Set)` — synonym for `Set`; placeholder for future proper set-theory support. Has `is_proper = False`.
- `Object(Symbol)` — base class for abstract category objects.
- `Morphism(Basic)` — abstract base for morphisms (cannot be instantiated directly).
  - Properties: `domain`, `codomain`.
  - `compose(other)` / `__mul__` — composes morphisms in standard mathematical order (`g.compose(f)` = g∘f).
- `IdentityMorphism(Morphism)` — identity morphism on a single object; acts as the identity under composition.
- `NamedMorphism(Morphism)` — morphism distinguished by a name string. Two named morphisms are equal iff they share domain, codomain, and name.
- `CompositeMorphism(Morphism)` — morphism built from a sequence of composable morphisms.
  - Constructor takes components in diagram order (f, g for g∘f).
  - Automatically flattens nested composites and strips identities during construction.
  - `flatten(new_name)` — forgets composite structure, returning a `NamedMorphism` with the overall domain/codomain.
- `Category(Basic)` — an abstract category with a name, optional objects class, and a `FiniteSet` of commutative diagrams. `hom()` and `all_morphisms()` raise `NotImplementedError`.
- `Diagram(Basic)` — a collection of morphisms (premises + conclusions) that forms a monoid under composition.
  - Constructor accepts lists or dicts of morphisms (with optional property sets); automatically closes under composition and adds identity morphisms.
  - `hom(A, B)` — returns `(premises_hom, conclusions_hom)` morphism sets between two objects.
  - `is_subdiagram(diagram)` — checks containment of premises and conclusions with matching properties.
  - `subdiagram_from_objects(objects)` — extracts a sub-diagram restricted to a given object set.

## Diagram Drawing

### `diagram_drawing.py`

Lays out diagram objects on a 2D grid and renders them as Xy-pic LaTeX strings.

- `_GrowableGrid` — internal dynamically-resizable 2D array supporting row/column prepend and append.
- `DiagramGrid` — analyses a `Diagram` and places its objects on a grid to minimise morphism crossings.
  - Supports two layout strategies:
    - **Generic** (default): decomposes the morphism graph into triangles and iteratively welds them onto a fringe.
    - **Sequential** (`layout="sequential"`): depth-first search from the minimum-degree vertex, producing near-linear layouts.
  - Handles `groups` (logical groupings of objects laid out independently then composed), `transpose`, and disconnected diagrams.
  - Key internal helpers:
    - `_build_skeleton` — builds undirected edge graph from morphisms, adding juxtaposed edges for triangle decomposition.
    - `_list_triangles` / `_drop_redundant_triangles` — enumerates and filters triangles by edge significance.
    - `_find_triangle_to_weld` / `_weld_triangle` — iteratively attaches triangles to the placement fringe.
    - `_grow_pseudopod` — extends the grid when no direct triangle welding is possible.
    - `_handle_groups` — recursively lays out logical groups, then composes them into a master grid.
    - `_get_connected_components` — splits disconnected diagrams for independent layout.
- `ArrowStringDescription` — data class holding all parameters for one Xy-pic `\ar` command (curving, looping, direction, label position, style).
- `XypicDiagramDrawer` — given a `Diagram` + `DiagramGrid`, produces a complete `\xymatrix{...}` string.
  - `draw(diagram, grid, masked=None, diagram_format="")` — main entry point; processes each morphism, applies formatters, pushes labels to outer edges, and emits the Xy-pic string.
  - Supports `arrow_formatters` (per-property) and `default_arrow_formatter` for customising arrow appearance.
  - Internal morphism processing distinguishes loop, horizontal, vertical, and diagonal arrows, choosing curving direction to avoid collisions.
- `xypic_draw_diagram(...)` — convenience function combining `DiagramGrid` + `XypicDiagramDrawer.draw`.
- `preview_diagram(...)` — convenience function combining `xypic_draw_diagram` with `sympy.printing.preview` for direct PNG/PDF output. Requires `latex`, `dvipng`, and `pyglet`.

### `__init__.py`

Re-exports public API: `Object`, `Morphism`, `IdentityMorphism`, `NamedMorphism`, `CompositeMorphism`, `Category`, `Diagram`, `DiagramGrid`, `XypicDiagramDrawer`, `xypic_draw_diagram`, `preview_diagram`.
