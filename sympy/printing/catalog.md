# sympy/printing — Output & Code Generation

Converts SymPy expression trees into human-readable or machine-consumable string representations. Every printer subclasses `Printer` and dispatches via `_print_<ClassName>` methods resolved through the expression's MRO.

## Glossary

- **Printer dispatch**: The base `Printer._print()` walks `type(expr).__mro__` looking for `_print_<ClassName>` methods; the first match wins.
- **printmethod**: A string attribute on each `Printer` subclass (e.g. `"_sympystr"`, `"_latex"`). If an object defines a method with that name, the object can control its own printing.
- **CodePrinter**: Intermediate base class for all language-targeted code printers; adds loop generation, assignment handling, reserved-word escaping, and unsupported-expression tracking.

---

## Infrastructure

### `printer.py`
Base class for the entire printing subsystem.

- **`Printer`** — abstract base; provides `doprint()`, `_print()` dispatcher, `_as_ordered_terms()`, `set_global_settings()`, and the `order` property.

### `defaults.py`
Mixin that gives SymPy objects sensible `__str__` / `__repr__` via `sstr`.

- **`DefaultPrinting`** — sets `__str__ = __repr__ = sstr(self)`.

### `precedence.py`
Operator-precedence tables consumed by every printer to decide when parentheses are needed.

- `PRECEDENCE` — base precedence map (Lambda < Xor < Or < And < Relational < Add < Mul < Pow < Func < Not < Atom).
- `PRECEDENCE_VALUES` — per-class overrides.
- `PRECEDENCE_FUNCTIONS` — callables for types whose precedence depends on sign/value (Integer, Mul, Rational, Float, PolyElement, FracElement).
- `precedence(item)` — returns effective precedence for an object.
- `precedence_traditional(item)` — variant for LaTeX/pretty printing where Integral/Sum/Product/Limit/Derivative get `Mul`-level precedence.

### `conventions.py`
Shared helpers for symbol-name decomposition and partial-derivative detection.

- `split_super_sub(text)` — splits `"a_x^1"` into `('a', ['1'], ['x'])`.
- `requires_partial(expr)` — returns `True` when ∂ should be used instead of d.

### `codeprinter.py`
Abstract base for all target-language code printers.

- **`CodePrinter(StrPrinter)`** — extends `StrPrinter` with:
  - `doprint(expr, assign_to)` — top-level entry; wraps result with constant declarations and unsupported-expression comments.
  - `_doprint_loops` — generates nested loops for `Indexed` tensor expressions.
  - `_sort_optimized` / `_rate_index_position` — loop-order heuristics (subclass must implement).
  - `_print_Assignment` — handles `Piecewise` RHS, `MatrixSymbol` element-wise expansion, and `Indexed` contraction.
  - `_print_Function` — dispatches known functions; inlines `Lambda`-backed implementations.
  - `_print_Mul` — sign-aware numerator/denominator splitting.
  - Many `_print_*` aliases routed to `_print_not_supported`.
- **`AssignmentError`** — raised when loop accumulation lacks a target variable.

### `__init__.py`
Public re-exports: `pretty`, `pprint`, `latex`, `mathml`, `ccode`, `fcode`, `jscode`, `julia_code`, `mathematica_code`, `octave_code`, `preview`, `srepr`, `sstr`, `sstrrepr`, `print_tree`, `TableForm`, etc.

---

## String / Repr Printers

### `str.py`
Default human-readable string representation (the output of `str(expr)`).

- **`StrPrinter(Printer)`** — `printmethod = "_sympystr"`. ~70 `_print_*` methods covering all core types (Add, Mul, Pow, Rational, Float, Symbol, Matrix, Poly, Relational, Piecewise, sets, tensors, categories, differential geometry, etc.).
- `sstr(expr, **settings)` — convenience wrapper.
- **`StrReprPrinter(StrPrinter)`** — like `StrPrinter` but prints Python `str` values with `repr()` quoting.
- `sstrrepr(expr, **settings)` — convenience wrapper.

### `repr.py`
Generates strings satisfying `eval(srepr(expr)) == expr`.

- **`ReprPrinter(Printer)`** — `printmethod = "_sympyrepr"`. Prints `Integer(3)`, `Rational(1, 2)`, `Symbol('x', positive=True)`, etc.
- `srepr(expr, **settings)` — convenience wrapper.

### `python.py`
Produces executable Python code (variable declarations + expression).

- **`PythonPrinter(ReprPrinter, StrPrinter)`** — collects referenced symbols/functions; escapes Python keywords.
- `python(expr)` — returns a string like `x = Symbol('x')\ne = x**2 + 1`.
- `print_python(expr)`

---

## Pretty Printing (2D ASCII/Unicode)

### `pretty/__init__.py`
Re-exports from `pretty.py` and calls `pprint_try_use_unicode()` on import.

### `pretty/pretty.py`
The 2D ASCII-art / Unicode pretty-printer (~2100 lines).

- **`PrettyPrinter(Printer)`** — `printmethod = "_pretty"`. Returns `prettyForm` objects (2D string pictures). Covers:
  - Arithmetic (Add, Mul, Pow with stacked fractions and roots).
  - Calculus (Integral with ∫ glyph, Sum with Σ, Product with Π, Limit, Derivative with ∂/d).
  - Matrices (bordered box layout).
  - Piecewise, sets, sequences, logic, quantum operators, categories, and more.
- `pretty(expr, **settings)` / `pprint(expr)` / `pretty_print(expr)` — convenience wrappers.
- `pager_print(expr)` — pipes output through system pager.

### `pretty/pretty_symbology.py`
Unicode/ASCII glyph lookup tables and helpers.

- `pretty_use_unicode(flag)` / `pretty_try_use_unicode()` — global toggle.
- Greek-letter Unicode mapping (`greek_unicode`), sub/superscript digit tables, box-drawing and fraction characters.
- `pretty_symbol(symb_name)` — renders `"x_1^prime"` with Unicode sub/superscripts.
- `xsym`, `xobj`, `vobj`, `hobj` — vertical/horizontal box-drawing object constructors.
- `annotated(letter)` — wraps a letter with parenthetical annotation for products/sums.

### `pretty/stringpict.py`
2D string-picture algebra used by the pretty-printer.

- **`stringPict`** — an ASCII picture stored as equal-length lines with a baseline. Supports vertical stacking (`above`, `below`), horizontal concatenation (`right`, `left`, `next`), and rendering (`render`).
- **`prettyForm(stringPict)`** — adds operator-binding information so the pretty-printer can decide when to parenthesize. Provides `__add__`, `__mul__`, `__pow__`, `__div__` for composing sub-expressions.

---

## LaTeX

### `latex.py`
Full-featured LaTeX printer (~2070 lines).

- **`LatexPrinter(Printer)`** — `printmethod = "_latex"`. Extensive settings: `mode` (plain/inline/equation/equation*), `itex`, `fold_frac_powers`, `fold_func_brackets`, `fold_short_frac`, `long_frac_ratio`, `mul_symbol`, `inv_trig_style`, `mat_str`, `mat_delim`, `symbol_names`.
- Key capabilities:
  - Greek letters, accents, bold/calligraphic/fraktur modifiers via `translate()` and `modifier_dict`.
  - Fractions with smart short/long splitting.
  - `\sqrt`, `\int`, `\sum`, `\prod`, `\lim`, `\frac{d}{dx}` with partial-derivative detection.
  - Bessel functions, Airy functions, hypergeometric/Meijer-G, orthogonal polynomials, elliptic integrals.
  - Set notation (∪, ∩, ∖, ∈, ℕ, ℤ, ℝ, ℂ), intervals, sequences.
  - Matrix environments, transforms (Fourier, Laplace, Mellin), category-theory diagrams.
  - Polynomial rings, quotient rings/modules, free modules.
- `translate(s)` — maps symbol name strings to LaTeX (Greek, modifiers, etc.).
- `latex(expr, **settings)` / `print_latex(expr)` — convenience wrappers.

---

## MathML

### `mathml.py`
Produces MathML (Content markup) XML.

- **`MathMLPrinter(Printer)`** — `printmethod = "_mathml"`. Builds `xml.dom.minidom` DOM nodes.
  - `doprint(expr)` — returns XML string.
  - `mathml_tag(e)` — maps SymPy class names to MathML element names.
  - Handles Mul (with division), Add (with subtraction), Pow (roots), Derivative (partial), Integral, Sum, Relational, Symbol (with Unicode Greek + sub/superscript markup).
  - `apply_patch()` / `restore_patch()` — monkey-patches `minidom.toprettyxml` whitespace bug.
- `mathml(expr)` / `print_mathml(expr)` — convenience wrappers.

---

## Code Generation

### `ccode.py`
C99 code printer.

- **`CCodePrinter(CodePrinter)`** — maps SymPy functions to `<math.h>` names (`sin`, `cos`, `fabs`, `tgamma`, …). Handles `Pow` → `pow()`/`sqrt()`, `Piecewise` → `if/else` or ternary, `Indexed` → flat array indexing, `For` loops, `AugmentedAssignment`, pointer dereference.
- `known_functions` dict, `reserved_words` list.
- `indent_code(code)` — auto-indents based on `{`/`}`.
- `ccode(expr, assign_to, **settings)` / `print_ccode(expr)` — convenience wrappers.

### `fcode.py`
Fortran 77/90/95/2003/2008 code printer.

- **`FCodePrinter(CodePrinter)`** — supports fixed/free source format, Fortran line-wrapping at column 72, `cmplx()` for complex numbers, `merge()` for inline piecewise (F95+), column-major matrix traversal.
  - `_wrap_fortran(lines)` — splits long lines with continuation markers.
  - `indent_code(code)` — Fortran-aware indentation with `do`/`end do`, `if`/`end if`.
- `fcode(expr, assign_to, **settings)` / `print_fcode(expr)` — convenience wrappers.

### `jscode.py`
JavaScript code printer.

- **`JavascriptCodePrinter(CodePrinter)`** — maps to `Math.*` functions. `Pow` → `Math.pow()`/`Math.sqrt()`, `Piecewise` → `if/else` or ternary.
- `jscode(expr, assign_to, **settings)` / `print_jscode(expr)` — convenience wrappers.

### `julia.py`
Julia code printer.

- **`JuliaCodePrinter(CodePrinter)`** — element-wise operators (`.*`, `.^`, `./`) for non-numeric operands vs scalar operators for numbers/`MatrixSymbol`. Column-major matrix traversal. Supports large function set (Bessel, Airy, erf, special functions).
- `julia_code(expr, assign_to, **settings)` — convenience wrapper.

### `octave.py`
Octave/MATLAB code printer.

- **`OctaveCodePrinter(CodePrinter)`** — similar Hadamard operator logic as Julia. Maps SymPy functions to Octave names (e.g., `gammaln`, `sinint`, `dirac`). Handles `Piecewise` via logical masking (inline) or `if/elseif/else/end`.
- `octave_code(expr, assign_to, **settings)` — convenience wrapper.

### `mathematica.py`
Wolfram Mathematica code printer.

- **`MCodePrinter(CodePrinter)`** — `Pow` → `^`, functions use `[…]` syntax (`Sin[x]`, `Exp[x]`), lists as `{…}`. Integral/Sum wrapped in `Hold[…]`.
- `mathematica_code(expr, **settings)` — convenience wrapper.

---

## Lambda / NumPy / NumExpr Printers

### `lambdarepr.py`
Printers for `lambdify` and array-oriented backends.

- **`LambdaPrinter(StrPrinter)`** — produces Python expressions suitable for `eval()`/`lambdify`. Handles `Piecewise` as nested ternary, `Sum` as `builtins.sum(… for …)`, boolean operators as Python `and`/`or`/`not`.
- **`NumPyPrinter(LambdaPrinter)`** — vectorized: `Piecewise` → `select()`, relationals → `equal()`/`less()` etc., `And`/`Or`/`Not` → `logical_and`/`logical_or`/`logical_not`, `MatMul` → `.dot()`, `Min`/`Max` → `amin`/`amax`.
- **`NumExprPrinter(LambdaPrinter)`** — wraps output in `evaluate('…', truediv=True)`. Only supports functions in `_numexpr_functions` dict; blacklists matrices and containers.
- `lambdarepr(expr)` — convenience wrapper.

---

## LLVM JIT Compilation

### `llvmjitcode.py`
Compiles SymPy expressions to native code via LLVM IR (requires `llvmlite`).

- **`LLVMJitPrinter(Printer)`** — emits LLVM IR instructions (`fadd`, `fmul`, `fdiv`, calls to `pow`/`sqrt`/math library).
- **`LLVMJitCallbackPrinter(LLVMJitPrinter)`** — variant for array-parameter callbacks (e.g. `scipy.integrate`).
- **`LLVMJitCode`** — orchestrates module/function creation, parameter mapping, IR generation, and LLVM compilation (`parse_assembly` → optimization passes → `create_mcjit_compiler`).
- **`LLVMJitCodeCallback(LLVMJitCode)`** — callback variant writing results into output arrays.
- **`CodeSignature`** — describes C-level function signature (return type, arg ctypes, input/output arg indices).
- `llvm_callable(args, expr, callback_type=None)` — public API; returns a `ctypes.CFUNCTYPE` callable. Supports `'scipy.integrate'` and `'cubature'` callback signatures, and CSE-optimized expression tuples.

---

## Theano Backend

### `theanocode.py`
Converts SymPy expressions to Theano computation graphs (requires `theano`).

- **`TheanoPrinter(Printer)`** — maps SymPy types to Theano ops via `mapping` dict (~40 entries covering arithmetic, trig, logic, matrices). Caches tensor variables for symbols.
  - Supports `MatMul` → `tt.dot`, `BlockMatrix` → `tt.join`, `Piecewise` → `tt.switch`, `Derivative` → `tt.Rop`.
- `theano_code(expr, cache, **kwargs)` — returns a Theano variable.
- `dim_handling(inputs, dim, dims, broadcastables)` — sets up broadcastable dimensions.
- `theano_function(inputs, outputs, dtypes, cache, **kwargs)` — builds and returns a compiled `theano.function`.

---

## Visualization & Miscellaneous

### `dot.py`
Graphviz DOT-language printer for expression trees.

- `purestr(x)` — canonical `Type(args…)` string.
- `styleof(expr, styles)` — merge CSS-like style dicts by type.
- `dotnode(expr, …)` / `dotedges(expr, …)` — emit node/edge declarations.
- `dotprint(expr, styles, atom, maxdepth, repeat, labelfunc)` — returns a complete DOT `digraph{…}` string.

### `gtk.py`
Renders MathML in a GtkMathView widget.

- `print_gtk(x, start_viewer=True)` — writes MathML to a temp file and launches `mathmlviewer`.

### `preview.py`
Renders expressions as PNG/DVI/PS/PDF/SVG via LaTeX compilation.

- `preview(expr, output, viewer, euler, packages, filename, outputbuffer, preamble, dvioptions, outputTexFile, **latex_settings)` — compiles LaTeX → DVI → target format. Viewer auto-detection; supports `pyglet`, `BytesIO`, `"file"` output, and custom preambles.

### `tree.py`
Text-based tree view of expression structure with assumptions.

- `pprint_nodes(subtrees)` — indented `+-` tree formatter.
- `print_node(node)` — class name + str + assumptions dump.
- `tree(node)` — recursive tree string.
- `print_tree(node)` — prints `tree(node)`.

### `tableform.py`
Tabular data display with customizable alignment, headings, and formatting.

- **`TableForm`** — accepts 2D data (or `Matrix`); configurable headings (`"automatic"` or lists), column alignments (left/right/center), format functions, zero-wiping, padding for ragged rows.
  - `as_matrix()` — returns a `Matrix`.
  - `as_latex()` — LaTeX `tabular` environment.
  - `_sympystr(p)` — ASCII table with column-width alignment.
  - `_latex(printer)` — LaTeX rendering hook.
