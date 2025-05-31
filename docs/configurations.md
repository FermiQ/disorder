# `configurations` Module

## Overview

The `configurations` module in `src/configurations.f90` is designed to handle and generate irreducible atomic configurations based on group theory and combinatorial calculations. It plays a crucial role in determining unique atomic arrangements under symmetry operations, which is essential in materials science simulations, particularly for disordered systems or alloys.

## Key Components

The module is structured around a main subroutine `irrconfig` which internally calls several helper subroutines for different stages of the configuration generation process.

*   **`irrconfig(iconf, deg, eqamat, group, k, lpro)`**:
    *   The main subroutine that orchestrates the generation of irreducible configurations.
    *   It initializes data structures, preprocesses input, eliminates redundant configurations, and postprocesses the results.
    *   `iconf`: Output array storing the irreducible configurations.
    *   `deg`: Output array storing the degeneracy of each configuration.
    *   `eqamat`: Input, likely a matrix defining equivalent atomic positions or symmetry operations.
    *   `group`: Input, defining site groups or types.
    *   `k`: Input, array related to the number of atoms of different types or within specific sites.
    *   `lpro`: Input, logical flag to enable/disable progress reporting.

*   **`preprocessing` (internal to `irrconfig`)**:
    *   Initializes variables, allocates arrays, and sets up parameters needed for the configuration search.
    *   Calls `confnum` to calculate the total number of possible configurations.
    *   Calls `matrixE` to precompute a matrix used in indexing or combinatorial calculations.
    *   Calls `lastconf` to determine the lexicographically last configuration.

*   **`eliminating` (internal to `irrconfig`)**:
    *   The core logic for identifying and eliminating symmetrically equivalent or redundant configurations.
    *   It iterates through potential configurations, applying symmetry operations and pruning the search space.
    *   Uses nested loops (`i2` to `i5`) and boolean flags (`occ2`, `occ3`, etc.) to manage the search.
    *   Calls `binary` and `multinary` subroutines.

*   **`postprocessing` (internal to `irrconfig`)**:
    *   Finalizes the list of irreducible configurations and their degeneracies.
    *   Allocates and populates the output arrays `iconf` and `deg`.
    *   Cleans up temporary data structures (linked list `head`, `p`).
    *   Calls `degeneracy_correction`.

*   **`binary` (internal to `irrconfig`)**:
    *   Likely handles the generation and checking of configurations for a binary (two-component) part of the system or a specific step in the elimination process.
    *   Interacts with `eqamat` and uses helper functions like `int2cconf` and `cconf2int`.

*   **`multinary(n, im, occ, deg, ib, ie)` (internal to `irrconfig`)**:
    *   Similar to `binary`, but generalized for multi-component or multi-stage configuration processing.
    *   `n`: Integer indicating the current component/stage.
    *   `im`: Index of the current configuration being processed.
    *   `occ`: Boolean array marking processed configurations.
    *   `deg`: Calculated degeneracy.
    *   `ib`, `ie`: Start and end indices for configuration arrays.

*   **`trialclose(deg)` (internal to `irrconfig`)**:
    *   Adds a valid irreducible configuration (`aconf`) and its degeneracy (`deg`) to a linked list (`p`).
    *   Updates progress if `lpro` is enabled.

*   **`degeneracy_correction` (internal to `irrconfig`)**:
    *   Applies corrections to the calculated degeneracies based on group information and configuration details.

*   **`confnum(nc, ncm, nct, ncp, ib, ie, m, k, group, ng, kg)`**:
    *   Calculates various combinatorial numbers:
        *   `nc`: Total number of configurations.
        *   `ncm`: Number of combinations for each component/shell.
        *   `nct`: Number of combinations related to site groups.
        *   `ncp`: Helper array for indexing multi-dimensional configuration spaces.
    *   `ib`, `ie`: Start and end indices for sub-configurations.
    *   `m`: Array derived from `k`, possibly representing remaining atoms to be placed.
    *   `group`, `ng`, `kg`: Relate to site groupings.

*   **`matrixE(E, m, k)`**:
    *   Computes a matrix `E` (likely related to binomial coefficients or look-up tables for `nchoosek`) used for efficiently converting between integer representations and actual configuration arrays.
    *   `m`, `k`: Input arrays defining dimensions for combinations.

*   **`lastconf(lconf, m, k)`**:
    *   Determines the lexicographically last configuration (`lconf`) given the number of atoms of each type (`k`) and available sites (`m`).

*   **`int2cconf(ic, cconf, nc, E, m, k, lconf)`**:
    *   Converts an integer index (`ic`) into a "canonical" configuration array (`cconf`).
    *   Uses the precomputed `E` matrix and `lconf`.
    *   `nc`: Total number of configurations.
    *   `m`, `k`: Integers defining the scope of the conversion.

*   **`cconf2aconf(conf, fconf, na)`**:
    *   Converts a "canonical" configuration (`conf`) to an "actual" configuration (`aconf` which is returned via `conf` argument) by mapping it based on fixed/forbidden sites (`fconf`).
    *   `na`: Total number of atoms/sites.

*   **`aconf2cconf(conf, fconf, na)`**:
    *   Converts an "actual" configuration (`conf`) back to a "canonical" configuration (`cconf` which is returned via `conf` argument) considering fixed/forbidden sites (`fconf`).

*   **`cconf2int(ic, cconf, lconf, nc, E, k)`**:
    *   Converts a "canonical" configuration array (`cconf`) back into its unique integer index (`ic`).
    *   Uses the precomputed `E` matrix and `lconf`.

## Important Variables/Constants

*   **`datalink` (type)**: A derived data type used to create a linked list for storing configurations temporarily before the final array allocation. It contains `iconf` (an allocatable integer array for the configuration) and `deg` (integer for degeneracy).
*   **`aconf`, `oconf`, `lconf`**: Allocatable integer arrays used extensively within `irrconfig` and its subroutines to store and manipulate atomic configurations during generation and testing.
*   **`E` (integer(8), allocatable, dimension(:,:))**: A key matrix, likely storing precomputed values for `nchoosek` (combinations), used for efficient mapping between integer indices and configuration arrays.
*   **`k` (integer(2), intent(in))**: An input array that seems fundamental, likely defining the number of atoms of different types or the number of atoms to be placed in different shells/sublattices.
*   **`group` (integer(2), intent(in))**: An input array defining groupings of sites, used for symmetry considerations and degeneracy calculations.
*   **`eqamat` (integer(2), intent(in))**: Input matrix representing symmetry operations or equivalency relations between sites.
*   **`valid` (logical(1), allocatable, dimension(:,:))**: A boolean array used in `binary` and `multinary` to keep track of valid symmetry operations for a given configuration.

## Usage Examples

This module is intended to be used as part of a larger simulation package. The main entry point is the `irrconfig` subroutine.

```fortran
module my_simulation_program
  use configurations
  use functions ! For nchoosek, complement, etc.
  implicit none

  subroutine run_simulation
    integer(2), allocatable :: iconf(:,:), deg(:)
    integer(2) :: eqamat(N, M) ! Example dimensions
    integer(2) :: group(G), k_array(NK)
    logical(1) :: enable_progress

    ! ... Initialize eqamat, group, k_array ...
    ! eqamat would define how atom indices map to each other under symmetry
    ! group would define distinct groups of sites
    ! k_array would define how many atoms of each type, e.g., k(1) atoms in group 1, etc.
    enable_progress = .true.

    call irrconfig(iconf, deg, eqamat, group, k_array, enable_progress)

    ! ... iconf now contains the irreducible configurations ...
    ! ... deg contains their respective degeneracies ...

    ! Further processing with the generated configurations

  end subroutine run_simulation
end module my_simulation_program
```

*(Note: The exact initialization and meaning of `eqamat`, `group`, and `k` would depend on the specific conventions of the calling program that uses this `disorder` code base.)*

## Dependencies and Interactions

*   **`functions`**: This module is explicitly used (`use functions`). It likely provides fundamental mathematical or utility functions such_as:
    *   `nchoosek`: For calculating combinations (number of ways to choose k items from n).
    *   `sort`: For sorting arrays, used to bring configurations into a canonical form.
    *   `complement`: Possibly for finding unoccupied sites.
    *   `binsearch_2`, `binsearch_8`: For binary searching within arrays.
*   **`progress`**: Used by `irrconfig` (specifically in `preprocessing` and `trialclose`) if the `lpro` flag is true. This module (`cls_prog` type) is responsible for displaying progress updates during lengthy calculations.
*   **`stdout`**: Used by `irrconfig` (specifically in `preprocessing` and `postprocessing`) for printing informational messages to standard output (e.g., `stdout_3`, `stdout_4`).

The module operates by taking structural and symmetry information as input and systematically generates all unique configurations, which can then be used for further calculations (e.g., energy calculations for each configuration in a material property study).
