# sympy/parsing — Catalog

> Part of [SymPy](../catalog.md). Translates strings (Python-like, Mathematica, Maxima) into SymPy expressions.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package entry point; contains only a docstring describing the parsing package. |
| `sympy_parser.py` | The main SymPy expression parser: tokenizes Python-like strings and applies a configurable pipeline of token transformations (`auto_symbol`, `auto_number`, `factorial_notation`, `implicit_multiplication`, `implicit_application`, `convert_xor`, `convert_equals_signs`, `split_symbols`, `function_exponentiation`, `rationalize`, `lambda_notation`) before evaluation, anchored by the `parse_expr` function, `stringify_expr`/`eval_expr` helpers, and the `EvaluateFalseTransformer` AST rewriter. |
| `sympy_tokenize.py` | A customised fork of Python's `tokenize` module that adds support for `!`/`!!` (factorial) operators and repeating-decimal float literals, providing `generate_tokens`, `untokenize`, the `TokenError` exception, and the `Untokenizer` class. |
| `ast_parser.py` | Uses Python's `ast` module to parse a string into a SymPy expression, providing the `Transform` `NodeTransformer` subclass (which wraps literals in `Integer`/`Float` and undefined names in `Symbol`) and a `parse_expr` entry-point function. |
| `mathematica.py` | Translates Mathematica-syntax strings into SymPy expressions via a regex-based recursive-descent `parse` function and helper translators, with `mathematica` as the public entry point. |
| `maxima.py` | Converts Maxima CAS expressions into SymPy objects using regex substitutions and the `MaximaHelpers` class (wrapping `expand`, `float`, `trigexpand`, `sum`, `product`, `csc`, `sec`), exposed through the `parse_maxima` function. |
