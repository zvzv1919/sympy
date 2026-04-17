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
- `mgamma(mu)` — 4×4 Dirac gamma matrix γ^μ.
- `pat_matrix(x, y, z, a)` — Parallel Axis Theorem inertia matrix.
- `mdft(n)` — n×n discrete Fourier transform matrix.
- `minkowski_tensor` — 4×4 Minkowski metric tensor.

### [`paulialgebra.py`](paulialgebra.py)
Pauli matrix algebra via symbolic manipulation (not Matrix class).
- `Pauli(i)` — Symbol subclass representing σ_i; algebraic multiplication yields products and scalar I automatically.
- `delta(i, j)` — Kronecker delta helper; returns 1 if i == j, else 0.
- `epsilon(i, j, k)` — Levi-Civita symbol helper; returns +1 for even permutations of (1,2,3), −1 for odd, 0 otherwise (including repeated indices).
- `evaluate_pauli_product(arg)` — simplifies a product of Pauli matrices using algebraic rules.

### [`pring.py`](pring.py)
Quantum particle on a ring.
- `wavefunction(n, x)` — eigenfunctions. `energy(n, m, r)` — energy levels.

### [`qho_1d.py`](qho_1d.py)
One-dimensional quantum harmonic oscillator: wavefunctions, energies, and coherent-state overlaps.
- `psi_n(n, x, m, omega)` — spatial wavefunction ψ_n using Hermite polynomials.
- `E_n(n, omega)` — energy eigenvalue ℏω(n+½).
- `coherent_state(n, alpha)` — Fock-basis expansion coefficient ⟨n|α⟩ for a coherent (displaced vacuum / minimum-uncertainty) state; computes exp(−|α|²/2)·α^n/√(n!).

### [`secondquant.py`](secondquant.py)
Second quantization framework for many-body quantum mechanics with bosonic and fermionic creation/annihilation operators.
- `BosonicOperator`, `CreateBoson` (B†), `AnnihilateBoson` (B) — bosonic ladder operators with commutation relations.
- `FermionicOperator`, `CreateFermion` (Fd), `AnnihilateFermion` (F) — fermionic ladder operators with anticommutation relations.
- `NO` — normal-ordering bracket; constructor reorders operators into creation-before-annihilation form, returns S.Zero if identical fermion operators violate Pauli exclusion.
- `Commutator`, `AntiCommutator` — symbolic (anti)commutator expressions.
- `FockState`, `FockStateKet`, `FockStateBra` — Fock-space state vectors.
- `wicks(expr)` — applies Wick's theorem to expand operator products into normal-ordered contractions.
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
- **Core**: `qexpr.py` (base quantum expression), `state.py` (Ket/Bra/Wavefunction), `operator.py` (Operator/Hermitian/Unitary), `hilbert.py` (Hilbert spaces), `represent.py` (matrix representations).
- **Angular momentum / CG**: `cg.py` — Clebsch-Gordan coefficient symbolic expressions and simplification rules (not state construction); orthogonality-relation identities reduce CG products summed over j,m to Kronecker deltas.
- **Spin**: `spin.py` — spin operators (Jx, Jy, Jz, J±, J²), coupled/uncoupled states, Wigner-D/d matrices.
  - `J2Op` — total angular momentum squared (Casimir) operator; commutes with all component operators and applies eigenvalue ℏ²j(j+1).
  - `CoupledSpinState` — coupled state constructor with triangle-inequality validation on coupling schemes.
- **Gates**: `gate.py` — quantum gate classes (H, X, Y, Z, S/Phase, T, CNOT, SWAP, CGate); each gate stores target matrices, commutation relations between gates (e.g. T–S, S–Z commute → 0), and decomposition methods (e.g. SWAP decomposes into three CNOT gates).
- **Circuit plotting**: `circuitplot.py` — `CircuitPlot` for rendering circuits; `CreateCGate(name, latexname=None)` factory for dynamically creating controlled gates (defaults latexname to name if omitted); mock measurement gates `Mz`, `Mx`.
- **Circuit identity search**: `identitysearch.py` — `generate_gate_rules(gate_seq)` finds equivalent gate rewriting rules via BFS; returns trivial rule set when input is a plain numeric scalar. `generate_equivalent_ids()` finds equivalent gate identities.
- **Second-quantized QM operators**: `boson.py` (BosonOp, Fock states), `fermion.py` (FermionOp — fermionic ladder operators with anticommutation: {a†,a}=1 for same-mode, 0/2·ab for independent modes).
- **Operator ordering**: `operatorordering.py` — `normal_ordered_form()` rearranges products into creation-before-annihilation order; when two operators belong to different independent modes, the commutation correction term is set to zero.
- **Commutator algebra**: `commutator.py`, `anticommutator.py` — symbolic `Commutator` and `AntiCommutator` with `doit()` evaluation; these delegate to gate/operator `_eval_commutator_*` methods.
- **Algorithms**: `grover.py` (Grover's search), `shor.py` (Shor's factoring), `qft.py` (quantum Fourier transform).
- **Qubits**: `qubit.py` — `Qubit`, `IntQubit`, qubit-state manipulation and measurement.
- **Other**: `tensorproduct.py`, `density.py`, `innerproduct.py`, `matrixutils.py`, `matrixcache.py`, `circuitutils.py`, `piab.py` (particle in a box), `sho1d.py` (1-D SHO operators), `constants.py` (ℏ).
  - `pauli.py` — Pauli spin operators: SigmaX/Y/Z, SigmaPlus/SigmaMinus raising/lowering operators (nilpotent under positive-integer exponentiation).
  - `cartesian.py` — 1-D/3-D position and momentum eigenstates (XKet/XBra, PxKet/PxBra) with plane-wave inner products.

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
- `kane.py` — `KanesMethod`: Kane's equations of motion.
- `lagrange.py` — `LagrangesMethod`: Lagrangian mechanics.
- `particle.py` — `Particle`. `rigidbody.py` — `RigidBody`. `body.py` — unified `Body`.
- `functions.py` — kinematic/dynamic helper functions.
- `linearize.py` — linearization around operating points.

### [`hep/`](hep/catalog.md)
High-energy physics.
- `gamma_matrices.py` — Dirac gamma-matrix algebra and trace evaluation.

### [`unitsystems/`](unitsystems/catalog.md)
Dimensional analysis and unit systems (SI, CGS, natural, etc.).
- `dimensions.py` — `Dimension` class, dimension system definitions.
- `units.py` — unit definitions. `quantities.py` — physical quantity objects.
- `prefixes.py` — SI prefixes. `simplifiers.py` — unit expression simplification.
