# sympy/physics — Catalog

> Part of [SymPy](../catalog.md). Physics subpackages: quantum mechanics, optics, classical mechanics, units, hydrogen wave functions, vector algebra.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Top-level namespace for sympy.physics; publicly exposes mgamma (Dirac gamma matrices), msigma (Pauli spin matrices), minkowski_tensor (flat spacetime metric), and mdft (discrete Fourier transform matrix) by importing them from matrices submodule. |
| `gaussopt.py` | Deprecated shim that re-exports `sympy.physics.optics.gaussopt`; warns users to use the optics subpackage instead. |
| `hydrogen.py` | Hydrogen atom wavefunctions and energy levels: radial wavefunction R_nl, non-relativistic energy E_nl, and Dirac relativistic energy E_nl_dirac. |
| `matrices.py` | Physics-related matrices: Pauli matrices (msigma), Dirac gamma matrices (mgamma), Minkowski metric tensor, parallel axis theorem matrix, and discrete Fourier transform matrix (mdft). |
| `paulialgebra.py` | Implements Pauli algebra by subclassing Symbol, providing the Pauli class and evaluate_pauli_product for symbolic Pauli matrix multiplication. |
| `pring.py` | Quantum particle on a ring: wavefunction and energy functions for a 1D ring geometry. |
| `qho_1d.py` | One-dimensional quantum harmonic oscillator: spatial wavefunction psi_n, energy spectrum E_n, and coherent_state which computes the Fock-basis expansion coefficient ⟨n\|α⟩ (overlap between a number state and a displaced-vacuum/coherent state, i.e. eigenstate of the annihilation operator). |
| `secondquant.py` | Second quantization framework: fermionic/bosonic creation and annihilation operators (F, Fd, B, Bd), Fock states, the NO class (normal-ordering brackets that canonically sort operators and return zero when the Pauli exclusion principle is violated by duplicate fermion operators), Wick's theorem (contraction and wicks function), and commutator algebra. |
| `sho.py` | 3D isotropic simple harmonic oscillator: radial wavefunction R_nl and energy E_nl using associated Laguerre polynomials. |
| `unitsystems.py` | Deprecated shim that re-exports `sympy.physics.units`; warns users to use the units subpackage instead. |
| `unitsystems/__init__.py` | Package init for the legacy unitsystems subpackage; defines dimensional analysis and unit system infrastructure. |
| `unitsystems/dimensions.py` | Dimension class for physical dimensions and DimensionSystem for managing base/derived dimension sets; get_dim looks up a dimension by name, symbol, or object reference (returns None instead of raising an error when not found). |
| `unitsystems/prefixes.py` | Prefix class for unit prefixes (kilo, milli, etc.) in the legacy unitsystems framework. |
| `unitsystems/quantities.py` | Quantity class representing physical quantities with a unit and factor in the legacy unitsystems framework. |
| `unitsystems/simplifiers.py` | Dimensional simplification functions for the legacy unitsystems framework. |
| `unitsystems/units.py` | Unit class and UnitSystem class for the legacy unitsystems framework; UnitSystem provides get_unit for unit lookup. |
| `unitsystems/systems/__init__.py` | Package init for legacy unit system definitions. |
| `unitsystems/systems/mks.py` | Legacy MKS unit system definition. |
| `unitsystems/systems/mksa.py` | Legacy MKSA unit system definition extending MKS with electromagnetic dimensions. |
| `unitsystems/systems/natural.py` | Legacy natural unit system definition (c = hbar = 1). |
| `wigner.py` | Wigner 3j, 6j, 9j symbols, Clebsch-Gordan coefficients, Racah coefficients, and Gaunt coefficients for angular momentum coupling. |
| `continuum_mechanics/__init__.py` | Package init for continuum_mechanics; imports the Beam class. |
| `continuum_mechanics/beam.py` | Beam class for solving 2D beam bending problems using singularity functions, including load application, reaction forces, shear force, bending moment, slope, and deflection. |
| `hep/gamma_matrices.py` | Gamma (Dirac) matrices expressed as tensor objects for high-energy physics, with Kahane simplification and gamma trace algorithms. |
| `mechanics/__init__.py` | Package init for classical mechanics; imports and re-exports Kane's method, Lagrange's method, rigid body, particle, linearization, body, system, and vector modules. |
| `mechanics/body.py` | Body class that serves as a unified representation of either a RigidBody or Particle, with support for applied loads (forces and torques). |
| `mechanics/functions.py` | Utility functions for classical mechanics: inertia dyadic creation, linear/angular momentum, kinetic/potential energy, Lagrangian computation, mechanics printing, and symbolic substitution (msubs). |
| `mechanics/kane.py` | KanesMethod class implementing Kane's method for forming equations of motion, producing mass matrix and forcing vector representations. |
| `mechanics/lagrange.py` | LagrangesMethod class implementing Lagrange's method for deriving equations of motion from a Lagrangian, with support for constraints and non-conservative forces. |
| `mechanics/linearize.py` | Linearizer class for computing the linearized form of a dynamic system, handling dependent coordinates and speeds arising from constraints. |
| `mechanics/models.py` | Sample symbolic mechanical models (multi-mass spring-damper, n-link pendulum) for testing and examples. |
| `mechanics/particle.py` | Particle class representing a point mass with position, velocity, acceleration, and potential energy. |
| `mechanics/rigidbody.py` | RigidBody class representing an idealized rigid body with mass, center of mass, reference frame, and inertia dyadic. |
| `mechanics/system.py` | SymbolicSystem class that stores a complete symbolic dynamic system (equations of motion, bodies, loads) in explicit or implicit form. |
| `optics/__init__.py` | Package init for optics; imports and re-exports TWave, Gaussian optics, Medium, and optics utility functions. |
| `optics/gaussopt.py` | Gaussian optics: ray transfer matrices (FreeSpace, FlatRefraction, CurvedRefraction, FlatMirror, CurvedMirror, ThinLens), GeometricRay, BeamParameter, and conjugation relations. |
| `optics/medium.py` | Medium class representing an optical medium with refractive index, permittivity, permeability, and related electromagnetic properties. |
| `optics/utils.py` | Optics utility functions: refraction_angle, deviation, lens_makers_formula, mirror_formula, lens_formula, hyperfocal_distance, and transverse_magnification. |
| `optics/waves.py` | TWave class representing a transverse sinusoidal wave with amplitude, frequency, phase, wavelength, and wave speed properties. |
| `quantum/__init__.py` | Package init for quantum mechanics; imports and re-exports anticommutator, commutator, dagger, hilbert spaces, inner product, operators, states, tensor product, represent, qapply, and constants. |
| `quantum/anticommutator.py` | AntiCommutator class implementing the unevaluated anticommutator {A, B} = A*B + B*A for quantum operators. |
| `quantum/boson.py` | Bosonic quantum operators (BosonOp) and Fock/coherent states (BosonFockKet, BosonCoherentKet) satisfying [a, a^dagger] = 1. |
| `quantum/cartesian.py` | 1D Cartesian position and momentum operators (XOp, PxOp) and their eigenstates (XKet, PxKet), plus 3D position states. |
| `quantum/cg.py` | Clebsch-Gordan coefficients and Wigner 3j/6j/9j symbol classes with symbolic representations; cg_simp simplification including orthogonality relation checks that reduce products of CG coefficients (summed over total angular momentum) to Kronecker deltas. |
| `quantum/circuitplot.py` | Matplotlib-based plotting of quantum circuits: CircuitPlot, measurement gates (Mz, Mx), dynamic gate factory functions (CreateCGate for building controlled gates, CreateOneQubitGate metaclass for building single-qubit gates at runtime with optional LaTeX names), and label rendering helpers. |
| `quantum/circuitutils.py` | Primitive circuit operations: subcircuit finding/replacement using KMP algorithm, symbolic index conversion, and random circuit reduction/insertion. |
| `quantum/commutator.py` | Commutator class implementing the unevaluated commutator [A, B] = A*B - B*A for quantum operators. |
| `quantum/constants.py` | Quantum mechanical constants: defines hbar (reduced Planck's constant) as a singleton NumberSymbol. |
| `quantum/dagger.py` | Dagger class implementing the Hermitian conjugate operation for quantum objects (operators, states, matrices). |
| `quantum/density.py` | Density operator class for representing mixed quantum states, with support for entropy calculation, probability checking, and matrix representation. |
| `quantum/fermion.py` | Fermionic quantum operators (FermionOp) and Fock states (FermionFockKet) satisfying {c, c^dagger} = 1. |
| `quantum/gate.py` | Quantum gates acting on qubits: base Gate class with the core _apply_operator_Qubit method that applies a unitary matrix to a computational basis state (selecting a matrix column based on target-qubit bit values and constructing the output superposition by flipping bits), controlled gates (CGate, CNotGate), single-qubit gates (Hadamard, X, Y, Z, Phase, T), SwapGate, UGate, and gate sorting/simplification. |
| `quantum/grover.py` | Grover's quantum search algorithm: OracleGate, WGate, superposition_basis, grover_iteration, and apply_grover. |
| `quantum/hilbert.py` | Hilbert space classes: HilbertSpace, ComplexSpace, L2, FockSpace, with support for direct sum and tensor product of spaces. |
| `quantum/identitysearch.py` | Gate identity and equivalence search: generate_gate_rules (computes equivalent circuit rewrite rules via BFS, with special-case handling for plain numeric scalar inputs), generate_equivalent_ids, random search, scalar matrix checking, and GateIdentity class. |
| `quantum/innerproduct.py` | InnerProduct class representing the symbolic inner product between a Bra and a Ket. |
| `quantum/matrixcache.py` | MatrixCache class for storing small matrices in SymPy, NumPy, and SciPy sparse formats for efficient reuse in quantum computations. |
| `quantum/matrixutils.py` | Utility functions for interconversion between SymPy Matrix, NumPy ndarray, and SciPy sparse matrices, plus matrix tensor product and dagger operations. |
| `quantum/operator.py` | Base quantum operator classes: Operator, HermitianOperator, UnitaryOperator, IdentityOperator, OuterProduct, and DifferentialOperator. |
| `quantum/operatorordering.py` | Post-hoc reordering utilities (normal_ordered_form, normal_order) that rearrange existing operator product expressions into normal order; operates on expression trees rather than constructing normal-order brackets. |
| `quantum/operatorset.py` | Mapping between quantum operators and their corresponding eigenstates (operators_to_state, state_to_operators). |
| `quantum/pauli.py` | Pauli sigma operators (SigmaX, SigmaY, SigmaZ, SigmaPlus, SigmaMinus) and states (SigmaZKet/Bra) for two-level quantum systems, with qsimplify_pauli. |
| `quantum/piab.py` | Particle in a box (PIAB) Hamiltonian operator and eigenstates (PIABKet, PIABBra) with position-space representation. |
| `quantum/qapply.py` | qapply function that walks an expression tree and dispatches operator-state applications; delegates actual gate-to-qubit computation to the gate classes themselves. |
| `quantum/qasm.py` | QASM parser: Qasm class that converts QASM circuit description commands into SymPy quantum gate circuits. |
| `quantum/qexpr.py` | QExpr base class for quantum expressions, providing argument handling, Hilbert space association, and multi-format printing. |
| `quantum/qft.py` | Quantum Fourier Transform (QFT) and inverse QFT gate implementations, including the RkGate phase rotation gate. |
| `quantum/qubit.py` | Qubit classes (Qubit, IntQubit) and measurement functions (measure_all, measure_partial, measure_partial_oneshot, measure_all_oneshot), with matrix-to-qubit conversion. |
| `quantum/represent.py` | represent function for computing matrix representations of quantum operators and states in various bases, plus expectation value and inner product helpers. |
| `quantum/sho1d.py` | 1D quantum simple harmonic oscillator operators (RaisingOp, LoweringOp, NumberOp, Hamiltonian) and Fock states (SHOKet, SHOBra) with matrix representations. |
| `quantum/shor.py` | Shor's factoring algorithm implementation with the CMod controlled modular exponentiation gate, period finding, and continued fraction expansion. |
| `quantum/spin.py` | Quantum angular momentum: spin operators (Jx, Jy, Jz, J2, Jplus, Jminus), rotation operators (WignerD, Rotation), spin eigenstates (JxKet, JzKet, etc.), and coupling/uncoupling functions. |
| `quantum/state.py` | Dirac notation state classes: Ket, Bra, TimeDepKet, TimeDepBra, and Wavefunction for representing quantum states with dual relationships and basis representations. |
| `quantum/tensorproduct.py` | TensorProduct class for symbolic tensor products of quantum operators and states, with tensor_product_simp for simplification. |
| `units/__init__.py` | Package init for units; imports dimensions, unit systems, quantities, prefixes, convert_to, and all standard unit/constant definitions. |
| `units/definitions.py` | Definitions of all physical units (SI base units, derived units, CGS units, astronomical units, information units) and constants (speed of light, Planck's constant, gravitational constant, etc.). |
| `units/dimensions.py` | Dimension class for physical dimensions and DimensionSystem for managing dimensional dependencies; defines standard dimension systems (MKS, MKSA, SI). |
| `units/prefixes.py` | Prefix class for SI (yocto through yotta) and binary (kibi through exbi) unit prefixes, with prefix_unit helper for generating prefixed unit variants. |
| `units/quantities.py` | Quantity class representing physical quantities as atomic expressions with a dimension, scale factor, and abbreviation; supports dimensional analysis and unit conversion. |
| `units/unitsystem.py` | UnitSystem class representing a coherent set of units built on a DimensionSystem, providing unit lookup and consistency checking. |
| `units/util.py` | Unit utility functions: convert_to for converting expressions between units, dim_simplify (deprecated), and dimensional matrix solver for unit conversion. |
| `units/systems/__init__.py` | Package init for unit systems; imports MKS, MKSA, SI, and natural unit system definitions. |
| `units/systems/mks.py` | MKS (meter-kilogram-second) unit system definition with associated dimensions and prefixed unit variants. |
| `units/systems/mksa.py` | MKSA (meter-kilogram-second-ampere) unit system extending MKS with electromagnetic dimensions and units. |
| `units/systems/natural.py` | Natural unit system where c = hbar = 1, using velocity, action, and energy as base dimensions. |
| `units/systems/si.py` | SI unit system extending MKSA with amount of substance (mol), luminous intensity (cd), and temperature (K). |
| `vector/__init__.py` | Package init for vector algebra; imports and re-exports ReferenceFrame, Vector, Dyadic, Point, functions, printing, and field functions. |
| `vector/dyadic.py` | Dyadic class representing a second-order tensor (dyadic product) used for inertia tensors and other rigid body properties. |
| `vector/fieldfunctions.py` | Vector field operations: curl, divergence, gradient, is_conservative, is_solenoidal, scalar_potential, and scalar_potential_difference. |
| `vector/frame.py` | ReferenceFrame class for defining coordinate frames with orientation relationships (DCM), angular velocity, and coordinate symbols (CoordinateSym). |
| `vector/functions.py` | Core vector functions: cross, dot, express, time_derivative, outer product, kinematic_equations, get_motion_params, partial_velocity, and dynamicsymbols. |
| `vector/point.py` | Point class representing a point in a dynamic system with position, velocity (vel, set_vel), acceleration, two-point and one-point velocity theorems (v2pt_theory, v1pt_theory), and partial_velocity method for computing partial velocities with respect to generalized speeds. |
| `vector/printing.py` | Custom printers (VectorStrPrinter, VectorLatexPrinter, VectorPrettyPrinter) and init_vprinting for physics vector/dynamics notation. |
| `vector/vector.py` | Vector class for representing 3D vectors in reference frames, with operations for dot product, cross product, magnitude, normalization, and expression in different frames. |
