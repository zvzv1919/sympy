# sympy/printing — Catalog

> Part of [SymPy](../catalog.md). Output formatting: LaTeX, MathML, pretty-print, C/Fortran/JS/Julia/Rust/GLSL code generation, and more.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package entry point; re-exports public printing functions (`latex`, `pretty`, `ccode`, `fcode`, `mathematica_code`, etc.) from submodules. |
| `printer.py` | Defines the `Printer` base class that all printers inherit from, implementing the dispatch mechanism that resolves `_print_<ClassName>` methods by MRO. |
| `defaults.py` | Provides the `DefaultPrinting` mixin that gives SymPy objects readable `__str__` and `__repr__` via `sstr`. |
| `conventions.py` | Shared helper utilities for printers: `split_super_sub` parses a single symbol's *name string* (e.g., `"x_i__j"`) into its name/superscript/subscript text components for display, and `requires_partial` detects when partial-derivative notation is needed. Does not handle the rendering or formatting of mathematical objects like deltas or tensors. |
| `precedence.py` | Defines operator-precedence constants (`PRECEDENCE` dict) and the `precedence()` function used by printers to decide when parentheses are required. |
| `codeprinter.py` | `CodePrinter` base class for all code-generation printers (C, Fortran, JS, etc.), extending `StrPrinter` with assignment handling, loop generation, and the `@requires` decorator. |
| `str.py` | `StrPrinter` — the default human-readable string printer; provides `sstr()` and `sstrrepr()` convenience functions. |
| `repr.py` | `ReprPrinter` — produces `srepr()` output where `eval(srepr(expr)) == expr` holds, giving a round-trippable representation of expressions. |
| `python.py` | `PythonPrinter` — generates executable Python code strings (with `Symbol`/`Function` import preambles) via the `python()` function. |
| `pycode.py` | `PythonCodePrinter`, `MpmathPrinter`, `NumPyPrinter`, and `SciPyPrinter` — emit Python code targeting the standard math library, mpmath, NumPy, or SciPy respectively. |
| `latex.py` | `LatexPrinter` — converts SymPy expressions to LaTeX markup; provides the `latex()` convenience function. Contains hundreds of `_print_<Type>` methods with per-type formatting logic for rendering specific mathematical objects (e.g., KroneckerDelta, DiracDelta, LeviCivita, Piecewise, integrals, sums, matrices, sets). These methods control fine-grained typesetting decisions such as subscript/superscript layout, delimiter choice, and index separator style. |
| `mathml.py` | `MathMLPrinter` — renders expressions as MathML (Content markup) XML; provides `mathml()` and `print_mathml()`. |
| `ccode.py` | `C89CodePrinter` and `C99CodePrinter` — generate C89/C99 source code from expressions, mapping SymPy functions to their `math.h` equivalents. |
| `cxxcode.py` | `CXX98CodePrinter`, `CXX11CodePrinter`, and `CXX17CodePrinter` — generate C++ code, extending the C printers with C++ standard-library math functions and reserved words. |
| `fcode.py` | `FCodePrinter` — generates Fortran 77/90/95 code from expressions, handling column-width wrapping, implicit typing, and Fortran-specific math intrinsics. |
| `rcode.py` | `RCodePrinter` — converts expressions into R language code, mapping SymPy functions to R's built-in math functions. |
| `jscode.py` | `JavascriptCodePrinter` — emits JavaScript code using `Math.*` functions; provides `jscode()`. |
| `julia.py` | `JuliaCodePrinter` — generates Julia source code from expressions; provides `julia_code()`. Handles element-wise (Hadamard) vs matrix operations, `Piecewise` rendering (inline ternary or if-else), and validates that Piecewise has a default branch (raises error otherwise). |
| `octave.py` | `OctaveCodePrinter` — produces Octave/Matlab-compatible code from expressions; provides `octave_code()`. |
| `rust.py` | `RustCodePrinter` — generates Rust source code from expressions using `f64` methods; provides `rust_code()`. |
| `glsl.py` | `GLSLPrinter` — emits GLSL (OpenGL Shading Language) code from expressions, with options for operator vs. function style and matrix formatting. |
| `mathematica.py` | `MCodePrinter` — converts expressions to Wolfram Mathematica syntax; provides `mathematica_code()`. |
| `lambdarepr.py` | `LambdaPrinter`, `NumExprPrinter`, and `TensorflowPrinter` — generate Python strings suitable for `lambdify`, NumExpr, and TensorFlow respectively. |
| `dot.py` | `dotprint()` — renders the expression tree as a Graphviz DOT graph, with helper functions `purestr`, `styleof`, `attrprint`, `dotnode`, and `dotedges`. |
| `tree.py` | `print_tree()` and `tree()` — produce an indented ASCII tree representation of an expression's class hierarchy and assumptions. |
| `tableform.py` | `TableForm` — formats 2-D tabular data for display in ASCII, LaTeX, or other output modes, with support for headings and alignment. |
| `preview.py` | `preview()` — compiles an expression to LaTeX then renders it as PNG, DVI, PostScript, or PDF using an external TeX distribution and viewer. |
| `gtk.py` | `print_gtk()` — renders an expression via MathML in the Gtkmathview widget (requires libgtkmathview-bin). |
| `theanocode.py` | `TheanoPrinter` — translates SymPy expressions into Theano tensor-variable graphs for GPU-accelerated numerical computation. |
| `llvmjitcode.py` | `LLVMJitPrinter` and `llvm_callable()` — compiles SymPy expressions to native machine code via LLVM IR using the llvmlite library. |
| `pretty/__init__.py` | Sub-package entry point for the ASCII/Unicode 2-D pretty-printer; re-exports `pretty`, `pprint`, and related functions. |
| `pretty/pretty.py` | `PrettyPrinter` — the core 2-D pretty-printer with `_print_<Type>` methods that compose `prettyForm` objects for each expression type. Orchestrates layout of expressions but delegates symbol/character lookup to `pretty_symbology.py` and 2-D picture manipulation to `stringpict.py`. |
| `pretty/pretty_symbology.py` | Low-level Unicode/ASCII character-lookup layer for the pretty printer. Contains `pretty_atom()` which maps atom names (Pi, Infinity, etc.) to display characters and raises `KeyError` when in ASCII mode with no default. Contains `xobj()` which returns the raw character pieces for variable-height delimiters. Also provides Greek letter tables, sub/superscript digit mappings, `pretty_symbol()`, and `xsym()`. Does not decide when or whether to apply brackets — only supplies the characters. |
| `pretty/stringpict.py` | `stringPict` and `prettyForm` — data structures representing 2-D ASCII pictures with baseline tracking, providing operations like horizontal/vertical joining, alignment, and binding-power parenthesization. Contains `parens()` which wraps a picture in bracket delimiters and includes an ASCII-mode fallback that collapses multi-line brackets to single-line to avoid ugly tall parentheses. Does not handle character/symbol lookup. |
