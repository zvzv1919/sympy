# sympy/parsing — Expression Parsing

## Glossary

- **Transformation**: A callable `(tokens, local_dict, global_dict) -> tokens` that rewrites a token stream before evaluation. Transformations are composable and order-dependent.
- **Token**: A `(type, value)` pair produced by the tokenizer (e.g. `(NAME, 'x')`, `(OP, '+')`, `(NUMBER, '42')`).

---

## Core Expression Parsing

### sympy_parser.py

Main entry point for converting Python-like math strings into SymPy expressions via a tokenize → transform → eval pipeline.

**Primary API**

- `parse_expr(s, local_dict=None, transformations=..., global_dict=None, evaluate=True)` — Top-level parser.
  - Tokenizes `s`, applies each transformation in sequence, then evals the result.
  - `evaluate=False` rewrites the AST so all operators carry `evaluate=False`, preserving input order and suppressing automatic simplification.
- `stringify_expr(s, local_dict, global_dict, transformations)` — Tokenize + transform → Python source string. Building block of `parse_expr`.
- `eval_expr(code, local_dict, global_dict)` — Eval the code string produced by `stringify_expr`.

**Built-in Transformations** (each is a token-stream rewriter)

- `standard_transformations` — Tuple of `(lambda_notation, auto_symbol, auto_number, factorial_notation)`. Default for `parse_expr`.
- `auto_symbol` — Wraps undefined names in `Symbol(...)`.
- `auto_number` — Converts numeric literals to `Integer`, `Float`, `Rational`, or `I`-multiplied forms. Handles repeating-decimal notation `3.4[31]`.
- `rationalize` — Post-pass that replaces `Float` calls with `Rational`. Must run after `auto_number`.
- `lambda_notation` — Rewrites Python `lambda x: ...` to SymPy `Lambda((x,), ...)`.
- `factorial_notation` — Converts `!` / `!!` operators to `factorial` / `factorial2` calls.
- `convert_xor` — Maps `^` to `**`.
- `convert_equals_signs` — Maps `=` to `Eq(...)`, supports nesting.
- `implicit_multiplication` — Inserts `*` tokens between adjacent names/parenthesized groups (e.g. `3x` → `3*x`).
- `implicit_application` — Adds parentheses so `sin x` parses as `sin(x)`.
- `implicit_multiplication_application` — Convenience combo: `split_symbols` + `implicit_multiplication` + `implicit_application` + `function_exponentiation`.
- `function_exponentiation` — Allows `cos**2(x)` → `cos(x)**2`.
- `split_symbols` / `split_symbols_custom(predicate)` — Splits multi-char names into single chars for implicit multiplication (`xyz` → `x*y*z`). Greek names are kept intact.

**Internal Helpers**

- `_token_splittable(token)` — Predicate: True if a name can be split (no underscores, not a Greek letter name).
- `_token_callable(token, local_dict, global_dict)` — Predicate: True if the name resolves to a callable that isn't a Symbol.
- `_add_factorial_tokens(name, result)` — Wraps preceding expression in a `factorial`/`factorial2` call.
- `_group_parentheses(recursor)` — Decorator-style helper that groups tokens inside `()` into `ParenthesisGroup` lists and recurses on inner content.
- `_apply_functions` — Converts `NAME + ParenthesisGroup` into `AppliedFunction`.
- `_implicit_multiplication`, `_implicit_application` — Low-level token inserters used by the public transformations.
- `_flatten(result)` — Expands `AppliedFunction` objects back into flat token lists.
- `_transform_equals_sign` — Single-level `=` → `Eq(...)` rewriter.

**AST Rewriting for `evaluate=False`**

- `evaluateFalse(s)` — Parses Python source and rewrites binary ops via `EvaluateFalseTransformer`.
- `class EvaluateFalseTransformer(ast.NodeTransformer)` — Replaces `+`, `*`, `**`, `-`, `/`, `|`, `&`, `^` with explicit `Add(... evaluate=False)` etc. Handles `Sub` → negation, `Div` → `Pow(..., -1)`, and denests `Add`/`Mul`.

**Helper Classes**

- `class AppliedFunction` — Groups a function name token with its parenthesized arguments and optional exponent. Used as an intermediate representation during implicit-multiplication transforms.
- `class ParenthesisGroup(list)` — Tagged list subclass for parenthesized token groups.

### ast_parser.py

Lightweight AST-based parser that wraps Python numeric literals and undefined names in SymPy types. Simpler alternative to `sympy_parser.py` — no token transformations, no implicit multiplication.

- `parse_expr(s, local_dict)` — Parse string `s` as a Python expression, transform the AST, compile, and eval.
- `class Transform(NodeTransformer)` — Visits the AST to:
  - `visit_Num`: `int` → `Integer(...)`, `float` → `Float(...)`.
  - `visit_Name`: undefined names → `Symbol('...')` (skips names found in dicts or `True`/`False`).
  - `visit_Lambda`: rewrites Python lambda to SymPy `Lambda(Tuple(...), body)`.

### sympy_tokenize.py

Customized fork of Python's `tokenize` module. Adds support for `!` / `!!` operators and repeating-decimal float notation (`3.14[15]`).

- `generate_tokens(readline)` — Generator yielding `(type, string, start, end, line)` 5-tuples.
- `untokenize(iterable)` — Reconstruct source code from token stream.
- `tokenize(readline, tokeneater)` — Callback-based tokenization interface.
- `class Untokenizer` — Stateful helper for `untokenize`; handles whitespace reconstruction.
- `class TokenError` / `class StopTokenizing` — Exception types.

**Caveats**: The `Special` regex includes `\!\!` and `\!` patterns not present in the stdlib tokenizer — this is what allows factorial notation to work.

---

## External CAS Import

### mathematica.py

Converts Mathematica-syntax strings to SymPy expressions via regex-based rewriting.

- `mathematica(s)` — Public entry point: parse + sympify.
- `parse(s)` — Recursive regex matcher. Handles:
  - `f[x]` → `f(x)` function-call syntax.
  - `^` → `**` exponentiation.
  - Implied multiplication (`2x`, `(a)(b)`).
  - Nested parenthesized expressions.
- `translateFunction(s)` — Maps Mathematica function names to SymPy (e.g. `ArcSin` → `asin`).
- `translateOperator(s)` — Maps operators (`^` → `**`).

**Caveats**: The regex rules are fragile and order-dependent; complex nested expressions may not parse correctly.

### maxima.py

Converts Maxima CAS expressions to SymPy via regex substitution + `sympify`.

- `parse_maxima(str, globals=None, name_dict={})` — Strips trailing `;`, applies `sub_dict` replacements, extracts optional `var :` assignment, then `sympify`s. If `globals` is provided and an assignment `var : expr` is detected, the result is stored in `globals[var]`.
- `class MaximaHelpers` — Namespace of static methods callable during sympification:
  - `maxima_expand`, `maxima_float`, `maxima_trigexpand` — Delegate to SymPy `.expand()` / `.evalf()`.
  - `maxima_sum`, `maxima_product` — Wrap SymPy `Sum` / `product`.
  - `maxima_csc`, `maxima_sec` — `1/sin`, `1/cos`.
- `sub_dict` — Regex map: `%pi` → `pi`, `%e` → `E`, `%i` → `I`, `^` → `**`, `inf`/`minf` → `oo`/`-oo`, Maxima builtins → `MaximaHelpers` methods.

---

## Package Init

### \_\_init\_\_.py

Docstring-only. No public API exported.
