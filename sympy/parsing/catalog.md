# sympy/parsing — Catalog

> Part of [SymPy](../catalog.md). Translates strings (Python-like, Mathematica, Maxima) into SymPy expressions.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package entry point; contains only a docstring describing the parsing package. |
| `sympy_parser.py` | Token-level SymPy expression parser: takes an already-tokenized stream (from `sympy_tokenize`) and applies a configurable pipeline of token transformations before evaluation. Key transformations include `auto_number` (classifies and converts numeric literals—detects whether a NUMBER token is integer, float, hex, complex, or repeating decimal, e.g. guards against misreading hex `0xE5` as float due to `e`/`E`, and wraps results in `Integer`/`Float`/`Rational`/`I`), `auto_symbol`, `implicit_multiplication`, `implicit_application`, `convert_xor`, `convert_equals_signs`, `split_symbols`, `function_exponentiation`, `rationalize`, `factorial_notation`, and `lambda_notation`. Anchored by `parse_expr`, `stringify_expr`/`eval_expr` helpers, and the `EvaluateFalseTransformer` AST rewriter. Operates on `(token_type, token_string)` pairs, not AST nodes. |
| `sympy_tokenize.py` | Low-level lexical scanner (fork of Python `tokenize`). Breaks a character stream into raw typed tokens (NAME, NUMBER, STRING, OP, etc.) via `generate_tokens`—does NOT classify or convert numeric subtypes (hex vs float vs integer classification happens downstream in `sympy_parser.py`). Tracks parenthesis nesting (`parenlev`) for line-break classification, handles indentation (INDENT/DEDENT), backslash continuation, and adds `!`/`!!` (factorial) operator and repeating-decimal literal support. Provides `generate_tokens`, `untokenize`, `TokenError`, and `Untokenizer`. |
| `ast_parser.py` | AST-based string-to-SymPy parser (alternative to the token-level `sympy_parser.py`). The `Transform` `NodeTransformer` rewrites the AST node-by-node: `visit_Num` wraps int/float literals in `Integer`/`Float`, `visit_Name` decides per-identifier whether to pass through (local dict, known globals, boolean literals like `'True'`/`'False'`) or auto-wrap in `Symbol`, and `visit_Lambda` converts lambdas. Exposed via its own `parse_expr` function. |
| `mathematica.py` | Translates Mathematica-syntax strings into SymPy expressions via a regex-based recursive-descent `parse` function and helper translators, with `mathematica` as the public entry point. |
| `maxima.py` | Converts Maxima CAS expressions into SymPy objects using regex substitutions and the `MaximaHelpers` class (wrapping `expand`, `float`, `trigexpand`, `sum`, `product`, `csc`, `sec`), exposed through the `parse_maxima` function. |
