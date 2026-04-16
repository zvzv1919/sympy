# sympy/logic — Boolean Logic & Inference

## Boolean Algebra

### `boolalg.py`

Core boolean algebra module — defines all boolean types, connectives, normal form conversions, and simplification via the Quine–McCluskey algorithm.

**Base types:**

- `Boolean` — abstract base; overloads `&`, `|`, `~`, `>>`, `<<`, `^` for symbolic boolean ops.
- `BooleanAtom` → `BooleanTrue` / `BooleanFalse` — singleton truth values (`S.true`, `S.false`). Block arithmetic operators to prevent accidental numeric use.
- `BooleanFunction` — base class for `And`, `Or`, `Not`, etc. Provides `to_nnf()` and hooks into `simplify()`.

**Connectives (all subclass `BooleanFunction`):**

- `And`, `Or` — variadic lattice ops with relational short-circuiting (contradictory relationals collapse to `false`/`true`). Support `as_set()` for single-variable real sets.
- `Not` — negation; `eval()` flips relational operators (`<` ↔ `>=`, etc.). `to_nnf()` pushes negation inward (De Morgan, Implies, Xor, ITE).
- `Xor` — exclusive OR; flattens nested Xors, cancels duplicate args, handles complementary relationals.
- `Nand`, `Nor` — thin wrappers that immediately rewrite to `Not(And(...))` / `Not(Or(...))`.
- `Implies` — logical implication (`A >> B` ≡ `~A | B`). Simplifies when args are concrete or identical/complementary relationals.
- `Equivalent` — n-ary equivalence; reduces when concrete booleans or complementary relationals are present.
- `ITE` — if-then-else; supports symbolic differentiation via `diff()`.

**Normal form conversions:**

- `to_nnf(expr, simplify=True)` — Negation Normal Form (only `And`/`Or`/`Not`, negation on literals only).
- `to_cnf(expr, simplify=False)` — Conjunctive Normal Form. Optionally simplifies via `simplify_logic`.
- `to_dnf(expr, simplify=False)` — Disjunctive Normal Form. Same optional simplification.
- `is_nnf`, `is_cnf`, `is_dnf` — predicate checks for each form.
- `eliminate_implications(expr)` — rewrites `>>`, `<<`, `Equivalent` into `&`/`|`/`~` (delegates to `to_nnf`).

**Distribution helpers:**

- `distribute_and_over_or(expr)` — push `And` inside `Or` (→ CNF structure).
- `distribute_or_over_and(expr)` — push `Or` inside `And` (→ DNF structure).

**Quine–McCluskey minimization:**

- `SOPform(variables, minterms, dontcares=None)` — minimal Sum-of-Products from a truth table.
- `POSform(variables, minterms, dontcares=None)` — minimal Product-of-Sums from a truth table.
- `simplify_logic(expr, form=None, deep=True)` — simplify a boolean expression by extracting the truth table and rebuilding in SOP or POS, whichever is smaller (or forced via `form='cnf'`/`'dnf'`).

**Misc utilities:**

- `conjuncts(expr)` / `disjuncts(expr)`
- `is_literal(expr)`
- `to_int_repr(clauses, symbols)` — encode CNF clauses as sets of signed integers (used by DPLL solvers).
- `term_to_integer` / `integer_to_term` — binary-list ↔ integer conversions for truth table work.
- `truth_table(expr, variables, input=True)` — generator yielding `(input_combo, result)` pairs.
- `bool_map(bool1, bool2)` — find a variable mapping that makes two boolean expressions equivalent; uses a fingerprinting heuristic (`_finger`).

**Caveats:** Python's `&`, `|`, `~`, `>>`, `<<` are bitwise on `int`/`bool`; use `S.true`/`S.false` or the named classes to avoid surprises.

---

## Inference

### `inference.py`

Propositional inference: satisfiability checking, validity, entailment, model evaluation, and a simple knowledge base.

- `satisfiable(expr, algorithm="dpll2", all_models=False)` — SAT solver front-end. Converts to CNF, dispatches to `dpll` or `dpll2`. Returns a model dict or `False`; with `all_models=True` returns a generator.
- `valid(expr)` — tautology check (`¬expr` is unsatisfiable).
- `entails(expr, formula_set)` — checks if `formula_set ⊨ expr` (adds `¬expr` and checks unsatisfiability).
- `pl_true(expr, model, deep=False)` — evaluate an expression under a (possibly partial) assignment. Returns `True`/`False`/`None`. With `deep=True`, falls back to SAT/validity for undetermined cases.
- `literal_symbol(literal)` — strip negation from a literal.

**Knowledge base classes:**

- `KB` — abstract base with `tell`/`ask`/`retract`.
- `PropKB(KB)` — propositional KB; stores clauses in CNF, answers queries via `entails`.

---

## SAT Solvers

### `algorithms/dpll.py`

Original DPLL implementation operating on both symbolic and integer representations.

- `dpll_satisfiable(expr)` — entry point; converts to CNF integer repr, calls `dpll_int_repr`.
- `dpll(clauses, symbols, model)` — recursive DPLL on symbolic clauses.
- `dpll_int_repr(clauses, symbols, model)` — recursive DPLL on integer-encoded clauses (faster).
- `unit_propagate(clauses, symbol)` / `unit_propagate_int_repr(clauses, s)` — simplify clauses given a unit literal.
- `find_pure_symbol` / `find_pure_symbol_int_repr` — identify literals appearing with only one polarity.
- `find_unit_clause` / `find_unit_clause_int_repr` — find clauses with exactly one unbound literal.
- `pl_true_int_repr(clause, model)` — lightweight model evaluation for integer-encoded clauses.

### `algorithms/dpll2.py`

Optimized DPLL solver with clause learning, watched literals, and VSIDS heuristic.

- `dpll_satisfiable(expr, all_models=False)` — entry point; supports enumerating all models via a generator.

**`SATSolver` class** — the core solver object:
  - Watched-literal (sentinel) scheme for efficient unit propagation — avoids scanning all clauses on every assignment.
  - VSIDS branching heuristic (occurrence-count–based heap with decay).
  - Optional clause learning (`'simple'` or `'none'`): `_simple_add_learned_clause`, `_simple_compute_conflict`, `_simple_clean_clauses`.
  - `_find_model()` — main DPLL loop; yields model dicts, supports backtracking over all solutions.
  - `_assign_literal`, `_undo`, `_simplify`, `_unit_prop` — internal solver mechanics.

**`Level` class** — bookkeeping for a single decision level (decision literal, variable settings, flip state).

**Caveats:** `dpll2` is the default solver used by `satisfiable()`. The original `dpll` is retained but is significantly slower on large instances.

---

## Utilities

### `utilities/dimacs.py`

Parser for the DIMACS CNF file format (standard SAT benchmark format).

- `load(s)` — parse a DIMACS-formatted string into a SymPy `And`/`Or` expression. Variables become `Symbol("cnf_N")`.
- `load_file(location)` — convenience wrapper that reads a file and calls `load`.

---

## Package Init

### `__init__.py`

Re-exports the public API: `to_cnf`, `to_dnf`, `to_nnf`, `And`, `Or`, `Not`, `Xor`, `Nand`, `Nor`, `Implies`, `Equivalent`, `ITE`, `POSform`, `SOPform`, `simplify_logic`, `bool_map`, `true`, `false`, `satisfiable`.
