# Physics Module Catalog

## Architecture Overview

The `physics` package provides symbolic physics across classical mechanics, optics, quantum mechanics, and unit systems. Top-level `.py` files handle standalone physics topics (hydrogen atom, Wigner symbols, Pauli algebra, second quantization, harmonic oscillators). Submodules group larger domains: `quantum/` (abstract quantum mechanics, gates, spin, algorithms), `vector/` (reference-frame-aware 3-D vectors and kinematics), `optics/` (geometric/wave optics), `mechanics/` (Lagrangian/Kane dynamics), `hep/` (high-energy physics), and `unitsystems/` (dimensional analysis).

---

## Top-Level Files

### [`hydrogen.py`](hydrogen.py)
Radial wavefunctions for hydrogen-like atoms.
- `R_nl(n, l, r, Z=1)` — radial wavefunction with quantum numbers n, l and atomic number Z.

### [`matrices.py`](matrices.py)
Standard physics matrices as SymPy Matrix objects.
- `msigma(i)` — 2×2 Pauli spin matrix σ_i (i = 1, 2, 3).
- `mgamma(mu, lower=False)` — 4×4 Dirac gamma matrix γ^μ (standard/Dirac representation); `lower=True` returns γ_μ by negating spatial (1,2,3) and chiral (5) indices (metric signature +−−−).
- `pat_matrix(x, y, z, a)` — Parallel Axis Theorem inertia matrix.
- `mdft(n)` — n×n discrete Fourier transform matrix.
- `minkowski_tensor` — 4×4 Minkowski metric tensor.

### [`paulialgebra.py`](paulialgebra.py)
Pauli matrix algebra via pure symbolic manipulation (Symbol subclass, not quantum operators or states). No kets, bras, or operator framework — for those see `quantum/pauli.py`.
- `Pauli(i)` — Symbol subclass representing σ_i; algebraic multiplication yields products and scalar I automatically.
- `delta(i, j)` — Kronecker delta helper; returns 1 if i == j, else 0.
- `epsilon(i, j, k)` — Levi-Civita symbol helper; returns +1 for even permutations of (1,2,3), −1 for odd, 0 otherwise (including repeated indices).
- `evaluate_pauli_product(arg)` — simplifies a product of Pauli matrices using algebraic rules.

### [`pring.py`](pring.py)
Quantum particle on a ring.
- `wavefunction(n, x)` — eigenfunctions. `energy(n, m, r)` — energy levels.

### [`qho_1d.py`](qho_1d.py)
One-dimensional quantum harmonic oscillator: closed-form analytical wavefunctions and energy formulas (no operator algebra).
- `psi_n(n, x, m, omega)` — spatial wavefunction ψ_n using Hermite polynomials.
- `E_n(n, omega)` — energy eigenvalue ℏω(n+½).
- `coherent_state(n, alpha)` — Fock-basis expansion coefficient ⟨n|α⟩ for a coherent state; computes exp(−|α|²/2)·α^n/√(n!).

### [`secondquant.py`](secondquant.py)
Second quantization framework for many-body quantum mechanics — integer-occupation-number bosonic/fermionic operators (distinct from abstract quantum operators in `quantum/`).
- `BosonicOperator`, `CreateBoson` (B†), `AnnihilateBoson` (B) — bosonic ladder operators with commutation relations.
- `FermionicOperator`, `CreateFermion` (Fd), `AnnihilateFermion` (F) — many-body fermionic ladder operators with fixed anticommutation rules (not mode-labeled like `quantum/fermion.py`).
- `NO` — normal-ordering bracket; constructor reorders operators into creation-before-annihilation form, returns S.Zero if identical fermion operators violate Pauli exclusion.
- `Commutator`, `AntiCommutator` — many-body (anti)commutator wrappers (for the abstract quantum operator versions, see `quantum/commutator.py` and `quantum/anticommutator.py`).
- `FockState`, `FockStateKet`, `FockStateBra` — Fock-space state vectors.
- `wicks(expr)` — applies Wick's theorem to expand operator products into normal-ordered contractions.
- `Dagger` — Hermitian conjugate of creation/annihilation operators; `eval()` dispatches: reverses factor order for products (Mul), distributes over sums, conjugates base of powers, negates I.
- `apply_operators()` — applies operators to states. `evaluate_deltas()` — simplifies Kronecker delta products.
- `contraction(a, b)` — evaluates the contraction of two operators.
- `matrix_rep(op, basis)` — matrix representation in a Fock basis.

### [`sho.py`](sho.py)
3-D isotropic quantum harmonic oscillator.
- `R_nl(n, l, nu, r)` — radial wavefunctions with associated Laguerre polynomials.

### [`units.py`](units.py)
Legacy physical-units module with ~200 predefined units and `find_unit()` search.

### [`wigner.py`](wigner.py)
Exact angular-momentum coupling coefficients (returns rationals × √rational).
- `wigner_3j(j1,j2,j3,m1,m2,m3)` — Wigner 3-j symbol; includes selection-rule short-circuits and a guard that strips imaginary parts if the factorial square-root evaluates to complex.
- `wigner_6j`, `wigner_9j` — higher-order recoupling coefficients.
- `clebsch_gordan(j1,j2,j3,m1,m2,m3)` — Clebsch-Gordan coefficient (wrapper around wigner_3j).
- `racah(aa,bb,cc,dd,ee,ff)` — Racah W-coefficient.
- `gaunt(l1,l2,l3,m1,m2,m3)` — Gaunt coefficient (integral of three spherical harmonics).

### [`gaussopt.py`](gaussopt.py)
Deprecated — redirects to `sympy.physics.optics.gaussopt`.

---

## Submodules

### [`quantum/`](quantum/catalog.md)
Abstract quantum mechanics framework: states, operators, Hilbert spaces, representations, and quantum-information primitives.
- **Core**: `qexpr.py` (base quantum expression), `operator.py` (Operator/Hermitian/Unitary), `hilbert.py` (Hilbert spaces).
  - `represent.py` — `represent(expr, basis)`: converts quantum expressions to matrix form. Fallback chain: if `_represent()` raises NotImplementedError, tries `rep_innerproduct` for Ket/Bra or `rep_expectation` for Operator; re-raises if fallback also fails.
  - `state.py` — Ket/Bra/Wavefunction; `KetBase.__mul__`/`__rmul__` dispatch multiplication: Ket*Bra → OuterProduct, Bra*Ket → InnerProduct.
    - `StateBase._represent_default_basis` — determines default representation basis by querying which operators the state is an eigenstate of (lazy-imports `operatorset` to break circular dependency).
- **Angular momentum / CG**: `cg.py` — Clebsch-Gordan and Wigner coupling coefficient symbolic expressions, evaluation, and simplification (not state construction).
  - `Wigner3j` — symbolic Wigner 3-j coefficient; `is_symbolic` property checks if any parameter is non-numeric. `doit()` raises ValueError for symbolic params.
  - `Wigner6j`, `Wigner9j` — symbolic 6-j and 9-j coefficients with analogous `doit()`.
  - `CG` (subclass of `Wigner3j`) — Clebsch-Gordan coefficient; inherits `is_symbolic` check and ValueError guard.
  - `_check_cg()` — validates whether a candidate term matches a structural Wild pattern and expected sign convention (sign tuple comparison after substitution).
  - Simplification rules apply orthogonality-relation identities to reduce CG products summed over j,m to Kronecker deltas.
- **Spin**: `spin.py` — spin operators (Jx, Jy, Jz, J±, J²), coupled/uncoupled states, Wigner-D/d matrices, `Rotation` operator (Euler-angle unitary).
  - `J2Op` — total angular momentum squared (Casimir) operator; commutes with all component operators and applies eigenvalue ℏ²j(j+1).
  - `Rotation` — Euler-angle rotation operator; `_apply_operator_uncoupled` applies to kets: enumerates D-matrix elements for numeric j, returns symbolic Sum for symbolic j.
  - `CoupledSpinState` — coupled state constructor with triangle-inequality validation on coupling schemes.
  - `couple()`/`_couple()` — combines uncoupled spin states into coupled representation; validates custom coupling order: after two spaces couple, the result must be referenced by the smaller index (raises ValueError otherwise).
  - Numeric path enumerates configurations, filters non-physical ones via triangle inequality (|j1−j2|≤j3≤j1+j2) and |m|≤j checks before computing CG coefficients.
  - `uncouple()`/`_uncouple()` — decomposes coupled eigenstates into sums of tensor-product states weighted by CG coefficients; enumerates valid magnetic projection configurations for numeric quantum numbers.
- **Gates**: `gate.py` — quantum gate classes (H, X, Y, Z, S/Phase, T, CNOT, SWAP, CGate, UGate); each gate stores target matrices, commutation relations, and decomposition methods.
  - `CGate` — controlled gate; wraps an inner gate with control qubits. When inner gate is Hermitian: dagger/inverse return self; power with even exponent → identity, odd → self.
  - `Gate._eval_hilbert_space` — determines smallest Hilbert space from target qubit indices: ComplexSpace(2)^(max_target+1).
  - `gate_sort(circuit)` — bubble-sorts gates respecting commutation; swaps commuting gates freely, applies (−1)^(exp1·exp2) sign correction when anticommutator vanishes.
- **Circuit plotting**: `circuitplot.py` — `CircuitPlot` for rendering circuits; `CreateCGate(name, latexname=None)` factory for dynamically creating controlled gates (defaults latexname to name if omitted); mock measurement gates `Mz`, `Mx`.
- **Circuit identity search**: `identitysearch.py` — `generate_gate_rules(gate_seq)` finds equivalent gate rewriting rules via BFS; returns trivial rule set when input is a plain numeric scalar. `generate_equivalent_ids()` finds equivalent gate identities.
  - `ll_op`, `lr_op`, `rl_op`, `rr_op` — elementary rule-rewriting operations: each removes a gate from one end of one side of an equation and left/right-multiplies both sides by its dagger.
- **Second-quantized QM operators**: `boson.py` — bosonic creation/annihilation operator algebra and quantum states for bosonic modes.
  - `BosonOp` — bosonic ladder operator; custom `__mul__` separates commutative from non-commutative factors when multiplying into product expressions.
  - `BosonFockKet/Bra` — Fock number states with KroneckerDelta inner product; `BosonCoherentKet/Bra` — coherent states with Gaussian overlap inner product.
  - `fermion.py` — `FermionOp`: mode-labeled fermionic ladder operator.
    - `_eval_anticommutator_FermionOp`: returns 1 for {a†,a} same-name; None for same-name same-type; `independent` hint checked only for different names.
- **Operator ordering**: `operatorordering.py` — `normal_ordered_form()` rearranges products into creation-before-annihilation order; when two operators belong to different independent modes, the commutation correction term is set to zero.
- **Commutator algebra**: `commutator.py`, `anticommutator.py` — abstract quantum `Commutator`/`AntiCommutator` with `doit()` evaluation.
  - Delegates to operator `_eval_commutator_*`/`_eval_anticommutator_*` methods; falls back through NotImplementedError chain.
- **Algorithms**: `grover.py` (Grover's search), `shor.py` (Shor's factoring), `qft.py` (quantum Fourier transform).
- **Qubits**: `qubit.py` — `Qubit`, `IntQubit`, qubit-state manipulation, measurement, and partial trace.
  - `Qubit._eval_trace(bra, indices)` — partial trace over selected subsystem indices; sorts indices to trace from most-significant qubit, returns scalar for full trace or density operator for partial trace.
  - `matrix_to_qubit(matrix)` — converts a numerical column/row vector into a symbolic superposition of basis states; determines Ket vs Bra from matrix shape.
  - `measure_all`/`measure_partial` — ensemble and partial qubit measurement.
- **Operator application**: `qapply.py` — `qapply(e)` symbolically applies operators to states in an expression; dispatches by expression type (Add, Mul, TensorProduct, Density, Pow).
  - `qapply_Mul` handles products by trying lhs._apply_operator(rhs), then rhs._apply_operator(lhs), then forming InnerProduct for Bra·Ket pairs.
  - Dagger fallback: if `qapply_Mul` fails to simplify a Mul and `dagger=True`, takes Hermitian conjugate of the expression, re-applies, then conjugates back.
- **Density matrices**: `density.py` — `Density` class for mixed-state statistical ensembles (weighted collections of pure states).
  - `apply_op(op)` — applies an operator to each pure-state component while preserving weights; returns a new Density.
  - `doit()` — expands into outer-product form (Σ pᵢ|ψᵢ⟩⟨ψᵢ|). `states()`, `probs()` — extract components.
  - Module-level `entropy()` — von Neumann entropy; `fidelity()` — quantum state fidelity.
- **Other**: `tensorproduct.py`, `innerproduct.py`, `matrixcache.py`, `circuitutils.py`, `piab.py` (particle in a box), `constants.py` (ℏ).
  - `matrixutils.py` — matrix format conversion: `to_sympy`/`to_numpy`/`to_scipy_sparse` dispatch on input type (Matrix, ndarray, sparse, Expr); Expr inputs pass through unchanged.
  - Also: `flatten_scalar`, `matrix_dagger`, `matrix_tensor_product`, `matrix_zeros`.
  - `sho1d.py` — 1-D SHO operator algebra: `RaisingOp`/`LoweringOp` (ladder operators), `NumberOp`, `Hamiltonian`; base class enforces single-argument restriction (ValueError on multiple args). `LoweringOp` applied to ground state returns zero.
  - `pauli.py` — Pauli spin operators as quantum Operator subclasses (SigmaX/Y/Z, SigmaPlus/SigmaMinus) with optional string labels; operators with different labels commute (commutator returns zero).
    - Power simplification: `_eval_power` reduces exponent mod 2 (squaring any SigmaX/Y/Z yields identity).
    - `SigmaZKet`/`SigmaZBra` — two-level system states (n=0 or 1); operator application methods define action of each Pauli/ladder operator on states (e.g., raising operator on upper state → 0).
  - `qsimplify_pauli(e)` — simplifies products of Pauli operators by chaining pairwise reduction, splitting scalar coefficients from operator parts after each step.
  - `cartesian.py` — 1-D position/momentum eigenstates (XKet/XBra, PxKet/PxBra) with DiracDelta and plane-wave inner products; 3-D position eigenstates (PositionKet3D/PositionBra3D) with three-dimensional DiracDelta orthogonality.

### [`vector/`](vector/catalog.md)
Reference-frame-aware 3-D vector and dyadic algebra, kinematics, and calculus.
- `vector.py` — `Vector` class. `dyadic.py` — `Dyadic` class.
- `frame.py` — `ReferenceFrame`: orientation, angular velocity, DCM computation.
- `point.py` — `Point`: position, velocity (`vel()`), acceleration in reference frames; `partial_velocity(frame, *gen_speeds)` returns partial velocities (single speed → bare Vector; multiple → tuple of Vectors). Two-point (`v2pt_theory`) and one-point (`v1pt_theory`) velocity theorems.
- `functions.py` — module-level vector utilities: `dot`, `cross`, `express`, `outer`, and a standalone `partial_velocity(vel_vecs, gen_speeds)` function operating on velocity lists (distinct from Point.partial_velocity).
- `fieldfunctions.py` — scalar/vector field operations: gradient, divergence, curl.

### [`optics/`](optics/catalog.md)
Geometric and wave optics.
- `gaussopt.py` — ray transfer matrices, geometric/Gaussian beam propagation.
- `waves.py` — `TWave` class for transverse electromagnetic waves.
- `medium.py` — `Medium` class (refractive index, permittivity, permeability).
- `utils.py` — `refraction_angle()` (Snell's law vector form; returns 0 for total internal reflection), `deviation()` (angular deviation through a planar interface; returns None when total internal reflection occurs), `lens_makers_equation()`, `brewster_angle()`, `critical_angle()`, `lens_formula()`, `mirror_formula()`, `hyperfocal_distance()`.

### [`mechanics/`](mechanics/catalog.md)
Classical mechanics: particles, rigid bodies, equations of motion.
- `kane.py` — `KanesMethod`: Kane's equations of motion; computes generalized active forces (fr) and generalized inertia forces (fr*). Body list must contain only `RigidBody` or `Particle` (raises TypeError otherwise).
- `lagrange.py` — `LagrangesMethod`: generates equations of motion via Lagrange's method (EOM formulation, not energy computation).
  - `solve_multipliers(op_point)` — solves for Lagrange multiplier values at a given operating point by composing the mass matrix with constraint coefficients and LU-solving.
  - `to_linearizer()` — converts to `Linearizer` form; raises ValueError if an external dynamic symbol and its time derivative both appear in forcing terms.
- `particle.py` — `Particle`: point mass with `linear_momentum`, `angular_momentum(point, frame)`, `kinetic_energy` methods.
- `rigidbody.py` — `RigidBody`: rigid body with `angular_momentum(point, frame)` (H = I·ω + r×mv combining spin and translational contributions), `linear_momentum`, `kinetic_energy`, `potential_energy` methods.
- `body.py` — unified `Body` wrapping Particle or RigidBody.
- `functions.py` — system-level kinematic/dynamic functions for multi-body systems (computation, not EOM generation).
  - `angular_momentum(point, frame, *body)` — sums angular momenta of Particles/RigidBodies; validates Point and ReferenceFrame types.
  - `linear_momentum`, `kinetic_energy`, `potential_energy` — analogous system-level aggregators.
  - `Lagrangian(frame, *body)` — computes T−V (kinetic minus potential energy) for a collection of Particles/RigidBodies in a given frame; returns a scalar expression.
- `linearize.py` — `Linearizer`: first-order approximation of constrained multi-body EOM; handles dependent coordinates/speeds. Constructor detects when time-derivatives of q overlap with u symbols and substitutes Dummy variables to avoid conflicts.
- `models.py` — pre-built example multi-body systems for testing/demos. `n_link_pendulum_on_cart()` builds a 2-D n-link pendulum on a sliding cart; `specified` inputs list becomes None (not empty list) when both lateral force and joint torques are disabled. `multi_mass_spring_damper()` builds a chain of masses connected by springs and dampers.

### [`hep/`](hep/catalog.md)
High-energy physics.
- `gamma_matrices.py` — `GammaMatrixHead`: Dirac gamma-matrix algebra using tensor infrastructure.
  - `_trace_single_line` — evaluates fermion-line traces; returns hardcoded 4 (D=4 only) when the line contains only a spinor identity (delta) and no gamma matrices.
  - `_gamma_trace1` — computes trace of gamma-matrix products; returns 4 for empty trace (identity).
  - `_kahane_simplify` — cancels contracted gamma matrices using Kahane's algorithm.

### [`unitsystems/`](unitsystems/catalog.md)
Dimensional analysis and unit systems (SI, CGS, natural, etc.).
- `dimensions.py` — `Dimension` class: represents dimensional exponents (mass, length, time, …) as a filtered dict; constructor strips zero-valued exponents so `Dimension(length=1, mass=0) == Dimension(length=1)`. Supports mul/div/pow composition and dimensional equality checks.
- `units.py` — `Unit` class and `UnitSystem` (coherent unit set); `UnitSystem.__call__` dispatches on argument type: Dimension → base-dimension string, Unit → base-unit string, Quantity → formatted "factor unit" string.
- `quantities.py` — `Quantity`: physical quantity with numeric factor and unit.
- `prefixes.py` — `Prefix` class for SI/binary scale multipliers; arithmetic (`__mul__`, `__div__`) between two Prefixes looks up the combined factor in the global PREFIXES dict, returning the raw numeric factor if no predefined prefix matches.
- `simplifiers.py` — `dim_simplify`: algebraic simplification of compound `Dimension` expressions (products/powers); does not alter individual Dimension construction or equality.
