# categories — Module Catalog

Category theory foundations: objects, morphisms, diagrams (with premises/conclusions), and diagram visualization via Xy-pic.

## Package Interface

### [`__init__.py`](__init__.py)
Public API of the categories package. Re-exports all primitives from `baseclasses` and all visualization utilities from `diagram_drawing`.
- Known gap: functors are **not yet implemented**.
- Reference work: *Abstract and Concrete Categories — The Joy of Cats* (Adamek, Herrlich, Strecker).

## Core Data Model

### [`baseclasses.py`](baseclasses.py)
Defines all category-theory primitives: objects, morphisms, categories, and diagrams.
- `Class` — base set-theoretic class (synonym for `Set`).
- `Object` — abstract object in a category (subclass of `Symbol`).
- `Morphism` — base morphism with `domain`, `codomain`, and `compose`/`__mul__`.
- `IdentityMorphism` — identity morphism on a single object.
- `NamedMorphism` — morphism with a user-supplied `name` string.
- `CompositeMorphism` — morphism formed by composing a sequence of `components`; `flatten(new_name)` collapses to a `NamedMorphism`.
- `Category` — a named category containing objects and commutative diagrams; `hom`/`all_morphisms` not yet implemented.
- `Diagram` — a commutative diagram built from **premises** (assumed morphisms) and **conclusions** (derived morphisms), each mapping morphisms → property sets.
  - `__new__` — constructs diagram; enforces that conclusions may only reference objects already established by premises (new-entity arrows are silently discarded). Identity/composite closure is added for premises but **not** for conclusions.
  - `_add_morphism_closure` — adds a morphism and auto-generates identity morphisms and composites; raises `ValueError` if properties are assigned to an `IdentityMorphism`.
  - `premises`, `conclusions` — `Dict` of morphism → `FiniteSet` of properties.
  - `objects` — `FiniteSet` of all objects appearing in the diagram.
  - `hom(A, B)` — returns (premise morphisms, conclusion morphisms) between two objects.
  - `is_subdiagram(diagram)` — checks containment: all premises/conclusions of `diagram` exist in `self` with identical properties.
  - `subdiagram_from_objects(objects)` — extracts a sub-diagram restricted to a given set of objects, preserving properties.

## Diagram Visualization

### [`diagram_drawing.py`](diagram_drawing.py)
Lays out diagram objects on a 2-D grid and renders to Xy-pic LaTeX strings. Handles only **visual presentation**, not diagram semantics.
- `DiagramGrid` — places objects of a `Diagram` onto a grid using triangle-welding layout algorithm. Performs no semantic validation; operates on an already-constructed `Diagram`.
  - Preprocessing: for **layout purposes only**, removes identity/composite morphisms that have no properties and unions premises with conclusions into a single edge set.
  - Builds skeleton of edges, decomposes into triangles, and sorts triangles by a size metric before welding.
  - `_morphism_length` — returns 1 for simple morphisms, component count for `CompositeMorphism`; used to compute triangle min sizes.
  - Supports `groups` (object groupings) and layout `hints` (e.g. transpose, sequential layout for linear diagrams).
- `ArrowStringDescription` — data class describing a single arrow's visual attributes (label, curve, style, direction offsets).
- `XypicDiagramDrawer` — converts a `DiagramGrid` into Xy-pic markup; handles arrow styling, label placement, looped morphisms.
  - `_process_morphism` — builds the label string for each arrow: identity (`id_{…}`), composite (`\circ`-joined component names), or named morphism name.
- `xypic_draw_diagram(diagram)` — convenience function returning Xy-pic string.
- `preview_diagram(diagram)` — renders diagram to image via LaTeX/Xy-pic.
