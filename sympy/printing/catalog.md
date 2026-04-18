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
- `__init__` — merges configuration in three layers: (1) copies subclass `_default_settings`, (2) overlays `_global_settings` but only for keys already in defaults (unknown global keys silently ignored), (3) overlays caller-provided `settings` dict and raises `TypeError` for any unrecognized key. This asymmetry is intentional.
- `_print` walks the expression's MRO looking for `_print_<ClassName>` handlers; if none found, falls back to `self.emptyPrinter(expr)` (defaults to `str(expr)`).

### [`precedence.py`](precedence.py)
Operator precedence values (`PRECEDENCE` dict) and lookup functions that return a numeric precedence level for an expression. Does **not** make rendering-specific bracket decisions — those live in each printer (e.g., `LatexPrinter._needs_mul_brackets`).

### [`conventions.py`](conventions.py)
- `split_super_sub()` — parses symbol names into base + superscripts (`^`/`__`) + subscripts (`_`); trailing digits on the base name are auto-split into a leading subscript (e.g., `alpha11` → name `alpha`, sub `11`).
- `requires_partial()` — determines if partial derivative symbol (∂) is needed by counting non-integer free symbols; returns True when more than one exists. Falls back to checking the expression's variable list when free_symbols is not iterable.

### [`defaults.py`](defaults.py)
`DefaultPrinting` mixin providing `__str__`/`__repr__` using `sstr()`.

---

## Pretty Printing (2D ASCII/Unicode)

### [`pretty/pretty.py`](pretty/pretty.py)
`PrettyPrinter` — renders expressions as 2D visual text art (ASCII/Unicode box drawing). Contains all expression-specific `_print_*` handlers that **orchestrate layout** by composing `stringPict` objects with symbols from `pretty_symbology`.
- `_print_Mul` — decomposes a product into numerator/denominator factor lists and renders as a 2D stacked fraction with a horizontal bar; inserts `1` when numerator is empty.
  - Scope limited to fraction decomposition and layout — does **not** handle inline multiplication signs, negative-one substitution, or sign-collision avoidance.
  - Uses `evaluate=False` for non-`-1` negative exponents to suppress auto-simplification when negating the exponent for the denominator.
- `_print_MatrixElement` — renders matrix element access; when parent is a `MatrixSymbol` with numeric indices, produces a subscripted symbol (e.g., `A₁₂`); otherwise uses function-like bracket notation `A[i, j]`.
- `_print_MatrixSlice` — renders sub-range access as 2D pretty-printed layout with slice simplification (same rules as `StrPrinter`).
- `_print_Product` — builds the iterated product (∏) sign as 2D box art; computes sign width from function height.
- `_print_Sum` — builds the summation (∑) sign with upper/lower limits.
- `_print_Integral` — builds integral signs with limits and spacing; limit tuple length determines rendering: 2-element tuple → empty lower bound with only upper bound shown; 3-element tuple → both bounds shown.
- Special-case `_print_*` overrides for functions whose names collide with Greek/Unicode symbols (e.g., `_print_Chi` keeps Latin "Chi" instead of Greek χ, `_print_gamma`/`_print_lowergamma`/`_print_uppergamma` use explicit Γ/γ glyphs).
- `_print_Function` — renders applied callables; attaches the formatted name and argument list as attributes on the result form so they can be reassembled when exponentiation is applied.
- `_print_Subs` — renders evaluation-at-a-point notation: parenthesizes the expression, draws a vertical bar (`|`) to its right, and places variable=value assignment pairs as subscripts below the bar.
- `_print_DMP` / `_print_DMF` — renders dense multivariate polynomials; if `ring` is set, attempts `ring.to_sympy(p)` conversion — on `SympifyError`, falls back to `repr(p)`.
- Algebraic domain printers (`_print_RealField`, `_print_ComplexField`, `_print_FiniteField`, `_print_IntegerRing`, `_print_RationalField`) — render number domains as Unicode double-struck letters (ℤ, ℚ, ℝ, ℂ) or ASCII fallbacks (ZZ, QQ, RR, CC); non-default precision is appended as a subscript.
- `_print_Range` — renders discrete integer ranges; abbreviates with ellipsis (`…`) when the range has more than 4 elements or is infinite, showing only endpoints and step; shows all elements otherwise.
- `_print_matrix_contents` — grid layout for `MatrixBase` rendering: computes per-column max width, horizontally centers each cell with right-bias on odd padding (`wleft = delta//2`, `wright = delta - wleft`), vertical alignment is implicit via baselines; returns empty `prettyForm('')` for 0×0 matrices. Used by `_print_MatrixBase` which wraps result in brackets.
- Handles matrices, piecewise, sequences, sets, relational operators, containers (tuple, list, dict, set), and all standard math expressions. Sign insertion (`+`/`-`) between addition terms is delegated to `prettyForm.__add__` in `stringpict.py`.
- `_print_tuple` — single-element tuples append a trailing comma before parenthesizing, to distinguish from a mere parenthesized expression.
- `_print_Float` — when `full_prec` setting is `"auto"`, shows full precision only at the top print level (`_print_level == 1`); nested floats use reduced precision.

### [`pretty/pretty_symbology.py`](pretty/pretty_symbology.py)
Symbol/character primitives and Unicode↔ASCII abstraction layer. This is **not** a printer — it provides building blocks that `pretty.py` consumes.
- `xobj(symb, length)` — constructs multi-line spatial objects (brackets, braces, integral signs) of a given height; when a glyph has a defined center piece (e.g., curly braces), silently increments even lengths to odd to maintain center symmetry.
- `pretty_atom(atom_name, default=None)` — returns pretty representation of named atoms (pi, infinity, etc.); raises `KeyError('only unicode')` in ASCII mode when no default is provided.
- `pretty_symbol(symb_name)` — translates symbol names to Unicode glyphs (Greek letters, sub/superscripts). Delegates name decomposition (base/super/sub parsing) to `conventions.split_super_sub`. If any superscript character fails Unicode mapping, **both** super- and subscript prettification are abandoned and the name falls back to underscore-delimited ASCII. Passive lookup only — rendering overrides live in `pretty.py`'s `_print_*` methods.
- `xsym(sym)` — resolves operator characters (comparison `<=`/`>=`/`!=`, arithmetic `*`/`.`, arrows `-->`/`==>`, assignment `:=`/`+=`) to Unicode or ASCII display form via `_xsym` lookup table.
- `xstr(*args)` — string-type dispatcher: calls `unicode()` when unicode mode is on, `str()` when off. Governed by module-level `_use_unicode` flag.
- `pretty_use_unicode(flag)` — gets/sets the module-level `_use_unicode` flag controlling Unicode vs ASCII output mode.
- `vobj(symb, height)` / `hobj(symb, width)` — sole builders of multi-line delimiter glyphs; delegate to `xobj` for glyph assembly.
- Key data: `atoms_table` (atom→Unicode mapping), `_xobj_unicode`/`_xobj_ascii` (bracket/delimiter glyph tables).

### [`pretty/stringpict.py`](pretty/stringpict.py)
`stringPict` — 2D string canvas with baseline tracking. Subclass `prettyForm` adds binding strength for precedence-aware parenthesization.
- `next(*args)` — core static method for horizontal composition: computes new unified baseline and height across all blocks, pads each block with empty lines above/below to align baselines, then joins corresponding rows. All other horizontal combinators (`left`, `right`) delegate to this.
- Spatial combinators: `above`, `below`, `left`, `right`, `stack` — arrange sub-pictures relative to each other. `stack` accepts a special `LINE` sentinel that is replaced with a horizontal dash row spanning the maximum width of all composed elements.
- `parens(left, right, ifascii_nougly)` — wraps picture in pre-built delimiter glyphs (does not construct glyph shapes); in ASCII mode with `ifascii_nougly=True`, collapses height to 1 to avoid ugly tall brackets.
- `render()` — converts picture to display string; splits output exceeding terminal width into column-width segments. Multi-line pictures get blank-line spacers between segments; single-line pictures do not.
- `terminal_width()` — detects console column count; uses `curses.tigetnum` on Unix, falls back to Windows `kernel32.GetConsoleScreenBufferInfo` via ctypes on Windows.
- `prettyForm.__div__` — constructs stacked fractions via `stack(num, LINE, den)`; parenthesizes nested divisions. For negative numerators (NEG binding), pads the numerator with a trailing space to preserve visual alignment under the fraction bar.
- `prettyForm.__add__` — binding-aware addition; reuses existing minus signs to simplify `+ -x` forms.
- `prettyForm.__pow__` — renders exponentiation as 2D layout; when the base is a function (FUNC binding), uses a height heuristic: single-line exponents go inline above the function name, multi-line exponents wrap the base in parentheses.
- `prettyForm.__mul__` — assembles the inline visual representation of a product sequence: inserts multiplication symbols between factors, applies precedence-based parenthesization.
  - Detects `-1` factors and substitutes `-1 * x` → `-x`; inserts a space when consecutive leading minus signs would create visual ambiguity (dash collision).

### [`pretty/__init__.py`](pretty/__init__.py)
Public API: `pretty`, `pretty_print`/`pprint`, `pretty_use_unicode`.

---

## String / Code Printers

### [`str.py`](str.py)
`StrPrinter` — the default human-readable text formatter (`str(expr)`, `print()`). Generates **1D flat-text** string representations with precedence-based parenthesization. No 2D layout, fraction bars, or spatial arrangement.
- `_print_Add` — determines sign of each summand by checking if its printed string starts with `'-'`; strips leading `'-'` and rebuilds with `+`/`-` tokens; omits leading `+` for the first term.
- `_print_Mul` — splits factors into numerator/denominator lists; formats as `a*b/c` (single denominator, no parens) or `a*b/(c*d)` (multiple denominators wrapped in parentheses).
  - Uses `evaluate=False` for non-`-1` negative exponents when negating for the denominator; allows evaluation when exponent is exactly `-1` (negation yields 1, collapsing to base).
- `_print_MatrixSlice` — renders matrix sub-range access as flat 1D `A[start:stop:step, ...]` text; simplifies by omitting unit step, collapsing single-element ranges, and dropping zero start index.
- `_print_DMP` / `_print_DMF` — renders dense polynomials; if `ring` is set, attempts `ring.to_sympy(p)` conversion — on `SympifyError`, falls back to raw `ClassName(rep, dom, ring)` format.

### [`codeprinter.py`](codeprinter.py)
`CodePrinter` base class for code-generating printers. Extends `StrPrinter` with internal `doprint(assign_to)` for assignment-statement rendering; not called directly — language-specific convenience functions (e.g., `jscode()`, `fcode()`, `julia_code()`) are the public entry points.
- `_format_code(lines)` and `indent_code(code)` are **abstract stubs** (`NotImplementedError`); actual formatting/indentation logic lives in language-specific subclasses (e.g., `julia.py`, `fcode.py`).
- `_print_Mul` — splits factors into numerator/denominator lists based on negative rational exponents; renders as flat 1D text `a*b/c` or `a*b/(c*d)` (no 2D fraction bars).
- Logical operators (`_print_And`, `_print_Or`, `_print_Not`, `_print_Xor`, `_print_Equivalent`) — look up symbols in `self._operators` dict; And/Or/Not assume the operator always exists, but Xor and Equivalent check for `None` and fall back to `_print_not_supported` if the target language has no mapping.

### [`ccode.py`](ccode.py)
`CCodePrinter` — generates C code, mapping SymPy functions to C math library equivalents.
- `_print_Pow` — special-cases: exp==-1 → `1.0/x`, exp==0.5 → `sqrt(x)`, otherwise `pow(x, y)`.
- `_print_Rational` — emits long-double literals (`p.0L/q.0L`).
- `_print_Indexed` — flattens multi-dimensional array access into a single linear index using row-major (C-style) linearization.

### [`fcode.py`](fcode.py)
`FCodePrinter` — generates Fortran code with language-specific operators and formatting (source format, precision, contraction).
- `fcode()` — top-level API; accepts `assign_to`, `precision`, `source_format` ('fixed'/'free'), `standard` (66/77/90/95/2003/2008), `human`, `contract`, and `user_functions`.
- Column-major matrix traversal and 1-based loop index adjustment (adds 1 to both lower and upper bounds).
- `_print_Add` — separates terms into pure-real, pure-imaginary, and mixed; when imaginary parts exist, wraps real+imaginary in `cmplx(re, im)` and appends the mixed (symbolic) portion with sign detection (leading `-` check).
- `indent_code` — auto-indents by nesting level; in free format, adds extra padding for continuation lines; in fixed format, pads leading columns and wraps via `_wrap_fortran`.
- `_wrap_fortran` — enforces Fortran fixed-format line length (72 chars) by wrapping long lines with continuation markers.
- Loop syntax: `do VAR = start, stop` / `end do`.

### [`jscode.py`](jscode.py)
`JavascriptCodePrinter` — generates JavaScript (browser-side scripting language) code from expressions, mapping SymPy functions to `Math.*` equivalents.
- `_print_Pow` — special-cases: exp==-1 → `1/x`, exp==0.5 → `Math.sqrt(x)`, otherwise `Math.pow(x, y)`.
- `jscode(expr)` — main public entry point; instantiates `JavascriptCodePrinter` and delegates via `doprint`. Accepts `assign_to`, `precision`, `human`, `contract`, and `user_functions`.
- `human=False` returns a tuple `(symbols_to_declare, not_supported_functions, code_text)` instead of a single string.
- Piecewise expressions emit if/else blocks when `assign_to` is given, ternary operators otherwise; requires a default `(expr, True)` branch.
- `indent_code` — auto-indents generated code using regex-matched block keywords (`if`/`else`/`function`/`for`/`while`/`end`); empty or newline-only lines are passed through without indentation.

### [`julia.py`](julia.py)
`JuliaCodePrinter` — generates Julia code from expressions for a scientific computing language.
- Column-major matrix traversal and 1-based loop index adjustment (adds 1 to both lower and upper bounds). Loop syntax: `for VAR = start:stop` / `end`.
- Emits element-wise dot operators (`.^`, `./`, `.*`) **by default** for regular `Symbol` operands to support vectorized code; uses standard operators (`^`, `/`, `*`) only for pure numbers or `MatrixSymbol` operands.
- `julia_code()` — top-level API; returns Julia-syntax string with dot-operator rules, assignment support, and custom function dispatch.
- `_print_Assignment` — overrides base: when inline=False and RHS is Piecewise, decomposes into per-branch assignments and re-wraps as a new Piecewise for multi-line `if/elseif` output.
- `_print_Pow` — special-cases exponents ½, −½, −1 with `sqrt` and scalar (`/`) vs element-wise (`./`) division based on whether the base is numeric.
- `_print_Piecewise` — dual-mode conditional output: inline emits nested ternary `(cond) ? (expr) :` chains; block mode emits `if/elseif/else/end`. Requires last branch to have a True guard.
- `indent_code` — auto-indents generated code using regex-matched block keywords; lines that both close and open blocks (e.g., `elseif`, `else`) decrease indent before the line and increase after.

### [`octave.py`](octave.py)
`OctaveCodePrinter` — generates Octave/MATLAB code from expressions for an array-based numerical computing environment with 1-based indexing.
- Column-major matrix traversal and 1-based loop index adjustment (adds 1 to both lower and upper bounds). Loop syntax: `for VAR = start:stop` / `end`.
- Emits element-wise dot operators (`.^`, `./`, `.*`) for symbolic operands; uses standard operators (`^`, `/`, `*`) only for pure numeric operands. Matrix types (`MatPow`) always use standard `^`.
- `octave_code()` — top-level API; returns Octave/Matlab-syntax string. Accepts `assign_to`, `precision`, `human`, `contract`, `inline`, and `user_functions`.
- `_print_Mul` — decides between scalar (`*`, `/`) and element-wise (`.*`, `./`) operators based on whether each operand is a pure number; handles imaginary-number shorthand.
- `_print_Pow` — chooses `.^` (element-wise) or `^` (scalar) based on whether all args are numbers; special-cases exponents ½, −½, −1 with `sqrt` and element-wise vs scalar division.
- `_print_MatrixBase` — renders 2D arrays with shape-dependent formatting: 0×0 → empty literal, zero-row or zero-col → `zeros(r,c)`, 1×1 → scalar, row vectors use space-separated syntax, column vectors use semicolon-separated syntax.
- `_print_Piecewise` — dual-mode conditional output: inline emits nested element-wise multiply `(cond).*(expr) + (~cond).*(...)`; block mode emits `if/elseif/else/end`. Raises `ValueError` if no default `(expr, True)` branch is provided.

### [`repr.py`](repr.py)
`ReprPrinter` — the executable-code output formatter; generates eval-able `repr()` strings (`srepr`) for round-trip fidelity: `eval(srepr(expr)) == expr`. Not a language code-generator — produces Python constructor calls.
- `_print_Symbol` — includes declared assumption properties (e.g., `positive=True`, `commutative=False`) in output so symbols round-trip with their metadata.
- `_print_MatrixBase` — emits constructor-call strings for matrices; special-cases matrices with zero rows XOR zero cols (emits explicit dimension args with empty list).

### [`lambdarepr.py`](lambdarepr.py)
`LambdaPrinter` — generates Python lambda-compatible string representations for use with `lambdify`.
- `_print_Piecewise` — converts to nested ternary expressions (`(e1) if (c1) else (e2) if (c2) else None`); final fallback is `None`.
- `NumPyPrinter` subclass — vectorized NumPy output; prints sequences as tuples (for numba nopython compatibility).
  - `_print_DotProduct` — emits `dot(a, b)` with automatic transpose to ensure 1×n by n×1 orientation when vector shapes don't match.
  - `_print_MatMul` — chains `.dot()` calls for matrix multiplication.
- `NumExprPrinter` subclass — generates string expressions for `numexpr.evaluate()` (string-based evaluation, not namespace-based).
  - `_numexpr_functions` dict maps SymPy function names to numexpr equivalents; unmapped functions checked for `_imp_` fallback before raising `TypeError`.
  - Blacklists all Matrix types.

### [`python.py`](python.py)
`PythonPrinter` — generates executable Python code strings.

---

## Markup Printers

### [`latex.py`](latex.py)
`LatexPrinter` — converts expressions to LaTeX markup strings (e.g., `\frac{x}{y}`). Produces **1D markup text**, not spatial/visual rendering.
- `latex()` — top-level API function; accepts formatting options including `long_frac_ratio` (default 2) which sets the numerator-to-denominator width ratio threshold before wide fractions are broken apart into separate factors.
- `__init__` — mode-dependent defaults: `mode='inline'` auto-enables `fold_short_frac` (compact `a/b` instead of `\frac{a}{b}` for simple fractions).
- Configures `mul_symbol_latex` and `mul_symbol_latex_numbers` (numeric factors default to centered dot even when general symbol is a space).
- `_print_Add` — iterates ordered terms; when a term has a negative leading coefficient, emits ` - ` and negates the term to produce clean `a - b` instead of `a + -b`. Wraps terms in `\left(...\right)` when they are relational expressions (via `_needs_add_brackets`).
- `_print_Mul` — renders products; uses two distinct separator settings: `mul_symbol_latex` between general factors and `mul_symbol_latex_numbers` between adjacent numeric factors (detected via regex on rendered terms). Renders fractions via `\frac{}{}` when a denominator is present.
- `_needs_mul_brackets` — decides whether an expression needs parentheses inside a Mul; position-sensitive: container objects (Integral, Sum, Product, Piecewise) need brackets only when **not** the last factor.
- `_print_Integral` — renders integration signs; uses compact `\iint`/`\iiint`/`\iiiint` for ≤4 bound-free variables, otherwise emits separate `\int` per limit with optional `\limits` in equation mode.
- Polynomial domain printers (`_print_Poly`, `_print_ComplexRootOf`, `_print_RootSum`, `_print_PolynomialRing`, `_print_FractionField`) — renders algebraic objects; `_print_ComplexRootOf` shortens the class name to `CRootOf` for display.
- Special function renderers (`_print_hyper`, `_print_meijerg`, Airy, Bessel, orthogonal polynomials, zeta, polylog, etc.) — each maps to standard LaTeX notation; Meijer G uses a `\begin{matrix}` layout for its four parameter groups with `\middle|` separator for the argument.
- Matrix operations (`_print_Adjoint`, `_print_Transpose`, `_print_MatPow`) conditionally wrap inner expressions in `\left(...\right)` based on whether the argument is a plain `MatrixSymbol` or a compound expression.
- `_print_MatMul` / `_print_HadamardProduct` — parenthesize operands that are sums or mixed-type products.
- `translate(s)` — module-level helper converting symbol names to LaTeX: looks up Greek letters, then recursively strips accent/modifier suffixes (hat, dot, prime, etc.) longest-first; guards against empty base by requiring remaining string length > modifier length.

### [`mathml.py`](mathml.py)
`MathMLPrinter` — generates MathML XML markup using DOM, prioritizing content markup.
- `mathml_tag(e)` — resolves an expression to its MathML tag name: walks the class MRO against a translate dict; if no ancestor matches, falls back to lowercasing the class name.

---

## Visualization & Display

### [`preview.py`](preview.py)
`preview()` — compiles LaTeX to PNG/DVI/PS/PDF via system LaTeX and displays with a viewer.
- Viewer selection/validation: auto-detects system viewers; handles deprecated `"StringIO"` viewer name (warns, converts to `"BytesIO"`); requires `outputbuffer` for BytesIO viewers.
- Supports `viewer="file"` for file output and `outputbuffer` for in-memory stream output.

### [`dot.py`](dot.py)
`dotprint()` — generates Graphviz DOT notation for expression tree visualization.

### [`tableform.py`](tableform.py)
`TableForm` — renders user-supplied 2D data (lists of lists) as aligned tables (ASCII, LaTeX, HTML); not used for `Matrix`/`MatrixBase` pretty-printing (that is `PrettyPrinter._print_matrix_contents`).
- `TableForm.__init__` — normalizes ragged 2D data; `pad` kwarg controls fill for short rows and None entries. When `pad=None` (default), short rows are space-filled but explicit None entries are preserved as-is; when `pad` is given, both None and short-row gaps use that character.
- Supports column alignments (left/center/right via string aliases `'l'`/`'r'`/`'c'`/`'<'`/`'>'`/`'^'`), row/column headings ("automatic" or custom labels), per-column format strings or callables, and `wipe_zeros`.
- Has its own `_latex` method that renders the table as a LaTeX `tabular` environment; when a per-column callable returns `None`, falls back to `printer._print(x)` for that cell.
- If row headings are present and alignment count == data columns + 1, the first alignment is peeled off as the row heading alignment; otherwise row heading defaults to right-justified.

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
`TheanoPrinter` — converts expressions to Theano computational graph variables (`tt.TensorVariable`).
- `_print_Piecewise` — converts piecewise to nested `tt.switch`; single-branch fallback is `np.nan`.
- `_print_Derivative` — symbolic differentiation via `tt.Rop`.
- `theano_function(inputs, outputs)` — end-to-end: builds Theano function from SymPy expressions with dimension/broadcasting handling.

### [`llvmjitcode.py`](llvmjitcode.py)
`LLVMJitPrinter` — JIT-compiles expressions to machine code via LLVM IR using llvmlite.
- `_print_Pow` — optimizes common exponent special cases for native code: exp==-1 → fdiv, exp==0.5 → sqrt intrinsic, exp==2 → fmul self-multiply; general case calls external `pow`.
- `LLVMJitCode` — manages LLVM module/engine lifecycle; wraps compiled IR into callable `ctypes` function pointers.
