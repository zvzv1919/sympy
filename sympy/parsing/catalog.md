# sympy/parsing — Catalog

> Part of [SymPy](../catalog.md). Translates strings (Python-like, Mathematica, Maxima) into SymPy expressions.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package entry point; contains only a docstring describing the parsing package. |
| `sympy_parser.py` | The high-level SymPy expression parser: takes an already-tokenized stream (from `sympy_tokenize`) and applies a configurable pipeline of token-level transformations (`auto_symbol`, `auto_number`, `factorial_notation`, `implicit_multiplication`, `implicit_application`, `convert_xor`, `convert_equals_signs`, `split_symbols`, `function_exponentiation`, `rationalize`, `lambda_notation`) before evaluation. Anchored by the `parse_expr` function, `stringify_expr`/`eval_expr` helpers, and the `EvaluateFalseTransformer` AST rewriter. Does not perform lexical scanning or newline/indentation classification. |
| `sympy_tokenize.py` | The low-level lexical scanner: a customised fork of Python's `tokenize` module. Its `generate_tokens` generator breaks a character stream into typed tokens (NAME, NUMBER, STRING, OP, etc.) while tracking parenthesis/bracket nesting depth (`parenlev`) to classify line breaks — emitting NEWLINE (logical statement terminator) when `parenlev == 0` and NL (non-significant continuation whitespace) when inside grouped expressions (`parenlev > 0`). Also handles indentation (INDENT/DEDENT), continued statements via backslash, and adds support for `!`/`!!` (factorial) operators and repeating-decimal float literals. Provides `generate_tokens`, `untokenize`, `TokenError`, and the `Untokenizer` class. |
| `ast_parser.py` | Uses Python's `ast` module to parse a string into a SymPy expression, providing the `Transform` `NodeTransformer` subclass (which wraps literals in `Integer`/`Float` and undefined names in `Symbol`) and a `parse_expr` entry-point function. |
| `mathematica.py` | Translates Mathematica-syntax strings into SymPy expressions via a regex-based recursive-descent `parse` function and helper translators, with `mathematica` as the public entry point. |
| `maxima.py` | Converts Maxima CAS expressions into SymPy objects using regex substitutions and the `MaximaHelpers` class (wrapping `expand`, `float`, `trigexpand`, `sum`, `product`, `csc`, `sec`), exposed through the `parse_maxima` function. |
