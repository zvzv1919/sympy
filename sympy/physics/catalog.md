# sympy/physics — Physics

Symbolic physics: quantum mechanics, classical mechanics, optics, unit/dimension systems, angular momentum coupling, second quantization, and standard physical constants/matrices.

## Glossary

- **Dyadic**: A tensor-like object used to represent inertia in `vector`/`mechanics`.
- **Fock state**: Quantum state described by occupation numbers (bosons) or occupied orbits (fermions).
- **Normal ordering**: Rearranging creation/annihilation operators so all creators are to the left.
- **Wigner symbols**: Coupling coefficients for angular momenta (3j, 6j, 9j).
- **Ray transfer matrix**: 2×2 ABCD matrix describing paraxial optical elements.

---

## Root-Level Files

### `__init__.py`

Package init — re-exports `units` and `matrices` (mgamma, msigma, minkowski_tensor, mdft).

### `hydrogen.py`

Hydrogen atom wavefunctions and energies in Hartree atomic units.

- `R_nl(n, l, r, Z=1)` — Radial wavefunction R_{nl} using associated Laguerre polynomials.
- `E_nl(n, Z=1)` — Non-relativistic energy of state (n, l). Independent of l.
- `E_nl_dirac(n, l, spin_up, Z, c)` — Relativistic (Dirac) energy including fine structure. Rest mass energy excluded.

### `sho.py`

3D isotropic harmonic oscillator wavefunctions and energies.

- `R_nl(n, l, nu, r)` — Radial wavefunction using associated Laguerre polynomials. `nu = m*omega/(2*hbar)`.
- `E_nl(n, l, hw)` — Energy: `(2n + l + 3/2) * hw`.

### `qho_1d.py`

1D quantum harmonic oscillator.

- `psi_n(n, x, m, omega)` — Wavefunction using Hermite polynomials.
- `E_n(n, omega)` — Energy: `hbar * omega * (n + 1/2)`.
- `coherent_state(n, alpha)` — Returns `<n|alpha>` for coherent states.

### `pring.py`

Particle on a ring (1D periodic potential).

- `wavefunction(n, x)` — Normalized wavefunction `exp(i*n*x) / sqrt(2*pi)`.
- `energy(n, m, r)` — Energy: `n^2 * hbar^2 / (2*m*r^2)`.

### `matrices.py`

Standard physics matrices.

- `msigma(i)` — Pauli matrix σ_i (i = 1, 2, 3).
- `pat_matrix(m, dx, dy, dz)` — Parallel axis theorem matrix for translating inertia.
- `mgamma(mu, lower=False)` — Dirac gamma matrix γ^μ (standard representation), μ ∈ {0,1,2,3,5}.
- `minkowski_tensor` — Minkowski metric with signature (+, −, −, −).
- `mdft(n)` — n×n discrete Fourier transform matrix.

### `paulialgebra.py`

Algebraic (non-matrix) Pauli algebra via `Symbol` subclassing.

- `class Pauli(Symbol)` — Algebraic Pauli matrix with `__mul__` encoding σ_i σ_j = δ_ij + i ε_ijk σ_k.
- `evaluate_pauli_product(arg)` — Evaluates a symbolic product of Pauli matrices.
- `delta(i, j)`, `epsilon(i, j, k)` — Kronecker delta and Levi-Civita helpers.

### `wigner.py`

Exact computation of angular momentum coupling coefficients.

- `wigner_3j(j1, j2, j3, m1, m2, m3)` — Wigner 3-j symbol (exact rational × √rational).
- `clebsch_gordan(j1, j2, j3, m1, m2, m3)` — Clebsch-Gordan coefficient via 3-j.
- `wigner_6j(j1..j6)` — Wigner 6-j symbol.
- `wigner_9j(j1..j9)` — Wigner 9-j symbol.
- `racah(aa, bb, cc, dd, ee, ff)` — Racah W-coefficient.
- `gaunt(l1, l2, l3, m1, m2, m3)` — Gaunt coefficient (integral over 3 spherical harmonics).
- `class Wigner3j(Function)` — Unevaluated symbolic wrapper; `.doit()` computes numerically.
- `dot_rot_grad_Ynm(j, p, l, m, theta, phi)` — Dot product of rotational gradients of spherical harmonics.

### `secondquant.py`

Second quantization operators and states for bosons and fermions (Fetter & Walecka formulation).

- **Operators**: `class Dagger`, `class SqOperator`, `class BosonicOperator`, `class FermionicOperator`
  - `AnnihilateBoson` / `CreateBoson` (aliases `B`, `Bd`)
  - `AnnihilateFermion` / `CreateFermion` (aliases `F`, `Fd`)
  - Fermionic operators carry `is_q_creator`, `is_q_annihilator`, `is_above_fermi`, `is_below_fermi` properties for Wick's theorem.
- **States**: `FockState`, `BosonState`, `FermionState`, `FockStateBosonKet/Bra`, `FockStateFermionKet/Bra` (aliases `BKet`, `BBra`, `FKet`, `FBra`)
- **Wick's theorem**:
  - `wicks(e, **kw_args)` — Normal ordered form via Wick's theorem. Supports `keep_only_fully_contracted`, `simplify_kronecker_deltas`, `simplify_dummies`.
  - `contraction(a, b)` — Computes contraction between two fermionic operators.
  - `NO(arg)` — Normal ordering brackets.
  - `Commutator` — `[A, B]` with auto-evaluation for single second-quantization operators.
- **Index utilities**:
  - `evaluate_deltas(e)` — Einstein-summation KroneckerDelta evaluation.
  - `substitute_dummies(expr)` — Collect terms by consistent dummy relabeling.
  - `AntiSymmetricTensor` — Tensor with automatically antisymmetric index groups.
  - `PermutationOperator` / `simplify_index_permutations` — Factor out index permutation symmetries.
- **Basis sets**: `VarBosonicBasis`, `FixedBosonicBasis`, `matrix_rep(op, basis)`.

### `units.py`

Legacy flat unit system (~200 units, SI-based). `Unit` subclasses `AtomicExpr`.

- Defines base units (m, kg, s, A, K, mol, cd) and derived units (N, J, W, Pa, etc.).
- Physical constants: `c`, `G`, `planck`, `hbar`, `avogadro_number`, `boltzmann`, `eV`, etc.
- `find_unit(quantity)` — Search for units by name string or matching base dimensions.

### `gaussopt.py`

**Deprecated** — redirects to `sympy.physics.optics.gaussopt`.

---

## HEP (High Energy Physics) — `hep/`

### `hep/gamma_matrices.py`

Symbolic Dirac gamma matrix algebra using the tensor module.

- `class GammaMatrixHead(TensorHead)` — Wraps a `TensorHead` for γ matrices in D dimensions. Singleton-cached per `(dim, eps_dim)`.
  - `simplify_gpgp(ex)` — Simplifies `G(i)*p(-i)*G(j)*p(-j) → p(i)*p(-i)`.
  - `simplify_lines(ex)` — Simplifies products along gamma matrix lines/traces.
  - `gamma_trace(t)` — Evaluate trace of a single gamma matrix line.
  - `_kahane_simplify(coeff, tids)` — Kahane's algorithm for canceling contracted gamma matrices.
  - `_trace_single_line(t)` — Recursive trace evaluation (uses identity for 2 and 4 gammas, recurses for more).
- `GammaMatrix` — Pre-built 4D instance: `GammaMatrixHead()`.
- `DiracSpinorIndex` — 4D spinor `TensorIndexType`.
- `_LorentzContainer` — Helper caching Lorentz `TensorIndexType` by dimension.

---

## Classical Mechanics — `mechanics/`

### `mechanics/particle.py`

- `class Particle` — Point mass with name, Point, and mass. Methods: `linear_momentum`, `angular_momentum`, `kinetic_energy`; property `potential_energy`.

### `mechanics/rigidbody.py`

- `class RigidBody` — Rigid body container (name, mass center, frame, mass, inertia dyadic + point). Methods: `linear_momentum`, `angular_momentum`, `kinetic_energy`; properties `central_inertia`, `potential_energy`.

### `mechanics/body.py`

- `class Body(RigidBody, Particle)` — Unified representation. Auto-generates frame, mass center, and inertia symbols when not provided. Creates a `Particle` if only mass center + mass given, else `RigidBody`.
  - `apply_force(vec, point=None)` — Append a (Point, Vector) load.
  - `apply_torque(vec)` — Append a (frame, Vector) load.

### `mechanics/functions.py`

Mechanics helper functions.

- `inertia(frame, ixx, iyy, izz, ixy=0, iyz=0, izx=0)` — Build an inertia Dyadic in a frame.
- `inertia_of_point_mass(mass, pos_vec, frame)` — Inertia dyadic of a point mass about O.
- `linear_momentum(frame, *body)` — System linear momentum.
- `angular_momentum(point, frame, *body)` — System angular momentum.
- `kinetic_energy(frame, *body)` — System kinetic energy.
- `potential_energy(*body)` — System potential energy.
- `Lagrangian(frame, *body)` — T − V.
- `msubs(expr, *sub_dicts, smart=False)` — Mechanics-aware substitution; ignores terms inside `Derivative`; optional `smart` mode handles 0/0 cases via `sin/cos` rewriting.
- `find_dynamicsymbols(expression, exclude=None)` — Finds all `dynamicsymbols` in an expression.
- Printing aliases: `mprint`, `msprint`, `mpprint`, `mlatex`, `mechanics_printing`.

### `mechanics/kane.py`

Kane's method for equations of motion.

- `class KanesMethod` — Book-keeping object for Kane's EoM formulation.
  - `kanes_equations(bodies, loads=None)` — Form Fr + Fr* = 0. Returns `(Fr, Fr*)`.
  - `rhs(inv_method=None)` — First-order form `[qdot, udot]^T = f(q, u, t)`.
  - `linearize(**kwargs)` — Linearize about symbolic operating point. Returns `(M, A, B, r)` or `(A, B, r)`.
  - `to_linearizer()` — Returns a `Linearizer` instance for efficient repeated linearization.
  - Properties: `mass_matrix`, `mass_matrix_full`, `forcing`, `forcing_full`, `auxiliary_eqs`.

### `mechanics/lagrange.py`

Lagrange's method for equations of motion.

- `class LagrangesMethod` — Generates EoM from a Lagrangian and generalized coordinates, with optional holonomic/nonholonomic constraints and non-conservative forces.
  - `form_lagranges_equations()` — Returns the EoM vector.
  - `rhs(inv_method=None)` — Numerically-solvable form.
  - `linearize(q_ind, qd_ind, q_dep, qd_dep, **kwargs)` — Linearization.
  - `solve_multipliers(op_point, sol_type)` — Solve for Lagrange multipliers at an operating point.
  - Properties: `mass_matrix`, `mass_matrix_full`, `forcing`, `forcing_full`.

### `mechanics/linearize.py`

- `class Linearizer` — General-form linearizer for dynamic systems with dependent coordinates/speeds.
  - `linearize(op_point=None, A_and_B=False, simplify=False)` — Returns `(M, A, B)` or `(A, B)` for state-space form.
  - Handles holonomic (configuration) and non-holonomic (velocity) constraints via permutation and coefficient matrices C_0, C_1, C_2.
- `permutation_matrix(orig_vec, per_vec)` — Compute permutation matrix between two orderings.

### `mechanics/models.py`

Pre-built symbolic models for testing.

- `multi_mass_spring_damper(n=1, apply_gravity, apply_external_forces)` — n-DOF serial mass-spring-damper. Returns `KanesMethod`.
- `n_link_pendulum_on_cart(n=1, cart_force, joint_torques)` — 2D n-link pendulum on a sliding cart under gravity. Returns `KanesMethod`.

---

## Optics — `optics/`

### `optics/medium.py`

- `class Medium(Symbol)` — Optical medium defined by permittivity, permeability, and/or refractive index. Properties: `intrinsic_impedance`, `speed`, `refractive_index`, `permittivity`, `permeability`.

### `optics/waves.py`

- `class TWave(Expr)` — Transverse sine wave `A * cos(kx − ωt + φ)`. Supports superposition via `__add__`. Properties: `frequency`, `wavelength`, `amplitude`, `phase`, `speed`, `angular_velocity`, `wavenumber`. Rewrite methods: `sin`, `cos`, `exp`, `pde`.

### `optics/gaussopt.py`

Gaussian optics: ray transfer matrices and conjugation relations.

- **Ray transfer matrices** (all subclass `RayTransferMatrix(Matrix)`):
  - `FreeSpace(d)`, `FlatRefraction(n1, n2)`, `CurvedRefraction(R, n1, n2)`, `FlatMirror()`, `CurvedMirror(R)`, `ThinLens(f)`.
- `class GeometricRay(Matrix)` — 2×1 `[height, angle]` representation.
- `class BeamParameter(Expr)` — Gaussian beam complex parameter. Properties: `q`, `radius`, `w`, `w_0`, `divergence`, `gouy`, `waist_approximation_limit`.
- **Conjugation utilities**:
  - `waist2rayleigh(w, wavelen)` / `rayleigh2waist(z_r, wavelen)` — Convert between waist and Rayleigh range.
  - `geometric_conj_ab`, `geometric_conj_af`, `geometric_conj_bf` — Geometric conjugation relations.
  - `gaussian_conj(s_in, z_r_in, f)` — Gaussian beam conjugation; returns `(s_out, z_r_out, m)`.
  - `conjugate_gauss_beams(wavelen, waist_in, waist_out, f=f)` — Find optical setup conjugating object/image waists.

### `optics/utils.py`

Geometric optics formulas.

- `refraction_angle(incident, medium1, medium2, normal, plane)` — Snell's law for 3D vectors; supports `Ray3D` or `Matrix` input.
- `deviation(incident, medium1, medium2, normal, plane)` — Angle of deviation due to refraction.
- `lens_makers_formula(n_lens, n_surr, r1, r2)` — Focal length of a thin lens.
- `mirror_formula(focal_length, u, v)` — Mirror equation; provide any 2 of 3 parameters.
- `lens_formula(focal_length, u, v)` — Thin lens equation; provide any 2 of 3 parameters.
- `hyperfocal_distance(f, N, c)` — Hyperfocal distance from focal length, f-number, and circle of confusion.

---

## Quantum Mechanics — `quantum/`

### `quantum/qexpr.py`

Base classes for quantum expressions.

- `class QExpr(Expr)` — Base class for all quantum expressions. Handles label management, Hilbert space association, printing, and representation dispatch.
- `class QuantumError(Exception)`
- `split_commutative_parts(e)`, `split_qexpr_parts(e)`

### `quantum/state.py`

Quantum states: kets, bras, and wavefunctions.

- `class StateBase(QExpr)` — Abstract base with dual-state logic.
- `class Ket(State, KetBase)` / `class Bra(State, BraBase)` — Time-independent states.
- `class TimeDepKet` / `class TimeDepBra` — Time-dependent states.
- `class Wavefunction(Function)` — Position-space wavefunction with normalization and probability.

### `quantum/operator.py`

Quantum operators.

- `class Operator(QExpr)` — General quantum operator.
- `class HermitianOperator(Operator)` — Self-adjoint operator.
- `class UnitaryOperator(Operator)` — Unitary operator.
- `class IdentityOperator(Operator)` — Identity with special simplification rules.
- `class OuterProduct(Operator)` — `|a><b|` outer product.
- `class DifferentialOperator(Operator)` — Operator defined by an arbitrary function/derivative expression applied to wavefunctions.

### `quantum/dagger.py`

- `class Dagger(adjoint)` — Hermitian conjugate; wraps `sympy.functions.adjoint`.

### `quantum/commutator.py`

- `class Commutator(Expr)` — `[A, B]` with auto-expansion for sums/products and `.doit()` evaluation.

### `quantum/anticommutator.py`

- `class AntiCommutator(Expr)` — `{A, B}` anticommutator.

### `quantum/innerproduct.py`

- `class InnerProduct(Expr)` — Unevaluated `<a|b>` inner product.

### `quantum/tensorproduct.py`

- `class TensorProduct(Expr)` — Tensor product of quantum expressions with `flatten`, `doit`, and representation support.
- `tensor_product_simp(e)` — Simplify tensor product expressions by combining factors.

### `quantum/hilbert.py`

Hilbert space representations.

- `HilbertSpace`, `ComplexSpace(dim)`, `L2(interval)`, `FockSpace`
- `TensorProductHilbertSpace`, `DirectSumHilbertSpace`, `TensorPowerHilbertSpace`

### `quantum/represent.py`

Representation of quantum objects in a basis.

- `represent(expr, **options)` — Main entry point; dispatches to `_represent_<BasisClass>` methods.
- `rep_innerproduct(expr)`, `rep_expectation(expr)` — Compute inner products and expectation values in a basis.
- `integrate_result(orig_expr, result)` — Integrate over continuous basis indices.
- `enumerate_states(*args)` — Generate a sequence of basis states.
- `get_basis(expr)` — Determine appropriate basis for an expression.

### `quantum/qapply.py`

- `qapply(e, **options)` — Apply operators to states in a quantum expression. Distributes over `Add`, handles `Mul` chains, `TensorProduct`, `InnerProduct`, and `Pow`.

### `quantum/constants.py`

- `class HBar(NumberSymbol)` — Singleton for ℏ.

### `quantum/cartesian.py`

Cartesian position/momentum operators and states.

- Operators: `XOp`, `YOp`, `ZOp`, `PxOp`.
- States: `XKet`/`XBra` (1D), `PxKet`/`PxBra` (momentum), `PositionKet3D`/`PositionBra3D`.

### `quantum/spin.py`

Angular momentum (spin) algebra — the largest file in the quantum sub-package.

- **Operators**: `JxOp`, `JyOp`, `JzOp`, `J2Op`, `JplusOp`, `JminusOp`.
- **Rotation**: `class Rotation(UnitaryOperator)` — Euler angle rotation operator with Wigner-D matrix representation.
- **Wigner-D**: `class WignerD(Expr)` — Symbolic Wigner-D function.
- **States**: `JxKet/Bra`, `JyKet/Bra`, `JzKet/Bra` — Uncoupled spin states.
- **Coupled states**: `CoupledSpinState`, `JzKetCoupled/BraCoupled`, etc.
- `couple(expr)` / `uncouple(expr)` — Transform between coupled and uncoupled representations.

### `quantum/cg.py`

Clebsch-Gordan coefficients and simplification.

- `class CG(Wigner3j)` — Clebsch-Gordan coefficient.
- `class Wigner3j`, `class Wigner6j`, `class Wigner9j` — Symbolic wrappers.
- `cg_simp(e)` — Simplify expressions with CG coefficients using Varshalovich identities.

### `quantum/gate.py`

Quantum gates for circuit-based quantum computation.

- **Base**: `Gate(UnitaryOperator)`, `OneQubitGate`, `TwoQubitGate`.
- **Controlled**: `CGate(Gate)` — General controlled gate. `CGateS` — variant using smaller matrix.
- **Standard gates**: `HadamardGate`, `XGate`, `YGate`, `ZGate`, `PhaseGate`, `TGate`, `CNotGate`, `SwapGate`.
- `UGate(Gate)` — General single-qubit unitary from a 2×2 matrix.
- `IdentityGate`

### `quantum/qubit.py`

Qubit states.

- `class Qubit(Ket)` / `class QubitBra(Bra)` — Multi-qubit computational basis states (e.g. `Qubit('01')`).
- `class IntQubit` / `class IntQubitBra` — Qubit from integer value and number of bits.

### `quantum/qft.py`

Quantum Fourier Transform.

- `class QFT(Fourier)` / `class IQFT(Fourier)` — QFT and inverse QFT gates.
- `class RkGate(OneQubitGate)` — Phase rotation gate R_k used in QFT decomposition.

### `quantum/grover.py`

Grover's search algorithm.

- `class OracleGate(Gate)` — Oracle marking target states.
- `class WGate(Gate)` — Grover diffusion operator.
- `superposition_basis(nqubits)` — Equal superposition over all basis states.
- `grover_iteration(qstate, oracle)` — One Grover iteration.
- `apply_grover(oracle, nqubits, iterations)` — Full Grover search.

### `quantum/shor.py`

Shor's factoring algorithm.

- `class CMod(Gate)` — Controlled modular exponentiation gate.
- `shor(N)` — Symbolic implementation of Shor's algorithm.
- `period_find(a, N)` — Order-finding subroutine.

### `quantum/boson.py`

Bosonic creation/annihilation operators and Fock/coherent states.

- `class BosonOp(Operator)` — Bosonic operator with commutation `[a, a†] = 1`.
- `BosonFockKet/Bra`, `BosonCoherentKet/Bra`

### `quantum/fermion.py`

Fermionic creation/annihilation operators and Fock states.

- `class FermionOp(Operator)` — Fermionic operator with `{c, c†} = 1`.
- `FermionFockKet/Bra`

### `quantum/pauli.py`

Pauli spin operators (quantum framework, not the algebraic `paulialgebra.py`).

- `SigmaX`, `SigmaY`, `SigmaZ`, `SigmaPlus`, `SigmaMinus` — Pauli operators with optional label for multi-spin systems.
- `SigmaZKet`/`SigmaZBra` — Eigenstates of σ_z.
- `qsimplify_pauli(e)` — Simplify expressions containing Pauli operators.

### `quantum/sho1d.py`

1D quantum harmonic oscillator (operator formalism).

- `RaisingOp` / `LoweringOp` — a† and a operators.
- `NumberOp` — Number operator N = a†a.
- `Hamiltonian` — H = ℏω(N + 1/2).
- `SHOKet`/`SHOBra` — Number states |n⟩.

### `quantum/piab.py`

Particle in a box (1D infinite square well).

- `PIABHamiltonian(HermitianOperator)` — Hamiltonian.
- `PIABKet`/`PIABBra` — Energy eigenstates.

### `quantum/density.py`

Density matrices.

- `class Density(HermitianOperator)` — Density operator from state-probability pairs.
- `entropy(density)` — Von Neumann entropy.
- `fidelity(state1, state2)` — Fidelity between two quantum states.

### `quantum/operatorordering.py`

Normal and antinormal ordering of bosonic operators.

- `normal_ordered_form(expr)` — Rewrite in normal ordered form using commutation.
- `normal_order(expr)` — Apply normal ordering (moves all a† left of a).

### `quantum/operatorset.py`

Mapping between operators and states.

- `operators_to_state(operators)` — Find the state class associated with operators.
- `state_to_operators(state)` — Find operators associated with a state class.

### `quantum/qasm.py`

QASM (Quantum Assembly Language) parser.

- `class Qasm` — Parses QASM instructions into gate operations.
- `read_qasm(lines)` / `read_qasm_file(filename)` — Parse QASM from string or file.

### `quantum/circuitutils.py`

Circuit manipulation utilities.

- `find_subcircuit(circuit, subcircuit)` — KMP-based subcircuit search.
- `replace_subcircuit(circuit, subcircuit, replace)` — Replace a subcircuit.
- `convert_to_symbolic_indices` / `convert_to_real_indices` — Map between symbolic and real qubit indices.
- `random_reduce(circuit, gate_ids)` / `random_insert(circuit, choices)` — Stochastic circuit simplification.

### `quantum/circuitplot.py`

Circuit visualization (matplotlib-based).

- `Mz`, `Mx` — Measurement gates for plotting.
- `CreateOneQubitGate`, `CreateCGate` — Factory functions for custom gate visuals.
- `render_label(label)`, `labeller(n)` — Label rendering helpers.

### `quantum/identitysearch.py`

Search for gate identities in quantum circuits.

- `bfs_identity_search(gate_list, nqubits, max_depth)` — BFS search for gate sequences equivalent to identity.
- `random_identity_search(gate_list, numgates, nqubits)` — Random search variant.
- `generate_gate_rules(gate_seq)` / `generate_equivalent_ids(gate_seq)` — Enumerate equivalent gate sequences.
- `class GateIdentity(Basic)` — Represents a gate sequence equal to identity.
- `is_scalar_sparse_matrix` / `is_scalar_nonsparse_matrix` — Check if a circuit's matrix representation is scalar.

### `quantum/matrixutils.py`

Matrix backend utilities (sympy / numpy / scipy.sparse interop).

- `to_sympy`, `to_numpy`, `to_scipy_sparse` — Convert matrices between backends.
- `matrix_tensor_product(*product)` — Tensor product dispatched to the appropriate backend.
- `matrix_eye(n)`, `matrix_zeros(m, n)` — Backend-aware identity/zero matrices.
- `matrix_dagger(e)`, `flatten_scalar(e)`, `matrix_to_zero(e)`

### `quantum/matrixcache.py`

- `class MatrixCache` — Caches small gate matrices across backends.

---

## Vector — `vector/`

Foundation for `mechanics`: reference frames, vectors, dyadics, points, and vector calculus.

### `vector/frame.py`

- `class ReferenceFrame` — 3D reference frame with orientation, angular velocity, angular acceleration. Supports rotations via `orient(parent, rot_type, amounts)` (Axis, Body, Space, Quaternion, DCM). Provides `.x`, `.y`, `.z` basis vectors and `[0]`, `[1]`, `[2]` coordinate scalars.
- `class CoordinateSym(Symbol)` — Coordinate scalar bound to a frame and index.

### `vector/vector.py`

- `class Vector` — Symbolic vector built from frame-basis components. Operations: `+`, `-`, `^` (cross), `&` (dot), `|` (outer → Dyadic), `magnitude()`, `normalize()`, `express(frame)`, `dt(frame)` (time derivative), `diff(var, frame)`.

### `vector/dyadic.py`

- `class Dyadic` — Second-order tensor (outer products of basis vectors). Operations: `+`, `&` (dot with Vector or Dyadic), `express(frame)`, `dt(frame)`, `to_matrix(ref_frame)`.

### `vector/point.py`

- `class Point` — Point in a dynamic system storing position, velocity, and acceleration relative to other points.
  - `set_pos`, `pos_from` — Position relationships.
  - `set_vel`, `vel` — Velocity in a frame.
  - `set_acc`, `acc` — Acceleration in a frame.
  - `v1pt_theory`, `v2pt_theory`, `a1pt_theory`, `a2pt_theory` — Kinematic theorems for computing velocities/accelerations.

### `vector/functions.py`

Convenience functions and `dynamicsymbols`.

- `cross`, `dot`, `express`, `outer` — Wrappers around `Vector`/`Dyadic` methods.
- `time_derivative(expr, frame, order=1)` — Time derivative in a frame.
- `kinematic_equations(speeds, coords, rot_type, rot_order)` — Generate kinematic differential equations.
- `get_motion_params(frame, **kwargs)` — Integrate/differentiate to fill in position, velocity, acceleration.
- `partial_velocity(vel_vecs, gen_speeds, frame)` — Compute partial velocities.
- `dynamicsymbols(names)` — Create functions of time `t`: `q(t)`, `u(t)`, etc.

### `vector/fieldfunctions.py`

Vector calculus operations on scalar/vector fields.

- `curl(vect, frame)` — ∇ × F
- `divergence(vect, frame)` — ∇ · F
- `gradient(scalar, frame)` — ∇f
- `is_conservative(field)` — Check if curl is zero.
- `is_solenoidal(field)` — Check if divergence is zero.
- `scalar_potential(field, frame)` — Compute scalar potential of a conservative field.
- `scalar_potential_difference(field, frame, P1, P2, O)` — Potential difference between two points.

### `vector/printing.py`

Custom printers for vector/mechanics expressions.

- `VectorStrPrinter`, `VectorLatexPrinter`, `VectorPrettyPrinter` — Print `dynamicsymbols` without `(t)` suffix.
- `vprint`, `vsprint`, `vpprint`, `vlatex` — Convenience print functions.
- `init_vprinting(**kwargs)` — Initialize pretty printing for vector expressions.

---

## Unit Systems — `unitsystems/`

Group-theoretical dimensional analysis framework (separate from the legacy `units.py`).

### `unitsystems/dimensions.py`

- `class Dimension(Expr)` — Physical dimension as a vector of base-dimension exponents. Supports `mul`, `div`, `pow`, `add` (dimensional analysis). Dict-like access to exponents.
- `class DimensionSystem` — A set of base dimensions + derived dimensions with methods to compute dimensional dependencies. Can be extended via `.extend(base, dims)`.

### `unitsystems/units.py`

- `class Unit(Expr)` — Unit = dimension + scale factor + optional prefix/abbreviation. All arithmetic operates on factor and dimension.
- `class Constant(Unit)` — Physical constant (a Unit where the factor is a measured value).
- `class UnitSystem` — Collection of base units + derived units. Methods: `get_unit(dim)`, `print_unit_base(unit)`. Extendable via `.extend()`.

### `unitsystems/quantities.py`

- `class Quantity(Expr)` — Physical quantity = numeric factor × Unit. Supports `add`, `sub`, `mul`, `div`, `pow`, `convert(unit)`.

### `unitsystems/prefixes.py`

- `class Prefix` — SI/binary prefix (name, abbreviation, base^exponent). Combinable with other prefixes.
- `PREFIXES` — Dict of standard SI prefixes (yocto to yotta).
- `BIN_PREFIXES` — Dict of binary prefixes (kibi to yobi).
- `prefix_unit(unit, prefixes)` — Generate all prefixed variants of a unit.

### `unitsystems/simplifiers.py`

- `dim_simplify(expr)` — Recursively simplify dimension expressions.
- `qsimplify(expr)` — Recursively simplify quantity expressions (converts Unit to Quantity as needed).

### `unitsystems/systems/mks.py`

MKS (meter-kilogram-second) system definition. Defines base dimensions (length, mass, time), derived dimensions (velocity, force, energy, etc.), base units (m, kg, s), derived units (N, J, W, Pa, Hz), and constants (G, c).

### `unitsystems/systems/mksa.py`

MKSA (meter-kilogram-second-ampere) extension of MKS. Adds `current` base dimension and electromagnetic derived dimensions/units (V, ohm, S, F, H, C, T, Wb) plus constant Z_0.

### `unitsystems/systems/natural.py`

Natural unit system (c = 1, ℏ = 1). Base dimensions: action, energy, velocity. Base units: ℏ, eV, c. Length, mass, time become derived.

---

## Appendix

**Caveats:**

- `sympy.physics.gaussopt` (root level) is **deprecated** in favor of `sympy.physics.optics.gaussopt`.
- `secondquant.py` uses a standalone `Dagger` and `Commutator` unrelated to the `quantum/` sub-package versions.
- The `units.py` (root level) is a legacy flat system; `unitsystems/` provides a more structured group-theoretical alternative. They are not interchangeable.
- `mechanics/` re-exports the entire `vector/` sub-package — `from sympy.physics.mechanics import *` includes all vector classes.
- `quantum/spin.py` is ~2100 lines and the most complex file; it handles both uncoupled and coupled angular momentum states with full representation logic.
