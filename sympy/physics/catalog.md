# sympy/physics — Catalog

> Part of [SymPy](../catalog.md). Physics subpackages: quantum mechanics, optics, classical mechanics, units, hydrogen wave functions, vector algebra.

## Python Files

| File | Summary |
|------|---------|
| `__init__.py` | Package init for sympy.physics; imports units and physics matrices (mgamma, msigma, minkowski_tensor, mdft). |
| `gaussopt.py` | Deprecated shim that re-exports `sympy.physics.optics.gaussopt`; warns users to use the optics subpackage instead. |
| `hydrogen.py` | Hydrogen atom wavefunctions and energy levels: radial wavefunction R_nl, non-relativistic energy E_nl, and Dirac relativistic energy E_nl_dirac. |
| `matrices.py` | Physics-related matrices: Pauli matrices (msigma), Dirac gamma matrices (mgamma), Minkowski metric tensor, parallel axis theorem matrix, and discrete Fourier transform matrix (mdft). |
| `paulialgebra.py` | Implements Pauli algebra by subclassing Symbol, providing the Pauli class and evaluate_pauli_product for symbolic Pauli matrix multiplication. |
| `pring.py` | Quantum particle on a ring: wavefunction and energy functions for a 1D ring geometry. |
| `qho_1d.py` | One-dimensional quantum harmonic oscillator: wavefunction psi_n, energy E_n, and coherent state functions. |
| `secondquant.py` | Second quantization operators and states for bosons and fermions, including creation/annihilation operators (F, Fd, B, Bd), Fock states, Wick's theorem, the NO class (normal-ordered product container with methods for iterating quasi-creators/annihilators and removing operators at specific positions via get_subNO), contraction functions, and commutator algebra. FermionState (FKet/FBra) provides `up`/`down` methods that apply creation/annihilation to fermionic occupation states with Fermi-level–aware logic: above-fermi indices create particles, below-fermi indices remove holes, and general (unrestricted) indices resolve ambiguity by introducing a KroneckerDelta with a restricted dummy index. This is distinct from `quantum/fermion.py`, which defines abstract FermionOp operators and single-mode Fock states without Fermi-level semantics. Note: for reordering generic bosonic/fermionic operator expressions into normal form, see `quantum/operatorordering.py`. |
| `sho.py` | 3D isotropic simple harmonic oscillator: radial wavefunction R_nl and energy E_nl using associated Laguerre polynomials. |
| `unitsystems.py` | Deprecated shim that re-exports `sympy.physics.units`; warns users to use the units subpackage instead. |
| `wigner.py` | Pure numerical computation of Wigner 3j, 6j, 9j symbols, Clebsch-Gordan coefficients, Racah coefficients, and Gaunt coefficients for angular momentum coupling. These are low-level functions that require numeric arguments; for symbolic wrapper classes (Wigner3j, CG) with parameter validation, doit(), and pretty-printing, see `quantum/cg.py`. |
| `continuum_mechanics/__init__.py` | Package init for continuum_mechanics; imports the Beam class. |
| `continuum_mechanics/beam.py` | Beam class for solving 2D beam bending problems using singularity functions, including load application, reaction forces, shear force, bending moment, slope, and deflection. |
| `hep/gamma_matrices.py` | Gamma (Dirac) matrices expressed as tensor objects for high-energy physics, with Kahane simplification and gamma trace algorithms. |
| `mechanics/__init__.py` | Package init for classical mechanics; imports and re-exports Kane's method, Lagrange's method, rigid body, particle, linearization, body, system, and vector modules. |
| `mechanics/body.py` | Body class that serves as a unified representation of either a RigidBody or Particle, with support for applied loads (forces and torques). |
| `mechanics/functions.py` | Utility functions for classical mechanics: inertia dyadic creation, linear/angular momentum, system-level kinetic/potential energy (sums over a collection of bodies/particles), Lagrangian computation, find_dynamicsymbols (discovers dynamic symbols in expressions), mechanics printing (re-exported from `vector/printing.py`), and symbolic substitution (msubs). Note: individual body/particle classes (`mechanics/rigidbody.py`, `mechanics/particle.py`) have their own `kinetic_energy` methods for single-object energy; this module's functions aggregate across multiple bodies. `dynamicsymbols` itself is defined in `vector/functions.py` and re-exported here. |
| `mechanics/kane.py` | KanesMethod class implementing Kane's method for forming equations of motion: computes generalized active forces (_form_fr) and generalized inertia forces (_form_frstar) including translational (mass × acceleration) and rotational (inertia tensor rate, angular acceleration, and gyroscopic cross-product) contributions, distinguishing rigid bodies from particles. Produces mass matrix and forcing vector representations. Its `linearize()` method delegates to the `Linearizer` class in `mechanics/linearize.py`. |
| `mechanics/lagrange.py` | LagrangesMethod class implementing Lagrange's method for deriving equations of motion from a Lagrangian, with support for constraints and non-conservative forces. |
| `mechanics/linearize.py` | Linearizer class for computing the linearized form of a dynamic system, handling dependent coordinates and speeds arising from constraints. The `linearize()` method accepts operating-point dicts (single or iterable of dicts, merged internally), an `A_and_B` flag to choose between returning raw (M, A, B) matrices or the fully solved state-space (A, B) form, and a `simplify` option. Internally builds Jacobian-based block sub-matrices guarded by dimension variables. This is the primary implementation; `KanesMethod.linearize` and `LagrangesMethod.linearize` delegate here. |
| `mechanics/models.py` | Sample symbolic mechanical models (multi-mass spring-damper, n-link pendulum) for testing and examples. |
| `mechanics/particle.py` | Particle class representing a point mass with position, velocity, acceleration, and potential energy. |
| `mechanics/rigidbody.py` | RigidBody class representing an idealized rigid body with mass, center of mass, reference frame, and inertia dyadic. Provides its own `kinetic_energy(frame)` method that computes both rotational (inertia dyadic · angular velocity) and translational (mass · velocity²) components for *this specific body*. See also `mechanics/functions.py` for system-level energy utilities and `mechanics/particle.py` for point-mass kinetic energy. |
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
| `quantum/cg.py` | Symbolic wrapper classes for angular momentum coupling coefficients: Wigner3j and CG (Clebsch-Gordan) classes with parameter validation (is_symbolic check raises ValueError for non-numeric args in doit()), pretty-printing, LaTeX output, and simplification via cg_simp. The doit() method delegates to numerical functions in `wigner.py`. |
| `quantum/circuitplot.py` | Matplotlib-based plotting of quantum circuits, including CircuitPlot, measurement gates (Mz, Mx), and gate creation helpers. |
| `quantum/circuitutils.py` | Primitive circuit operations: subcircuit finding/replacement using KMP algorithm, symbolic-to-real index conversion, random_reduce (tries to remove identity patterns from a circuit, returns original if none found), and random_insert. See `quantum/identitysearch.py` for discovering gate identities; this module applies/removes them from circuits. |
| `quantum/commutator.py` | Commutator class implementing the unevaluated commutator [A, B] = A*B - B*A for quantum operators. |
| `quantum/constants.py` | Quantum mechanical constants: defines hbar (reduced Planck's constant) as a singleton NumberSymbol. |
| `quantum/dagger.py` | Dagger class implementing the Hermitian conjugate operation for quantum objects (operators, states, matrices). |
| `quantum/density.py` | Density operator class for representing mixed quantum states, with support for entropy calculation, probability checking, and matrix representation. |
| `quantum/fermion.py` | Fermionic quantum operators (FermionOp) and single-mode Fock states (FermionFockKet) satisfying {c, c^dagger} = 1. For multi-mode fermionic occupation states with Fermi-level logic (creation/annihilation via `up`/`down`), see `secondquant.py`. |
| `quantum/gate.py` | Quantum gates acting on qubits: base Gate class, controlled gates (CGate, CNotGate), single-qubit gates (Hadamard, X, Y, Z, Phase, T), SwapGate, UGate, and gate sorting/simplification. |
| `quantum/grover.py` | Grover's quantum search algorithm: OracleGate, WGate, superposition_basis, grover_iteration, and apply_grover. |
| `quantum/hilbert.py` | Hilbert space classes: HilbertSpace, ComplexSpace, L2, FockSpace, TensorProductHilbertSpace, DirectSumHilbertSpace, and TensorPowerHilbertSpace (exponentiation via `**` with eval() validation for zero/unit powers and symbolic exponent checking). Handles space algebra, not operator/state tensor products (see `quantum/tensorproduct.py` for those). |
| `quantum/identitysearch.py` | Gate identity discovery algorithms: BFS and random search for finding equivalent quantum gate sequences (GateIdentity class, scalar matrix checking). Only searches for identities; to apply or remove identities from circuits, see `quantum/circuitutils.py`. |
| `quantum/innerproduct.py` | InnerProduct class representing the symbolic inner product between a Bra and a Ket. |
| `quantum/matrixcache.py` | MatrixCache class for storing small matrices in SymPy, NumPy, and SciPy sparse formats for efficient reuse in quantum computations. |
| `quantum/matrixutils.py` | Utility functions for interconversion between SymPy Matrix, NumPy ndarray, and SciPy sparse matrices, plus matrix tensor product and dagger operations. |
| `quantum/operator.py` | Base quantum operator classes: Operator, HermitianOperator, UnitaryOperator, IdentityOperator, OuterProduct, and DifferentialOperator. |
| `quantum/operatorordering.py` | Functions for reordering quantum operator expressions into normal ordered form: normal_ordered_form and helpers (_normal_ordered_form_factor, _normal_ordered_form_terms) that handle bosonic commutator and fermionic anti-commutator correction terms during reordering, with an `independent` flag controlling whether cross-mode corrections are included or omitted. See `secondquant.py` for the NO container class. |
| `quantum/operatorset.py` | Mapping between quantum operators and their corresponding eigenstates (operators_to_state, state_to_operators). |
| `quantum/pauli.py` | Pauli sigma operators (SigmaX, SigmaY, SigmaZ, SigmaPlus, SigmaMinus) and states (SigmaZKet/Bra) for two-level quantum systems, with qsimplify_pauli. |
| `quantum/piab.py` | Particle in a box (PIAB) Hamiltonian operator and eigenstates (PIABKet, PIABBra) with position-space representation. For general wavefunction coordinate handling and default integration bounds, see `quantum/state.py`. |
| `quantum/qapply.py` | qapply function for symbolically applying quantum operators to states in an expression tree. |
| `quantum/qasm.py` | QASM parser: Qasm class that converts QASM circuit description commands into SymPy quantum gate circuits. |
| `quantum/qexpr.py` | QExpr base class for quantum expressions, providing argument handling, Hilbert space association, adjoint evaluation (_eval_adjoint with fallback to Dagger wrapping and Hilbert space preservation), and multi-format printing utilities. The adjoint logic lives here, not in `quantum/dagger.py`. |
| `quantum/qft.py` | Quantum Fourier Transform (QFT) and inverse QFT gate implementations, including the RkGate phase rotation gate. |
| `quantum/qubit.py` | Qubit classes (Qubit, IntQubit) and measurement functions (measure_all, measure_partial, measure_partial_oneshot, measure_all_oneshot) that accept a format parameter ('sympy', 'numpy', 'scipy.sparse') — currently only 'sympy' is implemented (raises NotImplementedError for others). Also includes qubit-to-matrix and matrix-to-qubit conversion. |
| `quantum/represent.py` | represent function for computing matrix representations of quantum operators and states in various bases, plus expectation value and inner product helpers. |
| `quantum/sho1d.py` | 1D quantum simple harmonic oscillator operators (RaisingOp, LoweringOp, NumberOp, Hamiltonian) and Fock states (SHOKet, SHOBra) with matrix representations. |
| `quantum/shor.py` | Shor's factoring algorithm implementation with the CMod controlled modular exponentiation gate, period finding, and continued fraction expansion. |
| `quantum/spin.py` | Quantum angular momentum: spin operators (Jx, Jy, Jz, J2, Jplus, Jminus), rotation operators (WignerD, Rotation), spin eigenstates (JxKet, JzKet, etc.), and coupling/uncoupling functions. |
| `quantum/state.py` | Dirac notation state classes: Ket, Bra, TimeDepKet, TimeDepBra, and Wavefunction for representing quantum states with dual relationships and basis representations. Wavefunction stores coordinate variables and their integration limits; coordinates passed as bare symbols (without explicit bounds) default to `(-oo, oo)`. Also provides `is_normalized` and `norm` for probability checking. |
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
| `unitsystems/__init__.py` | Legacy unit-systems package (separate from `units/`). Imports Dimension, DimensionSystem, Unit, Constant, UnitSystem, Quantity, and dim_simplify. |
| `unitsystems/dimensions.py` | Legacy Dimension class and DimensionSystem for the old unitsystems package. |
| `unitsystems/prefixes.py` | Legacy Prefix class for the old unitsystems package. |
| `unitsystems/quantities.py` | Legacy Quantity class for the old unitsystems package, representing a physical quantity with factor, unit, and uncertainty; provides `__mul__`, `__add__`, `convert_to`, and related arithmetic. |
| `unitsystems/simplifiers.py` | Simplification functions for the legacy unitsystems package: `dim_simplify` recursively reduces dimensional expressions (Mul, Pow, Add of Dimension objects), discarding non-dimensional numeric factors from products; `qsimplify` recursively reduces quantity expressions, separating Quantity/Unit args from plain numeric args and combining them via dedicated methods. Distinct from `units/util.py` which has a deprecated dim_simplify shim. |
| `unitsystems/units.py` | Legacy Unit and Constant classes for the old unitsystems package. |
| `unitsystems/systems/__init__.py` | Package init for legacy unit system definitions. |
| `unitsystems/systems/mks.py` | Legacy MKS unit system definition. |
| `unitsystems/systems/mksa.py` | Legacy MKSA unit system definition. |
| `unitsystems/systems/natural.py` | Legacy natural unit system definition. |
| `vector/__init__.py` | Package init for vector algebra; imports and re-exports ReferenceFrame, Vector, Dyadic, Point, functions, printing, and field functions. |
| `vector/dyadic.py` | Dyadic class representing a second-order tensor (dyadic product) used for inertia tensors and other rigid body properties. |
| `vector/fieldfunctions.py` | Vector field operations: curl, divergence, gradient, is_conservative, is_solenoidal, scalar_potential, and scalar_potential_difference. |
| `vector/frame.py` | ReferenceFrame class for defining coordinate frames with orientation relationships (DCM), angular velocity, and coordinate symbols (CoordinateSym). |
| `vector/functions.py` | Core vector functions: cross, dot, express, time_derivative, outer product, kinematic_equations, get_motion_params, partial_velocity, and dynamicsymbols (canonical definition — creates time-dependent symbolic functions, handles single-name vs comma-separated names differently). Re-exported by `mechanics/functions.py`. |
| `vector/point.py` | Point class representing a point in a dynamic system with position, velocity, and acceleration relative to other points. |
| `vector/printing.py` | Custom printers (VectorStrPrinter, VectorLatexPrinter, VectorPrettyPrinter) and display functions (vsprint, vprint, vpprint, vlatex) that strip explicit time arguments from dynamic symbols and use prime notation for time derivatives (e.g., `u1(t)` → `u1`, `Derivative(u2(t), t)` → `u2'`). Also provides init_vprinting for physics vector/dynamics notation. |
| `vector/vector.py` | Vector class for representing 3D vectors in reference frames, with operations for dot product, cross product, magnitude, normalization, and expression in different frames. |
