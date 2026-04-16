# sympy/codegen — Code Generation

AST types for representing code structures (assignments, code blocks, loops) used by SymPy's code printers and code generation utilities.

## Package Init

### `__init__.py`
Re-exports the public API: `Assignment`, `aug_assign`, `CodeBlock`, `For`.

## AST Nodes

### `ast.py`
Defines the Abstract Syntax Tree types used to represent code constructs (assignments, blocks, loops) for code generation.

**`Assignment(Relational)`** — Variable assignment (`lhs := rhs`).
- Validates that `lhs` is an assignable type (`Symbol`, `MatrixSymbol`, `MatrixElement`, `Indexed`).
- Validates shape compatibility between lhs and rhs for matrix assignments.

**`AugmentedAssignment(Assignment)`** — Base class for `op=` compound assignments.
- Concrete subclasses: `AddAugmentedAssignment` (`+=`), `SubAugmentedAssignment` (`-=`), `MulAugmentedAssignment` (`*=`), `DivAugmentedAssignment` (`/=`), `ModAugmentedAssignment` (`%=`).

**`aug_assign(lhs, op, rhs)`** — Convenience factory that dispatches to the correct `AugmentedAssignment` subclass based on the operator string.

**`CodeBlock(Basic)`** — Ordered sequence of statements (currently assignments only).
- `left_hand_sides` / `right_hand_sides` — Tuples of lhs/rhs extracted from contained assignments.
- `topological_sort(assignments)` — Class method; reorders assignments so every variable is assigned before it is used (uses graph-based topological sort).
- `cse(...)` — Performs common-subexpression elimination across the block's right-hand sides and returns a new topologically-sorted `CodeBlock`.

**`For(Basic)`** — For-loop construct with `target`, `iterable`, and `body` (a `CodeBlock`).

#### Caveats
- All assignment and augmented-assignment classes are registered into `Relational.ValidRelationOperator` at module level as a side effect.
- `CodeBlock` currently only supports `Assignment` nodes; `cse()` does not support augmented assignments or duplicate lhs variables.
