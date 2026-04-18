# Physics Module Catalog

## Architecture Overview

The `physics` package provides symbolic physics across classical mechanics, optics, quantum mechanics, and unit systems. Top-level `.py` files handle standalone physics topics (hydrogen atom, Wigner symbols, Pauli algebra, second quantization, harmonic oscillators). Submodules group larger domains: `quantum/` (abstract quantum mechanics, gates, spin, algorithms), `vector/` (reference-frame-aware 3-D vectors and kinematics), `optics/` (geometric/wave optics), `mechanics/` (Lagrangian/Kane dynamics), `hep/` (high-energy physics), and `unitsystems/` (dimensional analysis).

---

## Top-Level Files

### [`hydrogen.py`](hydrogen.py)
Hydrogen-like atom wavefunctions and energy levels.
- `R_nl(n, l, r, Z=1)` — radial wavefunction with quantum numbers n, l and atomic number Z.
- `E_nl(n, Z=1)` — non-relativistic energy in Hartree units; depends only on n (not l). Raises ValueError for n < 1.
- `E_nl_dirac(n, l, spin_up=True, Z=1, c=137.036...)` — relativistic Dirac energy (rest mass excluded). Raises ValueError when l=0 and spin_up=False (no spin-down state for s-orbitals).

### [`matrices.py`](matrices.py)
Standard physics matrices as SymPy Matrix objects.
- `msigma(i)` — 2×2 Pauli spin matrix σ_i (i = 1, 2, 3); raises `IndexError` for any i outside {1, 2, 3}.
- `mgamma(mu, lower=False)` — 4×4 Dirac gamma matrix γ^μ (standard/Dirac representation); `lower=True` returns γ_μ by negating spatial (1,2,3) and chiral (5) indices (metric signature +−−−).
- `pat_matrix(m, dx, dy, dz)` — 3×3 Parallel Axis Theorem correction matrix for translating an inertia tensor by displacement (dx, dy, dz) for a body of mass m; returns m·[off-diagonal: −dᵢdⱼ, diagonal: sum of squared perpendicular components].
- `mdft(n)` — n×n discrete Fourier transform matrix.
- `minkowski_tensor` — 4×4 Minkowski metric tensor.

### [`paulialgebra.py`](paulialgebra.py)
Pauli matrix algebra via pure symbolic manipulation (Symbol subclass, not quantum operators or states). No kets, bras, or operator framework — for those see `quantum/pauli.py`.
- `Pauli(i)` — Symbol subclass representing σ_i (i must be 1, 2, or 3; raises `IndexError` otherwise); algebraic multiplication yields products and scalar I automatically.
  - `_eval_power` reduces exponent mod 2 for positive integers (σ²=1); returns None (Symbol fallback) for non-positive/non-integer exponents.
- `delta(i, j)` — Kronecker delta helper; returns 1 if i == j, else 0.
- `epsilon(i, j, k)` — Levi-Civita symbol helper; returns +1 for even permutations of (1,2,3), −1 for odd, 0 otherwise (including repeated indices).
- `evaluate_pauli_product(arg)` — simplifies a product of commutative Pauli Symbol objects (σ₁σ₂→iσ₃) using index-based algebraic rules; no labels, no non-commutative operator framework.

### [`pring.py`](pring.py)
Quantum particle on a ring (circular-path constraint).
- `wavefunction(n, x)` — angular eigenfunctions (complex exponentials); no integer validation on quantum number n.
- `energy(n, m, r)` — quantized kinetic energy levels (n²ℏ²/2mr²); raises ValueError if n is not an integer.

### [`qho_1d.py`](qho_1d.py)
One-dimensional quantum harmonic oscillator: closed-form analytical wavefunctions and energy formulas (no operator algebra).
- `psi_n(n, x, m, omega)` — spatial wavefunction ψ_n using Hermite polynomials.
- `E_n(n, omega)` — energy eigenvalue ℏω(n+½).
- `coherent_state(n, alpha)` — Fock-basis overlap ⟨n|α⟩ between number state and coherent state (eigenstate of annihilation/lowering operator); computes exp(−|α|²/2)·α^n/√(n!).

### [`secondquant.py`](secondquant.py)
Second quantization framework for many-body quantum mechanics — integer-occupation-number bosonic/fermionic operators (distinct from abstract quantum operators in `quantum/`).
- `BosonicOperator`, `CreateBoson` (B†), `AnnihilateBoson` (B) — bosonic ladder operators with commutation relations.
- `FermionicOperator`, `CreateFermion` (Fd), `AnnihilateFermion` (F) — many-body fermionic ladder operators with fixed anticommutation rules and Fermi-level orbital properties (not mode-labeled like `quantum/fermion.py`).
  - `apply_operator(state)` — applies to FockStateKet directly; for Mul (product) expressions, checks if the first non-commutative factor is a FockStateKet and acts on it while preserving commutative prefactors; otherwise returns plain product.
  - `is_restricted` — returns +1 (above_fermi), −1 (below_fermi), or 0 (general/unrestricted) based on orbital index symbol assumptions.
  - `is_above_fermi`/`is_below_fermi` — whether the index *allows* values above/below Fermi level (general indices allow both).
- `NO` — normal-ordering bracket for `secondquant` operators (CreateBoson/AnnihilateBoson, CreateFermion/AnnihilateFermion); reorders into creation-before-annihilation form.
  - Returns S.Zero if identical fermion operators violate Pauli exclusion. For mode-labeled `BosonOp`/`FermionOp`, see `quantum/operatorordering.py`.
- `Commutator`, `AntiCommutator` — many-body (anti)commutator wrappers (for the abstract quantum operator versions, see `quantum/commutator.py` and `quantum/anticommutator.py`).
  - `Commutator.eval` — automatic simplification on construction: distributes over sums ([A+B,C]→[A,C]+[B,C]), extracts scalar prefactors ([xA,yB]→xy[A,B]), directly evaluates bosonic/fermionic single-operator pairs.
- `FockState`, `FockStateKet`, `FockStateBra` — Fock-space state vectors. `FermionState` subclass adds fermi-level logic: `_only_above_fermi(i)` returns True for symbolic indices without assumptions when no fermi level is set.
- `wicks(expr)` — applies Wick's theorem to expand operator products into normal-ordered contractions.
- `Dagger` — Hermitian conjugate for second-quantization operators (distinct from `quantum/dagger.py` which wraps `adjoint` for abstract quantum operators). `eval()` fallback for objects without `_dagger_()`: reverses factor order for products (Mul), distributes over sums, conjugates base of powers, negates I.
- `apply_operators()` — applies operators to states. `evaluate_deltas()` — simplifies Kronecker delta products.
- `contraction(a, b)` — evaluates the contraction of two operators.
- `matrix_rep(op, basis)` — matrix representation in a Fock basis.

### [`sho.py`](sho.py)
3-D isotropic quantum harmonic oscillator.
- `R_nl(n, l, nu, r)` — radial wavefunctions with associated Laguerre polynomials.

### [`units.py`](units.py)
Legacy physical-units module with ~200 predefined units, physical constants, and `find_unit()` search.
- `Unit` — base class for physical units (AtomicExpr subclass); stores `name` and `abbrev`. Sets `is_positive=True` so sqrt(unit²) simplifies to unit (not Abs(unit)). Equality (`__eq__`) compares only `name`, ignoring `abbrev` — two units with the same name but different abbreviations are considered equal.
- `find_unit(quantity)` — two modes: string input → substring match against module namespace; unit expression input → strips numeric coefficient via `as_coeff_Mul()`, compares the dimensional part against all defined symbols in the module. Results sorted by name length.

### [`wigner.py`](wigner.py)
Exact angular-momentum coupling coefficients (returns rationals × √rational).
- `wigner_3j(j1,j2,j3,m1,m2,m3)` — Wigner 3-j symbol; validates integer/half-integer inputs (doubles values and checks integrality, raises ValueError otherwise); returns 0 early for selection-rule violations (triangle inequality, m1+m2+m3≠0, |m|>j); guards against imaginary parts from factorial square-roots.
- `wigner_6j`, `wigner_9j` — higher-order recoupling coefficients.
- `clebsch_gordan(j1,j2,j3,m1,m2,m3)` — Clebsch-Gordan coefficient (wrapper around wigner_3j).
- `racah(aa,bb,cc,dd,ee,ff)` — Racah W-coefficient; uses `_big_delta_coeff` products internally; returns 0 immediately if any triangle inequality fails.
- `_big_delta_coeff(aa,bb,cc)` — triangle coefficient for three angular momenta; returns 0 (not error) when triangle inequality is violated (e.g., one j exceeds sum of other two).
- `gaunt(l1,l2,l3,m1,m2,m3)` — Gaunt coefficient (integral of three spherical harmonics).

### [`gaussopt.py`](gaussopt.py)
Deprecated — redirects to `sympy.physics.optics.gaussopt`.

---

## Submodules

### [`quantum/`](quantum/catalog.md)
Abstract quantum mechanics framework: states, operators, Hilbert spaces, representations, and quantum-information primitives.
- **Core**: `qexpr.py` (base quantum expression `QExpr`), `operator.py` (non-commuting Operator base class, HermitianOperator, UnitaryOperator, IdentityOperator, OuterProduct, DifferentialOperator), `hilbert.py` (Hilbert spaces).
  - `hilbert.py` — `HilbertSpace.__contains__` checks membership by comparing space *classes* (not instances) to support symbolic dimensions.
    - `TensorProductHilbertSpace.eval` — validates all arguments are HilbertSpace or TensorPowerHilbertSpace (raises TypeError otherwise); merges adjacent like spaces into `TensorPowerHilbertSpace`: two powers with same base → combined exponent; plain space adjacent to its power → exponent+1; two identical plain spaces → power of 2.
    - `TensorPowerHilbertSpace.eval` — validates exponent when raising a Hilbert space to a power: single-atom exponent must be a non-negative Integer or Symbol (raises ValueError otherwise); multi-term exponent (e.g. n+42) only checks each atom is Integer or Symbol but does NOT enforce non-negativity of integer components.
  - `operator.py` also defines `OuterProduct` (|ket⟩⟨bra| dyadic); `__new__` distributes over addition — when either argument is a sum of states, expands into a sum of individual OuterProduct terms. `_eval_adjoint` returns OuterProduct(Dagger(bra), Dagger(ket)) — swaps and daggers both components.
  - `DifferentialOperator` (d/dx applied to wavefunctions): applies to Wavefunction by substituting the placeholder function with the wavefunction's expression, then preserving the original coordinate bounds (args[1:]) in the resulting Wavefunction.
  - `qexpr.py` — `QExpr._eval_adjoint`: if parent Expr adjoint returns None, wraps in Dagger; then propagates `hilbert_space` onto the result if it is a QExpr instance. `_qsympify_sequence` normalizes constructor args: strings → Symbol (prevents 'pi' becoming a numeric constant), sequences → recursive Tuple, Matrix passthrough, else sympify.
  - `represent.py` — `represent(expr, basis)`: converts quantum expressions to matrix form. Fallback chain: if `_represent()` raises NotImplementedError, tries `rep_innerproduct` for Ket/Bra or `rep_expectation` for Operator; re-raises if fallback also fails.
    - `get_basis(expr, **options)` — resolves representation basis: if basis option is a StateBase instance, returns it directly; if an Operator (instance or class), maps via `operators_to_state` and instantiates via `_make_default` if result is a class rather than instance. When no basis given: Ket → default of own class, Bra → default of `dual_class()`, Operator → operator-to-state mapping.
    - `enumerate_states(state, ...)` — generates indexed copies of an abstract vector (Ket/Bra) given a list of indices or a start index + count; delegates to `state._enumerate_state()`; returns empty list if the state raises NotImplementedError.
    - `_sympy_to_scalar` — converts SymPy scalar expressions to native Python types (int/float/complex) for numpy/scipy compatibility; handles Integer, Float, Rational, Number, NumberSymbol, and imaginary unit I (→ complex). Raises TypeError for non-numeric expressions.
  - `state.py` — Ket/Bra/Wavefunction with multiplication dispatch on both sides: `KetBase.__mul__` (Ket*Bra → OuterProduct, else Expr.__mul__), `BraBase.__mul__` (Bra*Ket → InnerProduct, else Expr.__mul__), `BraBase.__rmul__` (Ket*Bra → OuterProduct, non-ket*Bra falls back to Expr.__rmul__).
    - `StateBase` — abstract base for all quantum states; `dual` property constructs the conjugate-class counterpart (via `dual_class()`) with same Hilbert space and args. `_eval_adjoint` delegates to `dual`, so the Hermitian conjugate of a ket is its bra and vice versa.
    - `Wavefunction` — continuous-basis representation (Function subclass). Constructor converts Python tuples to Tuple objects before parent call to avoid type-check errors.
    - `Wavefunction.norm` — L2 norm: integrates |expr|² over each coordinate variable within its bounds, returns sqrt of result. `is_normalized` compares norm to 1.0.
    - `Wavefunction.normalize()` — returns a rescaled Wavefunction with unit norm; raises NotImplementedError if norm is infinite (non-normalizable function).
    - `BraBase._represent` — default bra representation: delegates to the dual ket's `_represent` and applies Dagger (conjugate transpose) to the result.
    - `StateBase._represent_default_basis` — determines default representation basis by querying which operators the state is an eigenstate of (lazy-imports `operatorset` to break circular dependency).
- **Angular momentum / CG**: `cg.py` — Clebsch-Gordan and Wigner coupling coefficient symbolic expressions, evaluation, and simplification (not state construction).
  - `Wigner3j` — symbolic Wigner 3-j coefficient; `is_symbolic` property checks if any parameter is non-numeric. `doit()` raises ValueError for symbolic params.
  - `Wigner6j`, `Wigner9j` — symbolic 6-j and 9-j coefficients with analogous `doit()`.
  - `CG` (subclass of `Wigner3j`) — Clebsch-Gordan coefficient; inherits `is_symbolic` check and ValueError guard.
  - `_check_cg()` — validates whether a candidate term matches a structural Wild pattern and expected sign convention (sign tuple comparison after substitution).
  - Simplification rules apply orthogonality-relation identities to reduce CG products summed over j,m to Kronecker deltas.
- **Spin**: `spin.py` — spin operators (Jx, Jy, Jz, J±, J²), coupled/uncoupled states, Wigner-D/d matrices, `Rotation` operator (Euler-angle unitary).
  - `SpinOpBase._apply_op` — general operator-on-ket dispatch: rewrites ket into operator's own basis, then branches: single State → eigenvalue multiply, Sum → delegate to `_apply_operator_Sum`, else → fallback to `qapply`; raises NotImplementedError if qapply cannot simplify.
  - `SpinOpBase._apply_operator_TensorProduct` — distributes operator action across each factor of a tensor-product (multi-particle) state; restricted to coordinate-basis operators (Jx, Jy, Jz) only — raises NotImplementedError for J+, J−, J².
  - `J2Op` — total angular momentum squared (Casimir) operator; commutes with all component operators and applies eigenvalue ℏ²j(j+1). Supports rewrite as Cartesian components (Jx²+Jy²+Jz²) or as ladder operators via symmetrized form: Jz² + ½(J+J− + J−J+).
  - `Rotation` — Euler-angle rotation operator; applies to both uncoupled and coupled kets: enumerates D-matrix elements for numeric j, returns symbolic Sum for symbolic j (using Dummy variable by default, named symbol when `dummy=False`).
  - `SpinState._eval_innerproduct_J{x,y,z}Bra` — cross-basis inner products: when bra and ket belong to different component bases, uses the ket's matrix representation in the bra's basis; same-basis returns KroneckerDelta orthonormality.
  - `SpinState._rewrite_basis` — converts spin eigenstates between measurement bases (Jx↔Jy↔Jz); numeric j enumerates D-matrix elements, symbolic j returns Sum with auto-generated summation index that avoids collisions with symbols already in the state.
  - `CoupledSpinState` — coupled state constructor with triangle-inequality validation on coupling schemes; auto-generates a default sequential coupling order when none provided (space 1+2, then result+3, etc., with accumulated partial j sums).
    - `_build_coupled(jcoupling, length)` — parses a coupling scheme (list of (n1,n2,j) tuples) into paired subsystem index groups and intermediate j values; used by both the constructor and `uncouple()`.
    - `_eval_hilbert_space`: numeric total j → DirectSumHilbertSpace of ComplexSpaces; symbolic j → falls back to single ComplexSpace(2j+1).
    - `_represent_coupled_base` — matrix representation of a coupled state; computes starting row index differently for integer j (j²) vs half-integer j ((2j−1)(1+2j)/4) to place the block within the direct-sum Hilbert space vector.
  - `couple()`/`_couple()` — combines uncoupled spin states into coupled representation; validates custom coupling order: after two spaces couple, the result must be referenced by the smaller index (raises ValueError otherwise).
  - Numeric path enumerates configurations, filters non-physical ones via triangle inequality (|j1−j2|≤j3≤j1+j2) and |m|≤j checks before computing CG coefficients.
  - `uncouple()`/`_uncouple()` — decomposes spin eigenstates into sums of tensor-product states weighted by CG coefficients. Accepts both `CoupledSpinState` (extracts coupling from state) and plain `SpinState` (requires explicit j-values; auto-generates default sequential coupling scheme if none provided: space 1+2, then result+3, etc.).
    - Numeric j,m: enumerates valid magnetic projection configurations explicitly. Symbolic j,m: returns symbolic Sum over CG products.
- **Gates**: `gate.py` — quantum gate classes (H, X, Y, Z, S/Phase, T, CNOT, SWAP, CGate, UGate); each gate stores target matrices and decomposition methods.
  - `Gate._represent_ZGate` — Z-basis matrix representation; validates nqubits (raises QuantumError if zero or less than `min_qubits` for the gate) before building the representation matrix via `represent_zbasis`.
  - `OneQubitGate._eval_commutator` — short-circuits to zero when two single-qubit gates act on different targets OR are the same gate class; otherwise falls back to generic Operator commutator.
  - `Gate._apply_operator_Qubit` — applies a gate to a qubit state: selects a target-matrix column via bit-shifted index from target qubits, then flips bits to construct the output state superposition.
  - `CGate` — controlled gate; wraps an inner gate with control qubits. When inner gate is Hermitian: dagger/inverse return self; power with even exponent → identity, odd → self, **except** exp=−1 delegates to parent `Gate._eval_power` instead of returning self.
  - `Gate._eval_hilbert_space` — determines smallest Hilbert space from target qubit indices: ComplexSpace(2)^(max_target+1).
  - `gate_sort(circuit)` — bubble-sorts gates respecting commutation; swaps commuting gates freely, applies (−1)^(exp1·exp2) sign correction when anticommutator vanishes.
  - `gate_simp(circuit)` — recursive symbolic simplification of gate sequences: self-inverse gates (H, X, Y, Z) reduce exponent mod 2; PhaseGate²→ZGate, TGate²→PhaseGate (power-promotion chain). Calls gate_sort first, then iterates.
- **Circuit plotting**: `circuitplot.py` — `CircuitPlot` for rendering circuits; `CreateCGate(name, latexname=None)` factory for dynamically creating controlled gates (defaults latexname to name if omitted); mock measurement gates `Mz`, `Mx`.
- **Circuit identity search**: `identitysearch.py` — gate identity discovery and equivalent rewriting rules via BFS exploration.
  - `generate_gate_rules(gate_seq)` — finds equivalent gate rewriting rules via BFS; returns trivial rule set when input is a plain numeric scalar.
  - `generate_equivalent_ids(gate_seq)` — finds equivalent gate identities; returns `{Integer(1)}` immediately when input is a plain Number.
  - `bfs_identity_search(gate_list, nqubits, max_depth)` — BFS over operator product sequences to find those equivalent to a scalar; prunes trivially decomposable sequences via `is_reducible`.
  - `GateIdentity` — represents a gate sequence that multiplies to a scalar; stores equivalent permutations. `is_degenerate` checks if a candidate is a permutation of an existing identity.
  - `is_scalar_sparse_matrix(circuit, nqubits, identity_only, eps)` — checks if a gate sequence's scipy.sparse matrix form is a scalar matrix (bI); splits complex entries into real/imaginary parts and zeros near-zero values within eps tolerance before checking diagonal uniformity and trace. Handles edge case where `represent()` returns a plain int instead of a matrix.
  - `is_scalar_nonsparse_matrix` — dense-matrix variant (fallback when scipy unavailable); same edge case handling for scalar `represent()` returns. Checks diagonal + uniform trace.
  - `is_reducible(circuit, nqubits, begin, end)` — checks if a circuit interval contains a scalar subcircuit; only tests right-anchored subcircuits (grows leftward from `end`), so left-anchored-only reductions within the range may be missed.
  - `ll_op`, `lr_op`, `rl_op`, `rr_op` — elementary rule-rewriting operations on two-sided circuit equalities. First letter = source side (L/R), second = end removed: `ll_op` takes leftmost of left side; `lr_op` takes rightmost of left side; `rl_op` takes leftmost of right side, daggers it, prepends to left side; `rr_op` takes rightmost of right side. Each verifies unitarity before rewriting.
- **Second-quantized QM operators**: `boson.py` — bosonic creation/annihilation operator algebra and quantum states for bosonic modes.
  - `BosonOp` — mode-labeled bosonic ladder operator; `_eval_commutator_BosonOp` returns −1 for [a†,a] same-name, 0 with `independent` hint, None otherwise.
    - `__mul__`: when multiplied by a Mul expression, separates commutative and non-commutative factors, iteratively multiplies non-commutative parts, then recombines with commutative prefactor.
  - `BosonFockKet/Bra` — Fock number states with KroneckerDelta inner product; ladder action: a|n⟩→√n|n−1⟩, a†|n⟩→√(n+1)|n+1⟩.
  - `BosonCoherentKet/Bra` — coherent states (eigenstates of annihilation operator); ⟨α|β⟩ inner product returns 1 when α=β, else Gaussian overlap exp(−(|α|²+|β|²−2·conj(β)·α)/2).
  - `fermion.py` — `FermionOp`: mode-labeled fermionic ladder operator. `FermionFockKet`/`FermionFockBra`: single-mode Fock states restricted to n∈{0,1}; applying creation to occupied state → 0 (Pauli exclusion enforcement).
    - `_eval_anticommutator_FermionOp`: returns 1 for {a†,a} same-name; None for same-name same-type; `independent` hint checked only for different names.
- **Operator ordering**: `operatorordering.py` — two reordering modes for `BosonOp`/`FermionOp` (mode-labeled quantum/ operators):
  - `normal_ordered_form()` — equivalent reordering: preserves operator algebra by adding commutator/anticommutator correction terms during swaps.
  - `normal_order()` — non-equivalent reordering: simply swaps creation before annihilation, dropping commutator terms. Fermionic swaps always apply a sign flip (−1) regardless of whether operators share the same mode.
  - `_normal_ordered_form_factor` — core swap logic for equivalent form: bosonic swaps use Commutator; fermionic swaps use AntiCommutator (with sign flip).
  - Independent modes (`independent=True`): correction term set to zero for operators on different modes.
  - Recurses after each swap+expand; `recursive_limit` depth guard warns and aborts on excess.
- **Commutator algebra**: `commutator.py`, `anticommutator.py` — abstract quantum `Commutator`/`AntiCommutator` with `doit()` evaluation.
  - Delegates to operator `_eval_commutator_*`/`_eval_anticommutator_*` methods; falls back through NotImplementedError chain.
- **Algorithms**: `grover.py` (Grover's search: `OracleGate`, `WGate`, `grover_iteration`, `apply_grover`; default iteration count = floor(√2ⁿ·π/4) when not specified), `qft.py` (quantum Fourier transform gates and matrix representations).
  - `RkGate` — parametric phase-rotation gate R_k; constructor simplifies small k values: k=1→ZGate, k=2→PhaseGate, k=3→TGate (returns different gate type, not RkGate).
  - `Fourier` (QFT/IQFT) — `_represent_ZGate` builds Fourier matrix and embeds into full Hilbert space via tensor products with identity matrices on both sides when gate doesn't start at qubit 0 or total qubits exceed gate range.
  - `shor.py` — Shor's factoring. `CMod`: controlled modular-exponentiation gate; reads integer from upper register half, computes a^k mod N, writes into lower half.
- **Qubits**: `qubit.py` — `Qubit`, `IntQubit`, qubit-state manipulation, measurement, and partial trace. `QubitState.flip(*bits)` toggles specified bit positions using reversed indexing (dimension−i−1) to map user-facing LSB-right convention to internal tuple order.
  - `Qubit._represent_ZGate` — Z-basis column vector: iterates bit values in reverse to compute binary→integer index, sets a 1 at that position in a 2^n-length vector. Supports sympy, numpy, scipy.sparse formats.
  - `IntQubit` — integer-to-binary qubit encoding: single int arg → uses minimum bits needed; two-int args → second specifies bit width, raises ValueError if width is insufficient to represent the integer. `as_int()` reconstructs integer from stored binary tuple.
  - `Qubit._eval_trace(bra, indices)` — partial trace over selected subsystem indices; sorts indices to trace from most-significant qubit, returns scalar for full trace or density operator for partial trace.
  - `matrix_to_qubit(matrix)` — converts a numerical column/row vector into a symbolic superposition of basis states; determines Ket vs Bra from matrix shape. Raises QuantumError if vector length is not a power of 2.
  - `measure_all(qubit, format='sympy')` — full ensemble measurement: returns list of (basis-state, probability) pairs for all non-zero-amplitude outcomes. Accepts format parameter ('sympy', 'numpy', 'scipy.sparse') but only 'sympy' is implemented; others raise NotImplementedError.
  - `measure_partial(qubit, bits, format='sympy')` — partial measurement on a subset of qubits: uses `_get_possible_outcomes` to bin state-vector amplitudes into outcome groups via bitmask matching on measured-qubit indices; computes per-outcome probability via inner product and returns list of (post-collapse normalized state, probability) pairs. Same format limitation as `measure_all`.
  - `measure_all_oneshot(qubit)` — single-shot measurement: normalizes state, draws a random number, accumulates squared amplitudes until cumulative probability exceeds threshold, returns the selected basis state.
  - `measure_partial_oneshot(qubit, bits)` — single-shot partial measurement on selected qubits: computes outcome probabilities, randomly selects one outcome weighted by probability, returns the normalized post-collapse state.
- **Operator application**: `qapply.py` — `qapply(e)` symbolically applies operators to states in an expression; dispatches by expression type (Add, Mul, TensorProduct, Density, Pow).
  - `qapply_Mul` handles products: decomposes OuterProduct (ket-bra) by pushing ket onto args and using bra as new lhs; tries lhs._apply_operator(rhs), then rhs._apply_operator(lhs), then forms InnerProduct for Bra·Ket pairs.
  - Dagger fallback: if `qapply_Mul` fails to simplify a Mul and `dagger=True`, takes Hermitian conjugate of the expression, re-applies, then conjugates back.
- **Density matrices**: `density.py` — `Density` class for mixed-state statistical ensembles (weighted collections of pure states).
  - `apply_op(op)` — applies an operator to each pure-state component while preserving weights; returns a new Density.
  - `doit()` — expands into outer-product form (Σ pᵢ|ψᵢ⟩⟨ψᵢ|); when a pure state is a superposition (Add), expands and enumerates all pairwise cross-terms via cartesian product. `states()`, `probs()` — extract components.
  - Module-level `entropy()` — von Neumann entropy; `fidelity()` — quantum state fidelity.
- **Operator–state mapping**: `operatorset.py` — bidirectional mapping between operator classes and their eigenstate classes.
  - `state_to_operators(state)` — maps a state (class or instance) to its observable operator(s); for Bra states not directly in the registry, resolves via `dual_class()` to look up the corresponding Ket entry.
  - `operators_to_state(operators)` — inverse mapping: operator(s) → eigenstate.
- **Circuit utilities**: `circuitutils.py` — primitive circuit manipulation: `find_subcircuit`/`replace_subcircuit` (KMP-based subsequence search/replace in gate tuples), `random_reduce(circuit, gate_ids)` (randomly removes a known gate identity from a circuit; returns original circuit unchanged if no identity is found), `random_insert` (inserts a random identity into a circuit), `flatten_ids` (expands GateIdentity objects into sorted list of equivalent sequences), `convert_to_symbolic_indices`/`convert_to_real_indices`.
- **QASM parser**: `qasm.py` — `Qasm` class: parses text-based gate descriptions into a quantum circuit. `add()` dispatches each command: user-defined custom operations (`self.defs`) take priority over built-in methods; unrecognized commands are skipped with a print warning. Built-in commands: `x`, `z`, `h`, `s`, `t`, `measure`, `cnot`, `swap`, `cphase`, `toffoli`, `cx`.
- **Other**: `tensorproduct.py`, `matrixcache.py`, `piab.py` (particle in a box), `constants.py` (ℏ).
  - `innerproduct.py` — `InnerProduct` expression node (⟨bra|ket⟩); constructor validates types and stores bra/ket.
    - `doit()` — evaluation dispatch with fallback: first tries ket's inner-product method; if that raises NotImplementedError, tries conjugate of the dual's inner-product; if both fail, returns self unevaluated.
    - `_eval_conjugate` — conjugate of ⟨a|b⟩ returns InnerProduct(Dagger(ket), Dagger(bra)), i.e. swaps and daggers both components.
    - Concrete overlap formulas (DiracDelta, plane-wave, etc.) live in `_eval_innerproduct_*` methods on state classes, not here.
  - `matrixutils.py` — matrix format conversion: `to_sympy`/`to_numpy`/`to_scipy_sparse` dispatch on input type (Matrix, ndarray, sparse, Expr); Expr inputs pass through unchanged.
  - `flatten_scalar` — extracts the element from a 1×1 matrix (returns larger matrices unchanged); does NOT check whether a circuit is a scalar matrix (see `identitysearch.py` for that).
  - Also: `matrix_dagger`, `matrix_tensor_product`, `matrix_zeros`.
  - `sho1d.py` — 1-D SHO operator algebra and states: `RaisingOp`/`LoweringOp` (ladder operators), `NumberOp`, `Hamiltonian`; base class enforces single-argument restriction (ValueError on multiple args). Ladder operators define `_eval_commutator_*` methods implementing canonical commutation relations ([a, a†] = 1).
    - State application: `RaisingOp` on |n⟩ → √(n+1)|n+1⟩; `LoweringOp` on |n⟩ → √n|n−1⟩; `LoweringOp` on ground state → 0.
    - `SHOKet`/`SHOBra` — ket/bra states for 1-D SHO; each provides `_represent_NumberOp` to produce column/row vectors in the number basis.
    - Caveat: error conditions (n ≥ ndim, non-integer n) use `return ValueError(...)` instead of `raise`, silently returning the error object as a value.
    - Each operator provides `_represent_NumberOp` for matrix representation in the number basis; supports sympy, numpy, and scipy.sparse formats. For scipy.sparse, sqrt entries are cast to float before insertion.
    - Position-basis representation (`_represent_XOp`) raises NotImplementedError for all operators (LoweringOp, RaisingOp, NumberOp, Hamiltonian) — underlying position representation logic is unimplemented.
  - `pauli.py` — Pauli spin-½ operators as quantum Operator subclasses: SigmaX/Y/Z (components), SigmaPlus (raising), SigmaMinus (lowering); optional string labels for subsystem identification. Operators with different labels commute (commutator returns zero).
    - Power simplification: `_eval_power` reduces exponent mod 2 (squaring any SigmaX/Y/Z yields identity). `SigmaMinus`/`SigmaPlus` are nilpotent: any positive integer power → 0.
    - `SigmaZKet`/`SigmaZBra` — two-level system states (n=0 or 1); operator application methods define action of each Pauli/ladder operator on states (e.g., raising operator on upper state → 0).
  - `qsimplify_pauli(e)` — simplifies products of Pauli operators by chaining pairwise reduction via `_qsimplify_pauli_product(a, b)`.
    - Same-label pairs: applies algebraic identities (e.g. σxσy=iσz, σx²=1, raising/lowering decompositions).
    - Different-label pairs: operators commute; sorted by label name (canonical ordering), not algebraically simplified.
    - Splits scalar coefficients from operator parts after each step.
  - `cartesian.py` — 1-D position/momentum operators (XOp, PxOp) and eigenstates (XKet/XBra, PxKet/PxBra), plus 3-D position operators (YOp, ZOp) and eigenstates (PositionKet3D/PositionBra3D). XOp defines `_eval_commutator_PxOp` implementing the canonical commutation relation [X, Px] = iℏ.
    - `PxOp._represent_XKet` — position-basis representation of momentum operator; uses `options.pop("index", 1)` as default start index for basis enumeration.
    - `PxKet._eval_innerproduct_XBra` — computes Fourier-kernel plane-wave overlap exp(i·p·x/ℏ)/√(2πℏ). `XKet._eval_innerproduct_PxBra` — conjugate overlap.
    - Same-basis inner products return DiracDelta; 3-D position states return product of three DiracDeltas.

### [`vector/`](vector/catalog.md)
Reference-frame-aware 3-D vector and dyadic algebra, kinematics, and calculus.
- `vector.py` — `Vector` class: 3-D vector with frame-aware arithmetic; owns its own `__str__`, `_latex`, and `_pretty` rendering (not delegated to `printing.py`).
  - `__str__`/`_latex`/`_pretty` — custom coefficient formatting: wraps `Add` (sum) coefficients in parentheses for readability; extracts leading minus signs for sign-aware concatenation.
  - `Vector.doit(**hints)` — propagates keyword hints (e.g. `deep=False`) element-wise to each scalar coefficient; `simplify()` and `subs()` follow the same per-component pattern.
  - `Vector.applyfunc(f)` — applies a user-supplied function to each scalar component; raises TypeError if `f` is not callable.
  - `Vector.diff(var, frame)` — partial derivative in a frame; three branches: same-frame → direct diff, cross-frame no DCM dependency → diff in place.
  - Cross-frame with DCM dependency on var → re-expresses into derivative frame, differentiates, then converts back. `var_in_dcm` flag controls this.
- `dyadic.py` — `Dyadic` class: tensor product of two vectors (used for rigid-body inertia); owns its own `__str__`, `_pretty`, and `_latex` rendering (not delegated to `printing.py`).
  - `__str__` — zero-coefficient terms are silently skipped (fall through all branches without output).
  - `_pretty` — wraps `Add` (sum) coefficients in parentheses via `.parens()` for readability; simple scalars printed directly.
- `frame.py` — `ReferenceFrame`: orientation, angular velocity, DCM computation, `partial_velocity(frame, *gen_speeds)` returns partial angular velocities (single speed → bare Vector; multiple → tuple).
- `point.py` — `Point`: position, velocity (`vel()`), acceleration (`acc()`) in reference frames; `partial_velocity(frame, *gen_speeds)` returns partial velocities (single speed → bare Vector; multiple → tuple of Vectors). Two-point (`v2pt_theory`) and one-point (`v1pt_theory`) velocity theorems.
  - `acc(frame)` fallback: if acceleration not explicitly set, differentiates velocity; if velocity is also zero, returns zero vector.
- `functions.py` — module-level vector utilities: `dot`, `cross`, `express`, `outer`, `kinematic_equations`, and a standalone `partial_velocity(vel_vecs, gen_speeds)` function operating on velocity lists (distinct from Point.partial_velocity).
  - `kinematic_equations(speeds, coords, rot_type, rot_order)` — generates angular velocity-to-generalized-speed kinematic differential equations for body/space Euler-angle rotations or quaternion parameterization. Quaternion mode raises ValueError if a rotation order is specified or if coordinate count ≠ 4.
  - `dynamicsymbols(names, level=0)` — creates time-dependent symbolic functions (UndefinedFunction of `t`); `level` applies pre-differentiation. Single name → bare expression; multiple names → list. Stores time variable as `dynamicsymbols._t`.
  - `get_motion_params(frame, **kwargs)` — computes acceleration/velocity/position from any one given; integrates using `_process_vector_differential`, which short-circuits when the input vector is zero (returns boundary condition directly without integrating).
- `fieldfunctions.py` — scalar/vector field operations: gradient, divergence, curl.

### [`optics/`](optics/catalog.md)
Geometric and wave optics.
- `gaussopt.py` — ray transfer matrices, geometric/Gaussian beam propagation, and paraxial conjugation utilities.
  - `RayTransferMatrix` subclasses for optical elements: `FreeSpace(d)`, `FlatRefraction(n1,n2)`, `CurvedRefraction(R,n1,n2)` (lower-left element = (n1−n2)/(R·n2)), `FlatMirror`, `CurvedMirror(R)`, `ThinLens(f)`.
  - `RayTransferMatrix.__mul__` — type-dispatching multiplication: Matrix×BeamParameter extracts q, applies ABCD transform, reconstructs BeamParameter from real/imaginary parts; Matrix×GeometricRay returns GeometricRay.
  - `GeometricRay` — 2×1 column vector (height, angle) for geometric ray; constructor accepts two scalars or a single 2×1 Matrix. Raises ValueError if a single argument has wrong dimensions (e.g. 2×2).
  - `BeamParameter`: complex beam parameter — waist (w_0), Rayleigh range, divergence, Gouy phase, `waist_approximation_limit` (minimum waist for paraxial validity).
  - `geometric_conj_ab(a, b)` — computes focal distance from two conjugation distances (object/image); returns the finite distance when either input is infinity.
  - `geometric_conj_af`, `geometric_conj_bf` — conjugation relations given one distance and focal length.
- `waves.py` — `TWave`: transverse sinusoidal wave in 1-D (amplitude, frequency/time_period, phase, refractive index n).
  - Properties: `speed` (propagation velocity = c/n, medium-dependent), `wavelength` (= c/(f·n)), `angular_velocity`, `wavenumber`.
  - Constructor requires at least one of frequency or time_period (raises ValueError); validates mutual consistency when both given.
- `medium.py` — `Medium` class: electromagnetic propagation material with refractive index (n), permittivity (ε), permeability (μ), intrinsic impedance (√(μ/ε)), and wave speed.
  - Constructor derives missing parameter when n plus one of ε/μ are given; raises ValueError on inconsistency when all three are provided. Caveat: consistency-check condition has a typo (`permittivity != None and permittivity != None` instead of checking permeability), causing incorrect branching when n + μ given without ε.
- `utils.py` — `refraction_angle()` (Snell's law vector form; returns 0 for total internal reflection). Accepts incident/normal as Matrix, Ray3D, or sequence; when both are Ray3D and no plane is given, validates geometric intersection — raises ValueError if rays are not concurrent. When a Plane is given, computes intersection point and returns a Ray3D result.
  - `deviation()` (angular deviation; returns None for total internal reflection), `brewster_angle()`, `critical_angle()`, `hyperfocal_distance()`.
  - `lens_makers_formula(n_lens, n_surr, r1, r2)` — thin-lens focal length; accepts Medium objects or numeric indices.
  - `lens_formula(f, u, v)` — standard thin-lens equation 1/f=1/v−1/u; solves for whichever of f/u/v is missing. No paraxial ray tracing or infinity handling; for ray-based conjugation see `gaussopt.py`.
  - `mirror_formula()` — analogous to `lens_formula` for mirrors.

### [`mechanics/`](mechanics/catalog.md)
Classical mechanics: particles, rigid bodies, equations of motion.
- `kane.py` — `KanesMethod`: Kane's equations of motion (Kane & Levinson 1985).
  - Constructor takes an inertial ReferenceFrame, generalized coordinates/speeds, kinematic differential equations, and optional constraint/dependent-speed specs; validates frame type.
  - Constraint initialization: partitions velocity-constraint Jacobian into independent/dependent columns; when acceleration constraints are not explicitly provided, auto-derives them by time-differentiating the velocity constraints.
  - Computes generalized active forces (fr) and generalized inertia forces (fr*). When dependent speeds are present, projects the full force vector onto independent speeds using a constraint transformation matrix.
  - `to_linearizer()` — converts Kane's EOM into `Linearizer` form; decomposes equations into kinematic (f_0, f_1) and dynamic (f_2, f_3) components by zeroing different variable groups.
    - Partitions coordinates/speeds into independent vs dependent sets. Validates coefficient matrices contain no unexpected dynamic symbols.
    - Raises ValueError if an external dynamic symbol and its time derivative both appear in the auto-discovered forcing terms.
  - Body list must contain only `RigidBody` or `Particle` (raises TypeError otherwise).
  - `linearize(**kwargs)` — public linearization entry point; dispatches between legacy and new interfaces via `new_method` kwarg: if missing/False, emits SymPyDeprecationWarning and calls `_old_linearize()`; if True, delegates to `to_linearizer().linearize()` and appends the forcing-symbol vector `r` to the result.
  - Legacy `_old_linearize` (deprecated) — in-place linearization via manual chain-rule Jacobian decomposition. Validates that system matrices (K_kqdot, K_ku, etc.) contain no unexpected dynamic symbols outside the forcing vector; raises ValueError if found. Also rejects derivatives of unrecognized dynamic symbols in forcing terms. Branches into four cases based on holonomic/non-holonomic constraints, computing dqd/dqi and dud/dui via LU-solving constraint Jacobians.
- `lagrange.py` — `LagrangesMethod`: generates equations of motion via Lagrange's variational method (EOM formulation, not energy computation). Constructor validates frame argument: raises TypeError if a non-null value is not a ReferenceFrame instance.
  - Constraint unification: holonomic (position-level) constraints are time-differentiated, then stacked with nonholonomic (velocity-level) constraints into a single constraint matrix (`coneqs`).
  - `mass_matrix` — dynamic mass matrix, augmented with Lagrange multiplier coefficients when constraints exist (n×(n+m)).
  - `mass_matrix_full` — full block-structured coefficient matrix: identity block (kinematic qdot relations) on top, mass_matrix row in middle, differentiated constraint rows on bottom when constraints present.
  - `forcing` / `forcing_full` — generalized forcing vector; `forcing_full` augments with qdots and differentiated constraint forcing terms.
  - `solve_multipliers(op_point)` — solves for Lagrange multiplier values at a given operating point by composing the mass matrix with constraint coefficients and LU-solving.
  - `to_linearizer()` — converts to `Linearizer` form; raises ValueError if an external dynamic symbol and its time derivative both appear in forcing terms.
- `particle.py` — `Particle`: point mass with `linear_momentum`, `angular_momentum(point, frame)`, `kinetic_energy(frame)` (computes ½mv² via velocity dot product) methods.
- `rigidbody.py` — `RigidBody`: rigid body with `angular_momentum(point, frame)` (H = I·ω + r×mv), `linear_momentum`, `potential_energy`.
  - `inertia` setter — accepts (Dyadic, Point) tuple; applies parallel axis theorem in reverse to compute central inertia: subtracts point-mass contribution (`inertia_of_point_mass`) from given inertia.
  - `kinetic_energy(frame)` — individual body KE: ½I·ω² (rotational) + ½mv² (translational).
- `body.py` — unified `Body` wrapping Particle or RigidBody; constructor dispatches: mass given but no inertia → initializes as Particle; otherwise → RigidBody with symbolic inertia tensor.
- `functions.py` — system-level kinematic/dynamic functions for multi-body systems (computation, not EOM generation).
  - `angular_momentum(point, frame, *body)` — sums angular momenta of Particles/RigidBodies; validates Point and ReferenceFrame types.
  - `linear_momentum`, `kinetic_energy`, `potential_energy` — system-level aggregators that sum per-body contributions (delegate to each body's own method).
  - `Lagrangian(frame, *body)` — computes T−V (kinetic minus potential energy) for a collection of Particles/RigidBodies in a given frame; returns a scalar expression.
  - `find_dynamicsymbols(expression, exclude=None)` — finds all time-dependent (`dynamicsymbols`) in an expression; optional `exclude` kwarg must be iterable (raises TypeError if a bare symbol is passed instead of a list).
- `linearize.py` — `Linearizer`: first-order approximation of constrained multi-body EOM; handles dependent coordinates/speeds.
  - Constructor detects when time-derivatives of q overlap with u symbols and substitutes Dummy variables to avoid conflicts.
  - `_form_coefficient_matrices()` — builds projection matrices C_0, C_1, C_2 that account for holonomic/nonholonomic constraints.
    - Holonomic constraints present (l>0): C_0 is a Jacobian-based projection via LU-solve; absent: C_0 = identity.
    - Nonholonomic constraints present (m>0): C_1, C_2 computed from velocity-constraint Jacobian via LU-solve; absent: C_1 = zero, C_2 = identity.
  - `_form_block_matrices()` — conditionally computes Jacobian sub-blocks (M_qq, A_qq, M_uuc, etc.) based on nonzero dimension counts.
- `models.py` — pre-built example multi-body systems for testing/demos; **not exported by `__init__.py`** — must be imported explicitly (`from sympy.physics.mechanics.models import ...`). `n_link_pendulum_on_cart()` builds a 2-D n-link pendulum on a sliding cart; `specified` inputs list becomes None (not empty list) when both lateral force and joint torques are disabled. `multi_mass_spring_damper()` builds a chain of masses connected by springs and dampers.

### [`hep/`](hep/catalog.md)
High-energy physics.
- `gamma_matrices.py` — `GammaMatrixHead`: Dirac gamma-matrix algebra using tensor infrastructure.
  - `simplify_lines(ex)` — top-level simplifier: decomposes a product into independent open spinor chains, closed loops (traces), and non-gamma rest; simplifies each part separately and recombines.
  - `simplify_gpgp(ex)` — reduces contracted slash-notation pairs (G·p·G·p → p·p) by detecting adjacent gamma-momentum contractions sharing the same vector.
  - `extract_type_tens(expression)` — separates gamma-matrix tensors from non-gamma tensors in an expression; accepts single `Tensor` or `TensMul` only, raises `ValueError('wrong type')` for other expression types (e.g. `TensAdd` sums).
  - `_trace_single_line` — evaluates fermion-line traces; returns hardcoded 4 (D=4 only) when the line contains only a spinor identity (delta) and no gamma matrices.
  - `_gamma_trace1` — computes trace of gamma-matrix products; returns 4 for empty trace (identity).
  - `_kahane_simplify` — cancels contracted (dummy-index) gamma matrices using Kahane's algorithm; inserts virtual indices to handle consecutive dummy indices with no free indices between them.
    - Validates component ordering: raises ValueError if contracted gamma matrices are not adjacent (component position distance must be 1 or n−1). Also validates spinor-index free pairs (must be exactly 0 or 2 with correct slot positions).

### [`unitsystems/`](unitsystems/catalog.md)
Dimensional analysis and unit systems (SI, CGS, natural, etc.).
- `dimensions.py` — `Dimension` class: represents dimensional exponents (mass, length, time, …) as a filtered dict; constructor strips zero-valued exponents so `Dimension(length=1, mass=0) == Dimension(length=1)`. Supports mul/div/pow composition and dimensional equality checks.
  - `add(other)` — dimensional addition; raises TypeError for non-Dimension operand, raises ValueError when dimensions differ (e.g., length + time).
  - `sub(other)` — subtraction delegates to `add`; dimensions have no notion of ordering/magnitude, so subtraction is equivalent to addition when operands match.
  - `DimensionSystem.get_dim(dim)` — looks up a dimension in the system; accepts string (matches against name or symbol) or Dimension object (matches by identity in list); returns None if not found. `__getitem__` shortcut raises KeyError on miss.
  - `DimensionSystem.print_dim_base(dim)` — formats a dimension as a human-readable string in terms of basis dimensions, sorted by decreasing power; skips zero-power, omits exponent for power=1.
- `units.py` — `Unit` class and `UnitSystem` (coherent unit set).
  - `Unit` — measurement standard with dimension and scaling factor; arithmetic: `add`/`sub` (same-dimension only), `mul`/`div` (compose dimensions), `pow(other)` — numeric exponent → new Unit with raised dimension; symbolic (non-numeric) exponent → returns unevaluated `Pow(self, other)`.
  - `UnitSystem.__init__` — validates base-unit consistency via the underlying `DimensionSystem`, then reorders base units to match the dimension system's base-dimension ordering for deterministic storage.
  - `UnitSystem.get_unit(unit)` — looks up a unit in the system; string input matches only against `abbrev` (not full name — noted as TODO limitation); Unit input matches by identity; returns None if not found. `__getitem__` shortcut raises KeyError on miss.
  - `UnitSystem.__call__` dispatches on argument type: Dimension → base-dimension string, Unit → base-unit string, Quantity → formatted "factor unit" string.
- `quantities.py` — `Quantity`: physical quantity with numeric factor and unit. Arithmetic: `add`/`sub` (same-unit only, auto-converts), `mul`/`div`/`rdiv`, `pow`. `pow(other)` calls `evalf()` on the computed factor because symbolic Pow instances are incompatible with the Quantity constructor.
- `prefixes.py` — `Prefix` class for SI/binary scale multipliers; arithmetic (`__mul__`, `__div__`, `__rdiv__`) between two Prefixes looks up the combined factor in the global PREFIXES dict, returning the raw numeric factor if no predefined prefix matches. `__rdiv__` handles `1/prefix` by searching PREFIXES for the inverse factor.
- `simplifiers.py` — `dim_simplify`: recursive simplification of compound `Dimension` expressions (Add, Mul, Pow). Handles the CAS rewriting `Add(L,L)→Mul(2,L)` by stripping non-Dimension numeric factors from Mul before reducing. Also `qsimplify` for Quantity expressions.
- `systems/` — concrete unit-system definitions:
  - `mks.py` — meter-kilogram-second. Creates a separate prefix-free gram unit alongside the kilogram base unit.
    - Gram (not kilogram) is fed into the SI-prefix generation loop; applying prefixes to an already-prefixed unit would not produce correct scaled variants.
    - Derived units J/N/W/Pa carry factor=10³ to convert from gram-based to kilogram-based scaling.
  - `mksa.py` — MKS + ampere; defines electromagnetic Dimension objects (current, voltage, impedance, conductance, capacitance, inductance, charge, magnetic_density, magnetic_flux) and units (A/V/ohm/S/F/H/C/T/Wb).
  - `natural.py` — natural unit system (c=ℏ=1): redefines base dimensions as action, energy, velocity; length, mass, time become derived quantities. Base units: ℏ (action), eV (energy), c (velocity).
