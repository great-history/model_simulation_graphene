# Graphene Systems: Band Structure, Landau Levels, and Hartree–Fock

A collection of MATLAB research codes for the electronic structure and interaction effects of multilayer and twisted graphene systems. The repository includes tight-binding and continuum Hamiltonians, Landau-level calculations, and self-consistent Hartree–Fock studies.

The code is organized as model-specific functions and research scripts. Parameters, plotting settings, and data paths are generally configured directly in the scripts; there is no single application entry point.

## Models and calculations

| Area | Contents |
| --- | --- |
| Bernal bilayer graphene | Tight-binding and four-band continuum models; two- and four-band Landau-level models; layer asymmetry, trigonal warping, and example Ising spin–orbit terms |
| ABA trilayer graphene | Six-band continuum models, reduced Landau-level models, magnetic-field and displacement-field sweeps, density of states, and Landau-level crossing fits |
| ABC trilayer graphene | Two-band Landau-level calculations and interaction routines |
| Twisted graphene | Continuum models for twisted bilayer (TBG), twisted monolayer–bilayer (TMBG), and twisted double-bilayer graphene (TDBG), with valley-resolved band calculations |
| Hartree–Fock | Landau-level form factors, exchange integrals, Hartree and Fock matrices, spin-resolved density-matrix updates, and ABA-specific DIIS routines |
| Additional explorations | Velocity operators, band-overlap tracking, Berry-curvature and Wilson-loop scripts, and Stoner-model mean-field functions |

## Repository layout

```text
.
├── multilayer_graphene/
│   ├── bilayer_graphene/
│   │   ├── bilayer_hamitonians/       # Hamiltonian constructors (original spelling)
│   │   ├── interaction_effects/       # Bilayer Hartree–Fock calculations
│   │   ├── test_scripts/
│   │   ├── plot_funcs/
│   │   ├── plot_scripts/
│   │   └── utils/
│   ├── trilayer_ABA_graphene/
│   │   ├── classes/                  # trilayer_ABA_class
│   │   ├── trilayer_ABA_hamiltonian/
│   │   ├── interaction_effects/       # Including T2-subspace and DIIS routines
│   │   ├── trilayer_ABA_fitting_procedure/
│   │   ├── fitting/
│   │   ├── test_script/
│   │   ├── plot_funcs/
│   │   ├── utilities/
│   │   ├── code_test/
│   │   └── trilayer_landau_level_old/ # Earlier implementations
│   ├── trilayer_ABC_graphene/
│   ├── hartree_fock_package/          # Shared interaction helpers
│   └── utils/
├── twisted_multilayer_graphene/
│   ├── brillouin_zone/
│   ├── helper/
│   ├── multilayer_graphene/           # Rotated monolayer/bilayer building blocks
│   ├── velocity_operator/
│   └── Landauer_Buttiker/
└── mean_field_calculation/
    ├── stoner_model/                  # Mean-field functions and draft scripts
    └── vortex_pinning/                # Placeholder

```

## Getting started

The following examples assume MATLAB's current folder is the repository root. Use a fresh MATLAB session, or remove paths from previous model calculations, when switching between modules. These examples follow the source interfaces; they have not been runtime-validated as part of this README update.

### 1. Bilayer continuum bands

This small example constructs the four-band Hamiltonian in the `(A1, B1, A2, B2)` basis and plots the K-valley bands along a momentum cut. Hopping parameters and energies are in eV; `akx` and `aky` are dimensionless momenta.

```matlab
cd(fullfile('multilayer_graphene', 'bilayer_graphene'))
addpath('bilayer_hamitonians')

akx = linspace(-0.1, 0.1, 201);
energies = zeros(numel(akx), 4);
for ik = 1:numel(akx)
    [HK, HKp] = construct_bilayer_continuum_model( ...
        3.16, 0.381, 0.380, 0.140, 0, ...
        zeros(4), zeros(4), akx(ik), 0);
    energies(ik, :) = sort(real(eig(HK))).';
end

figure
plot(akx, energies)
xlabel('a k_x')
ylabel('Energy (eV)')
grid on
```

For a longer example with layer asymmetry and spin-dependent terms, see [bbg_continuum_test.m](multilayer_graphene/bilayer_graphene/bbg_continuum_test.m). Landau-level examples are in [test_scripts](multilayer_graphene/bilayer_graphene/test_scripts).

### 2. Twisted bilayer graphene bands

```matlab
cd('twisted_multilayer_graphene')
addpath('helper', 'brillouin_zone', 'multilayer_graphene', 'velocity_operator')
get_tbg_band
```

[get_tbg_band.m](twisted_multilayer_graphene/get_tbg_band.m) constructs and diagonalizes the K- and K′-valley continuum Hamiltonians along a momentum path and plots their bands. Its parameter block controls:

- `theta`: twist angle in radians, initialized using degrees times `pi / 180`.
- `w_aa`, `w_ab`: interlayer tunneling amplitudes.
- `onsite_a`, `onsite_b`: sublattice onsite energies.
- `q_trunc`: reciprocal-space truncation.
- `np_side_list`: sampling density along each path segment.

For `Nq = size(bm_sup_vecs, 2)`, each TBG valley Hamiltonian has dimension `4*Nq`; spin is not included in this matrix. Increase the truncation and momentum resolution to check convergence for the observable of interest.

Related entry scripts are [get_tmbg_band.m](twisted_multilayer_graphene/get_tmbg_band.m), [get_tdbg_band.m](twisted_multilayer_graphene/get_tdbg_band.m), and [k_mesh_band_structure_tbg.m](twisted_multilayer_graphene/k_mesh_band_structure_tbg.m). Review their parameter blocks and dependencies separately; the momentum-grid script explicitly starts a parallel pool.

### 3. ABA trilayer Landau levels

[trilayer_ABA_LL_model_test.m](multilayer_graphene/trilayer_ABA_graphene/trilayer_ABA_LL_model_test.m) demonstrates the `trilayer_ABA_class` workflow, including Landau-level sweeps, state selection, and density-of-states plots. More focused examples are in [test_script](multilayer_graphene/trilayer_ABA_graphene/test_script).

Before using the class, update the machine-specific `D:\matlab\graphene-package\...` paths in its constructor to your local checkout. The relevant folders are `utilities`, `trilayer_ABA_hamiltonian`, `trilayer_ABA_fitting_procedure`, `plot_funcs`, and `interaction_effects`, under `multilayer_graphene/trilayer_ABA_graphene`.

Check the energy convention at each interface: the example initializes hopping parameters with values such as `gamma0 = 3100`, while other modules use values around `3.1`. Energy units are not uniform across all scripts.

## Hartree–Fock workflows

Interaction code is split between the shared [hartree_fock_package](multilayer_graphene/hartree_fock_package) and the model-specific `interaction_effects` directories. A typical calculation consists of:

1. Selecting a Landau-level basis or low-energy subspace and a filling factor.
2. Computing form factors and exchange integrals for the chosen magnetic field and layer separation.
3. Constructing Hartree, Fock, single-particle, and spin-dependent contributions as appropriate to the model.
4. Diagonalizing the mean-field Hamiltonian, filling the lowest states, and rebuilding the density matrix.
5. Iterating while monitoring the density-matrix error and comparing converged solutions from different initial states.

The shared `Hartree_Fock_iteration.m` performs a spin-resolved density-matrix update and returns an error measure; the surrounding script controls the self-consistency loop. ABA-specific code also includes direct inversion in the iterative subspace (DIIS) helpers.

Useful starting points include the bilayer [interaction_effects](multilayer_graphene/bilayer_graphene/interaction_effects) scripts, [trilayer_ABC_LL_two_band.m](multilayer_graphene/trilayer_ABC_graphene/trilayer_ABC_LL_two_band.m), and the ABA [trilayer_ABA_T2_hf_diis_test.m](multilayer_graphene/trilayer_ABA_graphene/test_script/trilayer_ABA_T2_hf_diis_test.m).

The ABA T2 example loads precomputed state information and exchange integrals from `trilayer_ABA_data/*.mat`. That data directory is absent from this checkout. Prepare matching input data and check the `load(...)` paths before using this workflow; the example is not a standalone first-run demo.