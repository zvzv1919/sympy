# parsing — Module Catalog

Translates strings from various mathematical notations into SymPy expressions.

## Core Parsing

### [`sympy_parser.py`](sympy_parser.py)
Primary string-to-SymPy parser for Python-like expressions.
- `parse_expr()` — main entry point; applies token transformations, then evaluates.
- Token transformation functions (applied as pipeline): `auto_symbol`, `auto_number`, `lambda_notation`, `factorial_notation`, `convert_xor`, `convert_equals_signs`, `implicit_multiplication`, `implicit_application`, `split_symbols_custom`, `rationalize`.
- `_implicit_multiplication()` — inserts `*` tokens where multiplication is implied between adjacent tokens.
- `_implicit_application()` — inserts function application where implied.
- `AppliedFunction` — wrapper for a function token and its argument group.
- `stringify_expr()` — converts string to valid Python via token transformations.
- `eval_expr()` — evaluates the transformed code string.
- `evaluateFalse()` — rewrites AST to wrap operators with `evaluate=False`.
- `EvaluateFalseTransformer` — AST NodeTransformer that replaces binary ops (Add, Mul, Pow, Div, Sub, etc.) with SymPy calls using `evaluate=False`. Handles negated operand ordering in division and subtraction.

### [`ast_parser.py`](ast_parser.py)
Lightweight AST-based parser that wraps literal numbers as `Integer`/`Float` and undefined names as `Symbol`.
- `Transform` — NodeTransformer: `visit_Num` wraps numeric literals, `visit_Name` wraps unknown names as Symbols, `visit_Lambda` wraps lambdas.
- `parse_expr()` — simplified parse entry point (no token transformations, no `evaluate=False` support).
- **Caveat**: Does NOT support token-level transformations or `evaluate=False`; use `sympy_parser.parse_expr` for those.

### [`sympy_tokenize.py`](sympy_tokenize.py)
Python tokenizer producing 5-tuples of (type, string, start, end, line). Used by `sympy_parser`.
- `generate_tokens(readline)` — generator that yields token 5-tuples from a readline callable.
- `tokenize(readline, tokeneater)` — callback-based tokenization interface.
- `Untokenizer` — reconstructs source code from token streams. `add_whitespace()` enforces a row-ordering constraint (raises `ValueError` if row exceeds prev_row). `untokenize()` reassembles tokens into a string.
- `TokenError` — raised on malformed token streams.
- `untokenize(iterable)` — module-level convenience wrapper around `Untokenizer`.

## External Language Parsers

### [`mathematica.py`](mathematica.py)
Converts Wolfram Language / Mathematica notation to SymPy-compatible strings.
- `mathematica(s)` — parses and sympifies a Mathematica-syntax string.
- `parse(s)` — regex-based recursive descent parser handling function calls (`f[x]`), implied multiplication, and infix operators.
- `translateFunction(s)` — remaps Mathematica function names: inverse trig names starting with "Arc" become "a"-prefixed lowercase (e.g., ArcSin → asin); all others lowercased.
- `translateOperator(s)` — converts operators (e.g., `^` → `**`).

### [`maxima.py`](maxima.py)
Converts Maxima CAS notation to SymPy expressions.
- `parse_maxima(str)` — regex substitution of Maxima constants/functions, then sympifies.
- `MaximaHelpers` — helper methods for Maxima-specific functions (expand, trigexpand, sum, product, csc, sec).
