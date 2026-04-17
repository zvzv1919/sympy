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
- `_print_Mul` — decomposes products into numerator/denominator lists; treats exponent −1 differently from other negative rational exponents (evaluate vs evaluate=False).
- `_print_Product` — builds the iterated product (∏) sign as 2D box art; computes sign width from function height.
- `_print_Sum` — builds the summation (∑) sign with upper/lower limits.
- `_print_Integral` — builds integral signs with limits and spacing.
- Handles matrices, piecewise, sequences, sets, relational operators, and all standard math expressions.

### [`pretty/pretty_symbology.py`](pretty/pretty_symbology.py)
Symbol/character primitives and Unicode↔ASCII abstraction layer. This is **not** a printer — it provides building blocks that `pretty.py` consumes.
- `xobj(symb, length)` — constructs multi-line spatial objects (brackets, braces, integral signs) of a given height; handles even-height adjustment for centered middle pieces (e.g., curly braces).
- `pretty_atom(atom_name, default=None)` — returns pretty representation of named atoms (pi, infinity, etc.); raises `KeyError('only unicode')` in ASCII mode when no default is provided.
- `pretty_symbol(symb_name)` — translates symbol names to Unicode with Greek letters, sub/superscripts.
- `xsym(sym)` — resolves operator characters (comparison `<=`/`>=`/`!=`, arithmetic `*`/`.`, arrows `-->`/`==>`, assignment `:=`/`+=`) to Unicode or ASCII display form via `_xsym` lookup table.
- `vobj(symb, height)` / `hobj(symb, width)` — vertical/horizontal object constructors.
- Key data: `atoms_table` (atom→Unicode mapping), `_xobj_unicode`/`_xobj_ascii` (bracket/delimiter glyph tables).

### [`pretty/stringpict.py`](pretty/stringpict.py)
`stringPict` — 2D string canvas with baseline tracking. Subclass `prettyForm` adds binding strength for precedence-aware parenthesization.
- Spatial combinators: `above`, `below`, `left`, `right`, `stack` — arrange sub-pictures relative to each other.
- `parens(left, right, ifascii_nougly)` — wraps picture in parentheses; in ASCII mode with `ifascii_nougly=True`, collapses height to 1 to avoid ugly tall brackets.

### [`pretty/__init__.py`](pretty/__init__.py)
Public API: `pretty`, `pretty_print`/`pprint`, `pretty_use_unicode`.

---

## String / Code Printers

### [`str.py`](str.py)
`StrPrinter` — generates readable **1D flat-text** string representations with precedence-based parenthesization. No 2D layout or spatial arrangement.

### [`codeprinter.py`](codeprinter.py)
`CodePrinter` base class for code-generating printers. Extends `StrPrinter` with `doprint(assign_to)` for assignment statements and formatting hooks.

### [`ccode.py`](ccode.py)
`CCodePrinter` — generates C code, mapping SymPy functions to C math library equivalents.

### [`fcode.py`](fcode.py)
`FCodePrinter` — generates Fortran code with language-specific operators and formatting (source format, precision, contraction).

### [`jscode.py`](jscode.py)
`JavascriptCodePrinter` — generates JavaScript code from expressions.

### [`julia.py`](julia.py)
`JuliaCodePrinter` — generates Julia code from expressions.
- Distinguishes element-wise (`.^`, `./`, `.*`) vs scalar (`^`, `/`, `*`) operators based on whether operands are numeric.
- `_print_Pow` — special-cases exponents ½, −½, −1 with `sqrt` and appropriate division operators.

### [`octave.py`](octave.py)
`OctaveCodePrinter` — generates Octave/MATLAB code from expressions.
- Similar element-wise vs scalar operator distinction as Julia printer.

### [`lambdarepr.py`](lambdarepr.py)
`LambdaPrinter` — generates Python lambda-compatible string representations for use with `lambdify`.

### [`python.py`](python.py)
`PythonPrinter` — generates executable Python code strings.

---

## Markup Printers

### [`latex.py`](latex.py)
`LatexPrinter` — converts expressions to LaTeX markup strings (e.g., `\frac{x}{y}`). Produces **1D markup text**, not spatial/visual rendering.

### [`mathml.py`](mathml.py)
`MathMLPrinter` — generates MathML XML markup using DOM, prioritizing content markup.

---

## Visualization & Display

### [`preview.py`](preview.py)
`preview()` — compiles LaTeX to PNG/DVI/PS/PDF via system LaTeX and displays with a viewer.

### [`dot.py`](dot.py)
`dotprint()` — generates Graphviz DOT notation for expression tree visualization.

### [`tableform.py`](tableform.py)
`TableForm` — renders 2D data as aligned tables (ASCII, LaTeX, HTML).

### [`tree.py`](tree.py)
`tree()` / `print_tree()` — recursive text display of expression tree structure.

---

## Other

### [`repr.py`](repr.py)
`ReprPrinter` — generates eval-able `repr()` strings for expressions.

### [`mathematica.py`](mathematica.py)
`MathematicaCodePrinter` — generates Mathematica code from expressions.

### [`gtk.py`](gtk.py)
GTK-based MathML viewer for expressions.

### [`theanocode.py`](theanocode.py)
`TheanoPrinter` — converts expressions to Theano computational graph variables.

### [`llvmjitcode.py`](llvmjitcode.py)
`LLVMJitPrinter` — JIT-compiles expressions to machine code via LLVM IR.
