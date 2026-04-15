# sympy/parsing — Catalog

> Part of [SymPy](../catalog.md). Translates strings (Python-like, Mathematica, Maxima) into SymPy expressions.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package entry point; contains only a docstring describing the parsing package. |
| `sympy_parser.py` | The main SymPy expression parser: applies a configurable pipeline of token transformations (`auto_symbol`, `auto_number`, `factorial_notation`, `implicit_multiplication`, `implicit_application`, `convert_xor`, `convert_equals_signs`, `split_symbols`, `function_exponentiation`, `rationalize`, `lambda_notation`) before evaluation, anchored by the `parse_expr` function, `stringify_expr`/`eval_expr` helpers, and the `EvaluateFalseTransformer` AST rewriter. Uses `_re_repeated` to **convert** already-tokenized repeating-decimal strings into `Rational` values during the `auto_number` transformation — this is post-tokenization conversion, not lexical recognition. |
| `sympy_tokenize.py` | A customised fork of Python's `tokenize` module that is the **lexical analyzer**: defines all regular expression patterns for numeric literal recognition (integers, floats, imaginary numbers) via helper functions (`group`, `any`, `maybe`) and module-level regex variables (`Intnumber`, `Floatnumber`, `Imagnumber`, `Number`, etc.). Notably extends the standard Python lexer with `Repeatedfloat` (`r'\d*\.\d*\[\d+\]'`) for recurring/repeating decimal notation (e.g., `0.1[6]` for 0.1666…) and `!`/`!!` (factorial) operators. Provides `generate_tokens`, `untokenize`, the `TokenError` exception, and the `Untokenizer` class. See also: `sympy_parser.py` for post-tokenization conversion of repeating decimals. |
| `ast_parser.py` | Uses Python's `ast` module to parse a string into a SymPy expression, providing the `Transform` `NodeTransformer` subclass (which wraps literals in `Integer`/`Float` and undefined names in `Symbol`) and a `parse_expr` entry-point function. |
| `mathematica.py` | Translates Mathematica-syntax strings into SymPy expressions via a regex-based recursive-descent `parse` function and helper translators, with `mathematica` as the public entry point. |
| `maxima.py` | Converts Maxima CAS expressions into SymPy objects using regex substitutions and the `MaximaHelpers` class (wrapping `expand`, `float`, `trigexpand`, `sum`, `product`, `csc`, `sec`), exposed through the `parse_maxima` function. |
