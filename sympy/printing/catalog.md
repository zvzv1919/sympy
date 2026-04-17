# Module Catalog: printing

## Architecture Overview

The `printing` module converts SymPy expressions into various output formats. All printers inherit from `Printer` (in `printer.py`) and use a `_print` dispatch mechanism that routes expressions to type-specific `_print_*` methods.

The **pretty** subpackage (2D ASCII/Unicode output) has a strict layered architecture:
- `pretty/pretty.py` — layout orchestration: decides spatial arrangement of expression parts using `stringPict` objects.
- `pretty/pretty_symbology.py` — character/symbol generation: provides Unicode/ASCII primitives (brackets, delimiters, atoms, Greek letters) and constructs multi-line spatial characters.
- `pretty/stringpict.py` — 2D string canvas: represents multiline text pictures with baseline tracking and spatial combination methods.

---

## Base Infrastructure

### [`printer.py`](printer.py)
Base `Printer` class with `_print` dispatch mechanism routing expressions to `_print_*` methods. Manages settings and print-level tracking.
- `_print` walks the expression's MRO looking for `_print_<ClassName>` handlers; if none found, falls back to `self.emptyPrinter(expr)` (defaults to `str(expr)`).

### [`precedence.py`](precedence.py)
Operator precedence values (`PRECEDENCE` dict) and functions for determining when parentheses are needed.

### [`conventions.py`](conventions.py)
- `split_super_sub()` — parses symbol names into base + superscripts + subscripts.
- `requires_partial()` — checks if partial derivative notation is needed.

### [`defaults.py`](defaults.py)
`DefaultPrinting` mixin providing `__str__`/`__repr__` using `sstr()`.

---

## Pretty Printing (2D ASCII/Unicode)

### [`pretty/pretty.py`](pretty/pretty.py)
`PrettyPrinter` — renders expressions as 2D human-readable text art. Contains all expression-specific `_print_*` handlers that **orchestrate layout** by composing `stringPict` objects with symbols from `pretty_symbology`.
- `_print_Mul` — renders products as 2D stacked fractions with a horizontal bar; splits factors into numerator vs denominator lists, inserts `1` when numerator is empty. Uses `evaluate=False` for non-`-1` negative exponents to suppress auto-simplification when negating the exponent for the denominator.
- `_print_MatrixElement` — renders matrix element access; when parent is a `MatrixSymbol` with numeric indices, produces a subscripted symbol (e.g., `A₁₂`); otherwise uses function-like bracket notation `A[i, j]`.
- `_print_MatrixSlice` — renders sub-range access with start:stop:step notation; simplifies by omitting unit step, collapsing single-element ranges, and dropping zero start index.
- `_print_Product` — builds the iterated product (∏) sign as 2D box art; computes sign width from function height.
- `_print_Sum` — builds the summation (∑) sign with upper/lower limits.
- `_print_Integral` — builds integral signs with limits and spacing.
- Special-case `_print_*` overrides for functions whose names collide with Greek/Unicode symbols (e.g., `_print_Chi` keeps Latin "Chi" instead of Greek χ, `_print_gamma`/`_print_lowergamma`/`_print_uppergamma` use explicit Γ/γ glyphs).
- `_print_Function` — renders applied callables; attaches the formatted name and argument list as attributes on the result form so they can be reassembled when exponentiation is applied.
- Handles matrices, piecewise, sequences, sets, relational operators, containers (tuple, list, dict, set), and all standard math expressions. Sign insertion (`+`/`-`) between addition terms is delegated to `prettyForm.__add__` in `stringpict.py`.
- `_print_tuple` — single-element tuples append a trailing comma before parenthesizing, to distinguish from a mere parenthesized expression.
- `_print_Float` — when `full_prec` setting is `"auto"`, shows full precision only at the top print level (`_print_level == 1`); nested floats use reduced precision.

### [`pretty/pretty_symbology.py`](pretty/pretty_symbology.py)
Symbol/character primitives and Unicode↔ASCII abstraction layer. This is **not** a printer — it provides building blocks that `pretty.py` consumes.
- `xobj(symb, length)` — constructs multi-line spatial objects (brackets, braces, integral signs) of a given height; handles even-height adjustment for centered middle pieces (e.g., curly braces).
- `pretty_atom(atom_name, default=None)` — returns pretty representation of named atoms (pi, infinity, etc.); raises `KeyError('only unicode')` in ASCII mode when no default is provided.
- `pretty_symbol(symb_name)` — translates symbol names to Unicode glyphs (Greek letters, sub/superscripts). If any superscript character fails Unicode mapping, **both** super- and subscript prettification are abandoned and the name falls back to underscore-delimited ASCII. Passive lookup only — rendering overrides live in `pretty.py`'s `_print_*` methods.
- `xsym(sym)` — resolves operator characters (comparison `<=`/`>=`/`!=`, arithmetic `*`/`.`, arrows `-->`/`==>`, assignment `:=`/`+=`) to Unicode or ASCII display form via `_xsym` lookup table.
- `vobj(symb, height)` / `hobj(symb, width)` — vertical/horizontal object constructors.
- Key data: `atoms_table` (atom→Unicode mapping), `_xobj_unicode`/`_xobj_ascii` (bracket/delimiter glyph tables).

### [`pretty/stringpict.py`](pretty/stringpict.py)
`stringPict` — 2D string canvas with baseline tracking. Subclass `prettyForm` adds binding strength for precedence-aware parenthesization.
- Spatial combinators: `above`, `below`, `left`, `right`, `stack` — arrange sub-pictures relative to each other.
- `parens(left, right, ifascii_nougly)` — wraps picture in parentheses; in ASCII mode with `ifascii_nougly=True`, collapses height to 1 to avoid ugly tall brackets.
- `terminal_width()` — detects console column count; uses `curses.tigetnum` on Unix, falls back to Windows `kernel32.GetConsoleScreenBufferInfo` via ctypes on Windows.
- `prettyForm.__div__` — constructs stacked fractions via `stack(num, LINE, den)`; handles negative-numerator and nested-division parenthesization.
- `prettyForm.__add__` — binding-aware addition; reuses existing minus signs to simplify `+ -x` forms.
- `prettyForm.__mul__` — binding-aware multiplication; detects `-1` factors and substitutes `-1 * x` → `-x`; inserts a space when consecutive leading minus signs would collide.

### [`pretty/__init__.py`](pretty/__init__.py)
Public API: `pretty`, `pretty_print`/`pprint`, `pretty_use_unicode`.

---

## String / Code Printers

### [`str.py`](str.py)
`StrPrinter` — generates readable **1D flat-text** string representations with precedence-based parenthesization. No 2D layout, fraction bars, or spatial arrangement.
- `_print_Add` — determines sign of each summand by checking if its printed string starts with `'-'`; strips leading `'-'` and rebuilds with `+`/`-` tokens; omits leading `+` for the first term.

### [`codeprinter.py`](codeprinter.py)
`CodePrinter` base class for code-generating printers. Extends `StrPrinter` with `doprint(assign_to)` for assignment statements and formatting hooks.
- `_print_Mul` — splits factors into numerator/denominator lists based on negative rational exponents; renders as flat 1D text `a*b/c` or `a*b/(c*d)` (no 2D fraction bars).

### [`ccode.py`](ccode.py)
`CCodePrinter` — generates C code, mapping SymPy functions to C math library equivalents.
- `_print_Pow` — special-cases: exp==-1 → `1.0/x`, exp==0.5 → `sqrt(x)`, otherwise `pow(x, y)`.
- `_print_Rational` — emits long-double literals (`p.0L/q.0L`).
- `_print_Indexed` — flattens multi-dimensional array access into a single linear index using row-major (C-style) linearization.

### [`fcode.py`](fcode.py)
`FCodePrinter` — generates Fortran code with language-specific operators and formatting (source format, precision, contraction).

### [`jscode.py`](jscode.py)
`JavascriptCodePrinter` — generates JavaScript code from expressions.
- `jscode()` — top-level API; `human=False` returns a tuple `(symbols_to_declare, not_supported_functions, code_text)` instead of a single string.

### [`julia.py`](julia.py)
`JuliaCodePrinter` — generates Julia code from expressions for a scientific computing language.
- Emits element-wise dot operators (`.^`, `./`, `.*`) **by default** for regular `Symbol` operands to support vectorized code; uses standard operators (`^`, `/`, `*`) only for pure numbers or `MatrixSymbol` operands.
- `julia_code()` — top-level API; returns Julia-syntax string with dot-operator rules, assignment support, and custom function dispatch.
- `_print_Pow` — special-cases exponents ½, −½, −1 with `sqrt` and appropriate division operators.
- `_print_Piecewise` — dual-mode conditional output: inline emits nested ternary `(cond) ? (expr) :` chains; block mode emits `if/elseif/else/end`. Requires last branch to have a True guard.
- `indent_code` — auto-indents generated code using regex-matched block keywords; lines that both close and open blocks (e.g., `elseif`, `else`) decrease indent before the line and increase after.

### [`octave.py`](octave.py)
`OctaveCodePrinter` — generates Octave/MATLAB code from expressions.
- `_print_Mul` — decides between scalar (`*`, `/`) and element-wise (`.*`, `./`) operators based on whether each operand is a pure number; handles imaginary-number shorthand.
- `_print_Pow` — special-cases exponents ½, −½, −1 with `sqrt` and element-wise vs scalar division.
- `_print_Piecewise` — dual-mode conditional output: inline emits nested element-wise multiply `(cond).*(expr) + (~cond).*(...)`; block mode emits `if/elseif/else/end`. Requires last branch to have a True guard.

### [`repr.py`](repr.py)
`ReprPrinter` — generates eval-able `repr()` strings (`srepr`) for round-trip fidelity: `eval(srepr(expr)) == expr`.
- `_print_Symbol` — includes declared assumption properties (e.g., `positive=True`, `commutative=False`) in output so symbols round-trip with their metadata.
- `_print_MatrixBase` — emits constructor-call strings for matrices; special-cases matrices with zero rows XOR zero cols (emits explicit dimension args with empty list).

### [`lambdarepr.py`](lambdarepr.py)
`LambdaPrinter` — generates Python lambda-compatible string representations for use with `lambdify`.
- `_print_Piecewise` — converts to nested ternary expressions (`(e1) if (c1) else (e2) if (c2) else None`); final fallback is `None`.
- `NumPyPrinter` subclass — vectorized NumPy output; prints sequences as tuples (for numba nopython compatibility).
  - `_print_DotProduct` — emits `dot(a, b)` with automatic transpose to ensure 1×n by n×1 orientation when vector shapes don't match.
  - `_print_MatMul` — chains `.dot()` calls for matrix multiplication.

### [`python.py`](python.py)
`PythonPrinter` — generates executable Python code strings.

---

## Markup Printers

### [`latex.py`](latex.py)
`LatexPrinter` — converts expressions to LaTeX markup strings (e.g., `\frac{x}{y}`). Produces **1D markup text**, not spatial/visual rendering.
- `__init__` — mode-dependent defaults: `mode='inline'` auto-enables `fold_short_frac` (compact `a/b` instead of `\frac{a}{b}` for simple fractions).
- Configures `mul_symbol_latex` and `mul_symbol_latex_numbers` (numeric factors default to centered dot even when general symbol is a space).
- `_print_Add` — iterates ordered terms; when a term has a negative leading coefficient, emits ` - ` and negates the term to produce clean `a - b` instead of `a + -b`.
- `_print_Mul` — renders products; uses two distinct separator settings: `mul_symbol_latex` between general factors and `mul_symbol_latex_numbers` between adjacent numeric factors (detected via regex on rendered terms). Renders fractions via `\frac{}{}` when a denominator is present.
- `_print_Integral` — renders integration signs; uses compact `\iint`/`\iiint`/`\iiiint` for ≤4 bound-free variables, otherwise emits separate `\int` per limit with optional `\limits` in equation mode.
- Matrix operations (`_print_Adjoint`, `_print_Transpose`, `_print_MatPow`) conditionally wrap inner expressions in `\left(...\right)` based on whether the argument is a plain `MatrixSymbol` or a compound expression.
- `_print_MatMul` / `_print_HadamardProduct` — parenthesize operands that are sums or mixed-type products.

### [`mathml.py`](mathml.py)
`MathMLPrinter` — generates MathML XML markup using DOM, prioritizing content markup.

---

## Visualization & Display

### [`preview.py`](preview.py)
`preview()` — compiles LaTeX to PNG/DVI/PS/PDF via system LaTeX and displays with a viewer.
- Viewer selection/validation: auto-detects system viewers; handles deprecated `"StringIO"` viewer name (warns, converts to `"BytesIO"`); requires `outputbuffer` for BytesIO viewers.
- Supports `viewer="file"` for file output and `outputbuffer` for in-memory stream output.

### [`dot.py`](dot.py)
`dotprint()` — generates Graphviz DOT notation for expression tree visualization.

### [`tableform.py`](tableform.py)
`TableForm` — renders 2D data as aligned tables (ASCII, LaTeX, HTML).
- `TableForm.__init__` — normalizes ragged 2D data; `pad` kwarg controls fill for short rows and None entries. When `pad=None` (default), short rows are space-filled but explicit None entries are preserved as-is; when `pad` is given, both None and short-row gaps use that character.
- Supports column alignments (left/center/right), row/column headings ("automatic" or custom labels), per-column format strings or callables, and `wipe_zeros`.

### [`tree.py`](tree.py)
`tree()` / `print_tree()` — recursive text display of expression tree structure with assumption metadata.
- `pprint_nodes(subtrees)` — formats child strings with `+-` prefix; non-last children use `|` continuation lines, the last child uses spaces (visually distinguishes the final sibling).
- `print_node(node)` — emits class name, string form, and assumption properties; **skips properties whose value is None**.

---

## Other

### [`mathematica.py`](mathematica.py)
`MathematicaCodePrinter` — generates Mathematica code from expressions.

### [`gtk.py`](gtk.py)
GTK-based MathML viewer for expressions.

### [`theanocode.py`](theanocode.py)
`TheanoPrinter` — converts expressions to Theano computational graph variables.

### [`llvmjitcode.py`](llvmjitcode.py)
`LLVMJitPrinter` — JIT-compiles expressions to machine code via LLVM IR.
