# `disorder` Program

## Overview

The `disorder` program, located in `src/disorder.f90`, is the main executable for generating irreducible superstructures for disordered materials. It reads structural information and control parameters, determines symmetry-equivalent sites, groups them, and then generates all unique atomic configurations based on the specified substitutions. The output can include these configurations, symmetry matrices, and optionally, generated POSCAR files for each configuration.

## Main Program Logic

The program follows these main steps:

1.  **Initialization**:
    *   Sets default values for input parameters.
    *   Records the starting CPU time.
    *   Prints an initial banner (`stdout_0`).

2.  **Reading Inputs**:
    *   Reads the `INDSOD` file, which contains key parameters for the calculation, using a Fortran `NAMELIST` called `input`.
    *   Performs validation checks on the read parameters (e.g., `nsub`, `prec`, `subs`, `symb`).
    *   Reads the `SPOSCAR` file (a VASP-like structure file) to get the lattice vectors (`a`), atomic positions (`x`), atom types (`atom`), and number of atoms of each type (`natom`).
    *   Performs validation checks related to `SPOSCAR` data and `INDSOD` parameters (e.g., `site` value, consistency of `sum(k)` with `natom(site)`).
    *   Prints a summary of the input parameters (`stdout_1`).

3.  **Symmetry Analysis and Grouping**:
    *   Calls `eqamatrix` (from the `symmetry` module) to determine the equivalency matrix (`eqamat`) and space group matrices (`spgmat`). This identifies which atomic sites are equivalent under symmetry operations.
    *   Calls `grouping` (from the `groups` module) to group the equivalent sites based on the `eqamat` and other structural details. The `group` array is key for the configuration generation.

4.  **Irreducible Configuration Generation**:
    *   Calls `irrconfig` (from the `configurations` module) to generate the irreducible atomic configurations (`iconf`) and their degeneracies (`deg`). This is the core calculation step.

5.  **Output Generation**:
    *   Calls `output` (from the `outfiles` module) to write the results. This can include:
        *   The equivalency matrix (`eqamat`) if `leqa` is true.
        *   Space group matrices (`spgmat`) if `lspg` is true.
        *   The irreducible configurations (`iconf`) and their degeneracies (`deg`) if `lcfg` is true.
        *   Individual POSCAR files for each configuration if `lpos` is true.

6.  **Finalization**:
    *   Records the ending CPU time.
    *   Prints the total execution time (`stdout_5`).

## Input Parameters (from `INDSOD` file)

The `INDSOD` file uses a Fortran `NAMELIST` format under the group name `input`.

*   **`nsub` (Integer, Default: 2)**:
    *   The number of substituting elements/species on the disordered sublattice.
    *   Must be between 2 and 5 (inclusive).

*   **`subs` (Integer array, Dimension: 5)**:
    *   An array where `subs(1:nsub)` specifies the number of atoms for each substituting element.
    *   For example, if `nsub=2` and `subs=(/2, 6/)`, it means 2 atoms of type `symb(1)` and 6 atoms of type `symb(2)`.
    *   The sum `sum(subs(1:nsub))` must equal the total number of atoms on the substitution site (`natom(site)`).
    *   Elements must be greater than 0.

*   **`symb` (Character array, Dimension: 5, Element length: 2)**:
    *   An array where `symb(1:nsub)` specifies the chemical symbols for each substituting element.
    *   Example: `symb=(/'Li', 'Fe'/)`.
    *   The number of provided symbols must match `nsub`.

*   **`prec` (Real(8), Default: 1.0D-5)**:
    *   Precision parameter used in symmetry determination (e.g., for comparing atomic positions).
    *   Recommended not to be greater than 1.0D-2.

*   **`site` (Integer, Default: 1)**:
    *   The index of the atomic species in the `SPOSCAR` file that will be substituted.
    *   For example, if `SPOSCAR` lists atom types `Si`, `O`, and `site=1`, then `Si` atoms will be substituted.
    *   Must be a valid index within the atom types found in `SPOSCAR`.

*   **`leqa` (Logical, Default: .false.)**:
    *   If `.true.`, the program will output the equivalency matrix (`EQAMAT`).

*   **`lspg` (Logical, Default: .false.)**:
    *   If `.true.`, the program will output the space group matrices (`SPGMAT`).

*   **`lcfg` (Logical, Default: .true.)**:
    *   If `.true.`, the program will output the irreducible configurations (`CONFGL` and `CONFGD`).

*   **`lpos` (Logical, Default: .false.)**:
    *   If `.true.`, the program will generate `POSCAR` files for each irreducible configuration.

*   **`lpro` (Logical, Default: .false.)**:
    *   If `.true.`, enables progress reporting during the `irrconfig` calculation.

## Interactions with Other Modules

The `disorder` program acts as an orchestrator, utilizing several other modules:

*   **`structure`**:
    *   Used for reading the input structure file (`SPOSCAR`).
    *   Provides `readpos` subroutine to parse lattice parameters, atomic positions, and atom types.

*   **`symmetry`**:
    *   Used to determine symmetry operations and equivalent sites.
    *   Provides `eqamatrix` subroutine.

*   **`groups`**:
    *   Used to group atomic sites based on symmetry equivalence.
    *   Provides `grouping` subroutine.

*   **`configurations`**:
    *   The core module for generating irreducible configurations.
    *   Provides `irrconfig` subroutine.

*   **`outfiles`**:
    *   Handles the writing of output files (configurations, symmetry information, POSCARs).
    *   Provides `output` subroutine.

*   **`stdout`**:
    *   Used for printing messages (banner, input summary, timing) to standard output.
    *   Provides `stdout_0`, `stdout_1`, `stdout_5`.
    *   Also implicitly used by other modules for messages (e.g. `stderr` for error messages which is also part of `stdout` module).

## Usage Example (`INDSOD` file)

```fortran
&input
  nsub = 2
  subs = 2, 2  ! e.g., 2 atoms of symb(1), 2 atoms of symb(2)
  symb = 'Cu', 'Au'
  site = 1     ! Substitute the first atom type listed in SPOSCAR
  prec = 1.0d-6
  leqa = .true. ! Output EQAMAT
  lcfg = .true. ! Output configurations
  lpos = .true. ! Create POSCAR files for each configuration
  lpro = .true. ! Show progress
/
```
This `INDSOD` would configure a run for a binary alloy (e.g., Cu/Au) where the first atomic species in the `SPOSCAR` file is substituted. If there are 4 atoms of this species, they will be replaced by 2 Cu and 2 Au atoms in all symmetrically distinct ways.
